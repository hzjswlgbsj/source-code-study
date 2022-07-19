记录源码阅读和学习过程

## 方式
我阅读源码的步骤：
1. 翻译文档 
2. 根据 Usage 过一遍功能 
3. 分析目录结构
4. 以 Usage 里最简单的 API 为入口去找到单元测试，分析源码
5. 画出UML图
6. 尝试复刻（如果有必要）

所以本仓库中每一个源码阅读都是以如下结构来进行的
```
|—— xxx                   - 项目名称
├── analyze               - 以 Usage 里最简单的 API 为入口分析源码
├── directory-structure   - 分析目录结构
├── docs                  - 翻译并阅读文档
├── process               - 画出流程图
└── usage                 - 根据 Usage 过一遍功能
```

## 已读
- [CAC](https://github.com/cacjs/cac/blob/master/README.md)   - **C**ommand **A**nd **C**onquer 是一个用于构建 CLI 应用程序的JavaScript 库。

## 心路
我也是刚开始阅读源码，很多东西都是摸着石头过河，如有大佬降临，请务必指点迷津。