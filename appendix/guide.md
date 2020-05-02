# 编写指南

## 本地预览

本书可用 Gitbook 编译生成网页版浏览阅读，命令如下：

```
$ npm install -g gitbook-cli
$ gitbook install
$ gitbook serve
```

随后可打开 [http://localhost:4000/](http://localhost:4000/) 浏览。

## 文档编辑

本项目使用 Markdown (更准确地说是 [GitHub Flavored Markdown](https://github.github.com/gfm/), GFM) 的格式编写，可选择 [Typora](https://typora.io/) 或者其它的 Markdown 编辑器进行修改。对于 Markdown 不了解的请参阅这份 [简明指南](https://www.markdown.cn/)。

本项目还支持使用 mathjax 公式渲染。由于 GitHub 本身不对公式进行渲染，无法正常显示的可以安装 [GitHub with MathJax](https://chrome.google.com/webstore/detail/github-with-mathjax/ioemnmodlmafdkllaclgeombjnmnbima) 插件，以获得良好的阅读体验。

## 排版指南

本项目建议参照 [中文文案排版指北](https://github.com/sparanoid/chinese-copywriting-guidelines) 的排版规范，主要采种下列几点：

* 中英文之间需要增加空格
* 中文与数字之间需要增加空格
* 全角标点与其他字符之间不加空格
* 数字使用半角字符
* 专有名词使用正确的大小写

## 图片添加

图片统一添加到项目根目录的 `images` 文件夹，命名方式根据图片的内容，以全小写英文的方式命名，如 `double-spending-problem.png`。如果需进一步区分，可增加所在章节的前缀，如 `transaction-double-spending-problem.png`。

引用图片的方式建议增加 Alt Text，如 `![Double Spending Problem](/images/double-spending-problem.png)`

由于 Markdown 默认显示全宽图片无法指定大小，如果需要特别指定图片大小的场景，建议使用 HTML 代码来指定大小和居中布局。 `<div style="width: 50%; margin: auto">![alt-text-for-image](/images/double-spending-problem.png)</div>`

## 提交流程

本项目欢迎读者直接在 GitHub 上提交勘误，也可以发至邮箱 imcoddy@gmail.com 反馈。

请先将本项目 fork 至自己的 GitHub 帐户里面，并添加本项目为 `upstream` 以便能及时获取最近的更新：

```
$ git remote add upstream https://github.com/imcoddy/rebirth-of-bitcoin.git
```

每次编辑时从 `develop` 新建 `feature/summary-of-modification` 的新分支提交 Pullrequest。具体的操作方式如下：

```
$ git checkout develop
$ git pull upstream develop
$ git checkout -b feature/new-summary-of-modification (自行根据需要修改为合适的名称)
```

提交时直接将 `feature/summary-of-modification` 向 `imcoddy/rebirth-of-bitcoin` 的 `develop` 分支提交 Pullrequest 即可。
