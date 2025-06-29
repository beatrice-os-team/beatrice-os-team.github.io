---
title: "Live Code Platform"
date: 2025-06-29T20:51:15+08:00
weight: 3
---

# 源代码分析

## 工程结构规划 

```
.
├── doc
├── images
├── index.js # 前端启动 JavaScript 脚本
├── LICENSE # MIT
├── losu # 编译器 C 语言源代码
│   └── ...
├── Makefile # 项目构建脚本
├── package.json
├── package-lock.json
├── README.md
├── server # 前端服务端代码
└── www # 前端静态资源
    ├── assets
    ├── error.html
    ├── index.html
    ├── pages
    └── test.html
```

## 执行 Losu 编译器的函数

[editor.js](https://github.com/beatrice-os-team/live-code-platform/blob/main/www/assets/js/editor.js#L53-L64) 文件中的 `editor_run` 函数：

```javascript
let losu = await LosuLiveCode({
    print(text) {
        document.getElementById('editor-result').innerText +=
            `<span style="color:white">` + text + `</span><br>`;
    },
    printErr(text) {
        document.getElementById('editor-result').innerText +=
            `<span style="color:red">` + text + `</span><br>`;
    },
});
losu.ccall('run', [], ['string'], [this.editor.getValue()]);
```

1. emcc 将 Losu 编译器源码编译为 `LosuLiveCode` JavaScript 模块。
2. Hook STDOUT（`print`）和 STDERR（`printErr`），将输出流重定向到 DOM 元素。
3. 使用 ccall 运行 `run` 函数，此函数为运行 Losu 源文件的函数。
