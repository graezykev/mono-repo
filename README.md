# mono-repo

## Install PNPM

```sh
npm install -g pnpm
```

## Init Mono Repo

```sh
mkdir mono-repo && \
cd mono-repo && \
echo "node_modules\ndist" >> .gitignore && \
pnpm init
```

```sh
mkdir lib-web-ui && \
mkdir demo-lib-web-ui && \
mkdir app-next-js-1 && \
mkdir app-create-react-app-1
```

```sh
touch pnpm-workspace.yaml
```

```yaml
packages:
  - "lib-web-ui"
  - "demo-lib-web-ui"
  - "app-next-js-*"
  - "app-create-react-app-*"
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

`tailwind.config.js`:

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
touch src/index.css
```

```js
// _utils/class-name.ts
export function classNames(...classes: string[]):string {
  return classes.filter(Boolean).join(' ');
}
```

## Write a Component

```js
// button/index.tsx
import React from 'react';
import { classNames } from '../_utils/class-names';

type ButtonProps = {
  children: React.ReactNode;
  type?: 'button' | 'submit' | 'reset';
  className?: string;
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
/* index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```js
// index.ts
import './index.css'; // Import TailwindCSS styles

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

## Write the Demo

```sh
touch index.html && \
touch index.tsx && \
touch index.css && \
mkdir demo && \
mkdir demo/button && \
touch demo/button/index.tsx && \
```

`index.html`:

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
// index.tsx

import React from 'react'
import { createRoot } from 'react-dom/client'

import ButtonDemo from './demo/button'

import './index.css' // Import Demo Page styles

// TODO: avoid duplicate import
import './src/index.css' // Import styles from the library

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
/* index.css */
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

## Separate Demo Project

```js
// tab.tsx
import React, { useState } from 'react';

type TabProps = {
  label: string;
  children: React.ReactNode;
};

type TabsProps = {
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

```js
// index.tsx
import { Tabs, Tab } from 'lib-web-ui';

    <h2>My Tabs Component</h2>

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

## Generate TypeScript Declarations

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
+     // tsconfigPath: path.resolve(__dirname, 'tsconfig.json'),
+     // outDir: path.resolve(__dirname, 'dist'),
+     insertTypesEntry: true
+   })
  ],
```

```sh
npm run build
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

## Import Lib in CRA Project

> CRA: create-react-app <https://create-react-app.dev/>

```sh
pnpm create react-app app-create-react-app-1 && \
cd app-create-react-app-1
```

```js
// index.js
import 'lib-web-ui/dist/style.css'
```

```js
// app.js
import { Button } from 'lib-web-ui'
// ... <Button>click</Button>
```

```sh
npm start
```

## Import On Demand - Separate Entries

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
+   "./style.css": {
+     "import": "./dist/style.css",
+     "require": "./dist/style.css"
+   },
+   ".": {
+     "import": "./dist/index.es.js",
+     "require": "./dist/index.cjs.js"
+   },
+   "./button": {
+     "import": "./dist/button.es.js",
+     "require": "./dist/button.cjs.js"
+   }
+ },
```

```sh
npm publish
```

Go to CRA project:

`index.js`:

```diff
-import 'lib-web-ui/dist/style.css'
+import 'lib-web-ui/style.css'
```

`app.js`:

```diff
-import { Button }  from 'lib-web-ui'
+import Button  from 'lib-web-ui/button'
```

```sh
npm start
```

### TODO: Import styles on demand

## Import Lib in Next.js Project

Initiate a `Next.js` project with TS, TailwinCSS

## Watch src Change in Next.js Projects

Add the source path of `lib-web-ui` to the `Next.js` project's TailwindCSS configuration file `tailwind.config.ts`:

> The content section of your tailwind.config.js file is where you configure the paths to all of your HTML templates, JavaScript components, and any other source files that contain Tailwind class names.

```diff
const config: Config = {
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

## Testing

## Documentation & Showcase

## Playground

## CLI: create component

- add source template
- add `export` js
- add `export` in package.json
- add alias in projects
