在项目的源码中已经有一个 `example` 文件夹了，如果要快速了解 cac 的大致功能我选择直接将 `example` 文件夹下的所有的 demo 都跑一遍，然后做一点自己的理解。在源码的根目录直接使用 `node` 去执行示例文件即可。 

### 初体验
先执行他的简单示例：
```javascript
// examples/basic-usage.js
require('ts-node/register')
const cli = require('../src/index').cac()

cli.option('--type [type]', 'Choose a project type')

const parsed = cli.parse()

console.log(JSON.stringify(parsed, null, 2))
```

执行：
```
node examples/basic-usage.js --type foo
```

输出：
```
{
  "args": [],
  "options": {
    "--": [],
    "type": "foo"
  }
}
```

可以看到示例是自己引用了 `../src/index`，然后直接执行 `cac` 方法，进入 `../src/index` 可以看到 cac 就是 CAC 的一个实例
```
const cac = (name = '') => new CAC(name)
```
有了实例后便拥有了 cac 的能力了，示例代码展示了两个
- 添加了 `type` 选项，并赋值为 foo
- 使用 `cli.parse()` 以对象的形式输出了命令中的选项信息

说明 cac 最基础的功能是能够帮助注入一些 option 参数并解析成对象，方便做 cli 开发。


### 展示帮助信息和版本信息
示例代码：
```javascript
// examples/help.js
require('ts-node/register')
const cli = require('../src/index').cac()

cli.option('--type [type]', 'Choose a project type', {
  default: 'node',
})
cli.option('--name <name>', 'Provide your name')

cli.command('lint [...files]', 'Lint files').action((files, options) => {
  console.log(files, options)
})

cli.command('[...files]', 'Run files').action((files, options) => {
  console.log('run', files, options)
})

// Display help message when `-h` or `--help` appears
cli.help()
// Display version number when `-v` or `--version` appears
cli.version('0.0.0')

cli.parse()
```

执行：
```
node examples/help.js --version
```

输出：
```
help.js/0.0.0 darwin-x64 node-v15.3.0
```

执行：
```
node examples/help.js --help
```

输出：
```
help.js/0.0.0

Usage:
  $ help.js [...files]

Commands:
  lint [...files]  Lint files
  [...files]       Run files

For more info, run any command with the `--help` flag:
  $ help.js lint --help
  $ help.js --help

Options:
  --type [type]  Choose a project type (default: node)
  --name <name>  Provide your name 
  -h, --help     Display this message 
  -v, --version  Display version number 
```

是不是跟我们平常执行其他开源库的 `--help` 效果一模一样？

首先示例中注册了 type 和 name 选项，后面又注册了 两个自己的命令； `cli.help()` 的功能就是：通过 `-h` 和 `--help` 来显示帮助信息，这会展示所有的指令和 options。`cli.version('0.0.0')` 的功能是: 通过 `-v` 或 `--version` 展示版本信息，版本信息也会在 --help 中展示。最后必须执行 `cli.parse()` 去解析命令参数。


### 将选项附加到命令
示例代码
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

执行：
```
node examples/command-options.js rm --help  
```

输出：
```
command-options.js

Usage:
  $ command-options.js rm <dir>

Options:
  -r, --recursive  Remove recursively 
  -h, --help       Display this message 
```
示例中增加了 `rm` 命令， 并增加`-r` 选项，这里的命令是直接展示 help ，可以看到提示：
```
Usage:
  $ command-options.js rm <dir>
```

我们可以尝试执行
```
node examples/command-options.js rm -r foo
```
会输出
```
remove foo recursively
```

删除了 foo ，说明 cac 支持将选项附加到命令。

### 在代码中引用选项的名称
示例代码中没有这个demo，我们在 example 中添加`reference-option-name.js`，并加入如下代码
```javascript
require('ts-node/register')
const cli = require('../src/index').cac()

cli
  .command('dev', 'Start dev server')
  .option('--clear-screen', 'Clear screen')
  .action((options) => {
    console.log('clearScreen', options.clearScreen)
  })
cli.parse()
```

执行
```
node examples/reference-option-name.js dev --clear-screen   
```

输出
```
clearScreen true
```
可以看到选项中的`'--clear-screen'` ，在 action 中直接使用 `options.clearScreen` 是能够获取到的。


### 括号
通过文档可以知道，在 Command 中和 Option 中都支持 `[]` 和 `<>`，但是他们含义不同

在 Command 中：
`[]` -> 可选值
`<>` -> 必须值

在 Option 中：
`[]` -> string 或者 number
`<>` -> true

```
const cli = require('cac')()

// 这里的 folder 是必选的，options 的 level 是数字或者 string
cli
  .command('deploy <folder>', 'Deploy a folder to AWS')
  .option('--scale [level]', 'Scaling level')
  .action((folder, options) => {
    // ...
  })

// 这里的 project 是可选的，而 option 中的 dir 是必须的
cli
  .command('build [project]', 'Build a project')
  .option('--out <dir>', 'Output directory')
  .action((folder, options) => {
    // ...
  })

cli.parse()
```

示例代码里面展示了两种情况，具体我注释在里面了。

### 否定值
示例代码
```javscript
// examples/negated-option.js  
require('ts-node/register')
const cli = require('../src/index').cac()

cli.option('--no-clear-screen', 'Do not clear screen')

const parsed = cli.parse()

console.log(JSON.stringify(parsed, null, 2))

```

执行：

```
node examples/negated-option.js --no-clear-screen
```

输出：
```
{
  "args": [],
  "options": {
    "--": [],
    "clearScreen": false
  }
}
```

而执行：
```
node examples/negated-option.js
```

输出：
```
{
  "args": [],
  "options": {
    "--": [],
    "clearScreen": true
  }
}
```

### 可变参数
示例代码
```javascript
require('ts-node/register')
const cli = require('../src/index').cac()

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

输入：
```
node examples/variadic-arguments.js build a.js b.js c.js --foo
```

输出：
```
a.js
[ 'b.js', 'c.js' ]
{ '--': [], foo: true }
```

示例的意图是第一个参数是必选的，标识 entry ，而后面的 b.js c.js 都是可选的，最后能输出option，中间删掉一个 c.js 也是如此。

```
a.js
[ 'b.js' ]
{ '--': [], foo: true }
```

### 对象点操作选项
示例代码
```javascript
require('ts-node/register')
const cli = require('../src/index').cac()

cli
  .command('build', 'desc')
  .option('--env <env>', 'Set envs')
  .option('--foo-bar <value>', 'Set foo bar')
  .example('--env.API_SECRET xxx')
  .action((options) => {
    console.log(options)
  })

cli.help()

const parsed = cli.parse()

console.log(JSON.stringify(parsed, null, 2))
```

输入：
```
node examples/dot-nested-options.js build --env.foo foo --env.bar bar  
```

输出：
```
{
  "args": [],
  "options": {
    "--": [],
    "env": {
      "foo": "foo",
      "bar": "bar"
    }
  }
}
```

可以看到 env 参数被合并到同一个对象中了

### 默认的 Command
注册一个指令，如果没有触发任何指令，那么这个默认指令就会被触发，示例代码中没有这个demo，我们添加一个：
```javascript
require('ts-node/register')
const cli = require('../src/index').cac()

cli
  .command('[...files]', 'Build files')
  .option('--minimize', 'Minimize output')
  .action((files, options) => {
    console.log(files)
    console.log(options.minimize)
  })

const parsed = cli.parse()
console.log(JSON.stringify(parsed, null, 2))

```

输入：
```
node examples/default-command.js
```

输出：
```
{
  "args": [],
  "options": {
    "--": []
  }
}
```

### option 的数组值
```javascript
node cli.js --include project-a
# The parsed options will be:
# { include: 'project-a' }

node cli.js --include project-a --include project-b
# The parsed options will be:
# { include: ['project-a', 'project-b'] }
```

### 错误处理

全局错误处理方式：
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