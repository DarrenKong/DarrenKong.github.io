---
title: git 规范
tags:
  - 规范
categories:
  - Git
date: 2020-11-11 14:55:44
---

> Git 规范一般包括两点：分支管理规范、git commit 规范。

### 分支管理规范

一般项目分主分支（master）和其他分支。
当需要开发新功能或修复 Bug 时，从 master 分支开一个新的分支。例如项目要从客户端渲染改成服务端渲染，就开一个分支叫 ssr，开发完了再合并回 master 分支。
如果需要修复一个 Bug，可以从 master 分支开一个新分支，并用 Bug 号命名（一般小团队嫌麻烦，没这样做，除非有特别大的 Bug）。

### git commit 规范

```md
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

大致分为三个部分(使用空行分割):

1. 标题行: 必填, 描述主要修改类型和内容。
2. 主题内容: 描述为什么修改, 做了什么样的修改, 以及开发的思路等等。
3. 页脚注释: 可以写注释，Bug 号链接。

#### type: commit 的类型

- feat: 新功能、新特性
- fix: 修改 bug
- perf: 更改代码，以提高性能
- refactor: 代码重构（重构，在不影响代码内部行为、功能下的代码修改）
- docs: 文档修改
- style: 代码格式修改, 注意不是 css 修改（例如分号修改）
- test: 测试用例新增、修改
- build: 影响项目构建或依赖项修改
- revert: 恢复上一次提交
- ci: 持续集成相关文件修改
- chore: 其他修改（不在上述类型中的修改）
- release: 发布新版本
- workflow: 工作流相关文件修改

1. scope: commit 影响的范围, 比如: route, component, utils, build...
2. subject: commit 的概述
3. body: commit 具体修改内容, 可以分为多行.
4. footer: 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.

示例

##### fix（修复 Bug）

如果修复的这个 Bug 只影响当前修改的文件，可不加范围。如果影响的范围比较大，要加上范围描述。

例如这次 Bug 修复影响到全局，可以加个 global。如果影响的是某个目录或某个功能，可以加上该目录的路径，或者对应的功能名称。

```js
// 示例1
fix(global):修复checkbox不能复选的问题
// 示例2 下面圆括号里的 common 为通用管理的名称
fix(common): 修复字体过小的BUG，将通用管理下所有页面的默认字体大小修改为 14px
// 示例3
fix: value.length -> values.length
```

##### feat（添加新功能或新页面）

```js
feat: 添加网站主页静态页面

这是一个示例，假设对点检任务静态页面进行了一些描述。

这里是备注，可以是放BUG链接或者一些重要性的东西。
```

##### chore（其他修改）

chore 的中文翻译为日常事务、例行工作，顾名思义，即不在其他 commit 类型中的修改，都可以用 chore 表示。

```js
chore: 将表格中的查看详情改为详情
```

其他类型的 commit 和上面三个示例差不多，就不说了。

#### 验证 git commit 规范

验证 git commit 规范，主要通过 git 的 `pre-commit` 钩子函数来进行。当然，你还需要下载一个辅助工具来帮助你进行验证。

下载辅助工具

```sh
npm i -D husky
```

在 `package.json` 加上下面的代码

```json
"husky": {
  "hooks": {
    "pre-commit": "npm run lint",
    "commit-msg": "node script/verify-commit.js",
    "pre-push": "npm test"
  }
}
```

然后在你项目根目录下新建一个文件夹 `script`，并在下面新建一个文件 `verify-commit.js`，输入以下代码：

```js
const msgPath = process.env.HUSKY_GIT_PARAMS
const msg = require('fs')
.readFileSync(msgPath, 'utf-8')
.trim()

const commitRE = /^(feat|fix|docs|style|refactor|perf|test|workflow|build|ci|chore|release|workflow)(\(.+\))?: .{1,50}/

if (!commitRE.test(msg)) {
    console.log()
    console.error(`
        不合法的 commit 消息格式。
        请查看 git commit 提交规范：https://github.com/woai3c/Front-end-articles/blob/master/git%20commit%20style.md
    `)

    process.exit(1)
}
```

现在来解释下各个钩子的含义：

1. `"pre-commit": "npm run lint"`，在 `git commit` 前执行 `npm run lint` 检查代码格式。
2. `"commit-msg": "node script/verify-commit.js"`，在 `git commit` 时执行脚本 `verify-commit.js` 验证 commit 消息。如果不符合脚本中定义的格式，将会报错。
3. `"pre-push": "npm test"`，在你执行 `git push` 将代码推送到远程仓库前，执行 `npm test` 进行测试。如果测试失败，将不会执行这次推送。
