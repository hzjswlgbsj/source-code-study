简单分析一下cac的文件目录分别是做啥的，文中的树形结构是使用 `tree` 命令生成的，如果你是 mac 用户可以直接执行 `brew install tree` 来安装它。

根目录执行
```
tree -L 2 -I node_modules // -L标识层级，-I标识需要排除的文件（夹），全部option请查看官网
```

生成如下结构
```
|CAC
├── LICENSE                 开源协议文件
├── README.md               该项目的介绍文件，一般会包含简介、使用方法，简单的api等
├── circle.yml              CircleCI 的配置文件
├── examples                该项目的示例文件夹，其中包含基本上所有的API示例
├── index-compat.js         主入口，主要是为了兼容 cjs
├── jest.config.js          jest 配置文件
├── mod.js                  for deno
├── mod.ts                  for deno
├── mod_test.ts             for deno
├── package.json            NPM软件包的细节，同目录的package.json.md 有字段注释，[文档地址](https://docs.npmjs.com/cli/v7/configuring-npm/package-json/)
├── rollup.config.js        rollup 打包配置文件
├── .editorconfig           跨编辑器/IDE 规范编码风格，使用 yaml 风格
├── .gitattributes          当执行 git 动作时，.gitattributes 文件允许你指定由 git 使用的文件和路径的属性
├── .gitignore              配置此文件可以让 git 对某些特定文件不追踪变化
├── .prettierrc             prettier 的配置文件，详情参照官网
├── scripts                 项目中的脚本程序，比如node和deno转化
├── src                     源代码文件夹
│   ├── CAC.ts              CAC类的实现，继承EventEmitter，依赖 mri
│   ├── Command.ts          Command类的实现，为CAC提供命令注册和处理相关逻辑
│   ├── Option.ts           Option类的实现，为CAC提供解析用户选项的功能
│   ├── __test__            jest 测试用例文件夹
│   ├── deno.ts             兼容Deno
│   ├── index.ts            CAC 的出口文件，导出了 cac, CAC, Command
│   ├── node.ts             使用node环境获取参数process.argv
│   └── utils.ts            实现源码中会使用到的工具函数和一些公共函数
├── tsconfig.json           TS 配置文件，同目录的 tsconfig.json.md 有字段注释
└── yarn.lock               yarn 的依赖锁文件
```