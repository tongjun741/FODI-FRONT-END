---
tags:
    - IDE
    - VS Code
---

## eslint 格式化代码

本文用 Vue 项目做示范。

利用 `Vue-CLI`  创建项目时要将 ESlint 选上，下载完依赖后，用 VSCode 打开项目。

安装插件 ESLint，然后 `File` -> `Preference`-> `Settings`（如果装了中文插件包应该是 文件 -> 选项 -> 设置），搜索 eslint，点击 `Edit in setting.json`

![在这里插入图片描述](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/2901fe95f2cd48478945ac8df56c2cc5~tplv-k3u1fbpfcp-watermark.awebp)

将以下选项添加到配置文件

```js
"editor.codeActionsOnSave": {
    "source.fixAll": true,
},
"eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript"
],
"eslint.alwaysShowStatus": true,
"stylelint.validate": [
    "css",
    "less",
    "postcss",
    "scss",
    "vue",
    "sass"
],
复制代码
```

同时要确保 VSCode 右下角的状态栏 ESlint 是处于工作状态的。如果右下角看不到 Eslint 的标识，请按照上面讲过的步骤打开 `setting.json`，加上这行代码：

```js
"eslint.alwaysShowStatus": true,
复制代码
```

![image](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/606e8356a41f46d0884076328a45a572~tplv-k3u1fbpfcp-watermark.awebp)

配置完之后，VSCode 会根据你当前项目下的 `.eslintrc` 文件的规则来验证和格式化代码。

![img](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/52f6fb8444ab4441ba7c7f717ec30ec5~tplv-k3u1fbpfcp-watermark.awebp)

#### TypeScript

下载插件

```
npm install --save-dev typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin
复制代码
```

在 `.eslintrc` 配置文件，添加以下两个配置项：

```json
"parser": "@typescript-eslint/parser",
"plugins": [
    "@typescript-eslint"
],
复制代码
```

在根目录下的 `package.json` 文件的 `scripts` 选项里添加以下配置项：

```json
"scripts": {
  "lint": "eslint --ext .js,.ts,.tsx test/ src/",
},
复制代码
```

`test/` `src/` 是你要校验的目录。修改完后，现在 ts 文件也可以自动格式化了。

![img](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/f70118ab05094c8ebd51468f60a35a42~tplv-k3u1fbpfcp-watermark.awebp)

如果你使用 `Vue-CLI` 创建项目，并且想要格式化 TypeScript 的代码，则需要在 `.eslintrc.js` 文件添加或修改以下几项：

```js
parser: 'vue-eslint-parser',
plugins: [
    '@typescript-eslint',
],
parserOptions: {
    parser: '@typescript-eslint/parser',
    ecmaVersion: 2020,
},
复制代码
```

这样就可以格式化 `.js` `.ts` `.vue` 文件了。

## stylelint 格式化代码

下载依赖

```
npm install --save-dev stylelint stylelint-config-standard
复制代码
```

在项目根目录下新建一个 `.stylelintrc.json` 文件，并输入以下内容：

```json
{
    "extends": "stylelint-config-standard"
}
复制代码
```

VSCode 添加 `stylelint` 插件：

![在这里插入图片描述](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/4d92293ee56e43cabe9de92d90952492~tplv-k3u1fbpfcp-watermark.awebp)

然后就可以看到效果了。

![在这里插入图片描述](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/58dfc7cc46b14ba888e883499625394f~tplv-k3u1fbpfcp-watermark.awebp)

如果你想修改插件的默认规则，可以看[官方文档](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fstylelint%2Fstylelint%2Fblob%2F5a8465770b4ec17bb1b47f359d1a17132a204a71%2Fdocs%2Fuser-guide%2Frules%2Flist.md)，它提供了 170 项规则修改。例如我想要用 4 个空格作为缩进，可以这样配置：

```json
{
    "extends": "stylelint-config-standard",
    "rules": {
        "indentation": 4
    }
}
复制代码
```

如果你想格式化 `sass` `scss` 文件，则需要下载 `stylelint-scss` 插件：

```
npm i -D stylelint-scss
复制代码
```

然后就可以格式化 scss 文件了。

## 扩展

如何格式化 HTML、Vue（或其他后缀） 文件中的 HTML 代码？

`.vue` 文件的 HTML 代码可以使用 `eslint-plugin-vue` 插件来进行格式化：

```js
extends: [
    'plugin:vue/recommended', // 在 .eslintrc.js 文件中加上这一行代码
    '@vue/airbnb',
],
复制代码
```

其他的 HTML 文件需要利用 VSCode 自带的格式化，快捷键是 `shift + alt + f`。假设当前 VSCode 已经打开了一个 HTML 文件，按下 `shift + alt + f` 会提示你选择一种格式化规范。如果没提示，那就是已经有默认的格式化规范了，然后 HTML 文件的所有代码都会格式化，并且格式化规则还可以自己配置。

## 疑难问题

#### `Unknown word (CssSyntaxError)` 错误

解决方案为降级安装 VSCode 的 `stylelint` 插件，点击插件旁边的小齿轮，再点 `Install Another Version`，选择其他版本进行安装。

![image.png](/img-post/开发/IDE/VS Code/ESlint + Stylelint + VSCode自动格式化代码(2020).assets/916cb06bcb6b405aa999625c086a4636~tplv-k3u1fbpfcp-watermark.awebp)

选 `0.87.6` 版本安装就可以了，这时 css 自动格式化功能恢复正常。

#### 忽略 `.vue` 文件中的 HTML 模板验证规则无效

举个例子，如果你将 HTML 模板每行的代码文本长度设为 100，当超过这个长度后 eslint 将会报错。此时如果你还是想超过这个长度，可以选择忽略这个规则：

```js
/* eslint-disable max-len */
复制代码
```

注意，以上这行忽略验证的代码是不会生效的，因为这个注释是 JavaScript 注释，我们需要将注释改为 HTML 格式，这样忽略验证才会生效：

```html
<!-- eslint-disable max-len -->
```


作者：谭光志
链接：https://juejin.cn/post/6892000216020189198
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。