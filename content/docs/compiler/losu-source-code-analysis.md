---
title: "Losu Compiler Source Code Analysis"
date: 2025-06-29T21:43:14+08:00
weight: 3
---

## 项目简介

本项目是 OS 功能赛赛题，基于 Losu 语言，结合 Web Assembly 技术，开发一个编程教学环境，模拟并演示程序在 OS 中的执行过程，包括：

- 源代码编写（使用 Losu 编写源文件）
- 编译功能演示（生成 Losu 字节码文件）
    - **词法分析（Lexical Analysis）**
        将源代码分解为标记（Token），如标识符、运算符、字符串等。
    - **语法分析（Syntax Analysis）**
        根据语法规则构建抽象语法树（AST）。
    - **字节码生成**
        将 AST 转换为洛书虚拟机（Losu VM）可执行的字节码。



洛书 (Losu) 编程语言，全称 Language Of Systemd Units ，超轻量、跨平台、易扩展、支持中文代码、拥有中文文档和视频资料，致力于打造一门开源、高效、强大的编程语言。

洛书是一款：

- 图灵完备的编程语言，支持面向过程、面向对象与部分元编程的特性
- 全平台可用，支持 Windows、Linux、RTOS 等多种操作系统，解释器可以由 任何 遵循 c99 标准的编译器编译通过并运行于任何拥有 libc 的平台之上
- 高性能、低开销、零依赖，可以在低资源设备上运行 完整 的内核，具备裸机运行能力。

## 实验步骤

1. 使用 Losu 编写源代码
2. 词法分析
3. 语法分析

## 源代码编写

[Losu 语法手册](https://losu.tech/wiki/readme.mdv)

demo.els Losu 源代码示例：

```python
# 基类 people
class people:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# 派生类 chinese
class chinese(people):
    # 派生构造函数，自动调用基类构造函数
    def __init__(self, name, age, sex):
        self.sex = sex
    # 新增成员函数
    def print_info(self):
        print("chinese:", self.name, ' ' , self.age, ' ', self.sex)        
# 派生类 student
class student(chinese):
    # 派生构造函数，自动调用基类构造函数
    def __init__(self, name, age, sex, score):
        print('student:',name ,' arrive school')
        self.score = score
    # 新增成员函数
    def print_info(self):
        print("student:", self.name, ' ' , self.age, ' ', self.sex, ' ', self.score)

    # 析构函数
    def __del__(self):
        print('student:',self.name, ' leave school')
    # 重载运算符 
    def __call__(self):
        print('student:', self.name, ' call me')

print('type student is ', type(student))
let s1 = student('zhangsan', 18, 'male', 90)
let s2 = student('lisi', 19, 'female', 80)
s1.print_info()
s2.print_info()
s1()
s2()
```



## 词法分析器(Lexer)详细分析

### 核心数据结构

```c
/* syntax lex */
typedef struct losu_syntax_lex {
  int16_t current; /* uint8_t -> int16_t */
  struct losu_syntax_token tk;
  struct losu_syntax_token tkahead;
  struct losu_syntax_func* fs;
  losu_vm_t vm;
  struct losu_syntax_io* io;
  int32_t linenumber;
  int32_t lastline;
  struct losu_syntax_lex_idt {
    int32_t read;
    int32_t nowidt;
    int32_t size;
    int32_t tmp;
  } idt;
} losu_syntax_lex, *losu_syntax_lex_t;
```

### 词法分析核心函数：`losu_syntax_lex_next`

这是词法分析器的心脏函数，负责将源代码分解为Token流。

#### 主要功能模块：

1.  **缩进处理**

    ```c
    if (lex->idt.tmp > 0) {
        lex->idt.tmp--;
        lex->idt.nowidt--;
        return TOKEN_PASS;
    }
    ```

    -   处理Python风格的缩进语法
    -   管理代码块的开始和结束

2.  **空白字符和注释处理**

    ```c
    case '\r':
    case '\t':
    case ' ':
    case '\0': {
        next(lex);
        break;
    }
    case '#': {
        // 处理单行注释
        while (lex->current != '\n') {
            next(lex);
            if (lex->current == EOF) return TOKEN_EOZ;
        }
        break;
    }
    ```

    -   忽略空白字符
    -   支持 `#` 开头的单行注释

3.  **运算符识别**

    ```c
    case ':': {
        next(lex);
        if (lex->current == ':') return TOKEN_ASSIGN;  // ::
        if (lex->current == '>') return TOKEN_PIPE;    // :>
        if (lex->current == '<') {
            // 处理字符串嵌入语法 :<"..."
        }
        return ':';
    }
    ```

    -   支持复合运算符（如 `::`, `:>`, `==`, `!=` 等）
    -   采用前瞻技术确定运算符类型

4.  **字符串处理**

    ```c
    case '\'':
    case '\"': {
        readstr(lex, lex->current, tk);
        printf("%s %s\n", losu_syntax_keyword[TOKEN_STRING - 256].str, 
               lex->tk.info.s->str);
        return TOKEN_STRING;
    }
    ```

    -   支持单引号和双引号字符串
    -   内置调试输出功能

5.  **数字处理**

    ```c
    case '0': {
        next(lex);
        if (lex->current == 'x') {
            // 十六进制数字处理
        } else if (lex->current == 'b') {
            // 二进制数字处理
        } else if (lex->current == 'o') {
            // 八进制数字处理
        } else if (lex->current == 'a') {
            // ASCII字符处理
        }
    }
    
    case '1':
    case '2':
    case '3':
    case '4':
    case '5':
    case '6':
    case '7':
    case '8':
    case '9': {
    	readnum(lex, tk, 0);
    	printf("%s %f\n", losu_syntax_keyword[TOKEN_NUMBER - 256].str, lex->tk.info.num);
    	return TOKEN_NUMBER;
    }
    ```

    -   支持多种数字格式：
        -   `0x` 十六进制
        -   `0b` 二进制
        -   `0o` 八进制
        -   `0a` ASCII字符
        -   浮点数

6.  **文件嵌入功能**

    ```c
    case '`': {
        readstr(lex, lex->current, tk);
        // 读取文件内容并嵌入到代码中
        FILE* fp = fopen(filename, "rb");
        // ... 文件处理逻辑
    }
    ```

    -   支持使用反引号嵌入外部文件
    -   编译时将文件内容直接嵌入代码

7.  **关键字和标识符处理**

    ```c
    default: {
        if (!isalpha(lex->current) && lex->current != '_' && 
            lex->current < 0x80) {
            // ASCII字符处理
        }
        losu_object_string_t ts = losu_objstring_new(lex->vm, readname(lex));
        if (ts->marked > 0xff) {
            // 关键字处理
            return ts->marked;
        }
        // 标识符处理
        return TOKEN_NAME;
    }
    ```

### Token类型定义

```c
/* token */
typedef enum _losu_token {
  /** logic */
  TOKEN_AND = 256,
  TOKEN_OR,
  TOKEN_NOT,
  TOKEN_TRUE,
  TOKEN_FALSE,

  /** pass */
  TOKEN_PASS,

  /** if */
  TOKEN_IF,
  TOKEN_ELSE,
  TOKEN_ELSEIF,

  /** def */
  TOKEN_DEF,
  TOKEN_LAMBDA,
  TOKEN_ARG,

  /** let global  */
  TOKEN_LET,
  TOKEN_GLOBAL,
  TOKEN_CLASS,
  // TOKEN_MODULE,

  /* return break continue yield */
  TOKEN_RETURN,
  TOKEN_BREAK,
  TOKEN_CONTINUE,
  TOKEN_YIELD,

  /* with until for */
  TOKEN_WHILE,
  TOKEN_UNTIL,
  TOKEN_FOR,

  /* import */
  TOKEN_IMPORT,
  /* async */
  TOKEN_ASYNC,

  /* match case */
  TOKEN_MATCH,
  TOKEN_CASE,
  /* assert except */
  // TOKEN_ASSERT,
  TOKEN_EXCEPT,
  TOKEN_RAISE,

  /* == >= <= != :: ** |> */
  TOKEN_EQ,
  TOKEN_GE,
  TOKEN_LE,
  TOKEN_NE,
  TOKEN_ASSIGN,
  TOKEN_POW,
  TOKEN_PIPE,


  /* other */
  TOKEN_NAME,
  TOKEN_NUMBER,
  TOKEN_STRING,

  TOKEN_EOZ,
} _losu_token;
```

## 语法分析器(Parser)分析

### 核心函数：`losu_syntax_parse`

```c
losu_object_scode_t losu_syntax_parse(losu_vm_t vm, losu_syntax_io_t io) {
    losu_syntax_lex lex = {0};
    losu_syntax_func func = {0};
    losu_ctype_bool islast = 0;
    
    // 1. 初始化词法分析器
    __losu_syntaxpar_setin(vm, &lex, io);
    losu_syntax_lex_init(&lex);
    
    // 2. 创建函数上下文
    __losu_syntaxpar_newfunc(&lex, &func);
    
    // 3. 注册内置函数
    for (int i = 0; i < sizeof(losu_syntax_inlinefunc) / sizeof(const char*); i++)
        __losu_syntaxpar_newglovar(&lex, 
            losu_objstring_newconst(vm, losu_syntax_inlinefunc[i]));
    
    // 4. 开始语法分析
    __losu_syntaxpar_next(&lex);
    while (!islast && !__losu_syntaxpar_checkblock(lex.tk.token))
        islast = __losu_syntaxpar_stat(&lex);
    
    // 5. 检查语法正确性
    __losu_syntaxpar_checkcondition(&lex, 
        (lex.tk.token == TOKEN_EOZ), "invalid token");
    
    return func.fcode;
}
```

### 语法分析流程

1.  **初始化阶段**
    -   设置输入源
    -   初始化词法分析器
    -   创建函数上下文
2.  **预处理阶段**
    -   注册内置函数到全局符号表
    -   建立初始作用域
3.  **语法分析阶段**
    -   逐个处理语句(`__losu_syntaxpar_stat`)
    -   检查语法块结构
    -   构建抽象语法树
4.  **验证阶段**
    -   确保所有Token都被正确处理
    -   检查语法完整性

### 特色功能分析

#### 1. 缩进敏感语法

```c
// 缩进管理结构
typedef struct {
    int nowidt;    // 当前缩进级别
    int read;      // 读取的缩进数
    int size;      // 缩进大小(默认4)
    int tmp;       // 临时缩进计数
} losu_syntax_lex_idt;
```

洛书采用类似Python的缩进语法，通过精确的缩进计算来确定代码块边界。

#### 2. 文件嵌入机制

```c
case '`': {
    // 使用反引号可以在编译时嵌入文件内容
    readstr(lex, lex->current, tk);
    FILE* fp = fopen(filename, "rb");
    // 将文件内容作为字符串嵌入
}
```

这是一个独特的功能，允许在编译时将外部文件内容直接嵌入到代码中。

#### 3. 多种数字格式支持

-   **十六进制**：`0xFF`
-   **二进制**：`0b1010`
-   **八进制**：`0o777`
-   **ASCII**：`0aA` (字符A的ASCII值)
-   **浮点数**：`3.14`

#### 4. 灵活的运算符系统

```c
::    // 赋值
:>    // 管道运算符
|>    // 另一种管道运算符
==    // 相等比较
!=    // 不等比较
**    // 幂运算
...   // 参数展开
```

### 错误处理机制

编译器采用统一的错误处理机制：

```c
void losu_syntax_error(losu_syntax_lex_t lex, const char* fmt, ...) 
  // 格式化错误信息
  // 包含行号、列号等上下文信息
  // 提供清晰的错误描述
  char tmp[256] = {0};
  va_list ap;
  va_start(ap, fmt);
  vsnprintf(tmp, 256, fmt, ap);
  fprintf(stderr, "losu v%s\n\tsyntax error\n\t%s\n\tat line %d\n\tof %s\n",
          LOSU_RELEASE, tmp, lex->linenumber,
          lex->io->name != NULL ? lex->io->name : "<unknown>");
  va_end(ap);
  losu_sigmsg_throw(lex->vm, losu_signal_error);
}
```

## 实际应用示例

```python
# demo.els - Losu源代码示例
print("hello")
```

编译过程：

1.  **词法分析**：`print` → TOKEN_NAME, `"hello"` → TOKEN_STRING
2.  **语法分析**：构建函数调用语法树

