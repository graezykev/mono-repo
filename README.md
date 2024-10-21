# mono-repo

## Install PNPM

```sh
npm install -g pnpm
```

## Init Mono Repo

```sh
mkdir mono-repo && \
cd mono-repo && \
echo "node_modules\ndist\n.DS_Store\n.tmp\n" >> .gitignore && \
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
+   "@designgreat/lib-web-ui": "workspace:*"
  },
```

```sh
# link `lib-web-ui` (@designgreat/lib-web-ui) to pnpm-lock.yaml
pnpm install
```

## Init UI Component Library Project

```sh
cd lib-web-ui && \
echo "registry = 'https://registry.npmjs.org/'" >> .npmrc && \
pnpm init
```

`package.json`:

```diff
- "name": "lib-web-ui",
+ "name": "@designgreat/lib-web-ui",
+ "peerDependencies": {
+   "react": "^18.3.1",
+   "react-dom": "^18.3.1"
+ }
```

```sh
pnpm install
# add `importers` -> `lib-web-ui` (@designgreat/lib-web-ui) to pnpm-lock.yaml
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
import React, { MouseEventHandler } from 'react';

interface ButtonProps {
  type?: 'primary' | 'default' | 'dashed' | 'link';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: MouseEventHandler<HTMLButtonElement>;
}

const Button: React.FC<ButtonProps> = ({
  type = 'default',
  disabled = false,
  children,
  onClick,
}) => {
  const baseStyles = 'px-4 py-2 rounded focus:outline-none';
  const typeStyles = {
    primary: 'bg-blue-500 text-white hover:bg-blue-700',
    default: 'bg-white border border-gray-300 text-gray-700 hover:bg-gray-100',
    dashed: 'bg-white border-2 border-dashed border-gray-300 text-gray-700 hover:bg-gray-100',
    link: 'bg-transparent text-blue-500 hover:bg-gray-100',
  };
  const disabledStyles = 'opacity-50 cursor-not-allowed';

  const styles = `${baseStyles} ${typeStyles[type]} ${disabled ? disabledStyles : ''}`;

  return (
    <button
      className={styles}
      onClick={disabled ? undefined : onClick}
      disabled={disabled}
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

import { Button } from '../../src' // Import the Button component from the library

export default () => {
  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold mb-4">Button Library Demo</h1>
      <div className="space-y-4">
        <Button type="primary" onClick={() => alert('Primary Button Clicked')}>
          Primary Button
        </Button>
        <Button type="default" onClick={() => alert('Default Button Clicked')}>
          Default Button
        </Button>
        <Button type="dashed" onClick={() => alert('Dashed Button Clicked')}>
          Dashed Button
        </Button>
        <Button type="link" onClick={() => alert('Link Button Clicked')}>
          Link Button
        </Button>
        <Button type="primary" disabled>
          Disabled Primary Button
        </Button>
        <Button type="default" disabled>
          Disabled Default Button
        </Button>
      </div>

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
}

export type TabsProps = {
  children: React.ReactElement<TabProps>[];
}

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
npm publish # npm publish --access public
```

## Import On Demand - Multiple Entry Points

> we did not import the square method from the src/math.js module. That function is what's known as "dead code", meaning an unused export that should be dropped.

> In webpack, tree shaking works with both ECMAScript modules (ESM) and CommonJS, but it does not work with Asynchronous Module Definition (AMD) or Universal Module Definition (UMD).

```diff
-import { Button } from '@designgreat/lib-web-ui'
+import Button from '@designgreat/lib-web-ui/button'
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

### Demo template

```sh
mkdir scripts &&
touch scripts/get-npm-latest-version.ts &&
mkdir demo-lib-web-ui &&
cd demo-lib-web-ui &&
pnpm init && \
touch vite.config.ts && \
touch tsconfig.json && \
mkdir src && \
touch index.html && \
touch index.tsx && \
touch src/index.css
```

`package.json`:

```diff
  "scripts": {
+   "dev": "vite",
+   "build": "vite build"
  },
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
    "baseUrl": "./",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  },
  "include": [
    "src",
    "*.tsx"
  ]
}
```

`vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import fs from 'fs'
import path from 'path'
import { getLatestVersion } from '../scripts/get-npm-latest-version'

const tempDir = '.tmp'
const intermediateFiles: string[] = []

const pages = getHtmlEntries()

export default defineConfig({
  root: path.resolve(__dirname, tempDir),
  // base: 'https://your-cnd.com', // default: '/'
  plugins: [
    react(),
    generateComponentPages()
  ],
  build: {
    // sourcemap: true,
    rollupOptions: {
      input: pages
    },
    outDir: path.resolve(__dirname, 'dist')
  },
  resolve: { alias: { "@": path.resolve("src") } }
})

const htmlTemplate = fs.readFileSync(path.resolve(__dirname, 'index.html'), 'utf-8')
const tsxTemplate = fs.readFileSync(path.resolve(__dirname, 'index.tsx'), 'utf-8')

function generateComponentPages() {
  return {
    name: 'generate-component-pages',
    config() {

      deletePath(tempDir)

      const componentsDir = path.resolve(__dirname, 'src')
      
      traverseDir(componentsDir, (filePath, relativeDir) => {
        if (filePath.endsWith('.tsx')) {
          const componentName = path.basename(filePath, '.tsx')
          const capitalizedComponentName = capitalizeFirstLetter(componentName)
          const componentPath = `${relativeDir}/${componentName}`

          // Create variation TSX and HTML
          const { jsFilePath, htmlFilePath } = createPageFiles(`${tempDir}/${relativeDir}`, `${componentName}`, capitalizedComponentName, componentPath)
          intermediateFiles.push(jsFilePath, htmlFilePath)
          createPlayGroundProject(componentPath, jsFilePath, htmlFilePath, filePath)
        }
      })

      // Generate list html (index.html) to link all demos
      const listHtmlPath = path.resolve(__dirname, tempDir, 'index.html')
      const listHtmlContent = generateListHtml(componentsDir)
      // console.log('\nlist HTML:\n\n', listHtmlContent)
      fs.writeFileSync(listHtmlPath, listHtmlContent)
      intermediateFiles.push(listHtmlPath)
    },
    async buildEnd() {
      // console.log('\nintermediate files:\n\n', intermediateFiles)
      intermediateFiles.forEach((dir) => {
        deletePath(dir);
      });
      deletePath(tempDir)
    }
  }
}

function createPageFiles(
  relativeDir: string,
  suffix: string,
  componentName: string,
  componentPath: string
) {
  suffix = suffix === 'index' ? suffix : suffix + '/index'

  const randomStr = generateRandomString(5)
  const jsFileId = `${suffix}-${randomStr}`
  const jsFileName = `${relativeDir}/${jsFileId}.tsx`
  const jsFilePath = path.resolve(__dirname, jsFileName)
  ensureDirectoryExistence(jsFilePath)

  // Adjust the component path to be relative to the 'src' directory and use lowercase for the path
  const jsContent = tsxTemplate
    .replace(/import\s+ButtonDemo\s+from\s+['"].*?['"];?\s*/, `import ${componentName} from '@/${componentPath}';\n`)
    .replace(/<ButtonDemo\s*\/>/, `<${componentName} />`)

  fs.writeFileSync(jsFilePath, jsContent)

  // Create the modified HTML content
  const htmlFileName = `${relativeDir}/${suffix}.html`
  const htmlFilePath = path.resolve(__dirname, htmlFileName)
  const htmlContent = htmlTemplate
    .replace('<script type="module" src="index.tsx"></script>', `<script type="module" src="index-${randomStr}.tsx"></script>`)

  ensureDirectoryExistence(htmlFilePath)
  fs.writeFileSync(htmlFilePath, htmlContent)

  return { jsFilePath, htmlFilePath }
}

const playgroundDistDir = 'playground'

async function createPlayGroundProject(
  componentPath: string,
  jsFilePath: string,
  htmlFilePath: string,
  sourceFilePath: string
) {
  let playgroundProjectName:string
  if (componentPath.endsWith('index')) {
    playgroundProjectName = componentPath.split('/')[0]
  } else {
    playgroundProjectName = componentPath.replace('/', '-')
  }
  const playgroundProjectDir = path.resolve(__dirname, playgroundDistDir, 'dist', playgroundProjectName)

  deletePath(playgroundProjectDir)

  const playgroundTemplatePath = path.resolve(__dirname, playgroundDistDir)

  const playgroundPackageJSONTemplate = path.resolve(playgroundTemplatePath, 'package.json')
  const playgroundPackageJSONTartget = path.resolve(playgroundProjectDir, 'package.json')

  const playgroundViteConfigTemplate = path.resolve(playgroundTemplatePath, 'vite.config.ts')
  const playgroundViteConfigTarget = path.resolve(playgroundProjectDir, 'vite.config.ts')

  const playgroundTsConfigTemplate = path.resolve(playgroundTemplatePath, 'tsconfig.json')
  const playgroundTsConfigTarget = path.resolve(playgroundProjectDir, 'tsconfig.json')

  const htmlEntryFileName = path.basename(htmlFilePath)
  const htmlEntryFileTargetPath = path.resolve(playgroundProjectDir, htmlEntryFileName)

  const jsEntryFileName = path.basename(jsFilePath)
  const jsEntryFileTargetPath = path.resolve(playgroundProjectDir, jsEntryFileName)

  const jsSourceFilePath = path.relative(__dirname, sourceFilePath)
  const jsSourceFileTargetPath = path.resolve(playgroundProjectDir, jsSourceFilePath)

  const cssSourceFilePath = path.relative(__dirname, 'src/index.css')
  const cssSourceFileTargetPath = path.resolve(playgroundProjectDir, 'src/index.css')

  // console.log(
  //   '------\n',
  //   componentPath, '\n',
  //   jsFilePath, '\n',
  //   htmlFilePath, '\n',
  //   sourceFilePath, '\n',
  //   htmlEntryFileTargetPath, '\n',
  //   jsEntryFileTargetPath, '\n',
  //   jsSourceFileTargetPath, '\n'
  // )

  ensureDirectoryExistence(htmlEntryFileTargetPath)  
  fs.copyFileSync(htmlFilePath, htmlEntryFileTargetPath)
  ensureDirectoryExistence(jsEntryFileTargetPath)
  fs.copyFileSync(jsFilePath, jsEntryFileTargetPath)
  ensureDirectoryExistence(jsSourceFileTargetPath)
  fs.copyFileSync(sourceFilePath, jsSourceFileTargetPath)
  ensureDirectoryExistence(cssSourceFileTargetPath)
  fs.copyFileSync(cssSourceFilePath, cssSourceFileTargetPath)

  await copyModifyPackageJSON(playgroundPackageJSONTemplate, playgroundPackageJSONTartget)

  fs.copyFileSync(playgroundViteConfigTemplate, playgroundViteConfigTarget)
  fs.copyFileSync(playgroundTsConfigTemplate, playgroundTsConfigTarget)

  generatePlaygroundJs(playgroundProjectDir)
}

async function copyModifyPackageJSON(playgroundPackageJSONTemplate, playgroundPackageJSONTartget) {
  // fs.copyFileSync(playgroundPackageJSONTemplate, playgroundPackageJSONTartget)
  const json = JSON.parse(fs.readFileSync(playgroundPackageJSONTemplate, 'utf-8'))
  const version = await getLatestVersion('@designgreat/ui-component-web')
  // console.log('latest version: ', version)
  json.dependencies['@designgreat/ui-component-web'] = `^${version}`
  fs.writeFileSync(playgroundPackageJSONTartget, JSON.stringify(json, null, 2))
}

function traverseDir(dir: string, callback: (filePath: string, relativeDir: string) => void) {
  fs.readdirSync(dir).forEach((file) => {
    const filePath = path.join(dir, file)
    const stats = fs.statSync(filePath)
    if (stats.isDirectory()) {
      traverseDir(filePath, callback)
    } else if (stats.isFile()) {
      const relativeDir = path.relative(path.resolve(__dirname, 'src'), filePath)
      callback(filePath, path.dirname(relativeDir))
    }
  })
}

function capitalizeFirstLetter(string: string) {
  return string.charAt(0).toUpperCase() + string.slice(1)
}

function getHtmlEntries() {
  const entries: Record<string, string> = {}
  traverseDir(path.resolve(__dirname, 'src'), (filePath, relativeDir) => {
    if (filePath.endsWith('.tsx')) {
      const componentName = path.basename(filePath, '.tsx')
      if (componentName === 'index' || !filePath.endsWith('index.tsx')) {
        const htmlPath = `${relativeDir}/${componentName === 'index' ? 'index' : `${componentName}/index`}`
        // const htmlPath = `${relativeDir}/${componentName}`
        const htmlFileName = `${htmlPath}.html`
        entries[componentName === 'index' ? relativeDir : `${relativeDir}/${componentName}`] = path.resolve(__dirname, tempDir, htmlFileName)
      }
    }
  })
  entries.index = path.resolve(__dirname, tempDir, 'index.html')
  // console.log('\nentries:\n\n', entries)
  return entries
}

function ensureDirectoryExistence(filePath: string) {
  const dirname = path.dirname(filePath)
  if (fs.existsSync(dirname)) {
    return true
  }
  ensureDirectoryExistence(dirname)
  fs.mkdirSync(dirname)
}

function generateListHtml(componentsDir: string) {
  let listHtmlContent = '<!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"/><meta name="viewport" content="width=device-width, initial-scale=1.0"/><title>Component List</title></head><body><h1>Component List</h1><ul>'
  traverseDir(componentsDir, (filePath, relativeDir) => {
    if (filePath.endsWith('.tsx')) {
      const componentName = path.basename(filePath, '.tsx')
      if (componentName !== 'index') {
        // const htmlPath = `${relativeDir}/${componentName}.html`
        const htmlPath = `${relativeDir}/${componentName}/`
        listHtmlContent += `<li><a href="${htmlPath}">${relativeDir}/${componentName}</a></li>`
      } else {
        const htmlPath = `${relativeDir}/`
        listHtmlContent += `<li><a href="${htmlPath}">${relativeDir}</a></li>`
      }
    }
  })
  listHtmlContent += '</ul></body></html>'
  return listHtmlContent
}

function deletePath(filePath: string) {
  if (fs.existsSync(filePath)) {
    if (fs.statSync(filePath).isDirectory()) {
      fs.readdirSync(filePath).forEach((file) => {
        const currentPath = path.join(filePath, file);
        deletePath(currentPath);
      });
      fs.rmdirSync(filePath);
    } else {
      fs.unlinkSync(filePath);
    }
  }
}

function generateRandomString(length) {
  const characters = 'abcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  const charactersLength = characters.length;
  for (let i = 0; i < length; i++) {
      result += characters.charAt(Math.floor(Math.random() * charactersLength));
  }
  return result;
}

// Helper function to check if the file is text-based
function isTextFile(filePath) {
  const textExtensions = ['.js', '.html', '.tsx', '.jsx', '.css', '.ts', '.json'];
  return textExtensions.includes(path.extname(filePath));
}

// Recursive function to read all files in the directory
function readFiles(folder) {

  function readFilesRecursively(dir, fileContents = {}) {
    const files = fs.readdirSync(dir);

    files.forEach(file => {
      const fullPath = path.join(dir, file);
      const relativePath = path.relative(folder, fullPath);

      if (fs.statSync(fullPath).isDirectory()) {
        readFilesRecursively(fullPath, fileContents);
      } else if (isTextFile(fullPath)) {
        try {
          const content = fs.readFileSync(fullPath, 'utf8');
          fileContents[relativePath] = content;
        } catch (err) {
          console.error(`Error reading file: ${relativePath}`, err);
        }
      }
    });

    return fileContents;
  }

  return readFilesRecursively(folder)
}

// Main function to write JSON into playground.js
function generatePlaygroundJs(directory) {
  const fileContents = readFiles(directory);
  // const output = `const fileContents = ${JSON.stringify(fileContents, null, 2)};\n\nexport default fileContents;\n`;

  const packageJson = JSON.parse(fileContents['package.json'])

  const output = `
import sdk from 'https://unpkg.com/@stackblitz/sdk@1/bundles/sdk.m.js';

sdk.embedProject(
  'embed', // document.body
  {
    title: 'React Starter',
    description: 'A basic React project',
    template: 'node',
    dependencies: ${JSON.stringify({
      ...packageJson.dependencies,
      ...packageJson.devDependencies
    }, null, 2)},
    files: ${JSON.stringify(fileContents,null, 2)},
  },
  {
    clickToLoad: true,
    openFile: 'index.html',
    terminalHeight: 50,
  },
)
`

  fs.writeFileSync(path.join(directory, 'stackblitz-sdk.js'), output, 'utf8');
  // console.log('playground js has been generated.');
}

```

`/scripts/get-npm-latest-version.ts`:

```ts
const https = require('https');

function fetchPackageInfo(packageName: string): Promise<{version: string}> {
  const url = `https://registry.npmjs.org/${packageName}/latest`;

  return new Promise((resolve, reject) => {
    https.get(url, (res) => {
      let data = '';

      // A chunk of data has been received.
      res.on('data', (chunk) => {
        data += chunk;
      });

      // The whole response has been received.
      res.on('end', () => {
        if (res.statusCode === 200) {
          try {
            const jsonResponse = JSON.parse(data);
            resolve(jsonResponse);
          } catch (error) {
            reject(new Error('Error parsing response'));
          }
        } else if (res.statusCode === 404) {
          reject(new Error(`Package "${packageName}" not found.`));
        } else {
          reject(new Error(`Failed to retrieve data. Status code: ${res.statusCode}`));
        }
      });
    }).on('error', (err) => {
      reject(new Error(`Error fetching the package: ${err.message}`));
    });
  });
}

export async function getLatestVersion(packageName): Promise<string | undefined> {
  try {
    const packageInfo = await fetchPackageInfo(packageName);
    console.log(`The latest version of ${packageName} is: ${packageInfo.version}`);
    return packageInfo.version
  } catch (error) {
    console.error(error.message);
  }
}

// // Example usage
// const packageName = process.argv[2];
// if (packageName) {
//   getLatestVersion(packageName);
// } else {
//   console.error('Please provide a package name as an argument.');
// }

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

`index.tsx`:

```tsx
import '@/index.css'
import '@designgreat/lib-web-ui/style.css' // Import the CSS from the library

import React from 'react'
import { createRoot } from 'react-dom/client'

import ButtonDemo from '@/button'
// import OtherDemo from './other-path'
// ...


const container = document.getElementById('app') || document.body
const root = createRoot(container)

const App = () => (
  <div className="p-4">

    <ButtonDemo />

    {/*
      <OtherDemo />
      ...
    */}

  </div>
);

root.render(<App />)

```

`src/index.css`:

```css
html {
  background-color: aquamarine;
}
```

### Entry files for Each Component and their Variations

```txt
├── src
│   ├── index.css
│   │
│   ├── button
│   │   ├── index.tsx
│   │   └── variation1.tsx
│   │
│   ├── other-component
│   │   ├── index.tsx
│   │   ├── variation1.tsx
│   │   └── variation2.tsx
│   │
│   └── tab
│       └── index.tsx
│
├── index.html
├── index.tsx
│
├── package.json
│
└── vite.config.ts
```

`src/button/index.tsx`:

```tsx
import React from 'react';

import Button from '@designgreat/lib-web-ui/button'; // Import the Button component from the library
// import { Button }  from '@designgreat/lib-web-ui'; // Import the Button component in another way

export default function Demo() {
  return (
    <>
      <h2 className="text-2xl font-bold mb-4">Button Component Demo</h2>

      <div className="space-y-4">

        <Button type="default" onClick={() => alert('Default Button Clicked')}>
          Default Button
        </Button>

        <Button type="primary" onClick={() => alert('Primary Button Clicked')}>
          Primary Button
        </Button>
        
        <Button type="dashed" onClick={() => alert('Dashed Button Clicked')}>
          Dashed Button
        </Button>

        <Button type="link" onClick={() => alert('Link Button Clicked')}>
          Link Button
        </Button>
        <Button type="primary" disabled>
          Disabled Primary Button
        </Button>
        <Button type="default" disabled>
          Disabled Default Button
        </Button>
      </div>
    </>
  )
}
```

`src/button/variation1.tsx`:

```tsx
import React from 'react';

import Button from '@designgreat/lib-web-ui/button'

export default function Demo() {
  return (
    <>
      <h2 className="text-2xl font-bold mb-4">Primary Button Component Demo</h2>

      <div className="space-y-4">
        <Button type="primary" onClick={() => alert('Primary Button Clicked')}>
          Primary Button
        </Button>
      </div>
    </>
  )
}
```

`src/tab/index.tsx`:

```tsx
import React from 'react';

import { Tabs, Tab } from '@designgreat/lib-web-ui'; // Import Tab component

export default function Demo() {
  return (
    <>
      <h2>Tab Component Demo</h2>

      <Tabs>
        <Tab label="Tab 1" a="f">
          <div>Content of Tab 13</div>
        </Tab>
        <Tab label="Tab 2">
          <div>Content of Tab 21</div>
        </Tab>
        <Tab label="Tab 3">
          <div>Content of Tab 3</div>
        </Tab>
      </Tabs>
    </>
  )
}
```

```sh
npm run dev
```

Visit <http://localhost:5173/>

```sh
npm run build
```

### playground dir

```sh
mkdir playground && \
touch playground/package.json && \
touch playground/tsconfig.json && \
touch playground/vite.config.ts
```

`playground/package.json`:

```json
{
  "name": "lib-web-ui-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.html",
  "scripts": {
    "start": "vite",
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "vite",
    "build": "vite build"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "@designgreat/lib-web-ui": "^1.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.14.9",
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.1",
    "vite": "^5.3.1",
    "typescript": "^5.5.2"
  },
  "stackblitz": {
    "installDependencies": true,
    "startCommand": "npm start"
  }
}
```

`playground/tsconfig.json`:

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
    "baseUrl": "./",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  },
  "include": [
    "src",
    "*.tsx"
  ]
}
```

`playground/vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [
    react()
  ],
  resolve: { alias: { "@": path.resolve("src") } }
})
```

### Optional: Watch src code change

`vite.config.ts`:

```diff
export default defineConfig({
+ resolve: {
+   alias: {
+     '@designgreat/lib-web-ui': path.resolve(__dirname, '../lib-web-ui/src'),
+     '@designgreat/lib-web-ui/index.css': path.resolve(__dirname, '../lib-web-ui/src/index.css'),
+     '@designgreat/lib-web-ui/button': path.resolve(__dirname, '../lib-web-ui/src/button')
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

`app-cra-1/package.json`:

```diff
  "dependencies": {
    ...
-   "react": "^18.3.1",
-   "react-dom": "^18.3.1"
  },
```

```js
// index.js
import '@designgreat/lib-web-ui/style.css'
```

```js
// app.js
import { Button } from '@designgreat/lib-web-ui'
// import Button  from '@designgreat/lib-web-ui/button' // alternative import
// ... <Button>click</Button>
```

```sh
npm start
```

## Import Lib in Next.js Project

Initiate a `Next.js` project with TS, TailwinCSS

`app-next-js-a/package.json`:

```diff
  "dependencies": {
-   "react": "^18",
-   "react-dom": "^18"
  },
  "devDependencies": {
-   "typescript": "^5",
-   "@types/node": "^20",
-   "@types/react": "^18",
-   "@types/react-dom": "^18",
-   "postcss": "^8",
-   "tailwindcss": "^3.4.1"
  }
```

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

By default, `Next.js` looks for the module `@designgreat/lib-web-ui` from `lib-web-ui/dist` which is specified in the `main` field of `@designgreat/lib-web-ui`'s package.json.

So, we need to configure module alias in `next.config.mjs` to resolve the module `@designgreat/lib-web-ui` from the library source folder `src`:

```js
const nextConfig = {
  experimental: {
    // for `next dev --turbo`
    // not work for `next build`
    turbo: {
      resolveAlias: {
        '@designgreat/lib-web-ui': '../lib-web-ui/src',
        '@designgreat/lib-web-ui/button': '../lib-web-ui/src/button'
      },
    },
  },
  // for `next dev`
  // and for `next build`
  webpack: (config) => {
    config.resolve.alias['@designgreat/lib-web-ui'] = path.resolve(__dirname, '../lib-web-ui/src')
    config.resolve.alias['@designgreat/lib-web-ui/button'] = path.resolve(__dirname, '../lib-web-ui/src/button')
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
+     "@designgreat/lib-web-ui": [
+       "../lib-web-ui/src"
+     ],
+     "@designgreat/lib-web-ui/button": [
+       "../lib-web-ui/src/button"
+     ]
      ...
    }
```

## Documentation

### Essential Parts

`README.md`:

- Introduction
- Usage
  - Basic Examples / Variations
  - With Code snippets
  - With explanations
  - Advanced usage scenarios
  - Live code editor - CodeSandbox / StackBlitz
- API
  - Props
  - Styling
    - Color
    - Sizes
    - CSS Classes
- Design Token ??
- Accessibility
- FAQs?
- Contributing?
- License?

### Document Generation

- build docs website from folder structure
  - sidebar structure
  - MDX
  - Creating projects with the SDK (stackblitz)

### Use Docusaurus

```sh
pnpm create docusaurus doc-lib-web-ui classic --typescript
```

`doc-lib-web-ui/package.json`:

```diff
  "dependencies": {
    ...
-   "react": "^18.0.0",
-   "react-dom": "^18.0.0"
  },
  "devDependencies": {
    ...
-   "typescript": "~5.2.2"
  }
```

`pnpm-workspace.yaml`:

```diff
+- "doc-lib-web-ui"
```

```sh
pnpm install
```

```sh
cd doc-lib-web-ui && \
pnpm start
```

Add code below to `doc-lib-web-ui/docs/tutorial-basics/create-a-document.md`:

```tsx
import Button  from '@designgreat/lib-web-ui/button'; // Import the Button component from the library
import '@designgreat/lib-web-ui/style.css' // Import the CSS from the library

<div className="space-y-4">
  <Button type="primary" onClick={() => alert('Primary Button Clicked')}>
    Primary Button
  </Button>
  <Button type="default" onClick={() => alert('Default Button Clicked')}>
    Default Button
  </Button>
  <Button type="dashed" onClick={() => alert('Dashed Button Clicked')}>
    Dashed Button
  </Button>
  <Button type="link" onClick={() => alert('Link Button Clicked')}>
    Link Button
  </Button>
  <Button type="primary" disabled>
    Disabled Primary Button
  </Button>
  <Button type="default" disabled>
    Disabled Default Button
  </Button>
</div>
```

`docusaurus.config.ts`:

```diff
+  url: 'https://graezykev.github.io',
+  baseUrl: '/design-system', // For GitHub pages deployment, it is often '/<projectName>/'
+  organizationName: 'graezykev', // Usually your GitHub org/user name.
+  projectName: 'design-system', // Usually your repo name.
```

```sh
npm run build
```

```sh
npm run serve
```

Creat GitHub repository named `design-system`

Generate new SSH key (Mac):

```sh
ssh-keygen -t ed25519 -C "your_github_account_email@example.com"
```

```sh
cat ~/.ssh/id_ed25519.pub
```

Add a new SSH key to your GitHub accoun: GitHub -> Settings -> SSH and GPG keys -> New SSH key -> Paste key from the last step

```sh
GIT_USER=graezykev USE_SSH=true npm run deploy
```

Go to GitHub repository `design-system`:

Settings -> Pages -> Build and deployment -> Source -> Deploy from a branch -> Branch -> `gh-pages`

Visit <https://graezykev.github.io/design-system/>

## Playground

### Automatic Script

Read all text files under the demo folder

Get all the path names & text contents into a JSON with the format below:

```json
{
  files: {
    'path/to/file1.js': 'the file content',
    'path/to/file2.json': 'the file content',
    'path/to/file3.tsx': 'the file content',
    'other path': 'the file content'
  }
}
```

Translate the JSON to `a HTML form element` / `some JS code` according to the Stackblitz SDK

Insert the form/code to the corresponding MDX file

Watch the file changes and make the equivalent change to the MDX file

## Design Token

### What

> This specification was published by the Design Tokens Community Group. It is not a W3C Standard nor is it on the W3C Standards Track. <https://tr.designtokens.org/format/#sotd>
> Design tokens are a community movement.

<https://www.youtube.com/watch?v=wtTstdiBuUk>

They help establish a **common vocabulary** (platform-agnostic) across organisations like Designer, Developer(Web, Native App, React Native etc.), PM etc.

- Simply put, design token is A protocol, A Design Language, to translate Design to Development.
  - nicknames
    - Designers "choose" and Developers "use"

A (Design) Token is an information associated with a name, at minimum a **name/value** pair.

For example:

```css
color-text-primary: #000000;
font-size-heading-level-1: 44px;
```

- Manintain high consistency across product UI.
  - update (the value of the nickname)

- Compartmentalise
  - global token / atomic token
  - alias/semantic token

### Categary & Type

- Categary & Type
  - Color
    - font
    - background
    - border
    - outline
  - Size/dimension
    - font
    - width
    - height
    - border radius
  - Duration/Time
  - Number
    - line height
    - z index
  - Font Family
  - Font Weight
  - Font Style?
  - Cubic Bézier
  - Asset/File/Path/URL
    - fonts
    - svgs
    - icon fonts
  - others
- Composite Types
  - typography
    - Font Family
    - Font Size
    - Font Weight
    - Line Height
    - vertical align
    - Letter Spacing / Word Spacing
    - Text Color
    - Text Alignment
    - Text Decoration
    - Text Transform
    - Whitespace
    - word break
    - Margins / Paddings
    - text overflow
    - line clamp
    - text shadow
  - space
  - grid / layout
  - bg
  - border
    - width
    - style
    - color
  - box shadow
  - icon

### Design Token Structure

```js
const REM = 16

const Font_Family_Fallback = 'sans-serif'

const Font_Size_Base = REM // 16
const Font_Size_Medium_Small = Font_Size_Base * .875 // 12
const Font_Size_Small = Font_Size_Base * .75 // 12
const Font_Size_Heading1 = Font_Size_Base * 2.75 // 44
// Font_Size_Heading2~6 ...

const Font_Weight_Normal = 400
const Font_Weight_Bold = 700

const Text_Color_Base = '#000000'
const Background_Color_Base = '#FFFFFF'

const StyleDictionary = {
  color: {
    text: {
      base: {
        value: Text_Color_Base
      },
      link: { // LVHA
        link: {},
        visited: {},
        hover: {},
        active: {}
      }
    },
    decoration: {
      text: {
        base: {
        }
      }
    },
    background: {
      base: {
        value: Background_Color_Base
      },
      button: {
        primary: {
          hover: {},
          focus: {},
          active: {},
          disabled: {}
        }
      }
    },
    border: {},
    shadow: {
      text: {
        x: {},
        y: {}
      },
      box: {
        x: {},
        y: {}
      }
    }
  },
  size: {
    font: {
      base: {
        value: Font_Size_Base
      },
      paragraph: {
        value: Font_Size_Base
      },
      heading1: {
        value: Font_Size_Heading1
      },
      heading2: {}
    },
    height: {
      line: {
        base: {
          value: 1.15
        },
        paragragh: {
          value: 1.5
        },
        paragraghSparse: {
          value: 1.7
        },
        elementary: {
          value: 1
        },
        heading: {
          value: 1.2
        }
      },
      box: {}
    },
    width: {
      box: {},
      border: {},
      outline: {}
    },
    radius: {
      border: {}
    },
    thickness: {
      decoration: {
        text: {
          base: {
            value: 'auto' // default
          },
          fixed5: {
            value: 5 // px
          }
        }
      }
    },
    spacing: {
      letter: {
        base: {
          value: 'normal'
        }
      },
      word: {
        base: {
          value: 'normal'
        }
      },
      indent: {
        base: {
          value: 0
        }
      },
      margin: {},
      padding: {},
      position: {
        left: {},
        top: {},
        right: {},
        bottom: {},
        z: {}
      },
      offset: {
        outline: {
          value: 1
        },
        shadow: {
          text: {
            x: {},
            y: {},
            blurRadius: {}
          },
          box: {
            x: {},
            y: {},
            blurRadius: {},
            spreadRadius: {}
          }
        }
      }
    },
    lineClamp: {
      base: {
        value: 'none'
      },
      singleLine: {
        value: 1
      }
    },
    flexGrow: {
      base: {
        value: 0 // default
      }
    },
    flexShrink: {
      base: {
        value: 1 // default
      }
    },
    translate: {
      transform: {
        x: {},
        y: {},
        z: {},
        '3d': {}
      }
    },
    scale: {
      transform: {
        x: {
          base: {
            value: 1
          }
        },
        y: {},
        z: {},
        '3d': {}
      }
    },
    rotate: {
      transform: {
        x: {
          base: {
            value: '0deg'
          }
        },
        y: {},
        z: {},
        '3d': {}
      }
    },
    skew: {
      transform: {
        x: {
          base: {
            value: '0deg'
          }
        },
        y: {}
      }
    },
    perspective: {
      transform: {
        base: {
          value: 'none'
        }
      }
    },
    matrix: {
      transform: {
        a: {}, b: {}, c: {}, d: {}, tx: {}, ty: {}
      }
    }
  },
  fontFace: {
    base: {
      value: Font_Family_Fallback
    },
    Latin: {},
    zh_Hans: {},
    zh_Hans_SG: {},
    zh_Hant: {},
    zh_Hant_HK: {},
    zh_Hant_TW: {},
    Arabic: {},
    zh: {},
    Greek: {},
    Vietnamese: {},
    Hebrew: {}
  },
  fontFamily: {
    base: {
      value: Font_Family_Fallback
    },
    paragraph: {}
  },
  fontWeight: {
    base: {
      value: Font_Weight_Normal
    },
    bold: {
      value: Font_Weight_Bold
    }
  },
  textTransform: {
    base: {
      value: none // uppercase lowercase capitalize
    }
  },
  fontStretch: {
    base: {
      value: 'normal' // normal=100% condensed=75% expanded=125% ultra-expanded=200%
    }
  },
  align: {
    text: {
      horizontal: {
        centered: {
          value: 'center'
        }
      },
      vertical: {
        base: {
          value: 'baseline'
        },
        middle: {
          value: 'middle'
        },
        Superscript: {
          value: 'supper'
        },
        subscript: {
          value: 'sub'
        }
      }
    },
    box: {
      container: {
        mainAxis: { // justify-content
          // flex-start flex-end space-around space-between
          centered: {
            value: 'center'
          }
        },
        crossAxis: {
          nowrap: { // align-items
            // flex-start flex-end stretch baseline
            centered: {
              value: 'center'
            }
          },
          wrap: { // align-content
            // flex-start flex-end space-around space-between stretch
            centered: {
              value: 'center'
            }
          }
        }
      },
      items: { // align-self -- overrides --> align-items
        centered: {
          value: 'center'
        }
      }
    }
  },
  wordBreak: {
    base: {
      value: 'normal'
    },
    breakAll: {
      value: 'break-all'
    },
    breakWord: {
      value: 'break-word'
    }
  },
  whiteSpace: {
    base: {
      value: 'normal'
    },
    singleLine: {
      value: 'nowrap'
    },
    preserved: {
      value: 'pre'
    }
  },
  style: {
    font: {
      base: {
        value: 'normal'
      },
      annotation: {
        value: 'italic'
      },
      special: {
        value: 'oblique'
      }
    },
    border: {
      base: {
        value: 'solid'
      }
    },
    outline: {
      base: {
        value: 'none'
      }
    },
    decoration: {
      text: {
        underline: {
          value: 'solid'
        }
      }
    },
    list: {
      value: 'disc' // none circle square upper-roman lower-alpha
    }
  },
  position: {
    box: {},
    listStyle: {
      value: 'outside' // inside
    },
    line: {
      decoration: {
        text: {
          base: {
            value: 'none'
          },
          overline: {},
          underline: {},
          lineThrough: {
            value: 'line-through'
          }
        }
      }
    },
  },
  overflow: {
    box: {},
    text: {
      base: {
        value: 'ellipsis'
      },
      ellipsis: {
        value: 'ellipsis'
      }
    }
  },
  transition: {},
  asset: {}
}

const ellipsisWithLineClamp = (num) => {
  'text-overflow': 'ellipsis',
  overflow: 'hidden',
  'white-space': num <= 1 ? 'nowrap' : 'normal',
  display: '-webkit-box', // must
  '-webkit-line-clamp': num, // must
  '-webkit-box-orient': 'vertical' // must
}
```

### Translation Tools

Create a separate Figma file for your design tokens <https://diez.org/getting-started/figma.html>

#### Style Dictionary

```sh
cd lib-web-ui
```

```sh
# pnpm add -D style-dictionary@4.1.3
```

```sh
mkdir style-dictionary && cd style-dictionary
```

```sh
npx style-dictionary@4.1.3 init complete
```

<!-- rm -rf android ios README.md StyleDictionary.podspec LICENSE package.json -->

Use js because it's more programmable and extensible.

```sh
mv config.json sd.config.js # touch sd.config.js
```

`package.json`:

```diff
  "scripts": {
-   "build": "style-dictionary build",
+   "build": "style-dictionary build --config ./sd.config.js",
```

`sd.config.js`:

```js
export default {
  "source": ["tokens/**/*.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "buildPath": "css/",
      "prefix": "token",
      "files": [
        {
          "destination": "variables.css",
          "format": "scss/variables"
        }
      ],
      "actions": ["copy_assets"]
    },
    "jsts": {
      "transformGroup": "js",
      "buildPath": "jsts/",
      "files": [
        {
          "destination": "variables.js",
          "format": "javascript/module"
        },
        {
          "format": "typescript/module-declarations",
          "destination": "variables.d.ts"
        }
      ]
    },
    "ios": {
      "transformGroup": "ios",
      "buildPath": "ios/Classes/Generated/",
      "prefix": "StyleDictionary",
      "files": [
        {
          "destination": "StyleDictionarySize.h",
          "format": "ios/static.h",
          "options": {
            "className": "StyleDictionarySize",
            "type": "float"
          },
          "filter": {
            "attributes": {
              "category": "size"
            }
          }
        },
        {
          "destination": "StyleDictionarySize.m",
          "format": "ios/static.m",
          "options": {
            "className": "StyleDictionarySize",
            "type": "float"
          },
          "filter": {
            "attributes": {
              "category": "size"
            }
          }
        },
        {
          "destination": "StyleDictionaryIcons.h",
          "format": "ios/strings.h",
          "options": {
            "className": "StyleDictionaryIcons"
          },
          "filter": {
            "attributes": {
              "category": "content",
              "type": "icon"
            }
          }
        },
        {
          "destination": "StyleDictionaryIcons.m",
          "format": "ios/strings.m",
          "options": {
            "className": "StyleDictionaryIcons"
          },
          "filter": {
            "attributes": {
              "category": "content",
              "type": "icon"
            }
          }
        },
        {
          "destination": "StyleDictionaryColor.h",
          "format": "ios/colors.h",
          "options": {
            "className": "StyleDictionaryColor",
            "type": "StyleDictionaryColorName"
          },
          "filter": {
            "attributes": {
              "category": "color"
            }
          }
        },
        {
          "destination": "StyleDictionaryColor.m",
          "format": "ios/colors.m",
          "options": {
            "className": "StyleDictionaryColor",
            "type": "StyleDictionaryColorName"
          },
          "filter": {
            "attributes": {
              "category": "color"
            }
          }
        },
        {
          "destination": "StyleDictionaryProperties.h",
          "format": "ios/singleton.h",
          "options": {
            "className": "StyleDictionaryProperties"
          }
        },
        {
          "destination": "StyleDictionaryProperties.m",
          "format": "ios/singleton.m",
          "options": {
            "className": "StyleDictionaryProperties"
          }
        }
      ]
    },

    "android": {
      "transformGroup": "android",
      "buildPath": "android/styledictionary/src/main/res/values/",
      "files": [
        {
          "destination": "style_dictionary_colors.xml",
          "format": "android/colors"
        },
        {
          "destination": "style_dictionary_font_dimens.xml",
          "format": "android/fontDimens"
        },
        {
          "destination": "style_dictionary_dimens.xml",
          "format": "android/dimens"
        },
        {
          "destination": "style_dictionary_integers.xml",
          "format": "android/integers"
        },
        {
          "destination": "style_dictionary_strings.xml",
          "format": "android/strings"
        }
      ]
    },

    "android-asset": {
      "transformGroup": "android",
      "buildPath": "android/styledictionary/src/main/",
      "files": [
        {
          "destination": "assets/data/properties.json",
          "format": "json"
        }
      ],
      "actions": ["copy_assets"]
    }
  }
}

```

```sh
npm run build
# npx style-dictionary@4.1.3 build --config ./sd.config.js
```

##### Create 1st Design Token

```sh
rm -rf tokens && \
mkdir tokens && \
mkdir tokens/color
```

```sh
touch tokens/color/base.js
```

Use js to because it's more programmable and extensible.

`tokens/color/base.js`:

```js
export default {
  color: {
    base: {
      "white": { "value": "white", "type": "color" },
      "black": { "value": "#000000", "type": "color" }
    }
  }
}
```

Edit `sd.config.js`:

```diff
- "source": ["tokens/**/*.json"],
+ "source": ["tokens/**/*.json", "tokens/**/*.js"],
```

```sh
npm run build
```

##### Add Colors with HSL, RGBA, HSV

```diff
export default {
  color: {
    base: {
      ...
+     "red": { "value": "hsl(0, 100%, 50%)", "type": "color" },
+     "green": { "value": "#00ff00", "type": "color" },
+     "blue": { "value": "hsv(240, 100%, 100%)", "type": "color" },
+     "blue-transparent-50": { "value": "rgba(0, 0, 255, 50%)", "type": "color" },
+     "red-transparent-50": { "value": "#ff000080", "type": "color" },
```

`sd.config.js`:

```diff
    "jsts": {
-     "transformGroup": "js",
+     // "transformGroup": "js", // ['attribute/cti', 'name/pascal', 'size/rem', 'color/hex'],
+     "transforms": ['attribute/cti', 'name/pascal', 'size/rem', 'color/css'],
```

```sh
npm run build
```

### Color Palette

#### Base Colors

`tokens/color/base.js`:

```js
export default {
  color: {
    base: {
      "red": { "value": "#AE2E24", "type": "color" },
      "orange": { "value": "#A54800", "type": "color" },
      "yellow": { "value": "#7F5F01", "type": "color" },
      "green": { "value": "#216E4E", "type": "color" },
      "teal": { "value": "#206A83", "type": "color" },
      "blue": { "value": "#0055CC", "type": "color" },
      "purple": { "value": "#5E4DB2", "type": "color" },
      "magenta": { "value": "#943D73", "type": "color" },
      "lime": { "value": "#4C6B1F", "type": "color" }
    }
  }
}
```

#### Accent Colors / Color Shades

Derive from base colors.

Different shades of the base color.

`tokens/color/accent/blue.js`:

```js
export default {
  color: {
    accent: {
      "blue": { // https://mdigi.tools/color-shades/#0055cc
        "1": { "value": "#e5f0ff", "type": "color" },
        "2": { "value": "#b7d5ff", "type": "color" },
        "3": { "value": "#88baff", "type": "color" },
        "4": { "value": "#599eff", "type": "color" },
        "5": { "value": "#2a83ff", "type": "color" },
        "6": { "value": "#0068fb", "type": "color" },
        "7": { "value": "#0055CC", "type": "color" },
        "8": { "value": "#003c90", "type": "color" },
        "9": { "value": "#002355", "type": "color" },
        "10":{ "value": "#000b19", "type": "color" },
        "default": { "value": "{color.accent.blue.7}", "type": "color" }
      }
    }
  }
}

```

##### Auto-Generate Accent Colors / Color Shades

```sh
npm install tinycolor2
```

```sh
mkdir utils && touch utils/index.js
```

`utils/index.js`:

```js
import tinycolor2 from 'tinycolor2'

export function generateColorShades(
  name,
  colors,
  totalShades = 10,
  defaultShade = 7,
  darkestLightness = 0.05,
  lightestLightness = 0.95,
) {
  const darkerShades = totalShades - defaultShade
  const lighterShades = defaultShade - 1

  const rst = {}

  const value = colors[name].value
  rst[`${defaultShade}`] = value
  rst.default = rst[`${defaultShade}`]

  const color = tinycolor2(value)

  // console.warn('=============', name, value, color.toHsl(), color.toHslString(), color.getBrightness(), color.getAlpha())

  const hsl = color.toHsl()
  const lightness = hsl.l
  const lightGap = (lightestLightness - lightness) / lighterShades
  const darkGap = (lightness - darkestLightness) / darkerShades

  for (let from = 1; from <= darkerShades; from++) {
    const l = lightness - from * darkGap
    const newColor = tinycolor2(Object.assign({}, hsl, { l }))
    // console.log(newColor.toHex())
    rst[`${defaultShade + from}`] = `#${newColor.toHex()}`
  }

  for (let from = 1; from <= lighterShades; from++) {
    const l = lightness + from * lightGap
    const newColor = tinycolor2(Object.assign({}, hsl, { l }))
    // console.log(newColor.toHex())
    rst[`${defaultShade - from}`] = `#${newColor.toHex()}`
  }

  // console.log(rst)

  return rst
}

```

`tokens/color/accent/blue.js`:

```js
import tokens from '../base.js'
import { generateColorShades } from '../../../utils/index.js'

const name = 'blue'
const colors = tokens.color.base

const shades = generateColorShades(name, colors)
// console.log(shades)
const accents = Object.keys(shades).reduce((acc, level) => ({
  ...acc,
  [level]: {
    value: shades[level],
    type: 'color'
  }
}), {})
// console.log(accents)

export default {
  color: {
    accent: {
      [name]: accents
    }
  }
}

```

##### Auto-Generate Accent Colors from all Base Colors

```sh
rm tokens/color/accent/blue.js && \
touch tokens/color/accent/base.js
```

```js
import tokens from '../base.js'
import { generateColorShades } from '../../../utils/index.js'

const colors = tokens.color.base

const accent = Object.keys(colors).reduce((accent, name) => ({
  ...accent,
  [name]: (function () {
    const shades = generateColorShades(name, colors)
    // console.log(shades)
    const accents = Object.keys(shades).reduce((acc, level) => ({
      ...acc,
      [level]: {
        value: shades[level],
        type: 'color'
      }
    }), {})
    return accents
  }())
}), {})

// console.log(accent)

export default {
  color: {
    accent
  }
}

```

### Color roles

#### Brand/Primary Colors

The brand’s identity/overall look/feel.

Color meanings in branding <https://blog.tubikstudio.com/color-in-design-influence-on-users-actions/>

- Red — Confidence, youth, and power
- Blue — Trust, security, and stability
- Black — Reliable, sophisticated, and experienced
- ...

Usually choose 1 - 3 colors.

For example, Apple's website.

#### Secondary Colors

Used to support the primary colors.

- information - informative UI, information icon, progress etc.
- input - form fields
- discovery - something new
- inverse - UI elements that sit on bold emphasis backgrounds

##### Interaction states

hovered, pressed, selected, focused, or disabled

#### Semantic Colors

- Success - is often associated with **green**
- Warning - orange
- Error / Fail - red

#### Neutral Colors

- text
- backgrounds
- shapes
  - border
  - shadow
- disabled states

```sh
touch tokens/color/base-grey.js
```

`tokens/color/base-grey.js`:

```js
export default {
  color: {
    base: {
      "grey": { "value": "#172B4D", "type": "color" }
    }
  }
}

```

`tokens/color/accent/neutral.js`:

```js
export default {
  color: {
    accent: {
      "neutral": {
        "1": { "value": "#ffffff", "type": "color" },
        "2": { "value": "#dfe8f6", "type": "color" },
        "3": { "value": "#c0d0ec", "type": "color" },
        "4": { "value": "#a0b9e3", "type": "color" },
        "5": { "value": "#81a2d9", "type": "color" },
        "6": { "value": "#618ad0", "type": "color" },
        "7": { "value": "#4273c6", "type": "color" },
        "8": { "value": "#3360ac", "type": "color" },
        "9": { "value": "#2a4e8c", "type": "color" },
        "10": { "value": "#203d6d", "type": "color" },
        "11": { "value": "{color.base.grey}", "type": "color" }, // reference
        "12": { "value": "#060b14", "type": "color" },
        "default": { "value": "{color.accent.neutral.11}", "type": "color" } // reference
      }
    }
  }
}

```

##### Auto-Generate Accent Colors for Neutral Colors from Grey Color

`tokens/color/accent/neutral.js`:

```js
import tokens from '../base-grey.js'
import { generateColorShades } from '../../../utils/index.js'

const name = 'grey' // Auto-Generate Accent Colors for Neutral Colors from Grey Color

const colors = tokens.color.base

const shades = generateColorShades(name, colors, 12, 11, 0.05, 1) // make the lightest color white
// console.log(shades)
const accents = Object.keys(shades).reduce((acc, level) => ({
  ...acc,
  [level]: {
    value: shades[level],
    type: 'color'
  }
}), {})
// console.log(accents)

export default {
  color: {
    accent: {
      neutral: accents
    }
  }
}

```

#### Alpha Colors

Transparency or opacity. Transparency helps UI adapt to different background colors and elevations.

`tokens/color/alpha/neutral.js`:

```js
export default {
  color: {
    alpha: {
      "neutral": {
        "100": { "value": "#070a0e08", "type": "color" }, // "500": { "value": "{color.accent.neutral.1100}", "attributes": { "alpha": 0.03 } "type": "color" }
        "200": { "value": "#070a0e0F", "type": "color" }, // "500": { "value": "{color.accent.neutral.1100}", "attributes": { "alpha": 0.06 } "type": "color" }
        "300": { "value": "#070a0e24", "type": "color" }, // "500": { "value": "{color.accent.neutral.1100}", "attributes": { "alpha": 0.14 } "type": "color" }
        "400": { "value": "#070a0e4F", "type": "color" }, // "500": { "value": "{color.accent.neutral.1100}", "attributes": { "alpha": 0.31 } "type": "color" }
        "500": { "value": "#070a0e7D", "type": "color" } // "500": { "value": "{color.accent.neutral.1100}", "attributes": { "alpha": 0.49 } "type": "color" }
      }
    }
  }
}

```

`tokens/color/alpha/neutral-dark.js`:

```js
export default {
  color: {
    alpha: {
      "neutral": {
        "dark": {
          "100": { "value": "#f2f4f70A", "type": "color" }, // "500": { "value": "{color.accent.neutral.dark.1100}", "attributes": { "alpha": 0.04 }, "type": "color" }
          "200": { "value": "#f2f4f714", "type": "color" }, // "500": { "value": "{color.accent.neutral.dark.1100}", "attributes": { "alpha": 0.08 }, "type": "color" }
          "300": { "value": "#f2f4f729", "type": "color" }, // "500": { "value": "{color.accent.neutral.dark.1100}", "attributes": { "alpha": 0.16 }, "type": "color" }
          "400": { "value": "#f2f4f747", "type": "color" }, // "500": { "value": "{color.accent.neutral.dark.1100}", "attributes": { "alpha": 0.28 }, "type": "color" }
          "500": { "value": "#f2f4f780", "type": "color" }, // "500": { "value": "{color.accent.neutral.dark.1100}", "attributes": { "alpha": 0.5 }, "type": "color" }
        }
      }
    }
  }
}

```

- Mask layer

### Color Inverse

Means switching the colors to their opposite values on the color wheel.

For example, turning black text on a white background to white text on a black background. This is often used to create high-contrast versions of designs for better accessibility.

Emphasis levels <https://atlassian.design/foundations/color-new#emphasis-levels> are used to differentiate the importance of text or elements in a design.

Emphasis determines the amount of contrast a color has against the default surface. Emphasis can range from subtlest to boldest. Bolder colors have more contrast against the default surface, which adds more attention than subtle colors.

You might have high emphasis (bold and bright colors) for primary actions like buttons or headers, medium emphasis for secondary actions, and low emphasis (muted colors) for less critical information like captions.

- high emphasis
  - primary actions like buttons or headers
    - "Login" button
    - Save
    - Add to Cart
- medium emphasis
  - secondary actions
    - Forgot Password
    - Cancel
    - Add to Wishlist

- low emphasis
  - Placeholder Text
  - captions
  - Disabled Buttons
  - Footnotes and Fine Print

### Theme Color Conversion

Neutral colors should be set up in a way that makes it easy to convert a light theme to a dark theme.

<https://atlassian.design/foundations/color-new/color-palette-new#picking-colors-for-dark-mode>

Saturated colors should also be easy to **convert** from a light theme to a dark theme using symmetry.

For instance, if a button color is 700 in light theme, it will be 400 in dark theme. If a section message background is 100 in light theme, it will be 1000 in dark theme. Design tokens will handle these conversions for you.

<https://atlassian.design/foundations/color-new/color-palette-new#picking-colors-for-dark-mode>

```sh
touch tokens/color/base-grey-dark.js && \
touch tokens/color/accent/neutral-dark.js
```

`tokens/color/base-grey-dark.js`:

```js
export default {
  color: {
    base: {
      grey: {
        dark: { "value": "#C7D1DB", "type": "color" }
      }
    }
  }
}

```

`tokens/color/accent/neutral-dark.js`:

```js
export default {
  color: {
    "accent": {
      "neutral": {
        "dark": {
          "1": { "value": "#0a0d10", "type": "color" },
          "2": { "value": "#192027", "type": "color" },
          "3": { "value": "#29343f", "type": "color" },
          "4": { "value": "#384857", "type": "color" },
          "5": { "value": "#475b6f", "type": "color" },
          "6": { "value": "#576f87", "type": "color" },
          "7": { "value": "#67829e", "type": "color" },
          "8": { "value": "#7f96ad", "type": "color" },
          "9": { "value": "#97aabc", "type": "color" },
          "10": { "value": "#afbdcc", "type": "color" },
          "11": { "value": "{color.base.grey.dark}", "type": "color" }, // reference
          "12": { "value": "#eff2f5", "type": "color" },
          "default": { "value": "{color.accent.neutral.dark.11}", "type": "color" } // reference
        }
      }
    }
  }
}

```

#### Auto-Generate Accent Colors for Dark Mode Neutral Colors

`tokens/color/accent/neutral-dark.js`:

```js
import tokens from '../base-grey-dark.js'
import { generateColorShades } from '../../../utils/index.js'

const name = 'grey' // Auto-Generate Accent Colors for Dark Mode Neutral Colors from Dark Mode Grey Color

const colors = tokens.color.base[name]

const shades = generateColorShades('dark', colors, 12, 2)
// console.log(shades)
const accents = Object.keys(shades).reduce((acc, level) => ({
  ...acc,
  [
    isNaN(level) ?
      level : // default
      (12 - level + 1) // revert color shade level: 1<->12, 2<->11, 3<->10, ...
  ]: {
    value: shades[level],
    type: 'color'
  }
}), {})
// console.log(accents)

export default {
  color: {
    accent: {
      neutral: {
        dark: accents
      }
    }
  }
}

```

### Color Accessibility

<https://webaim.org/resources/contrastchecker/>

<https://webaim.org/resources/contrastchecker/?fcolor=141D2B&bcolor=F1F3F8>

<https://webaim.org/resources/contrastchecker/?fcolor=D9DFE6&bcolor=080B0D>

### Color Example

Apple’s brand colors are iconic and meticulously chosen. Primarily, Apple sticks to a minimalist color palette that includes silver, space gray, and gold for its devices. Their website reflects this too, using these colors in product imagery and accents. You’ll often see the crisp white background, which helps these brand colors pop. When they introduce new products or features, they sometimes incorporate vibrant, eye-catching colors like bright red for special editions (such as the (PRODUCT)RED line).

Apple's use of colors is a masterclass in branding. Let's break it down:

- Brand Colors

Apple's brand colors are clean and sophisticated. You'll mainly see **silver**, **space gray**, and **gold** — these exude luxury and high-tech vibes.

- Primary Colors

These are the main colors you'll associate with Apple's brand. The iconic white and black are staples, appearing in everything from the background to the text. Occasionally, you'll see **blue**, **green**, and **red** in promotional materials or special products, but they're used sparingly to maintain brand consistency.

- Neutral Colors

Apple excels in its use of neutral colors to create a minimalist and modern look. The website uses a lot of **white** backgrounds, with gray and **black** text. These colors provide a clean canvas that makes product images pop without distraction.

- Alpha Colors

On Apple’s site, for instance, alpha colors might be used for **overlay effects**, subtle shading, and creating a sense of depth, while still keeping the focus on the main content. Makes the whole experience feel slick and seamless!

- Semantic Colors

Semantic colors are used to convey specific meanings or statuses. For example, Apple uses **green** to indicate success or completion (like the green checkmark in the Apple Store) and **red** to signal errors or important notifications (like the red badge on app icons).

- Accent Colors

Accents are used to highlight key elements without overwhelming the overall design. On Apple’s site, you might see **blue** used for links and interactive elements, subtly guiding users' attention without clashing with the primary and neutral colors.

## Testing

## CLI: create component

- add source template
  - component name * (in camel case, snake case, pascal case or kebab case)
  - discription
  - props
- add `export` js
- add `export` in package.json
- add alias in projects
- write component/test/demo code by AI

## Auto Change Version

## Design System Layers

- Above Sea Level: React, Component Library, API Documents, Integration Guides, Demos, Unit Testing
- Below Sea Level: TailwindCSS, Icons, Design Tokens (overiding), Theming (choosing, modifying), Accessability, Mobile/PC/Responsive Design, Play Ground / Sandbox, Server Components, E2E Testing, Performance Optimization (Bundle size etc.)
- Below Seabed: Figma Integration, Auto (create) CLI, Document Auto-generation, Sandbox Auto-generation, Auto Version Management

### Desing System: Why

- reuse components, styles
- theme / style controling
- collaboration
- design audit
- onboarding
