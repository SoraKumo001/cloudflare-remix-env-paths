# cloudflare-remix-env-paths

Sample of switching import between Production and Development

- app/prisma/index.ts

```ts
export * from "@prisma/adapter-pg-worker";
import pg from "@prisma/pg-worker";
export const Pool = pg.Pool;
```

- app/prisma/index.dev.ts

```ts
export * from "@prisma/adapter-pg";
import pg from "pg";
export const Pool = pg.Pool;
```

- app/routes/\_index.tsx

```tsx
import { LoaderFunctionArgs } from "@remix-run/cloudflare";
import { Pool, PrismaPg } from "~/prisma";
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

- vite.config.ts

```ts
import {
  vitePlugin as remix,
  cloudflareDevProxyVitePlugin as remixCloudflareDevProxy,
} from "@remix-run/dev";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
import path from "path";

export default defineConfig({
  resolve: {
    alias:
      process.env.NODE_ENV === "development"
        ? {
            "~/prisma": path.resolve(__dirname, "./app/prisma/index.dev"),
          }
        : undefined,
  },
  plugins: [
    remixCloudflareDevProxy(),
    remix({
      future: {
        v3_fetcherPersist: true,
        v3_relativeSplatPath: true,
        v3_throwAbortReason: true,
      },
    }),
    tsconfigPaths(),
  ],
});
```
