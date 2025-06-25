---
title: "运行 WebAssembly"
date: 2025-06-25T12:30:32+08:00
weight: 2
---

{{< callout type="info" >}}
由于局限于技术栈以及其他原因，团队选择使用 [Expo](https://docs.expo.dev/) 进行前端开发。
{{< /callout >}}

## Emscripten Compiler Frontend (emcc)

[`emcc`](https://emscripten.org/docs/tools_reference/emcc.html) 可以将经过特殊处理的 C 语言源代码编译为 Wasm，使其能够在浏览器等支持 Wasm 运行的环境中运行。

将编译器的源码编译为 Wasm 后，前端可以直接使用编译器中的 C 语言函数。

对于输入输出流，比如最常见的 `stdin`、`stdout` 和 `stderr`，emcc 会使用 console 等 JavaScript 函数处理。

{{% steps %}}

### 一个简单的 C 语言程序源码如下

```c {filename="hello.c",linenos=table}
#include <stdio.h>

int test(int a) {
	printf("Hello, world!\n");
	return a;
}
```

这是一个简单的函数，有非 void 的形参和返回值，函数体内部存在 stdout 输出，足够作为测试用例。

### 对源码稍加改造，加入 emscripten 库

```c {filename="hello.c",linenos=table}
#include <emscripten.h>
#include <stdio.h>

EMSCRIPTEN_KEEPALIVE
int test(int a) {
	printf("Hello, world!\n");
	return a;
}
```

- `EMSCRIPTEN_KEEPALIVE` 宏告诉编译器保留此函数，使其在编译后能从 JavaScript 调用。

### 使用 emcc 命令编译

```bash
emcc hello.c -o hello.js
```

此时 emcc 会生成 hello.js 与 hello.wasm 两个文件：

- `hello.wasm`：c 语言源代码编译产物，包含 EMSCRIPTEN\_KEEPALIVE 宏保留的函数以及其他部分。
- `hello.js`：对 wasm 文件的进一步封装，使用 JavaScript 函数处理了标准输入输出流以及其他部分。

但是由于 Expo 的特殊性，本地不支持导入 Wasm 文件，因此需要让 emcc 将 Wasm 内嵌入 js 文件中。

```bash
emcc hello.c -o hello.js -s WASM=1 -s MODULARIZE=1 -s SINGLE_FILE=1 -s EXPORT_NAME="hello"
```

`EXPORT_NAME` 为导出模块名，c 源代码中的所有函数都会被封装到一个 JavaScript Module 内。

{{% /steps %}}

## 调用 Wasm

Expo TSX 代码如下：

```typescript {filename="HomeScreen.tsx",linenos=table}
import hello from "@/wasm/hello.js"; // 导入 Wasm

export default function HomeScreen() {
  const [text, setText] = useState("init");
  const [text2, setText2] = useState("init");

  return (
    <BaseScreen theme={theme} style={styles.container}>
      <TouchableOpacity onPress={async () => { // async 作用域
        const _hello: any = await hello({
          print: (text: string) => { // hook stdout
            console.log(`stdout: ${text}`);
            setText(text);
          },
        });
        setText2(_hello._test(114514));
      }}>
        <Text>{text}</Text>
        <Text>{text2}</Text>
      </TouchableOpacity>
    </BaseScreen>
  );
}
```

{{% steps %}}

### 使用 `import` 可以轻松导入 js 文件
### 使用 `<TouchableOpacity>` 创建一个异步作用域
### 通过 `await` 调用 hello 模块，并处理标准输出流 stdout

Q: 如何处理输入输出流？  
A: 在异步调用 Wasm 模块时，传入键值对参数，key 为流的名称，value 是一个 lambda。

| 标准流 | key             |
|:------:|:---------------:|
| stdin  | stdin （不常用）|
| stdout | print           |
| stderr | printErr        |

### 运行结果

点击 \<TouchableOpacity\> 后，第一个 \<Text\> 变为 **Hello, world!**，第二个 \<Text\> 变为 **114514**。

{{% /steps %}}
