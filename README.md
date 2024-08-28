# cloudflare-remix-env-paths

Sample of switching import between Production and Development

- app/routes/\_index.tsx

```tsx
import { Test } from "#/Test";

export default function Index() {
  return <Test />;
}
```

- app/libs/Test01.tsx

```tsx
export const Test = () => <>Production</>;
```

- app/libs/Test02.tsx

```tsx
export const Test = () => <>Development</>;
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
        "#/Test": ["./app/libs/Test01"]
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
      "#/Test": ["./app/libs/Test02"]
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
