```js
import { antfu } from '@antfu/eslint-config'
import eslintPluginTailwindcss from 'eslint-plugin-tailwindcss'

export default antfu({
  react: {
    overrides: {
      'antfu/consistent-list-newline': ['error', {
        ArrayExpression: false,
        ArrayPattern: false,
      }],
      'react-refresh/only-export-components': 'off',
    },
  },
  typescript: true,
  stylistic: {
    overrides: {
      'array-element-newline': 'off',
      'style/jsx-self-closing-comp': ['error', {
        component: true,
        html: true,
      }],
    },
  },
}, [
  {
    rules: {
      'unused-imports/no-unused-vars': 'warn',
      'no-console': 'warn',
    },
  },
  {
  // 2. Optional: extend an existing config preset
    ...eslintPluginTailwindcss.configs.recommended,
    settings: {
    // 3. Define the tailwindcss settings with the MANDATORY cssConfigPath
      tailwindcss: {
        cssConfigPath: './src/styles/tailwind.css',
      },
    },
    // 4. Optional: customize the rules to your needs
    rules: {
      'tailwindcss/classnames-order': 'warn',
      'tailwindcss/no-arbitrary-value': 'warn',
      'tailwindcss/no-custom-classname': [
        'warn',
        { whitelist: ['custom\\-*'] },
      ],
      'tailwindcss/no-contradicting-classname': 'warn',
    },
  },
  {
    files: ['**/tsconfig.json', '**/tsconfig.*.json', 'tsconfig.json', 'tsconfig.*.json'],
    rules: {
      'jsonc/sort-keys': ['error', {
        pathPattern: '^$',
        order: ['compilerOptions', 'include', 'exclude'],
      }],
    },
  }])

```