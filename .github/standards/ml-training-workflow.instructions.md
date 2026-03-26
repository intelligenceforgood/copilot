---
applyTo: "ml/containers/train-*/**,ml/scripts/submit_*.py,ml/src/ml/training/**"
---

# ML Training Workflow Standards

## Training Container Authoring

- **Coerce feature dtypes before XGBoost.** JSONL features can arrive as strings even when they look numeric. After building a feature DataFrame, always coerce: `df.apply(pd.to_numeric, errors="coerce").fillna(0)`. XGBoost's `DMatrix` rejects `object` dtype columns with a cryptic `KeyError: 'object'`.

- **Pin `classification_report` to all encoder classes.** When some classes have zero eval samples, `classification_report(y_true, y_pred, target_names=le.classes_)` raises `ValueError: Number of classes N does not match size of target_names M`. Always pass both extra params:
  ```python
  classification_report(
      y_eval, y_pred,
      labels=list(range(len(le.classes_))),
      target_names=le.classes_,
      zero_division=0,
  )
  ```

## Pipeline Submission

- **Upload the config to GCS before submitting.** If `submit_pipeline.py` constructs a `gs://` path for a config parameter, the training container will 404 unless the local file is actually uploaded first:
  ```python
  bucket.blob(f"configs/{Path(config_path).name}").upload_from_filename(config_path)
  ```

## Debugging Loop — Test Locally First

**Never iterate via cloud submissions.** Each build→push→submit→wait cycle takes ~10 minutes. Instead:

1. Download a small sample once:
   ```bash
   mkdir -p /tmp/xgb-test/data
   gsutil cat gs://i4g-ml-data/datasets/classification/v1/train.jsonl | head -100 > /tmp/xgb-test/data/train.jsonl
   gsutil cat gs://i4g-ml-data/datasets/classification/v1/eval.jsonl  | head -30  > /tmp/xgb-test/data/eval.jsonl
   gsutil cat gs://i4g-ml-data/datasets/classification/v1/test.jsonl  | head -30  > /tmp/xgb-test/data/test.jsonl
   cp pipelines/configs/classification_xgboost.yaml /tmp/xgb-test/
   ```

2. Run the script directly (~2s feedback):
   ```bash
   conda run -n ml python containers/train-xgboost/train.py \
     --config /tmp/xgb-test/classification_xgboost.yaml \
     --dataset /tmp/xgb-test/data \
     --output /tmp/xgb-test/output
   ```

3. Only build+push+submit once the local run exits cleanly.

## Monitoring Submitted Jobs

Poll pipeline state via the Python SDK (gcloud CLI doesn't have `pipeline-jobs describe`):
```python
from google.cloud import aiplatform
aiplatform.init(project="i4g-ml", location="us-central1")
job = aiplatform.PipelineJob.get("<resource_name>")
print(job.state)  # 3=RUNNING, 4=SUCCEEDED, 5=FAILED
```

To get the inner CustomJob ID from a failed pipeline, check the outer job logs for the `resource.labels.job_id` in the error URL, then pull errors with:
```bash
gcloud logging read 'resource.type="ml_job" resource.labels.job_id="<id>" severity>=DEFAULT' \
  --project=i4g-ml --format='value(timestamp, textPayload)' --order=asc --limit=500 2>&1 \
  | grep -A5 -E "Traceback|Error:|raise "
```
