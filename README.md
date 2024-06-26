# mono-repo

## Install PNPM

```sh
npm install -g pnpm
```

## Init Repo

```sh
mkdir mono-repo && \
cd mono-repo && \
echo "node_modules\ndist" >> .gitignore && \
pnpm init
```

```sh
mkdir lib-web-ui && \
mkdir application-next-js-1 && \
mkdir application-create-react-app-1
```

```sh
touch pnpm-workspace.yaml
```

```yaml
packages:
  - "lib-web-ui"
  - "application-next-js-*"
  - "application-create-react-app-*"
```

## Add Common Dependencies

```sh
# add the dependencies to the workspace root

pnpm add react react-dom -w

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
+   "ui-component-web": "workspace:*"
  },
```

```sh
# link `ui-component-web` to pnpm-lock.yaml
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

## Write Demo Component

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

## Write Entry Files

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

### Build the Library

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

## Demo Application

```sh
mkdir demo && \
mkdir demo/src && \
touch demo/src/index.html && \
touch demo/src/index.css && \
touch demo/src/main.tsx
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
  <script type="module" src="/src/main.tsx"></script>
</body>

</html>
```

```ts
// main.tsx

import React from 'react';
import { createRoot } from 'react-dom/client'

import { Button } from 'ui-component-web'; // Import the Button component from the library

import './index.css';

import 'ui-component-web/dist/style.css'; // Import styles from the library

const container = document.getElementById('app') || document.body
const root = createRoot(container)

const App = () => (
  <div className="p-4">
    <h1 className="text-2xl font-bold mb-4">Component Library Demo</h1>
    <Button onClick={()=>alert('click')}>Click Me</Button>
  </div>
);

root.render(<App />)
```

```css
/* index.css */
html {
  background-color: aquamarine;
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

## Import Lib in CRA Projects

> CRA: create-react-app <https://create-react-app.dev/>

`pnpm-workspace.yaml`:

```diff
packages:
  ...
+ - "application-create-react-app"
```

```sh
pnpm install
```

```sh
npx create-react-app application-create-react-app && \
cd application-create-react-app
```

```js
// index.js
import 'ui-component-web/dist/style.css'
```

```js
// app.js
import { Button } from 'ui-component-web'
// ... <Button>click</Button>
```

```sh
npm start
```

## Separate Entries

Import on demand:

```diff
-import { Button } from 'ui-component-web'
+import Button from 'ui-component-web/button'
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

`main.tsx`:

```diff
...
-import { Button }  from 'ui-component-web';
+import Button  from 'ui-component-web/button';
...
-import 'ui-component-web/dist/style.css'
+import 'ui-component-web/style.css'
...
```

```sh
npm run dev
```

# Watch dev
