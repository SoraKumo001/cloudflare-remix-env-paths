# cloudflare-remix-env-paths

Sample of switching import between Production and Development

- app/prisma/dev/index.ts

```ts
export * from "@prisma/adapter-pg";
import pg from "pg";
export const Pool = pg.Pool;
```

- app/prisma/prod/index.ts

```ts
export * from "@prisma/adapter-pg";
import pg from "pg";
export const Pool = pg.Pool;
```

- app/routes/\_index.tsx

```tsx
import { LoaderFunctionArgs } from "@remix-run/cloudflare";
import { Pool, PrismaPg } from "#prisma";
import { PrismaClient } from "@prisma/client";
import { useLoaderData } from "@remix-run/react";

export default function Index() {
  const values = useLoaderData<string[]>();
  return (
    <div>
      {values.map((v) => (
        <div key={v}>{v}</div>
      ))}
    </div>
  );
}

export async function loader({
  context,
}: LoaderFunctionArgs): Promise<string[]> {
  const pool = new Pool({
    connectionString: context.cloudflare.env.DATABASE_URL,
  });
  const adapter = new PrismaPg(pool);
  const prisma = new PrismaClient({ adapter });
  await prisma.test.create({ data: {} });
  return prisma.test.findMany().then((r) => r.map(({ id }) => id));
}
```

- tsconfig.json

  ```json
  {
    "include": [
      "**/*.ts",
      "**/*.tsx",
      "**/.server/**/*.ts",
      "**/.server/**/*.tsx",
      "**/.client/**/*.ts",
      "**/.client/**/*.tsx"
    ],
    "compilerOptions": {
      "lib": ["DOM", "DOM.Iterable", "ES2022"],
      "types": ["@remix-run/cloudflare", "vite/client"],
      "isolatedModules": true,
      "esModuleInterop": true,
      "jsx": "react-jsx",
      "module": "ESNext",
      "moduleResolution": "Bundler",
      "resolveJsonModule": true,
      "target": "ES2022",
      "strict": true,
      "allowJs": true,
      "skipLibCheck": true,
      "forceConsistentCasingInFileNames": true,
      "baseUrl": ".",
      "paths": {
        "~/*": ["./app/*"],
        "#prisma": ["./app/prisma/prod"]
      },

      // Vite takes care of building everything, not tsc.
      "noEmit": true
    }
  }
  ```

- tsconfig.dev.json

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "paths": {
      "#prisma": ["./app/prisma/dev"]
    }
  }
}
```

- vite.config.ts

```ts
import {
  vitePlugin as remix,
  cloudflareDevProxyVitePlugin as remixCloudflareDevProxy,
} from "@remix-run/dev";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [
    remixCloudflareDevProxy(),
    remix({
      future: {
        v3_fetcherPersist: true,
        v3_relativeSplatPath: true,
        v3_throwAbortReason: true,
      },
    }),
    // Switch tsconfig.js here
    tsconfigPaths({
      configNames:
        process.env.NODE_ENV === "production"
          ? ["tsconfig.json"]
          : ["tsconfig.dev.json"],
    }),
  ],
});
```
