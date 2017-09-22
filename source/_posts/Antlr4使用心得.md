---
title: Antlr4使用心得
date: 2017-09-22
categories: java
---



### 环境搭建

- 所需环境：

  1. idea
  2. antlr4
- 具体步骤：
  1. 安装idea；
     - 官网下载idea2017版，直接安装。
  2. 安装antlr4插件；
     - 打开idea，点击 file -> Settings; 找到plugin选项，点击查找antlr，安装antlr插件。
  3. 安装antlr4-jar包；
     - antlr4的idea插件是不带antlr-4.7.jar包的，需要自己手动下载。直接官网下载即可。
- 安装完成后，打开idea，创建新的java项目。新建文件时，可以手动新建*.g4文件，idea会自动识别。


### 开始使用

打开idea，创建新的项目 `Hello` ，新建g4文件`Hello.g4`,如下：

![Hello.g4](D:\learn\解释器构造\素材02.png)

- g4文件是antlr4的核心部分，有关antlr4的使用绝大部分都写在g4文件中，这里我们先使用官网的demo，一窥究竟：

  ![demo](D:\learn\解释器构造\素材03.png)

- `grammar` 定义了文法的名字，需要注意的是，这里必须与文件同名，否则会报错；

- Antlr的语法，关于构造文法的部分，与编译原理中使用的语法大致相同，上图中 `'Hello'` 匹配关键字Hello，后面跟着一个标识符`ID` ，ID使用正则表达式的方法表示匹配小写字母。`WS` 跳过空格、制表、回车符和换行符。

- 该语法构造完成后，可以进行简单的测试，使用我们之前安装好的Antlr4的插件 `ANTLR Preview`，输入测试语句后，会自动生成语法树。如下图所示：

  ![preview](D:\learn\解释器构造\素材04.png)

### 使用Antlr快速构造Calculator解释器

- 分析：
  - 需要实现最基本的四则运算；
  - 能够进行简单的赋值操作；
  - 使用`print` 函数打印内容。



<!--more-->

- 语法：

  - 四则运算的语法如下图：![四则运算语法](素材05.png)

    - 需要注意的是，该语法是有左递归的，并且有二义性。但是强大的`Antlr`帮我们解决了这个问题。`Antlr`**隐式地允许**我们指定运算符优先级，规则`expr`	中，乘除的规则在前，有利于解决运算符二义性的情况。

    - 同时，不同于其他语法分析器，`Antlr` **v4**是可以处理**直接**左递归的。直接左递归指的是直接调用在选项左边缘自身的规则，上图中`expr: expr op=('*'|'/') expr`便是直接左递归。但是它不能处理间接左递归，也就是说，`Antlr4`不能处理如下的情况：

      ```
      expr: atom op=('*'|'/') expr        # MulDiv //间接左递归无法处理
          | atom op=('+'|'-') expr        # AddSub
          ;
      term: FOL                           # fol
          | ID                            # id
          | '(' expr ')'                  # parens
          ;
      ```

  - 简单的赋值操作：如图

    ![赋值](素材06.png)

    - 图中的assign部分便是赋值的规则，`ID` 是一个词法规则名字，`expr` 对应的是一个语法规则名字，而图中的`'='` `';'`是单个的`token` 。

  - `print` 语法如上图所示，这里不再赘述。

- 词法规则

  - 上述已经简绍了完成计算器的语法结构，这里贴出相关的词法：

    ![词法](素材07.png)

    - `MUL DIV ADD SUB`分别是乘除加减token的词法规则；
    - `ID`：表示匹配例如`aaaa, a123 B123b`等形式，`Antlr`里面的正则匹配是**贪婪**的，需要注意；
    - `FOL`： 表示匹配整数和浮点数；
    - `NEWLINE`：表示匹配换行符； 
    - `WS`: 表示跳过制表符；需要注意的是，这里WS不能写成如`[ \r\n\t]+ -> skip`这种形式，原因是`Antlr`采用贪婪的正则匹配，并且在有两个词法规则可以匹配同一个字符串时，隐式匹配**能匹配最长字符串**的那个规则。举个例子：
      - `"print (1) ; \t\r\n"` 这样字符串中的`\t\r\n`并不会跳过`\t`匹配`NEWLINE`，它会直接匹配一个最长的符合规则的`WS`，然后Lexer的时候会报错，缺少`NEWLINE`。

- 测试

  - 使用`Antlr4`的插件，检查写好的语法是否有问题。如下图：

    ![语法树](素材08.png)

- 使用antlr插件，生成Lexer和Parser，如图：

  ![生成](素材09.png)

  - 生成如下六个文件：

    ```
    Calculator.tokens
    CalculatorBaseVisitor.java
    CalculatorLexer.java
    CalculatorLexer.tokens
    CalculatorParser.java
    CalculatorVisitor.java
    ```

    - 需要注意的是，`Antlr`默认自动生成的是Listener模式，即会生成`CalculatorListener.java`和`CalculatorBaseListener.java`，需要配置Antlr插件，勾选生成Visitor选项；

      ![配置antlr](素材10.png)

  - 下面要做的事情就是实现一个`MyVisitor`类，它通过遍历表达式语法分析树计算和返回值。

    - 创建`MyVisitor`类，继承`CalculatorBaseVisitor`类，相关代码如下所示：

      ```java
      public class MyVisitor extends CalculatorBaseVisitor<Float> {
          private HashMap<String, Float> tables = new HashMap<>();

          @Override
          public Float visitAssign(CalculatorParser.AssignContext ctx) {
              String id = ctx.ID().getText();
              Float value = visit(ctx.expr());
              tables.put(id,value);
              return value;
          }

          @Override
          public Float visitPrintRes(CalculatorParser.PrintResContext ctx) {
              System.out.println(visit(ctx.expr()));
              return 0f;
          }

          @Override
          public Float visitParens(CalculatorParser.ParensContext ctx) {
              return visit(ctx.expr());
          }

          @Override
          public Float visitMulDiv(CalculatorParser.MulDivContext ctx) {
              Float l = visit(ctx.expr(0));
              Float r = visit(ctx.expr(1));
              if(ctx.op.getType()==CalculatorLexer.MUL){
                  return l*r;
              }else{
                  if(r==0){
                      throw ctx.exception;
                  }
                	return l/r;
              }
          }

          @Override
          public Float visitAddSub(CalculatorParser.AddSubContext ctx) {
              Float l = visit(ctx.expr(0));
              Float r = visit(ctx.expr(1));
              if(ctx.op.getType()==CalculatorLexer.ADD){
                  return l+r;
              }else{
                  return l-r;
              }
          }

          @Override
          public Float visitId(CalculatorParser.IdContext ctx) {

              return tables.getOrDefault(ctx.ID().getText(), 0f);
          }

          @Override
          public Float visitFol(CalculatorParser.FolContext ctx) {

              return Float.parseFloat(ctx.FOL().getText());
          }

      }
      ```

    - 同时，我们要覆写继承自`CalculatorBaseVisitor<Float>`类与语句和表达式选项相关的方法，在这个计算器项目中，为以下几个方法：

      ```java
          @Override
          public Float visitAssign(CalculatorParser.AssignContext ctx){}

          @Override
          public Float visitPrintRes(CalculatorParser.PrintResContext ctx) {}

          @Override
          public Float visitParens(CalculatorParser.ParensContext ctx) {}

          @Override
          public Float visitMulDiv(CalculatorParser.MulDivContext ctx) {}

          @Override
          public Float visitAddSub(CalculatorParser.AddSubContext ctx){}

          @Override
          public Float visitId(CalculatorParser.IdContext ctx) {}

          @Override
          public Float visitFol(CalculatorParser.FolContext ctx) {}
      ```

  - 最后，编写启动函数，代码如下：

    ```java
    public class Calculator {

        public static void main(String[] args) {
            ANTLRInputStream inputStream = null;
            try {
                inputStream = new ANTLRInputStream(new FileInputStream("src/test.in"));
            } catch (IOException e) {
                e.printStackTrace();
            }

            CalculatorLexer lexer = new CalculatorLexer(inputStream);

            CommonTokenStream tokenStream = new CommonTokenStream(lexer);
            CalculatorParser parser = new CalculatorParser(tokenStream);
            ParseTree parseTree = parser.prog();
            MyVisitor visitor = new MyVisitor();
            visitor.visit(parseTree);
        }
    }
    ```


  - 我们使用一个`test.in`文件进行测试，下面是文件的内容：

    ```
    a=(10*356.1+1.1)/2-1024.1*1.1;
    print(a+1.2);
    b=(1024.1-22)/43-1;
    print(a*b);
    ```

    Outputs:

    > 655.74005
    > 14599.287


### 使用总结

以上是搭建Antlr环境，使用Antlr构造计算器语法的大体步骤，在使用Antlr的过程时，发现了Antlr的一些优点，包括但不限于：

- 具有很好的语法结构，我们可以很容易的使用自然语言描述，来构造一个语法；
- 同样的，可以使用正则表达式匹配标识符，迅速完成词法规则；
- 直观的语法树图，帮助测试和调试语法；
- 解决了直接左递归的问题；
- 可以隐式的指定运算符优先级，在一些简单的场景例如计算器的加减乘除上面十分实用；
- 可以在语法中嵌入操作。

由于自己的不当操作，也在最初使用Antlr的时候遇到了一些问题。

- 例如上文提到了`WS`与`NEWLINE`匹配相同字符的问题，Antlr默认选择能匹配最长字符的规则；
- Antlr使用LL(k)的文法，但具体内部构造，现在我还没有清楚，需要进一步研究源码和文档。


