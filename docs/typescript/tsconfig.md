# tsconfig

:::tip
如果一个目录下存在一个`tsconfig.json`文件，那么它意味着这个目录是`TypeScript`项目的根目录。 `tsconfig.json`文件中指定了用来编译这个项目的根文件和编译选项。
:::

使用tsconfig.json

1.不带任何输入文件的情况下调用`tsc`，编译器会从当前目录开始去查找`tsconfig.json`文件，逐级向上搜索父目录。
2.不带任何输入文件的情况下调用`tsc`，且使用命令行参数`--project`（或-p）指定一个包含`tsconfig.json`文件的目录。
当命令行上指定了输入文件时，`tsconfig.json`文件会被忽略。

## 示例

### 使用"files"属性
```js
  {
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "sourceMap": true
    },
    "files": [
        "core.ts",
        "sys.ts",
        "types.ts",
        "scanner.ts",
        "parser.ts",
        "utilities.ts",
        "binder.ts",
        "checker.ts",
        "emitter.ts",
        "program.ts",
        "commandLineParser.ts",
        "tsc.ts",
        "diagnosticInformationMap.generated.ts"
    ]
  }
```

### 使用"include"和"exclude"属性


```js
  {
    "compilerOptions": {
        "module": "system",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "outFile": "../../built/local/tsc.js",
        "sourceMap": true
    },
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
  }
```

`"compilerOptions"`可以被忽略，这时编译器会使用默认值。

`"files"`指定一个包含相对或绝对文件路径的列表。 `"include"`和`"exclude"`属性指定一个文件`glob`匹配模式列表。 支持的`glob`通配符有：

- `*` 匹配0或多个字符（不包括目录分隔符）
- `?` 匹配一个任意字符（不包括目录分隔符）
- `**/` 递归匹配任意子目录


如果一个`glob`模式里的某部分只包含*或.*，那么仅有支持的文件扩展名类型被包含在内（比如默认`.ts`，`.tsx`，和`.d.ts`， 如果 `allowJs`设置能`true`还包含`.js`和`.jsx`）。

如果`"files"`和`"include"`都没有被指定，编译器默认包含当前目录和子目录下所有的`TypeScript`文件（`.ts, .d.ts 和 .tsx`），排除在`"exclude"`里指定的文件。

`JS`文件（.js和.jsx）也被包含进来如果`allowJs`被设置成true。 
使用 `"outDir"`指定的目录下的文件永远会被编译器排除，除非你明确地使用`"files"`将其包含进来（这时就算用`exclude`指定也没用）。

使用`"include"`引入的文件可以使用`"exclude"`属性过滤。 然而，通过 `"files"`属性明确指定的文件却总是会被包含在内，不管`"exclude"`如何设置。 如果没有特殊指定， "exclude"默认情况下会排除`node_modules`，`bower_components`，`jspm_packages`和`<outDir>`目录。

## `@types，typeRoots和types`

默认所有可见的`"@types"`包会在编译过程中被包含进来。 `node_modules/@types`文件夹下以及它们子文件夹下的所有包都是可见的； 也就是说， `./node_modules/@types/，../node_modules/@types/`和`../../node_modules/@types/`等等。

如果指定了`typeRoots`，只有`typeRoots`下面的包才会被包含进来。 比如：

```js
{
   "compilerOptions": {
       "typeRoots" : ["./typings"]
   }
}
```

这个配置文件会包含所有`./typings`下面的包，而不包含`./node_modules/@types`里面的包。

如果指定了`types`，只有被列出来的包才会被包含进来。 比如：

```js
{
   "compilerOptions": {
        "types" : ["node", "lodash", "express"]
   }
}
```

这个`tsconfig.json`文件将仅会包含 `./node_modules/@types/node，./node_modules/@types/lodash和./node_modules/@types/express`。 `node_modules/@types/*`里面的其它包不会被引入进来。

### 使用extends继承配置

`tsconfig.json`文件可以利用`extends`属性从另一个配置文件里继承配置。

`extends`是`tsconfig.json`文件里的顶级属性（与`compilerOptions`，`files`，`include`，和`exclude`一样）。 `extends`的值是一个字符串，包含指向另一个要继承文件的路径。

在原文件里的配置先被加载，然后被来至继承文件里的配置重写。 如果发现循环引用，则会报错。

### compileOnSave


在最顶层设置`compileOnSave`标记，可以让`IDE`在保存文件的时候根据`tsconfig.json`重新生成文件。

```js
{
    "compileOnSave": true,
    "compilerOptions": {
        "noImplicitAny" : true
    }
}
```

要想支持这个特性需要`Visual Studio 2015`， `TypeScript1.8.4`以上并且安装`atom-typescript`插件。






