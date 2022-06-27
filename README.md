# vite-plugin-electron-renderer

Support use Node.js API in Electron-Renderer

[![NPM version](https://img.shields.io/npm/v/vite-plugin-electron-renderer.svg?style=flat)](https://npmjs.org/package/vite-plugin-electron-renderer)
[![NPM Downloads](https://img.shields.io/npm/dm/vite-plugin-electron-renderer.svg?style=flat)](https://npmjs.org/package/vite-plugin-electron-renderer)

## Install

```sh
npm i vite-plugin-electron-renderer -D
```

## Usage

vite.config.ts

```js
import renderer from 'vite-plugin-electron-renderer'

export default {
  plugins: [
    renderer(/* options */),
  ],
}
```

renderer.js

```ts
import { readFile } from 'fs'
import { ipcRenderer } from 'electron'

readFile(/* something code... */)
ipcRenderer.on('event-name', () => {/* something code... */})
```

## API

`renderer(options: Options)`

```ts
export interface Options {
  /**
   * Explicitly include/exclude some CJS modules  
   * `modules` includes `dependencies` of package.json, Node.js's `builtinModules` and `electron`  
   */
  resolve?: (modules: string[]) => typeof modules | undefined
}
```

## dependencies vs devDependencies

The simplest way

- Node.js package in `dependencies` of `package.josn`
- Web package in `devDependencies` of `package.josn`

In general, Vite may not be able to correctly build Node.js packages, especially C/C++ native modules, but Vite can load them as external packages.  
*通常的，Vite 可能不能正确的构建 Node.js 的包，尤其是 C/C++ 原生模块，但是 Vite 可以将它们以外部包的形式加载。*  

e.g.

Electron-Main

```js
import { ipcMain } from 'electron'
↓
const { ipcMain } = require('electron')
```

Electron-Renderer

```js
import { ipcRenderer } from 'electron'
↓
// Generate a virtual module by vite-plugin-reaolve
const electron = require('electron')
export const ipcRenderer = electron.ipcRenderer
↓
// The following code snippet will work in Electron-Renderer, 
// and it doesn't seem to have changed :)
import { ipcRenderer } from 'electron'
```

## How to work

Using Electron API in Electron-Renderer

```js
import { ipcRenderer } from 'electron'
↓
// Actually will redirect by `resolve.alias`
import { ipcRenderer } from 'vite-plugin-electron-renderer/electron-renderer.js'
```

[Using Node.js API and package in Electron-Renderer 👉](https://github.com/electron-vite/vite-plugin-electron-renderer/blob/4a2620d9ff9b3696cf55c1c5d4f2acdcf1ff806a/index.js#L37)

#### Config presets

1. Fist, the plugin will configuration something.
  *If you do not configure the following options, the plugin will modify their default values*

  * `base = './'`
  * `build.assetsDir = ''` -> *TODO: Automatic splicing `build.assetsDir`*
  * `build.emptyOutDir = false`
  * `build.cssCodeSplit = false`
  * `build.rollupOptions.output.format = 'cjs'`
  * `resolve.conditions = ['node']`
  * Always insert the `electron` module into `optimizeDeps.exclude`

2. The plugin transform Electron and Node.js built-in modules to ESModule format in `vite serve` phase.

3. Add Electron and Node.js built-in modules to Rollup `output.external` option in the `vite build` phase.
