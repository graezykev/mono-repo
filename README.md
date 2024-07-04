# mono-repo

## Install PNPM

```sh
npm install -g pnpm
```

## Init Mono Repo

```sh
mkdir mono-repo && \
cd mono-repo && \
echo "node_modules\ndist\n.DS_Store" >> .gitignore && \
pnpm init
```

```sh
mkdir lib-web-ui && \
mkdir demo-lib-web-ui && \
mkdir app-next-1 && \
mkdir app-cra-1
```

```sh
touch pnpm-workspace.yaml
```

```yaml
packages:
  - "lib-web-ui"
  - "demo-lib-web-ui"
  - "app-next-*"
  - "app-cra-*"
```

## Add Common Dependencies

```sh
# add the dependencies to the workspace root

pnpm add react react-dom next -w

pnpm add -D typescript -w

pnpm add -D @types/node @types/react @types/react-dom -w

pnpm add -D vite @vitejs/plugin-react vite-plugin-dts -w

pnpm add -D tailwindcss postcss autoprefixer -w
```

## PNPM link workspace

`package.json`:

```diff
  "dependencies": {
    ...
+   "lib-web-ui": "workspace:*"
  },
```

```sh
# link `lib-web-ui` to pnpm-lock.yaml
pnpm install
```

## Init UI Component Library Project

```sh
cd lib-web-ui && \
pnpm init
```

`package.json`:

```diff
+ "peerDependencies": {
+   "react": "^18.3.1",
+   "react-dom": "^18.3.1"
+ }
```

```sh
pnpm install
# add `importers` -> `lib-web-ui` to pnpm-lock.yaml
```

## Init TailwindCSS

```sh
npx tailwindcss init -p  --esm --ts
```

`tailwind.config.ts`:

```diff
  content: [
+   "./src/**/*.{js,jsx,ts,tsx}",
  ],
```

## Init Files

```sh
# create:
# - source directory `src`
# - entry js (to import and export all components)
# - entry css (to import TailwindCSS directives)
# - a demo component `Button`
# - helper js `_utils/...`
mkdir src && \
mkdir src/button && \
touch src/button/index.tsx && \
mkdir src/_utils && \
touch src/_utils/class-name.ts && \
touch src/index.ts && \
touch src/style.css
```

```js
// src/_utils/class-name.ts
export function classNames(...classes: string[]):string {
  return classes.filter(Boolean).join(' ');
}
```

## Write a Component

```js
// src/button/index.tsx
import React, { MouseEventHandler } from 'react'
import { classNames } from '../_utils/class-names'

export type ButtonProps = {
  children: React.ReactNode;
  type?: 'button' | 'submit' | 'reset';
  className?: string;
  onClick?: MouseEventHandler<HTMLButtonElement>;
};

const Button: React.FC<ButtonProps> = ({ children, type = 'button', className = '', ...props }) => {
  return (
    <button
      type={type}
      className={classNames(
        'px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm',
        'text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500',
        className
      )}
      {...props}
    >
      {children}
    </button>
  );
};

export default Button;
```

## Export the Component

```css
/* src/style.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```js
// src/index.ts
import './style.css'; // Import TailwindCSS styles

export { default as Button } from './button';
```

## Build the Library

```sh
touch vite.config.ts
```

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  build: {
    outDir: path.resolve(__dirname, 'dist'),
    lib: {
      entry: index: path.resolve(__dirname, 'src'),
      name: 'Web UI Library',
      fileName: (format, entryName) => `${entryName}.${format}.js`,
      formats: ['es', 'cjs']
    },
    rollupOptions: { // exclude basic libs (react, react-dom etc.) from building output
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
      },
    },
  },
  plugins: [
    react()
  ]
});
```

`package.json`:

```diff
  "scripts": {
    ...
+   "build": "vite build"
  },
```

```sh
npm run build
# check the output of `dist` folder
```

## TypeScript Declarations

Generate TypeScript Declarations

```sh
tsc --init # touch tsconfig.json
```

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "jsx": "react-jsx",
    "strict": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationDir": "./dist",
    "outDir": "./dist",
    "baseUrl": "./"
  },
  "include": [
    "src"
  ]
}
```

```sh
pnpm add -D vite-plugin-dts
```

`vite.config.ts`:

```diff
...

+import dts from 'vite-plugin-dts'

...
  plugins: [
    react(),
+   dts({
+     tsconfigPath: path.resolve(__dirname, 'tsconfig.json'),
+     outDir: path.resolve(__dirname, 'dist'),
+     // insertTypesEntry: true
+   })
  ],
```

```sh
npm run build
```

## Write the Demo

```sh
mkdir demo && \
touch demo/index.html && \
touch demo/index.tsx && \
touch demo/index.css && \
mkdir demo/button && \
touch demo/button/index.tsx
```

`demo/index.html`:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Component Library Demo</title>
</head>

<body>
  <div id="app"></div>
  <script type="module" src="index.tsx"></script>
</body>

</html>
```

```ts
// demo/index.tsx

import React from 'react'
import { createRoot } from 'react-dom/client'

import ButtonDemo from './demo/button'

import './src/index.css' // Import styles from the library

import './index.css' // Import styles specific to Demo Pages

const container = document.getElementById('app') || document.body
const root = createRoot(container)

const App = () => (
  <div>
    <ButtonDemo />
  </div>
)

root.render(<App />)
```

```css
/* demo/index.css */
html {
  background-color: aquamarine;
}
```

```js
// demo/button/index.tsx

import { Button } from '../../src' // Import the Button component from the library // TODO: tsconfig -> path

export default () => {
  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold mb-4">Button Library Demo</h1>
      <Button onClick={()=>alert('click')}>Click Me</Button>
    </div>
  )
}
```

`vite.config.ts`:

```diff
...
export default defineConfig({
+ root: path.resolve(__dirname, 'demo'),
  build: {
    ...
```

`package.json`:

```diff
  "scripts": {
    ...
+   "dev": "vite",
    ...
  },
```

```sh
npm run dev
```

## Write another Component

`lib-web-ui/src/tab/index.tsx`:

```js
// tab/index.tsx
import React, { useState } from 'react'

export type TabProps = {
  label: string;
  children: React.ReactNode;
};

export type TabsProps = {
  children: React.ReactElement<TabProps>[];
};

export const Tab: React.FC<TabProps> = ({ label, children }) => {
  return <div>{children}</div>;
};

export const Tabs: React.FC<TabsProps> = ({ children }) => {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      <div className="flex cursor-pointer border-b">
        {children.map((tab, index) => (
          <div
            key={index}
            className={`p-4 ${activeTab === index ? 'border-b-2 border-blue-500' : ''}`}
            onClick={() => setActiveTab(index)}
          >
            {tab.props.label}
          </div>
        ))}
      </div>
      <div className="p-4">
        {React.Children.map(children, (child, index) => {
          if (index === activeTab) return child;
          return null;
        })}
      </div>
    </div>
  );
};
```

`lib-web-ui/src/index.tsx`:

```diff
+export { Tab, Tabs } from './tab'
```

## Publish

`package.json`:

```diff
+ "type": "module",
+ "main": "dist/index.cjs.js",
+ "module": "dist/index.es.js",
+ "types": "dist/index.d.ts",
+ "files": [
+   "dist"
+ ],
```

```sh
npm publish
```

## Import On Demand - Multiple Entry Points

> we did not import the square method from the src/math.js module. That function is what's known as "dead code", meaning an unused export that should be dropped.

> In webpack, tree shaking works with both ECMAScript modules (ESM) and CommonJS, but it does not work with Asynchronous Module Definition (AMD) or Universal Module Definition (UMD).

```diff
-import { Button } from 'lib-web-ui'
+import Button from 'lib-web-ui/button'
```

To avoid importing all components in case projects using the library without **tree shaking**.

`vite.config.ts`:

```diff
...
-     entry: path.resolve(__dirname, 'src'),
+     entry: {
+       index: path.resolve(__dirname, 'src'),
+       button: path.resolve(__dirname, 'src/button'),
+       tab: path.resolve(__dirname, 'src/tab')
+     },
...
-     fileName: (format) => `index.${format}.js`,
+     fileName: (format, entryName) => `${entryName}.${format}.js`,
```

```sh
npm run build
```

`package.json`:

```diff
+ "exports": {
+   ".": {
+     "import": "./dist/index.es.js",
+     "require": "./dist/index.cjs.js",
+     "types": "./dist/index.d.ts"
+   },
+   "./style.css": {
+     "import": "./dist/style.css",
+     "require": "./dist/style.css"
+   },
+   "./button": {
+     "import": "./dist/button.es.js",
+     "require": "./dist/button.cjs.js",
+     "types": "./dist/button/index.d.ts"
+   },
+   "./tab": {
+     "import": "./dist/tab.es.js",
+     "require": "./dist/tab.cjs.js",
+     "types": "./dist/tab/index.d.ts"
+   }
+ },
```

```sh
npm publish
```

### TODO: Import styles on demand

## Separate Demo Project

```sh
pnpm init && \
touch vite.config.ts && \
mkdir src && \
touch src/index.html && \
touch src/index.tsx && \
touch src/index.css
```

`vite.config.ts`:

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
// import path from 'path';

export default defineConfig({
  plugins: [
    react()
  ]
});
```

`src/index.html`:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Component Library Demo</title>
</head>

<body>
  <div id="app"></div>
  <script type="module" src="/src/index.tsx"></script>
</body>

</html>
```

`src/index.tsx`:

```js
import React from 'react';
import { createRoot } from 'react-dom/client'

import Button  from 'ui-component-web/button'; // Import the Button component from the library
// import { Button }  from 'ui-component-web'; // Import the Button component in another way
import { Tabs, Tab } from 'ui-component-web'; // Import Tab component

import './index.css';

import 'ui-component-web/style.css' // Import the CSS from the library

const container = document.getElementById('app') || document.body
const root = createRoot(container)

const App = () => (
  <div className="p-4">

    <h2 className="text-2xl font-bold mb-4">Component Library Demo</h2>

    <Button onClick={()=>alert('click')}>Click Me</Button>

    <h2>My Tabs Component f</h2>

    <Tabs>
      <Tab label="Tab 1">
        <div>Content of Tab 1</div>
      </Tab>
      <Tab label="Tab 2">
        <div>Content of Tab 2</div>
      </Tab>
      <Tab label="Tab 3">
        <div>Content of Tab 3</div>
      </Tab>
    </Tabs>

  </div>
);

root.render(<App />)
```

`index.css`:

```css
html {
  background-color: aquamarine;
}
```

### Optional: Watch src code change

`vite.config.ts`:

```diff
export default defineConfig({
+ resolve: {
+   alias: {
+     'ui-component-web': path.resolve(__dirname, '../ui-component-web/src'),
+     'ui-component-web/index.css': path.resolve(__dirname, '../ui-component-web/src/index.css'),
+     'ui-component-web/button': path.resolve(__dirname, '../ui-component-web/src/button')
+   },
  },
```

Init TailwindCSS:

```sh
npx tailwindcss init -p  --esm --ts
```

`tailwind.config.ts`:

```diff
...
+import libBaseTailwindConfig from '../lib-web-ui/tailwind.config'
...
const config: Config = {
+ ...libBaseTailwindConfig
  content: [
    ...
+   "../lib-web-ui/src/**/*.{js,jsx,ts,tsx}"
  ],
```

## Import Lib in CRA (create-react-app) Project

> CRA: create-react-app <https://create-react-app.dev/>

```sh
pnpm create react-app app-cra-1 && \
cd app-cra-1
```

```js
// index.js
import 'lib-web-ui/dist/style.css'
```

```js
// app.js
import { Button } from 'lib-web-ui'
// import Button  from 'lib-web-ui/button' // alternative import
// ... <Button>click</Button>
```

```sh
npm start
```

## Import Lib in Next.js Project

Initiate a `Next.js` project with TS, TailwinCSS

## Watch src Change in Next.js Projects

Add the source path of `lib-web-ui` to the `Next.js` project's TailwindCSS configuration file `tailwind.config.ts`:

> The `content` section of your `tailwind.config.ts` file is where you configure the paths to all of your HTML templates, JavaScript components, and any other source files that contain Tailwind class names.

```diff
...
+import libBaseTailwindConfig from '../lib-web-ui/tailwind.config'
...
const config: Config = {
+ ...libBaseTailwindConfig // optimise this by deep merge
  content: [
    ...
+   "../lib-web-ui/src/**/*.{js,jsx,ts,tsx}"
  ],
```

By default, `Next.js` looks for the module `lib-web-ui` from `lib-web-ui/dist` which is specified in the `main` field of `lib-web-ui`'s package.json.

So, we need to configure module alias in `next.config.mjs` to resolve the module `lib-web-ui` from the library source folder:

```js
const nextConfig = {
  experimental: {
    // for `next dev --turbo`
    // not work for `next build`
    turbo: {
      resolveAlias: {
        'lib-web-ui': '../lib-web-ui/src',
        'lib-web-ui/button': '../lib-web-ui/src/button'
      },
    },
  },
  // for `next dev`
  // and for `next build`
  webpack: (config) => {
    config.resolve.alias['lib-web-ui'] = path.resolve(__dirname, '../lib-web-ui/src')
    config.resolve.alias['lib-web-ui/button'] = path.resolve(__dirname, '../lib-web-ui/src/button')
    return config;
  },
};
```

This also enables `Next.js` to watch the changes of `lib-web-ui/src` while developing.

Also, configure `tsconfig.json` to resolve modules correctly:

```diff
  "compilerOptions": {
    ...
    "paths": {
      ...
+     "lib-web-ui": [
+       "../lib-web-ui/src"
+     ],
+     "lib-web-ui/button": [
+       "../lib-web-ui/src/button"
+     ]
      ...
    }
```

## Documentation & Showcase

## Playground

## Testing

## CLI: create component

- add source template
- add `export` js
- add `export` in package.json
- add alias in projects

## Design Token

## Auto Change Version
