# Зависимости

Описание основных используемых технологий и файлов конфигурации для формирования общего представления о созданном решении.

## 📋 Обзор зависимостей

Более побробно можно ознакомитсья в `package.json` файле в коревой папке проекта.

::: details Окружение

- **Typescript** <Badge type="tip" text="~5.8.3" /> - язык программирования
- **Node.js** <Badge type="tip" text="~22.16.0" /> - серверная среда выполнения

  :::

::: details Frontend

- **Vite** <Badge type="tip" text="^6.3.5" /> - сборщик проекта
- **VitePress** <Badge type="tip" text="^1.6.3" /> - документирование решения
- **Eslint** <Badge type="tip" text="^9.25.0" /> - проверка на ошибки до исполнения кода
- **Prettier** <Badge type="tip" text="^3.5.3" /> - форматирование кода
- **React** <Badge type="tip" text="^18.0.0" /> - создание UI компонентов
- **React DOM** <Badge type="tip" text="^18.0.0" /> - отрисовка react-компонентов в браузере
- **React Router** <Badge type="tip" text="^6.0.0" /> - маршрутизация на стороне клиента
- **Tailwind CSS** <Badge type="tip" text="^4.1.8" /> - стилизация копонентов
- **Radix UI** <Badge type="tip" text="^1.1.14" /> - использование готовых копонентов
- **Axios** <Badge type="tip" text="^1.10.0" /> - написание запросов на сервер
  :::

::: details Backend

- **Strapi** <Badge type="tip" text="5.16.0" /> - система управления контентом
  :::

## ⚙️ Файлы конфигурации

::: details Frontend
::: code-group

```ts [vite.config.ts]
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react-swc';
import path from 'path';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      '@': path.join(__dirname, './src'),
      '@app': path.join(__dirname, 'src/app'),
      '@styles': path.join(__dirname, 'src/styles'),
      '@pages': path.join(__dirname, 'src/pages'),
      '@features': path.join(__dirname, 'src/features'),
      '@entities': path.join(__dirname, 'src/entities'),
      '@widgets': path.join(__dirname, 'src/widgets'),
      '@shared': path.join(__dirname, 'src/shared'),
      '@store': path.join(__dirname, 'src/store'),
      '@api': path.join(__dirname, 'src/api'),
    },
  },
});
```

```js [.eslint.config.js]
import js from '@eslint/js';
import globals from 'globals';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  { ignores: ['dist'] },
  {
    extends: [js.configs.recommended, ...tseslint.configs.recommended],
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
    },
  },
);
```

```json [.prettierrc]
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./src/app/styles/index.css",
  "singleQuote": true,
  "printWidth": 80,
  "quoteProps": "preserve",
  "useTabs": false,
  "tabWidth": 2,
  "bracketSameLine": true,
  "arrowParens": "avoid",
  "singleAttributePerLine": true
}
```

```json [components.json]
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/app/styles/index.css",
    "baseColor": "neutral",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "iconLibrary": "lucide"
}
```

```json [tsconfig.json]
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@app/*": ["./src/app/*"],
      "@styles/*": ["./src/styles/*"],
      "@pages/*": ["./src/pages/*"],
      "@features/*": ["./src/features/*"],
      "@entities/*": ["./src/entities/*"],
      "@widgets/*": ["./src/widgets/*"],
      "@shared/*": ["./src/shared/*"],
      "@store/*": ["./src/store/*"],
      "@api/*": ["./src/api/*"]
    }
  }
}
```

:::

::: details Backend
Директория конфигурации `config/`
::: code-group

```ts [admin.ts]
export default ({ env }) => ({
  auth: {
    secret: env('ADMIN_JWT_SECRET'),
  },
  apiToken: {
    salt: env('API_TOKEN_SALT'),
  },
  transfer: {
    token: {
      salt: env('TRANSFER_TOKEN_SALT'),
    },
  },
  secrets: {
    encryptionKey: env('ENCRYPTION_KEY'),
  },
  flags: {
    nps: env.bool('FLAG_NPS', true),
    promoteEE: env.bool('FLAG_PROMOTE_EE', true),
  },
});
```

```ts [api.ts]
export default {
  rest: {
    defaultLimit: 25,
    maxLimit: 100,
    withCount: true,
  },
};
```

```ts [database.ts]
import path from 'path';

export default ({ env }) => {
  const client = env('DATABASE_CLIENT', 'sqlite');

  const connections = {
    mysql: {
      connection: {
        host: env('DATABASE_HOST', 'localhost'),
        port: env.int('DATABASE_PORT', 3306),
        database: env('DATABASE_NAME', 'strapi'),
        user: env('DATABASE_USERNAME', 'strapi'),
        password: env('DATABASE_PASSWORD', 'strapi'),
        ssl: env.bool('DATABASE_SSL', false) && {
          key: env('DATABASE_SSL_KEY', undefined),
          cert: env('DATABASE_SSL_CERT', undefined),
          ca: env('DATABASE_SSL_CA', undefined),
          capath: env('DATABASE_SSL_CAPATH', undefined),
          cipher: env('DATABASE_SSL_CIPHER', undefined),
          rejectUnauthorized: env.bool(
            'DATABASE_SSL_REJECT_UNAUTHORIZED',
            true,
          ),
        },
      },
      pool: {
        min: env.int('DATABASE_POOL_MIN', 2),
        max: env.int('DATABASE_POOL_MAX', 10),
      },
    },
    postgres: {
      connection: {
        connectionString: env('DATABASE_URL'),
        host: env('DATABASE_HOST', 'localhost'),
        port: env.int('DATABASE_PORT', 5432),
        database: env('DATABASE_NAME', 'strapi'),
        user: env('DATABASE_USERNAME', 'strapi'),
        password: env('DATABASE_PASSWORD', 'strapi'),
        ssl: env.bool('DATABASE_SSL', false) && {
          key: env('DATABASE_SSL_KEY', undefined),
          cert: env('DATABASE_SSL_CERT', undefined),
          ca: env('DATABASE_SSL_CA', undefined),
          capath: env('DATABASE_SSL_CAPATH', undefined),
          cipher: env('DATABASE_SSL_CIPHER', undefined),
          rejectUnauthorized: env.bool(
            'DATABASE_SSL_REJECT_UNAUTHORIZED',
            true,
          ),
        },
        schema: env('DATABASE_SCHEMA', 'public'),
      },
      pool: {
        min: env.int('DATABASE_POOL_MIN', 2),
        max: env.int('DATABASE_POOL_MAX', 10),
      },
    },
    sqlite: {
      connection: {
        filename: path.join(
          __dirname,
          '..',
          '..',
          env('DATABASE_FILENAME', '.tmp/data.db'),
        ),
      },
      useNullAsDefault: true,
    },
  };

  return {
    connection: {
      client,
      ...connections[client],
      acquireConnectionTimeout: env.int('DATABASE_CONNECTION_TIMEOUT', 60000),
    },
  };
};
```

```ts [middlewares.ts]
export default [
  'strapi::logger',
  'strapi::errors',
  'strapi::security',
  {
    name: 'strapi::cors',
    config: {
      origin: ['http://94.232.40.253', 'http://localhost:5174'],
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
      headers: '*',
    },
  },
  'strapi::poweredBy',
  'strapi::query',
  'strapi::body',
  'strapi::session',
  'strapi::favicon',
  'strapi::public',
];
```

```ts [server.ts]
export default ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  app: {
    keys: env.array('APP_KEYS'),
  },
});
```

```json [tsconfig.json]
{
  "compilerOptions": {
    "module": "CommonJS",
    "moduleResolution": "Node",
    "lib": ["ES2020"],
    "target": "ES2019",
    "strict": false,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "incremental": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "noEmitOnError": true,
    "noImplicitThis": true,
    "outDir": "dist",
    "rootDir": "."
  },
  "include": ["./", "./**/*.ts", "./**/*.js", "src/**/*.json"],

  "exclude": [
    "node_modules/",
    "build/",
    "dist/",
    ".cache/",
    ".tmp/",
    "src/admin/",
    "**/*.test.*",
    "src/plugins/**"
  ]
}
```

:::
