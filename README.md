# React + TypeScript + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Oxc](https://oxc.rs)
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/)

## React Compiler

The React Compiler is not enabled on this template because of its impact on dev & build performances. To add it, see [this documentation](https://react.dev/learn/react-compiler/installation).

## Expanding the ESLint configuration

If you are developing a production application, we recommend updating the configuration to enable type-aware lint rules:

```js
export default defineConfig([
  globalIgnores(['dist']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      // Other configs...

      // Remove tseslint.configs.recommended and replace with this
      tseslint.configs.recommendedTypeChecked,
      // Alternatively, use this for stricter rules
      tseslint.configs.strictTypeChecked,
      // Optionally, add this for stylistic rules
      tseslint.configs.stylisticTypeChecked,

      // Other configs...
    ],
    languageOptions: {
      parserOptions: {
        project: ['./tsconfig.node.json', './tsconfig.app.json'],
        tsconfigRootDir: import.meta.dirname,
      },
      // other options...
    },
  },
])
```


## Application Setup

### Technology Stack
- React 18.2.0
- TypeScript
- Vite 8.0.1
- Node.js 24.21.0
- npm 11.19.0

### Installation

```bash
npm install
```

### Development Server

```bash
npm run dev
```

### Local Application URL

http://localhost:5173/

### Production Build

```bash
npm run build
```

### Build Output

`dist/`

### Environment Variables

No environment variables are currently required.

### Application Validation

- Application successfully loaded in Google Chrome.
- HTTP response verified using `curl.exe -I http://localhost:5173`.
- HTTP Status: 200 OK.
- Production build completed successfully.

### Application Configuration Summary

| Configuration | Value |
|---|---|
| Framework | React |
| Build Tool | Vite |
| Development Port | 5173 |
| Start Command | npm run dev |
| Build Command | npm run build |
| Output Directory | dist |
| Environment Variables | None identified |
Webhook automatic build test
