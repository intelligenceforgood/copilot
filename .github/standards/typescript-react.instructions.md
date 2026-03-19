---
applyTo: "**/*.ts,**/*.tsx,**/*.jsx"
---

# TypeScript / React Standards

- **TypeScript required** for all new code. No `.js` files except build configs.
- **Formatter:** Prettier via `pnpm format`. Always run after editing.
- **Naming:** `camelCase` for variables/functions/properties/hooks. `PascalCase` for components/classes/types/interfaces/enum members. `UPPER_SNAKE_CASE` for constants.
- **File names:** `kebab-case.ts` for utilities, `PascalCase.tsx` for React components (follow directory convention).
- **Types:** Interfaces over `type` aliases for object shapes. Discriminated unions for status branching. Interfaces mirror API schemas with camelCase field names.
- **React:** Functional components with hooks only. `React.FC` only when children needed. Derived state → custom hooks. Style via CSS modules or design system.
- **Exports:** Named exports preferred. Default exports only for Next.js pages/layouts.
- **Error handling:** Never swallow errors. Use Error boundaries. `try/catch` must log or surface.
- **Null handling:** Prefer `undefined` over `null`. Use `?.` and `??`.
- **No manual casing translation.** Use framework-native serialization.
- **HTML/CSS:** Semantic HTML. Keyboard accessibility. `alt` on images. CSS Modules preferred. No `!important`.
