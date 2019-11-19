# 迁移到Typescript: 为第三方NPM模块编写声明文件

> 原文：[Migrating to Typescript: Write a declaration file for a third-party NPM module](https://medium.com/@chris_72272/migrating-to-typescript-write-a-declaration-file-for-a-third-party-npm-module-b1f75808ed2)

> 译者注：为了使用淘宝百川的 NodeJS SDK，尝试编写该SDK的声明文件。

> 对应代码仓库 [gnu4cn/ts-baichuan](https://github.com/gnu4cn/ts-baichuan)


假定现在手头上就有一个带有很多NPM模块的node.js应用。这对于 Vanilla JavaScript来将不是问题。而对于以静态类型检查器为优势的 TypeScript来讲，要利用上这一优势，就需要开始将类型注记（type annotations）添加进代码，包括那些第三方的npm模块。

TypeScript使用声明文件（declaration files）来知悉模块的众多类型与函数签名。将这些声明文件添加到项目中的流程已经改变，因此在网上搜索到的那些信息可能已经过时。

在 TypeScript 2.2 中，有着一种相当直接的方式来添加某个流行npm模块的声明文件。要做的只是：

```console
$ npm install --save-dev @types/module
// 比如：
$ npm install --save lodash
$ npm install --save-dev @types/lodash
```

`npm` 将创建出`node_modules/@types`目录，其下是各个有着 `index.d.ts`文件的模块的子目录。`index.d.ts`文件并不包含任何代码。他只是一个描述模块接口，比如模块的各个类与各种类型的文件。仍需导入真实模块。

## 对于那些没有任何什么文件的模块，又该怎么办呢

不可避免地会发现，使用到某个在npm中没有任何声明文件的模块。此时若要利用上静态类型检查，就需要编写自己的声明文件了。

在解决这个问题上我（原文作者）花了好几个小时。TypeScript官方文档并没有给出如何来编写这种声明文件的方法，在网上找到的信息通常也只是适用于旧版本的TypeScript。因此就写下本文，从而希望后来者可以节省为找到如何编写第三方模块的声明文件的方法，而耗费几个小时的时间。

首先来看看 `tsconfig.json`，它里面有个`typeRoots`属性，默认没有设置，不过正是这个属性配置了要到那里去搜索各个声明文件。默认搜索的是`node_moduels/@types`。那么就是只在`node_modules`这个无法将自己的文件放入的地方搜索。

因此首先就要加入一个新的、计划要将自己的声明文件存入的目录到项目。在本示例中，就使用`@types`作为那个目录了，当然也可以取自己想要的其他名字。

_tsconfig.json_

```json
{
    "compilerOptions": {
        "outDir": "./built",
        "allowJs": true,
        "noImplicitAny": true,
        "strictNullChecks": true,
        "target": "es6",
        "module": "commonjs"
    },
    "include": [
        "./src/**/*"
    ],
    "exclude": [
        "node_modules"
    ]
}
```

该配置文件开启了`noImplicitAny`选项，意味着 __必须__ 显示地加上类型注记。若在项目较大且需要逐步迁移时，应该将此选项关闭。

同时该配置文件还加入了 `typeRoots: ["@types", "./@types"]`。这就告诉了 TypeScript编译器同时在 `node_modules/@types`与这里的定制目录`./@types`中查找 `.d.ts`文件。请注意所有的原始 JavaScript文件都已被移入 `src` 文件夹，以保证TypeScript的编译。


> 更新（2018-02-01）：如同本文评论中所支出的那样，在近期版本的TypeScript中，已经不需要在 `tsconfig.json`中指定 `typeRoot` 选项了。


现在就可以创建自己的定制声明文件了。在本示例中，将给出如何为 npm 模块`dir-obj`编写声明文件的方法，因为这就是笔者要解决的问题。

## 以创建一个新项目开始

```console
$ mkdir ~/dev/myproject
$ cd ~/dev/myproject
$ mkdir src
$ mkdir built
$ vim tsconfig.json
# Add outDir, include, and set noImplicitAny
<paste>
{
"compilerOptions": {
"outDir": "./built",
"module": "commonjs",
"target": "es6",
"noImplicitAny": true,
"sourceMap": false
},
"include": [
"src/**/*"
]
}
</paste>

$ vim src/index.ts

<paste>
import * as dirObj from 'dir-obj';
const project = dirObj.readDirectory(__dirname + '/..', {
fileTransform: (file: dirObj.File) => {
return file.fullpath;
}
});
console.log(JSON.stringify(project, null, 2));
</paste>
```

这个简单的文件将读取项目目录结构，并输出到各个文件的完整路径。

要将此 TypeScript 文件编译为 ES5 的 JavaScript 文件，只需从项目根处运行：

```console
$ tsc -p .
```

这里的 `-p` 选项告诉 `tsc` 在当前目录查找 `tsconfig.json` 文件。

__警告！无法找到模块的声明文件__

__Warning! Could not find a declaration file for module__


```console
src/index.ts(1,25): error TS7016: Could not find a declaration file for module 'dir-obj'. '/Users/chris/dev/personal/typescript-examples/node_modules/dir-obj/index.js' implicitly has an 'any' type.
```

在当前设置下，`tsc`无法就代码是否有效进行静态类型检查。为此就需要加入一个声明文件。

```console
$ mkdir src/@types
$ mkdir src/@types/dir-obj
$ vim src/@types/dir-obj/index.d.ts
```

这里在`src`目录中创建了定制的 `@types` 目录，因此这些文件在编译过程中将自动包含进去。

接下来就要给模块`dir-obj`添加一个声明文件了。声明文件必须位于一个与 npm 模块匹配的目录中。


## 创建声明文件

```typescript
/// <reference types="node" />

declare module 'dir-obj' {
    import { Stats } from "fs";

    export interface readOptions {
        filter?: RegExp | Filter,
        dirTransform?: DirTransform,
        fileTransform?: FileTransform
    }

    export type Filter = (file: File) => boolean;
    export type DirTransform = (file: File, value: any) => any;
    export type FileTransform = (file: File) => any;

    export function readDirectory(dir: string, options?: readOptions): object;

    export class File {
        key: string;
        readonly path: string;
        readonly fullpath: string;
        readonly ext: string;
        readonly name: string;
        readonly basename: string;

        constructor(dir: string, file: string);

        readonly attributes: Stats;
        readonly isDirectory: boolean;
        readonly isRequirable: boolean;
    }
}
```

这里是以`declare module 'dir-obj'`开始的这个声明文件，其显式地指明要进行描述的模块。

该声明文件剩余部分则是一个原始 JavaScript 源代码中可用的函数与类的清单，只不过这些函数与类只是加入了类型信息。至于如何去阅读JavaScript源代码并找出其中的函数与类，以及怎样编写类型定义，不属于本文讨论的内容，希望你可以循着这两点信息自己去完成。

现在来编译项目：

```console
$ tsc -p .
```

就不会有编译错误了。

[GitHub 上完整的示例](https://github.com/cjthompson/typescript-examples)
