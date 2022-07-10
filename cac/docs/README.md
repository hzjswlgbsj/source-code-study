## 介绍
CAC（Command And Conquer）是一个用于构建 CLI 应用程序的 JavaScript 库

## 特性
- **超级轻量：** 没有依赖，只有一个文件。
- **简单易学：** 要构建简单的cli，您只需要学习 4 个 api: `cli.option`、 `cli.version`、`cli.help`、`cli.parse`。
- **但如此强大：** 拥有如下特性: 默认命令、类似git的子命令、必须参数和选项的验证、可变参数、对象参数选项、自动生成帮助消息等等。
- **开发者友好：** 使用 Typescript 编写。

## 安装
```
yarn add cac
```
## 用法
### 简单解析
使用CAC作为简单的参数解析器：
```javascript
// examples/basic-usage.js
const cli = require('cac')()

cli.option('--type <type>', 'Choose a project type', {
  default: 'node',
})

const parsed = cli.parse()

console.log(JSON.stringify(parsed, null, 2))
```
![](https://user-images.githubusercontent.com/8784712/48981576-2a871000-f112-11e8-8151-80f61e9b9908.png)

### 显示帮助信息和版本
```javascript
// examples/help.js
const cli = require('cac')()

cli.option('--type [type]', 'Choose a project type', {
  default: 'node',
})
cli.option('--name <name>', 'Provide your name')

cli.command('lint [...files]', 'Lint files').action((files, options) => {
  console.log(files, options)
})

// Display help message when `-h` or `--help` appears
cli.help()
// Display version number when `-v` or `--version` appears
// It's also used in help message
cli.version('0.0.0')

cli.parse()
```
![](https://user-images.githubusercontent.com/8784712/48979012-acb20d00-f0ef-11e8-9cc6-8ffca00ab78a.png)


### 命令的具体选项
可以将选项附加到命令。
```javascript
const cli = require('cac')()

cli
  .command('rm <dir>', 'Remove a dir')
  .option('-r, --recursive', 'Remove recursively')
  .action((dir, options) => {
    console.log('remove ' + dir + (options.recursive ? ' recursively' : ''))
  })

cli.help()

cli.parse()
```
当命令被使用时，命令的选项将被验证。任何未知选项都将被报告为错误。但是，如果基于 action 的命令没有定义为 action，则不会验证选项。如果您真的想使用未知选项，请使用[command.allowUnknownOptions](https://github.com/cacjs/cac/blob/master/README.md#commandallowunknownoptions)。

![](https://user-images.githubusercontent.com/8784712/49065552-49dc8500-f259-11e8-9c7b-a7c32d70920e.png)

### 选项名称中的破折号
应该在 options 代码中使用 camelCase 而不是 kebab-case
```javascript
cli
  .command('dev', 'Start dev server')
  .option('--clear-screen', 'Clear screen')
  .action((options) => {
    console.log(options.clearScreen)
  })
```
事实上 `——clear-screen` 和 `——clearScreen` 都被映射到 `options.clearScreen`。

### 方括号
当在命令名中使用方括号时，尖括号表示必需的命令参数，而方括号表示可选参数。

在选项名称中使用括号时，角度括号表示需要一个字符串 / 数值，而方括号则表示该值也可以是正确的。
```javascript
const cli = require('cac')()

cli
  .command('deploy <folder>', 'Deploy a folder to AWS')
  .option('--scale [level]', 'Scaling level')
  .action((folder, options) => {
    // ...
  })

cli
  .command('build [project]', 'Build a project')
  .option('--out <dir>', 'Output directory')
  .action((folder, options) => {
    // ...
  })

cli.parse()
```

### 否定选项
要允许一个值为 `false` 的选项，您需要手动指定一个否定选项
```javascript
cli
  .command('build [project]', 'Build a project')
  .option('--no-config', 'Disable config file')
  .option('--config <path>', 'Use a custom config file')
```
这将允许 CAC 将 `config` 的默认值设置为 `true`，您可以使用 `——no-config` 标志将其设置为 `false`。


### 可变参数
命令的最后一个参数可以是可变参数，而且只能是最后一个参数。要使一个参数可变，你必须添加…到参数名的开头，就像JavaScript中的rest操作符一样。这里有个例子
```javascript
const cli = require('cac')()

cli
  .command('build <entry> [...otherFiles]', 'Build your app')
  .option('--foo', 'Foo option')
  .action((entry, otherFiles, options) => {
    console.log(entry)
    console.log(otherFiles)
    console.log(options)
  })

cli.help()

cli.parse()
```
![](https://user-images.githubusercontent.com/8784712/48979056-47125080-f0f0-11e8-9d8f-3219e0beb0ed.png)

### 对象点操作符选项
对象点操作选项被解析后会被合并到一个中
```javascript
const cli = require('cac')()

cli
  .command('build', 'desc')
  .option('--env <env>', 'Set envs')
  .example('--env.API_SECRET xxx')
  .action((options) => {
    console.log(options)
  })

cli.help()

cli.parse()
```
![](https://user-images.githubusercontent.com/8784712/48979771-6ada9400-f0fa-11e8-8192-e541b2cfd9da.png)

### 默认命令
注册一个不匹配其他命令时将使用的命令。

```javascript
const cli = require('cac')()

cli
  // Simply omit the command name, just brackets
  .command('[...files]', 'Build files')
  .option('--minimize', 'Minimize output')
  .action((files, options) => {
    console.log(files)
    console.log(options.minimize)
  })

cli.parse()
```

### 提供一个数组作为选项值
```javascript
node cli.js --include project-a
# The parsed options will be:
# { include: 'project-a' }

node cli.js --include project-a --include project-b
# The parsed options will be:
# { include: ['project-a', 'project-b'] }
```

### 错误处理
全局处理命令错误"
```javascript
try {
  // Parse CLI args without running the command
  cli.parse(process.argv, { run: false })
  // Run the command yourself
  // You only need `await` when your command action returns a Promise
  await cli.runMatchedCommand()
} catch (error) {
  // Handle error here..
  // e.g.
  // console.error(error.stack)
  // process.exit(1)
}
```

### 在 Typescript 环境使用
首先，您需要在项目中安装 @types/node作为DEV依赖性：
```
yarn add @types/node --dev
```

然后，一切都在开箱即用：
```javascript
const { cac } = require('cac')
// OR ES modules
import { cac } from 'cac'
```

### 在 Deno 环境使用
```javascript
import { cac } from 'https://unpkg.com/cac/mod.ts'

const cli = cac('my-program')
```

## 谁在使用CAC（这个就不翻译了）
- [VuePress](https://github.com/vuejs/vuepress): 📝 Minimalistic Vue-powered static site generator.
- [SAO](https://github.com/egoist/sao): ⚔️ Futuristic scaffolding tool.
- [DocPad](https://github.com/docpad/docpad): 🏹 Powerful Static Site Generator.
- [Poi](https://github.com/egoist/poi): ⚡️ Delightful web development.
- [bili](https://github.com/egoist/bili): 🥂 Schweizer Armeemesser for bundling JavaScript libraries.
- [Lad](https://github.com/ladjs/lad): 👦 Lad scaffolds a Koa webapp and API framework for Node.js.
- [Lass](https://github.com/lassjs/lass): 💁🏻 Scaffold a modern package boilerplate for Node.js.
- [Foy](https://github.com/zaaack/foy): 🏗 A lightweight and modern task runner and build tool for general purpose.
- [Vuese](https://github.com/vuese/vuese): 🤗 One-stop solution for vue component documentation.
- [NUT](https://github.com/nut-project/nut): 🌰 A framework born for microfrontends
- Feel free to add yours here...

## 参考
**如果您想获得更深入的API参考资料，可以查看源代码[生成的文档](https://cac-api-doc.egoist.sh/classes/_cac_.cac.html)**。
下面是一个简短的概述。

### CLI 实例
CLI 实例是通过调用 `cac` 函数来创建的：
```JavaScript
const cac = require('cac')
const cli = cac()
```

#### cac(name?)
创建一个 CLI 实例，可选地指定将在帮助和版本消息中显示的程序名称。当未设置时，我们使用 `argv [1]` 的 basename。

#### cli.command(name, description, config?)
- Type: `(name: string, description: string) => Command`

创建一个命令实例。

该选项还接受附加命令配置的第三个参数配置:
- `config.allowUnknownOptions`: `boolean` 在这个命令中允许未知选项。
- `config.ignoreOptionDefaultValue`: `boolean` 不要在解析选项中使用选项的默认值，而只会在帮助消息中显示它们。

#### cli.option(name, description, config?)
Type: `(name: string, description: string, config?: OptionConfig) => CLI`

添加一个全局选项

该选项还接受其他选项 `config` 的第三个参数配置：
- `config.default`: 选项的默认值。
- `config.type: any[]` 当设置为`[]`时，该选项值返回数组类型。您还可以使用转换函数，如`[String]`，它将使用`String`调用选项值。

#### cli.parse(argv?)
- Type: `(argv = process.argv) => ParsedArgv`

```typescript
interface ParsedArgv {
  args: string[]
  options: {
    [k: string]: any
  }
}
```

当调用此方法时， `cli.rawArgs` `cli.args` `cli.options` `cli.matchedCommand` 也将可用。

#### cli.version(version, customFlags?)
- Type: `(version: string, customFlags = '-v, --version') => CLI`

当出现 `-v`，`——version`标志时，输出版本号。

#### cli.help(callback?)
Type: `(callback?: HelpCallback) => CLI`
出现 `-h`，`——help` 标志时输出帮助消息。
可选的 `callback` 允许在显示帮助文本之前进行后处理：
```
type HelpCallback = (sections: HelpSection[]) => void

interface HelpSection {
  title?: string
  body: string
}
```

#### cli.outputHelp()
- Type: `() => CLI`
输出帮助信息

#### cli.usage(text)
- Type: `(text: string) => CLI`
添加全局用法文本。子命令不使用它。

### Command 实例
Command 实例通过调用 `cli.command` 方法创建
```javascript
const command = cli.command('build [...files]', 'Build given files')
```

#### command.option()
基本上和 `cli.option` 一样，但这将选项添加到特定的命令。

#### command.action(callback)
Type: `(callback: ActionCallback) => Command`

当命令匹配用户输入时，使用回调函数作为命令操作。
```typescript
type ActionCallback = (
  // Parsed CLI args
  // The last arg will be an array if it's a variadic argument
  ...args: string | string[] | number | number[]
  // Parsed CLI options
  options: Options
) => any

interface Options {
  [k: string]: any
}
```

#### command.alias(name)
- Type: `(name: string) => Command`

在此命令中添加一个别名名称，此处的 `name` 不能包含括号。

#### command.allowUnknownOptions()
- Type: `() => Command`

在此命令中允许未知选项，默认情况下，当使用未知选项时，CAC 将记录错误。

#### command.example(example)
- Type: `(example: CommandExample) => Command`
添加一个示例，该示例将显示在帮助消息的末尾。

```typescript
type CommandExample = ((name: string) => string) | string
```

#### command.usage(text)
Type: `(text: string) => Command`

为该命令添加用法文本。

### 事件
监听命令
```javascript
// Listen to the `foo` command
cli.on('command:foo', () => {
  // Do something
})

// Listen to the default command
cli.on('command:!', () => {
  // Do something
})

// Listen to unknown commands
cli.on('command:*', () => {
  console.error('Invalid command: %s', cli.args.join(' '))
  process.exit(1)
})
```

## 常见问题

### 名称如何写和发音？
CAC，或CAC，读作C-A-C。
这个项目献给我们可爱的C.C. sama。也许CAC也代表C&C:P

### 为什么不使用 Commander.js
CAC 与 Commander.js 非常相似，而后者不支持 DOT 嵌套选项，即类似 `-env.api_secret foo`。此外，您也不能在 Commander.js 中使用未知选项。


也许还有更多...


基本上，我制作了 CAC，以满足我对 POI，SAO 和所有 CLI 应用等 CLI 应用程序的需求。它很小，简单但功能强大：P

### 项目状态
[状态良好](https://github.com/cacjs/cac#project-stats)

### 如何贡献
1. Fork 仓库代码!
2. 创建你的分支: `git checkout -b my-new-feature`
3. 提交你的修改: `git commit -am 'Add some feature'`
4. Push 代码到你的远程分支: `git push origin my-new-feature`
5. 提交一个 pull request :D