# Tiny Interpreter

一个最小的 Lisp 风格解释器，用于理解程序如何被求值。

## 核心接口与不变量

### 接口

```python
# 词法分析
lexer = Lexer(source_code)
tokens = lexer.tokenize()

# 语法分析
parser = Parser(tokens)
ast = parser.parse()

# 求值
evaluator = Evaluator()
result = evaluator.eval(ast_node, environment)

# 一体化接口
evaluator = Evaluator()
result = evaluator.run(source_code)
```

### 关键不变量

1. **词法作用域**：变量查找遵循静态作用域规则，内层环境可以访问外层环境的变量
2. **闭包捕获**：闭包捕获定义时的环境，而不是调用时的环境
3. **括号匹配**：所有左括号必须有对应的右括号
4. **类型安全**：运算符只能应用于正确类型的操作数

## 快速开始

### 安装

```bash
# 克隆仓库
git clone https://github.com/first-principles-cs/tiny-interpreter.git
cd tiny-interpreter

# 安装依赖
make install
```

### 运行

```bash
# 运行 REPL
python -m src.tiny_interpreter.main

# 运行文件
python -m src.tiny_interpreter.main examples/factorial.lisp
```

### 测试

```bash
# 运行所有测试
make test

# 运行测试并生成覆盖率报告
make test-cov
```

## 架构概览

Tiny Interpreter 由四个主要组件组成：

### 1. 词法分析器（Lexer）

**职责**：将字符流转换为 token 流

**Token 类型**：
- `LPAREN`, `RPAREN`: 括号
- `NUMBER`: 整数
- `SYMBOL`: 标识符
- `BOOLEAN`: `#t` 或 `#f`

### 2. 语法分析器（Parser）

**职责**：将 token 流转换为抽象语法树（AST）

**AST 节点类型**：
- `Number`: 数字字面量
- `Boolean`: 布尔字面量
- `Symbol`: 符号（变量名或函数名）
- `SExpression`: S-表达式（列表）

### 3. 环境模型（Environment）

**职责**：管理变量绑定，支持词法作用域

**数据结构**：
- 环境是一个字典 + 父环境的引用
- 支持嵌套作用域

### 4. 求值器（Evaluator）

**职责**：对 AST 进行求值

**求值规则**：
- 自求值：数字、布尔值
- 变量查找：符号
- 特殊形式：`define`, `lambda`, `if`, `quote`, `begin`
- 函数调用：先求值参数，再应用函数

## 语言特性

### 数据类型

- **整数**：`42`, `-10`
- **布尔值**：`#t`, `#f`
- **符号**：`foo`, `bar-baz`
- **列表**：`(1 2 3)`

### 特殊形式

```lisp
; 定义变量
(define x 42)

; 定义函数
(define square (lambda (x) (* x x)))

; 条件表达式
(if (< x 10) "small" "large")

; 引用（不求值）
(quote (1 2 3))

; 顺序执行
(begin
  (define x 1)
  (define y 2)
  (+ x y))
```

### 内置函数

**算术运算**：`+`, `-`, `*`, `/`

**比较运算**：`=`, `<`, `>`, `<=`, `>=`

**列表操作**：`cons`, `car`, `cdr`, `list`, `null?`

**类型判断**：`number?`, `boolean?`, `list?`

## 示例程序

### 递归：阶乘

```lisp
(define factorial
  (lambda (n)
    (if (= n 0)
        1
        (* n (factorial (- n 1))))))

(factorial 5)  ; => 120
```

### 闭包：计数器

```lisp
(define make-counter
  (lambda (init)
    (lambda ()
      (begin
        (define result init)
        (define init (+ init 1))
        result))))

(define counter (make-counter 0))
(counter)  ; => 0
(counter)  ; => 1
(counter)  ; => 2
```

### 高阶函数

```lisp
(define apply-twice
  (lambda (f x)
    (f (f x))))

(define add1 (lambda (x) (+ x 1)))

(apply-twice add1 5)  ; => 7
```

## 失败模型与测试

### 失败模型

本系统考虑以下失败场景：

1. **词法错误**：非法字符、格式错误的 token
2. **语法错误**：括号不匹配、非法的 AST 结构
3. **运行时错误**：未定义的变量、类型错误、参数数量不匹配

### 测试策略

- **单元测试**：测试每个组件（lexer, parser, evaluator）
- **集成测试**：端到端测试示例程序
- **边界测试**：空输入、嵌套深度、错误处理

### 运行测试

```bash
# 运行所有测试
pytest tests/ -v

# 运行特定测试
pytest tests/test_lexer.py -v

# 生成覆盖率报告
pytest tests/ --cov=src/tiny_interpreter --cov-report=html
```

## 设计文档

详细的设计文档请参考 [docs/design.md](./docs/design.md)。

## 许可证

本项目采用 MIT License，详见 [LICENSE](./LICENSE)。

## 关联课程

本项目是 [first-principles-cs](https://github.com/first-principles-cs/guide) 课程体系的一部分，对应 [Module B: 语言与执行](https://github.com/first-principles-cs/guide/blob/main/courses/module-b-language-execution.md)。
