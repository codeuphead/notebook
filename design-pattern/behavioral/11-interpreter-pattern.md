# Interpreter 模式

## 语法规则也是类

## 角色

- AbstractExpression

定义了语法树节点的共同接口

- TerminalExpression

终结符表达式

- NonterminalExpression

非终结符表达式

- Context

为解释器进行语法解释提供了必要的信息

- Client

调用 TerminalExpression NonterminalExpression

## 要点

- 其他迷你型语言
  - 正则表达式
  - 检索表达式
  - 批处理语言

- 跳过标记还是读取标记

## 相关

- Composite 模式

NonterminalExpression 多是递归结构，因此常会使用 Composite 模式来实现

- Flyweight 模式

有时会使用来共享 TreminalExpression

- Visitor

推导出语法树后，有时会使用 Visitor 来访问语法树的各个节点