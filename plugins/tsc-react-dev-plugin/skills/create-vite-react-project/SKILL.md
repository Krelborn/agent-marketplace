---
name: create-vite-react-project
description: Scaffold a production-ready Vite + React + TypeScript project from scratch with Prettier, Vitest, React Testing Library, and VS Code settings pre-configured. Use this skill whenever the user wants to create a new React project, start a new Vite project, scaffold a TypeScript React app, set up a React project with testing, or mentions "new project", "vite project", "react project", "scaffold", or "bootstrap" in the context of creating a fresh web application. Also trigger when the user asks for a project template or starter with React and TypeScript.
---

# Create Vite React TypeScript Project

Scaffold a complete Vite + React + TypeScript project with Prettier, Vitest, Testing Library, and VS Code configuration.

## Prerequisites

- Node.js and npm must be installed
- The user must specify a project name (ask if not provided)

## Steps

Follow these steps **exactly in order**. Every dependency must be installed via `npm install` — never write to `package.json` directly. This ensures the latest versions are always used.

### 1. Scaffold the Vite project

```bash
npm create vite@latest <project-name> -- --template react-ts
cd <project-name>
npm install
```

Wait for each command to complete before proceeding.

### 2. Install and configure Prettier

```bash
npm install --save-dev prettier
```

Create `.prettierrc` in the project root:

```json
{
  "singleQuote": false,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 120
}
```

### 3. Configure ESLint with Prettier

Install the Prettier ESLint config to disable formatting rules that conflict with Prettier:

```bash
npm install --save-dev eslint-config-prettier
```

Edit `eslint.config.js` to import and add `eslint-config-prettier`. Add the import at the top:

```js
import eslintConfigPrettier from "eslint-config-prettier"
```

Then add `eslintConfigPrettier` to the `extends` array inside the files config object (after the existing extends entries):

```js
extends: [
  js.configs.recommended,
  tseslint.configs.recommended,
  reactHooks.configs.flat.recommended,
  reactRefresh.configs.vite,
  eslintConfigPrettier,
],
```

Also update the `globalIgnores` call to include the coverage directory:

```js
globalIgnores(["coverage", "dist"]),
```

### 4. Create VS Code workspace settings

Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### 5. Update .gitignore

Append the coverage output directory to `.gitignore`:

```
coverage
```

The Vite template ignores `.vscode` by default, but we want to commit our workspace settings. Add this **after** the `.vscode` ignore entry (or at the end of the file if `.vscode` isn't listed):

```
!.vscode/settings.json
```

### 6. Install Vitest and Testing Library

```bash
npm install --save-dev vitest jsdom @vitest/coverage-v8 @testing-library/dom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

### 7. Configure Vitest in vite.config.ts

Replace the contents of `vite.config.ts` with:

```ts
/// <reference types="vitest/config" />
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: ["./src/test/setupTests.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "html"],
    },
  },
})
```

### 8. Create the test setup file

Create `src/test/setupTests.ts`:

```ts
import "@testing-library/jest-dom/vitest"
```

### 9. Configure TypeScript

There are three tsconfig files to manage. The root `tsconfig.json` is a solution-style config that references the others.

**tsconfig.app.json** — Edit the existing file to **add** exclusions for test files. Add these entries to the `"exclude"` array (create the array if it doesn't exist):

```json
"exclude": [
  "src/**/*.test.ts",
  "src/**/*.test.tsx",
  "src/test/*"
]
```

Do NOT change any other settings in `tsconfig.app.json`.

**tsconfig.test.json** — Create this new file. It should copy the `compilerOptions` from `tsconfig.app.json` but add `"types": ["vitest/globals"]` to the compiler options, and include only test files:

```json
{
  "compilerOptions": {
    // Copy all compilerOptions from tsconfig.app.json, then add:
    "types": ["vitest/globals"]
  },
  "include": [
    "src/**/*.test.ts",
    "src/**/*.test.tsx",
    "src/test/*"
  ]
}
```

IMPORTANT: Read the current `tsconfig.app.json` first to copy its `compilerOptions` accurately. Do not guess or hardcode them — they may change between Vite versions.

**tsconfig.json** — Edit the root solution config to add a reference to `tsconfig.test.json`:

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" },
    { "path": "./tsconfig.test.json" }
  ]
}
```

### 10. Add test scripts to package.json

Use `npm pkg set` to add the scripts — do NOT edit `package.json` directly:

```bash
npm pkg set scripts.test="vitest run"
npm pkg set scripts.test:coverage="vitest run --coverage"
```

### 11. Strip boilerplate and create Hello World

**Delete** these files:
- `src/App.css`
- `src/index.css`
- `src/assets/react.svg`

**Replace** `src/App.tsx` with:

```tsx
function App() {
  return (
    <div>
      <h1>Hello World</h1>
    </div>
  )
}

export default App
```

**Replace** `src/main.tsx` with:

```tsx
import { StrictMode } from "react"
import { createRoot } from "react-dom/client"
import App from "./App"

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

### 12. Create the demo test

Create `src/App.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react"
import App from "./App"

describe("App", () => {
  it("renders hello world", () => {
    render(<App />)
    expect(screen.getByText("Hello World")).toBeInTheDocument()
  })
})
```

### 13. Format all files with Prettier

```bash
npx prettier --write .
```

### 14. Run the tests to verify everything works

```bash
npm test
```

Confirm the test passes. If it fails, diagnose and fix before reporting success.

### 15. Build the project

```bash
npm run build
```

Confirm the build succeeds with no errors. If it fails, diagnose and fix before reporting success.

## Completion

Once all steps are done and tests pass, inform the user that the project is ready and summarise what was set up:

- Vite + React + TypeScript scaffolded
- Prettier configured with format-on-save in VS Code, ESLint integrated with eslint-config-prettier
- Vitest with globals, jsdom environment, and v8 coverage
- React Testing Library with jest-dom matchers
- TypeScript configured with separate app and test configs
- Boilerplate stripped, Hello World app with passing test
- `npm test` and `npm run test:coverage` scripts ready