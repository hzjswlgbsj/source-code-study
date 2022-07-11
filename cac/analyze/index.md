## 代码流程
从一个简单的 example case 来走一遍源码流程，对代码组织方式和流程有个印象。
```javascript
// example/basic-usage.js
require('ts-node/register')

const cli = require('../src/index').cac()

cli.option('--type [type]', 'Choose a project type')

const parsed = cli.parse()

console.log(JSON.stringify(parsed, null, 2))

```
### 第一步：得到实例
`../src/index` 文件中导出的 `cac` 其实是一个箭头函数，返回的正是 CAC 类的实例，执行 cac 便得到了一个 「cli」
```
const cac = (name = '') => new CAC(name)
```

### 第二步：注册 option
示例代码直接调用了 cli 的option方法，直接进入可以查看实现
```javascript
/**
   * Add a global CLI option.
   *
   * Which is also applied to sub-commands.
   */
  option(rawName: string, description: string, config?: OptionConfig) {
    this.globalCommand.option(rawName, description, config)
    return this
  }
```
注释告诉我们调用 option 后会添加一个全局的选项，当然这也适用于子命令。源码中可以看到他也是调用了 `this.globalCommand` 的 `option`。

继续看会发现 `this.globalCommand` 是在构造函数中实例化了 `Command` 类，我们继续转到 `Command` 类。

`Command` 类中的 `option` 方法是直接实例化了 `Option` 类，然后将这个实例 push 到 `Command` 类的成员变量 `options` 中缓存。

继续进入 `Option` 类，源码很简单，只有一个构造函数
```typescript
import { removeBrackets, camelcaseOptionName } from './utils'

interface OptionConfig {
  default?: any
  type?: any[]
}

export default class Option {
  /** Option name */
  name: string
  /** Option name and aliases */
  names: string[]
  isBoolean?: boolean
  // `required` will be a boolean for options with brackets
  required?: boolean
  config: OptionConfig
  negated: boolean

  constructor(
    public rawName: string,
    public description: string,
    config?: OptionConfig
  ) {
    this.config = Object.assign({}, config)

    // You may use cli.option('--env.* [value]', 'desc') to denote a dot-nested option
    rawName = rawName.replace(/\.\*/g, '')

    this.negated = false
    this.names = removeBrackets(rawName)
      .split(',')
      .map((v: string) => {
        let name = v.trim().replace(/^-{1,2}/, '')
        if (name.startsWith('no-')) {
          this.negated = true
          name = name.replace(/^no-/, '')
        }

        return camelcaseOptionName(name)
      })
      .sort((a, b) => (a.length > b.length ? 1 : -1)) // Sort names

    // Use the longest name (last one) as actual option name
    this.name = this.names[this.names.length - 1]

    if (this.negated && this.config.default == null) {
      this.config.default = true
    }

    if (rawName.includes('<')) {
      this.required = true
    } else if (rawName.includes('[')) {
      this.required = false
    } else {
      // No arg needed, it's boolean flag
      this.isBoolean = true
    }
  }
}

export type { OptionConfig }

```
主要处理外部传入的 name 、config的description 值，得到如下几个值
```typescript
 /** Option name */
  name: string
  /** Option name and aliases */
  names: string[]
  isBoolean?: boolean
  // `required` will be a boolean for options with brackets
  required?: boolean
  config: OptionConfig
  negated: boolean
```

### 第三步：解析
以实例代码中的 `cli.parse()` 为入口分析解析的过程

我们直接来到了 `CAC` 类的 `parse` 方法，这应该是比较核心的功能了，便于阅读我将源码复制过来
```typescript
/**
   * Parse argv
   */
  parse(
    argv = processArgs,
    {
      /** Whether to run the action for matched command */
      run = true,
    } = {}
  ): ParsedArgv {
    this.rawArgs = argv // 获取 argv

    // 从argv中解析出 name
    if (!this.name) {
      this.name = argv[1] ? getFileName(argv[1]) : 'cli'
    }

    let shouldParse = true

    // 遍历实例上保存的 commands 解析出用户传入的命令
    for (const command of this.commands) {

      // 依赖mri做命令行参数解析
      const parsed = this.mri(argv.slice(2), command)

      const commandName = parsed.args[0]
      // 如果找到匹配的command 的话就将shouldParse置成false
      if (command.isMatched(commandName)) {
        shouldParse = false
        const parsedInfo = {
          ...parsed,
          args: parsed.args.slice(1),
        }
        this.setParsedInfo(parsedInfo, command, commandName)

        // 使用EventEmitter 的 `emit` 方法将这个动作抛出
        this.emit(`command:${commandName}`, command)
      }
    }

    // 上面如果没有匹配成功的话就从默认命令里面去匹配
    if (shouldParse) {
      // Search the default command
      for (const command of this.commands) {
        if (command.name === '') {
          shouldParse = false
          const parsed = this.mri(argv.slice(2), command)
          this.setParsedInfo(parsed, command)
          this.emit(`command:!`, command)
        }
      }
    }

    // 如果还是没有匹配成功的话，再用mri解析一遍
    if (shouldParse) {
      const parsed = this.mri(argv.slice(2))
      this.setParsedInfo(parsed)
    }

    if (this.options.help && this.showHelpOnExit) {
      this.outputHelp()
      run = false
      this.unsetMatchedCommand()
    }

    if (this.options.version && this.showVersionOnExit && this.matchedCommandName == null) {
      this.outputVersion()
      run = false
      this.unsetMatchedCommand()
    }

    const parsedArgv = { args: this.args, options: this.options }

    if (run) {
      this.runMatchedCommand()
    }

    if (!this.matchedCommand && this.args[0]) {
      this.emit('command:*')
    }

    return parsedArgv
  }
```
从返回值来看，parse 的主要功能是将当前 cac 实例上的 option 和 command 解析出来以对象的形式输出：

```typescript
interface ParsedArgv {
  args: ReadonlyArray<string>
  options: {
    [k: string]: any
  }
}
```
解析 command 是直接遍历当前实例上的 `this.commands` ，这里依赖了 [mri](https://github.com/lukeed/mri) ,它是一个解析命令行参数的一个库。解析完成后通过 `setParsedInfo` 方法将数据 set 到 `this.args`，最后通过 EventEmitter 的 `emit` 方法将「这个动作」解析的结果抛出，用户可以直接使用实例的 `on` 方法（来自EventEmitter）监听该动作。

解析 option 的过程跟 command 基本上是一样的。

特别要注意的是：我发现每个类中的每个方法都返回了 `this` ,这使得 CAC 可以实现链式调用。

到这里简单的一个解析流程就已经走完了，我简单画出了 CAC 的 [模块依赖关系图](https://lib.sixtyden.com/CAC%E6%8B%93%E6%89%91%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB.png)。

![](https://lib.sixtyden.com/CAC%E6%8B%93%E6%89%91%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB.png)


以及 CAC 的 [源码流程图](https://www.processon.com/view/link/62caa78b0e3e7451a621c082)，这里仅仅以上面的示例代码为例，主要是理解 option 和 parse 流程.

![](https://lib.sixtyden.com/CAC%E6%BA%90%E7%A0%81%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

## 总结
- 需要实现链式调用在每个函数中返回 `this` 是最基本的
- 命令行工具需要依赖 Node 环境，Deno 可以做一个兼容
- 如果要做到代码灵活性够高就必须要做到不能写死代码，比如 CAC 源码中的 `outputHelp` 函数将 console 写死了，用户不能再自己做输出的动作了
- 写单元测试是很重要的