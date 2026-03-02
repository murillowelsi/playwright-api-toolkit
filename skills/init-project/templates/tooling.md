# ESLint and Prettier Setup

Install and configure ESLint (with TypeScript and Playwright plugins) and Prettier as the formatter.

> Uses ESLint flat config (`eslint.config.js`) — required for ESLint v9+.

---

## Install

```bash
npm install --save-dev \
  eslint \
  @typescript-eslint/parser \
  @typescript-eslint/eslint-plugin \
  eslint-plugin-playwright \
  prettier \
  eslint-config-prettier
```

---

## `eslint.config.js`

```js
const tseslint = require('@typescript-eslint/eslint-plugin');
const tsParser = require('@typescript-eslint/parser');
const playwright = require('eslint-plugin-playwright');
const prettierConfig = require('eslint-config-prettier');

module.exports = [
  {
    ignores: [
      'node_modules/**',
      'allure-results/**',
      'allure-report/**',
      'playwright-report/**',
      'test-results/**',
    ],
  },
  {
    files: ['src/**/*.ts', 'utils/**/*.ts'],
    plugins: {
      '@typescript-eslint': tseslint,
    },
    languageOptions: {
      parser: tsParser,
      parserOptions: {
        project: './tsconfig.json',
      },
    },
    rules: {
      ...tseslint.configs.recommended.rules,
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/consistent-type-imports': ['error', { prefer: 'type-imports' }],
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    },
  },
  {
    ...playwright.configs['flat/recommended'],
    files: ['src/tests/**/*.ts'],
  },
  prettierConfig,
];
```

`prettierConfig` must be last — it disables all ESLint formatting rules that conflict with Prettier.

---

## `.prettierrc`

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

---

## Verify

```bash
npm run lint
npm run format:check
```

Both must pass before the setup is complete.
