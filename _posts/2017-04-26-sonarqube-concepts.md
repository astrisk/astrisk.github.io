---
title:  "SONARQUBE-CONCEPTS"
date:   2014-09-10 22:37:00
categories: example
---

## **SonarQube 简介**

Sonar 是一个用于代码质量管理的开源平台，用于管理源代码的质量。Sonar 通过插件形式，可以支持包括Java/C#/C/C++/PL/SQL/Cobol/JavaScrip/Groovy 等等二十几种编程语言的代码质量管理与检测。

Sonar 可以从以下七个维度检测代码质量，而作为开发人员至少需要处理前5种代码质量问题：

1. **不遵循代码标准**  
    Sonar 可以通过PMD，CheckStyle，Findbugs 等等代码规则检测工具规范代码编写。
2. **潜在的缺陷**  
    Sonar 可以通过PMD，CheckStyle，Findbugs 等等代码规则检测工具检测出潜在的缺陷。
3. **糟糕的复杂度分布**  
    文件、类、方法等，如果复杂度过高将难以改变，这会使得开发人员难以理解它们，且如果没有自动化的单元测试，对于程序中的任何组件的改变都将可能导致需要全面的回归测试。
4. **重复**  
    显然程序中包含大量复制粘贴的代码是质量低下的，Sonar 可以展示源码中重复严重的地方。
5. **注释不足或者过多**  
    没有注释将使代码可读性变差，特别是当不可避免地出现人员变动时，程序的可读性将大幅下降。而过多的注释又会使得开发人员将精力过多地花费在阅读注释上，亦违背初衷。
6. **缺乏单元测试**  
    Sonar 可以很方便地统计并展示单元测试覆盖率。
7. **糟糕的设计**  
    通过 Sonar 可以找出循环，展示包与包、类与类之间的相互依赖关系，可以检测自定义的架构规则。通过 Sonar 可以管理第三方的 jar 包，可以利用 LCOM4 检测单个任务规则的应用情况，检测耦合。

---

## **Duplication**

- Duplicated blocks  重复块数
- Duplicated files  重复文件数
- Duplicated lines  重复行数
- Duplicated lines (%)  重复行占总行数的百分比

## **Documentation**

- Comment lines  注释行数  
    例如Java代码中包含Javadoc、多行注释、单行注释的总数。不包括空注释行、头文件中的注释（主要用于定义许可证）以及commented-out行。
- Public documented API (%)  添加注释的公有API百分比
- Public undocumented API  未添加注释API数
- Public API  公共类、公共方法（不包括访问器）以及公共属性（不包括public final static类型的）的数目。

## **Complexity**

Complexity 圈复杂度，也被称为McCabe度量。它简单归结为一个方法中’if’，‘for’，’while’等块的数目。每个方法默认有最小的值为1，按照控制流一直往下通过程序，一旦遇到以下关键字或者其同类的词，就加1：if,while,repeat,for,and,or,给case(switch)语句中的每一种情况加1.《代码大全》的作者里对圈复杂度的建议：

1. 0~5：子程序可能还不错。
2. 6~20：得想办法简化子程序。
3. 10+：把子程序得某一部分拆成另一个子程序并调用它。
   - Complexity 默认为各个文件复杂度的加和
   - Complexity /function 方法复杂度
   - Complexity /file 文件复杂度
   - Complexity /class 类复杂度

## **Coverage**

- Line coverage 行覆盖率
- Condition coverage 条件覆盖率
- Uncovered lines 未覆盖的行
- Uncovered branches 未覆盖的分支
- Lines to cover 覆盖的行
- Unit test errors error的unit test数目
- Unit test failures 失败的unit test数目
- Skipped unit tests skip的unit test 个数
- Unit test success (%) unit test 成功率
- Unit tests duration unit tests执行时间

## **Size**

- Lines 代码行数
- Statements 代码语句数目  
    比如Java语言规范中没有块定义的语句数目；此数目在遇到含有if, else, while, do, for, switch, break, continue, return, throw, synchronized, catch, finally等关键字的语句时增加。
- Functions 方法数目
- Classes 类数目
- Files 文件数目
- Directories 目录数目

## **Bug**

An issue that represents something wrong in the code. If this has not broken yet, it will, and probably at the worst possible moment. This needs to be fixed. Yesterday.

## **Vulnerability**

A security-related issue which represents a potential backdoor for attackers. [See also Security-related rules](http://docs.sonarqube.org/display/SONAR/Security-related+rules)

## **Code Smell**

A maintainability-related issue in the code. Leaving it as-is means that at best maintainers will have a harder time than they should making changes to the code. At worst, they'll be so confused by the state of the code that they'll introduce additional errors as they make changes.

## **Issue**

When a component does not comply with a coding rule, an issue is logged (was violation prior to SonarQube 3.6) on the snapshot.

An issue can be logged on a source file or a unit test file. There are 3 types of issue:

- Code Smell : an issue affecting your maintainability rating, preventing you to inject changes as fast as when you start from scratch
- Bug : an issue highlighting a real or potential point of failure in your software
- Vulnerability : an issue highlighting a security hole that can be used to attack your software
