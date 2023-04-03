---
title: React + TypeScript 最佳实践
date: 2022-08-24 17:05:54
tags: 
    - React
    - TypeScript
categories:
    - FE
---
> 本文译[https://www.sitepoint.com/react-with-typescript-best-practices/]

如今， `React` 和 `TypeScript` 是许多开发人员正在使用的两种很棒的技术。但是把他们结合起来使用就变得很棘手了，有时很难找到正确的答案。不要担心，本文我们来总结一下两者结合使用的最佳实践。

### `React` 和 `TypeScript` 如何一起使用

在开始之前，让我们回顾一下 `React` 和 `TypeScript` 是如何一起工作的。`React` 是一个 “用于构建用户界面的 `JavaScript` 库”，而 `TypeScript` 是一个 “可编译为普通 `JavaScript` 的 `JavaScript` 类型化超集” 。通过同时使用它们，我们实际上是使用 `JavaScript` 的类型化版本来构建 `UI`。

将它们一起使用的原因是为了获得静态类型化语言( `TypeScript` )对 `UI` 的好处：减少 `JS` 带来的 `bug`，让前端开发更安全。

#### `TypeScript` 会编译我的 `React` 代码吗？

一个经常被提到的常见问题是 `TypeScript` 是否编译你的 `React` 代码。`TypeScript` 的工作原理类似于下面的方式：

> `TS`：“嘿，这是你所有的 `UI` 代码吗？”
> `React`：“是的！”
> `TS`：“酷！我将对其进行编译，并确保你没有错过任何内容。”
> `React`：“听起来对我很好！”

因此，答案是肯定的！但是稍后，当我们介绍 `tsconfig.json` 配置时，大多数时候你都想使用 `"noEmit": true` 。这是因为通常情况下，我们只是利用 `TypeScript` 进行类型检查。

概括地说， `TypeScript` 编译你的 `React` 代码以对你的代码进行类型检查。在大多数情况下，它不会发出任何 `JavaScript` 输出。输出仍然类似于非 `TypeScript React` 项目。

#### `TypeScript` 可以与 `React` 和 `Webpack` 一起使用吗？

是的， `TypeScript` 可以与 `React` 和 `webpack` 一起使用。幸运的是，官方 `TypeScript` 手册对此提供了配置指南。

希望这能使你轻而易举地了解两者的工作方式。现在，进入最佳实践！

### 最佳实践

我们研究了最常见的问题，并整理了 `React with TypeScript` 最常用的一些写法和配置。这样，通过使用本文作为参考，你可以在项目中遵循最佳实践。

#### 配置

配置是开发中最无趣但是最重要的部分之一。我们怎样才能在最短的时间内完成这些配置，从而提供最大的效率和生产力？我们一起来讨论下面的配置
```
tsconfig.json
ESLint / Prettier
VS Code 扩展和配置
```

### 项目初始化
初始化一个 `React/TypeScript` 应用程序的最快方法是 `create-react-app` 与 `TypeScript` 模板一起使用。你可以运行以下面的命令：
```
$ yarn create react-app may-app --template typescript
```
这可以让你开始使用 `TypeScript` 编写 `React` 。一些明显的区别是：
`.tsx`：`TypeScript JSX` 文件扩展
`tsconfig.json`：具有一些默认配置的 `TypeScript` 配置文件
`react-app-env.d.ts`：`TypeScript` 声明文件，可以进行允许引用 `SVG` 这样的配置

#### tsconfig.json

幸运的是，最新的 `React/TypeScript` 会自动生成 `tsconfig.json` ，并且默认带有一些最基本的配置。我们建议你修改成下面的内容：
```json
{
  "compilerOptions": {
    "target": "es5", // 指定 ECMAScript 版本
    "lib": ["dom", "dom.iterable", "esnext"], // 要包含在编译中的依赖库文件列表
    "allowJs": true, // 允许编译 JavaScript 文件
    "skipLibCheck": true, // 跳过所有声明文件的类型检查
    "esModuleInterop": true, // 禁用命名空间引用 (import * as fs from "fs") 启用 CJS/AMD/UMD 风格引用 (import fs from "fs")
    "allowSyntheticDefaultImports": true, // 允许从没有默认导出的模块进行默认导入
    "strict": true, // 启用所有严格类型检查选项
    "forceConsistentCasingInFileNames": true, // 不允许对同一个文件使用不一致格式的引用
    "module": "esnext", // 指定模块代码生成
    "moduleResolution": "node", // 使用 Node.js 风格解析模块
    "resolveJsonModule": true, // 允许使用 .json 扩展名导入的模块
    "noEmit": true, // 不输出(意思是不编译代码，只执行类型检查)
    "jsx": "react-jsx", // 在.tsx文件中支持JSX
    "sourceMap": true, // 生成相应的.map文件
    "declaration": true, // 生成相应的.d.ts文件
    "noUnusedLocals": true, // 报告未使用的本地变量的错误
    "noUnusedParameters": true, // 报告未使用参数的错误
    "experimentalDecorators": true, // 启用对ES装饰器的实验性支持
    "incremental": true, // 通过从以前的编译中读取/写入信息到磁盘上的文件来启用增量编译
    "noFallthroughCasesInSwitch": true
  },
  "include": [
    "src" // *** TypeScript文件应该进行类型检查 ***
  ],
  "exclude": ["node_modules", "build"] // *** 不进行类型检查的文件 ***
}

```
> 建议来自 `react-typescript-cheatsheet` 社区

#### `ESLint` / `Prettier`
为了确保你的代码遵循项目或团队的规则，并且样式保持一致，建议你设置  `ESLint` 和 `Prettier` 。为了让它们配合的很好，请按照以下步骤进行设置。

1. 安装依赖
```sh
yarn add eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-plugin-react --dev
```
2. 在根目录下创建一个 `eslintrc.js` 文件并添加以下内容：
```js
module.exports =  {
  parser:  '@typescript-eslint/parser',  // 指定ESLint解析器
  extends:  [
    'plugin:react/recommended',  // 使用来自 @eslint-plugin-react 的推荐规则
    'plugin:@typescript-eslint/recommended',  // 使用来自@typescript-eslint/eslint-plugin的推荐规则
  ],
  parserOptions:  {
  ecmaVersion:  2018,  // 允许解析最新的 ECMAScript 特性
  sourceType:  'module',  // 允许使用 import
  ecmaFeatures:  {
    jsx:  true,  // 允许对JSX进行解析
  },
  },
  rules:  {
    // 自定义规则
    // e.g. "@typescript-eslint/explicit-function-return-type": "off",
  },
  settings:  {
    react:  {
      version:  'detect',  // 告诉 eslint-plugin-react 自动检测 React 的版本
    },
  },
};
```

3. 添加 Prettier 依赖
```sh
yarn add prettier eslint-config-prettier eslint-plugin-prettier --dev
```

4. 在根目录下创建一个 `.prettierrc.js` 文件并添加以下内容：
```js
module.exports = {
  semi: true,
  trailingComma: 'all',
  singleQuote: true,
  printWidth: 120,
  tabWidth: 2,
};

```

更新 `.eslintrc.js` 文件：
```JS
module.exports = {
  parser: '@typescript-eslint/parser', // 指定ESLint解析器
  extends: [
    'plugin:react/recommended', // 使用来自 @eslint-plugin-react 的推荐规则
    'plugin:@typescript-eslint/recommended', // 使用来自@typescript-eslint/eslint-plugin的推荐规则
    'prettier/@typescript-eslint', // 使用 ESLint -config-prettier 禁用来自@typescript-eslint/ ESLint 与 prettier 冲突的 ESLint 规则
    'plugin:prettier/recommended',
  ],
  parserOptions: {
    ecmaVersion: 2018, // 允许解析最新的 ECMAScript 特性
    sourceType: 'module', // 允许使用 import
    ecmaFeatures: {
      jsx: true, // 允许对JSX进行解析
    },
  },
  rules: {
    // 自定义规则
    // e.g. "@typescript-eslint/explicit-function-return-type": "off",
  },
  settings: {
    react: {
      version: 'detect', // 告诉 eslint-plugin-react 自动检测 React 的版本
    },
  },
};

```

#### VSCode 扩展和设置
我们添加了 `ESLint` 和 `Prettier` ，下一步就是在保存时自动修复/美化我们的代码。

首先，安装 `VSCode` 的 `ESLint extension` 和 `Prettier extension` 。这将使 `ESLint` 与您的编辑器无缝集成。

接下来，通过将以下内容添加到您的中来更新工作区设置 `.vscode/settings.json` ：
```json
{
  "editor.formatOnSave": true
}
```
保存时， `VS Code` 会发挥它的魔力并修复您的代码。很棒！

#### 组件

`React` 的核心概念之一是组件。在这里，我们将引用 `React v16.8` 以后的标准组件，这意味着使用 Hook 而不是类的组件。

通常，一个基本的组件有很多需要关注的地方。让我们看一个例子：
```js
import React from 'react'

// 函数声明式写法
function Heading(): React.ReactNode {
  return <h1>My Website Heading</h1>
}

// 函数扩展式写法
const OtherHeading: React.FC = () => <h1>My Website Heading</h1>
```
注意这里的关键区别。在第一个例子中，我们使用函数声明式写法，我们注明了这个函数返回值是 `React.ReactNode` 类型。相反，第二个例子使用了一个函数表达式。因为第二个实例返回一个函数，而不是一个值或表达式，所以我们我们注明了这个函数返回值是 `React.FC` 类型。

记住这两种方式可能会让人混淆。这主要取决于设计选择。无论您选择在项目中使用哪个，都要始终如一地使用它。

#### Props

我们将介绍的下一个核心概念是 `Props`。你可以使用 `interface` 或 `type` 来定义 `Props` 。让我们看另一个例子：
```js
import React from 'react'

interface Props {
  name: string;
  color: string;
}

type OtherProps = {
  name: string;
  color: string;
}

// Notice here we're using the function declaration with the interface Props
function Heading({ name, color }: Props): React.ReactNode {
  return <h1>My Website Heading</h1>
}

// Notice here we're using the function expression with the type OtherProps
const OtherHeading: React.FC<OtherProps> = ({ name, color }) =>
  <h1>My Website Heading</h1>

```
关于 `interface` 或 `type` ，我们建议遵循 `react-typescript-cheatsheet` 社区提出的准则：
> 在编写库或第三方环境类型定义时，始终将 `interface` 用于公共 `API` 的定义。
> 考虑为你的 `React` 组件的 `State` 和 `Props` 使用 `type` ，因为它更受约束。”

让我们再看一个示例：
```js
import React from 'react'

type Props = {
   /** color to use for the background */
  color?: string;
   /** standard children prop: accepts any valid React Node */
  children: React.ReactNode;
   /** callback function passed to the onClick handler*/
  onClick: ()  => void;
}

const Button: React.FC<Props> = ({ children, color = 'tomato', onClick }) => {
   return <button style={{ backgroundColor: color }} onClick={onClick}>{children}</button>
}
```
在此 `<Button />` 组件中，我们为 `Props` 使用 `type`。每个 `Props` 上方都有简短的说明，以为其他开发人员提供更多背景信息。`?` 表示 `Props` 是可选的。`children props` 是一个 `React`.`ReactNode` 表示它还是一个 `React` 组件。

通常，在 `React` 和 `TypeScript` 项目中编写 `Props` 时，请记住以下几点：

> 始终使用 `TSDoc` 标记为你的 `Props` 添加描述性注释 `/** comment */`。
> 无论你为组件 `Props` 使用 `type` 还是 `interfaces` ，都应始终使用它们。
> 如果 `props` 是可选的，请适当处理或使用默认值。

### Hooks

幸运的是，当使用 `Hook` 时， `TypeScript` 类型推断工作得很好。这意味着你没有什么好担心的。举个例子:
```js
// `value` is inferred as a string
// `setValue` is inferred as (newValue: string) => void
const [value, setValue] = useState('')
```

`TypeScript` 推断出 `useState` 钩子给出的值。这是一个 `React` 和 TypeScript 协同工作的成果。

在极少数情况下，你需要使用一个空值初始化 `Hook` ，可以使用泛型并传递联合以正确键入 `Hook` 。查看此实例：
```js
type User = {
  email: string;
  id: string;
}

// the generic is the < >
// the union is the User | null
// together, TypeScript knows, "Ah, user can be User or null".
const [user, setUser] = useState<User | null>(null);
```

下面是一个使用 userReducer 的例子：
```js
type AppState = {};
type Action =
  | { type: "SET_ONE"; payload: string }
  | { type: "SET_TWO"; payload: number };

export function reducer(state: AppState, action: Action): AppState {
  switch (action.type) {
    case "SET_ONE":
      return {
        ...state,
        one: action.payload // `payload` is string
      };
    case "SET_TWO":
      return {
        ...state,
        two: action.payload // `payload` is number
      };
    default:
      return state;
  }
}
```
可见，`Hooks` 并没有为 `React` 和 `TypeScript` 项目增加太多复杂性。

#### 常见用例
本节将介绍人们在将 `TypeScript` 与 `React` 结合使用时一些常见的坑。我们希望通过分享这些知识，您可以避免踩坑，甚至可以与他人分享这些知识。

#### 处理表单事件
最常见的情况之一是 `onChange` 在表单的输入字段上正确键入使用的。这是一个例子：
```js
import React from 'react'

const MyInput = () => {
  const [value, setValue] = React.useState('')

  // 事件类型是“ChangeEvent”
  // 我们将 “HTMLInputElement” 传递给 input
  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    setValue(e.target.value)
  }

  return <input value={value} onChange={onChange} id="input-example"/>
}
```

#### 扩展组件的 Props
有时，您希望获取为一个组件声明的 `Props`，并对它们进行扩展，以便在另一个组件上使用它们。但是你可能想要修改一两个属性。还记得我们如何看待两种类型组件 `Props`、`type` 或 `interfaces` 的方法吗?取决于你使用的组件决定了你如何扩展组件 `Props` 。让我们先看看如何使用 `type`:
```js
import React from 'react';

type ButtonProps = {
    /** the background color of the button */
    color: string;
    /** the text to show inside the button */
    text: string;
}

type ContainerProps = ButtonProps & {
    /** the height of the container (value used with 'px') */
    height: number;
}

const Container: React.FC<ContainerProps> = ({ color, height, width, text }) => {
  return <div style={{ backgroundColor: color, height: `${height}px` }}>{text}</div>
}
```
如果你使用 `interface` 来声明 `props`，那么我们可以使用关键字 `extends` 从本质上“扩展”该接口，但要进行一些修改：
```js
import React from 'react';

interface ButtonProps {
    /** the background color of the button */
    color: string;
    /** the text to show inside the button */
    text: string;
}

interface ContainerProps extends ButtonProps {
    /** the height of the container (value used with 'px') */
    height: number;
}

const Container: React.FC<ContainerProps> = ({ color, height, width, text }) => {
  return <div style={{ backgroundColor: color, height: `${height}px` }}>{text}</div>
}
```
两种方法都可以解决问题。由您决定使用哪个。就个人而言，扩展 `interface` 更具可读性，但最终取决于你和你的团队。

#### 第三方库
无论是用于诸如 `Apollo` 之类的 `GraphQL` 客户端还是用于诸如 `React Testing Library` 之类的测试，我们经常会在 `React` 和 `TypeScript` 项目中使用第三方库。发生这种情况时，你要做的第一件事就是查看这个库是否有一个带有 `TypeScript` 类型定义 `@types` 包。你可以通过运行：
```sh
#yarn
yarn add @types/<package-name>

#npm
npm install @types/<package-name>
```

例如，如果您使用的是 `Jest` ，则可以通过运行以下命令来实现：
```sh
#yarn
yarn add @types/jest

#npm
npm install @types/jest
```

这样，每当在项目中使用 `Jest` 时，就可以增加类型安全性。

该 `@types` 命名空间被保留用于包类型定义。它们位于一个名为 `DefinitelyTyped` 的存储库中，该存储库由 `TypeScript` 团队和社区共同维护。

### 总结
由于信息量大，以最佳方式一起使用 `React` 和 `TypeScript` 需要一些学习时间，但是从长远来看，其收益是巨大的。在本文中，我们介绍了`配置`、`组件`、`Props`、`Hook`、`常见用例` 和 `第三方库`。尽管我们可以更深入地研究各个领域，但这应涵盖帮助您遵循最佳实践所需的 `80％` 。

如果您希望看到它的实际效果，可以在 `GitHub` 上看到这个示例。

