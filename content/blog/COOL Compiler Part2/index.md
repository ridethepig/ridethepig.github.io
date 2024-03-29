+++
title = 'COOL Compiler-Part2'
date = '2023-03-21'
categories = ['编程']
tags = ['Compiler', '编译原理', '课程项目']
+++

## PA2

PA 2-5 正式写编译器。PA2 写词法分析器，首先读一遍 README 和 handout。

> 环境配置
> 
> 因为这个项目的结构非常的智障，导致需要进行一些配置才能让 `clangd` 正常工作。因为是 `Makefile` 项目，所以不能直接生成 `compile_commands.json`。
> 1. 安装 `apt install bear`，这个工具可以拦截 `make` 命令来生成上述的文件。
> 2. 在 PA2 目录下，运行 `make clean && bear -- make lexer`，然后 `clangd` 应该就不会找不到头文件之类的了

> 获取评测脚本
> 
> 现在已经没有办法在线提交测试了（除非花钱？），因此需要从一些奇怪的地方获取测试工具，这里我从 https://github.com/shootfirst/CS143 这个 repo 里面扒了评测脚本（测试数据已经包含在了脚本里了） `pa[2-5]-grading.pl`，测试的话直接用 `perl ./pa2-grading.pl` 就行了。
> 这个东西确实测出来一些自己没考虑到的边角问题，虽然我已经很努力的在编测例了。

写完了发现还是很折磨的，写了快两天的样子，一方面是对于 `flex` 工具和配套的基础设施不是很了解，另一方面是各种细节问题需要处理。

### 实现说明

比较简单的部分是那些只需要返回一个 token 类型的东西（关键字和符号这种），pattern 就是他们自己，然后 action 就一个 `{return (TOKEN)}`。

然后稍微想了一下的，标识符和 Int、Bool 常量，需要写一个简单的正则外加操作 String Table 和 `cool_yylval`。

需要折腾一会的是注释，需要用到 flex 的状态。
- 比较简单的是单行注释，检测到 `--` 就进入 `<SCOMMENT>`（Single-line Comment），然后检测到换行就回到 `INITIAL` 状态。这里是 `.` 和 `\n` 这一组互斥状态，EOF 不需要单独处理，可知已经覆盖了所有的情况。
- 然后是多行注释，不过这里需要支持 nested comment。这玩意的意思是说，需要在注释里面也要成对匹配 `(*` `*)`，不能简单的只考虑最外层或者最内层的符号对。这里需要多加一个变量 `comment_nest_level` 来维护嵌套的层数，流程也更复杂一些。
  首先遇到 `(*` 进入 `<NCOMMENT>`（Nested Comment），同时初始化嵌套层数；然后在该状态下过滤 `(*`，发现一个就叠一层，过滤 `*)` 发现一个减一层或者到底了就退出状态就行了；这里需要特殊处理 EOF，如果在该状态下匹配到 EOF，需要报错，但是因为反正输入流已经结束了，就不需要做 resume 了；遇到 `\n` 维护一下行号；剩下的就交给 `.`。前面其实不需要写 exclusion，因为最长匹配（`(* *)` 都是2个字符，`.` 和 `\n` 都是1个， EOF 不可能匹配到其他的东西上），所以可以简单的囊括所有的情况并维护优先级。
- 除了注释里面的部分，还需要考虑一个单独的 `*)`，需要匹配并报错（而不是`*` `)` 两个 token）。这个是4.1节第5条的要求。这样一来，我们就解决了嵌套注释不匹配的两种错误情况，即：左符号比右符号多 => `EOF in comment` 报错；右符号比左符号多（等价于单独的 `*)`） => `Unmatched *)` 报错。

最折腾的是字符串，因为它要考虑多行的情况，而且需要做特殊的 resume 处理。好消息是这些细节文档都有描述，坏消息是需要看好几遍才能完全理解。
- 为了方便处理出错恢复处理，用了两个状态 `<STRING>` 和 `<STRINGREC>`（String Recovery）（不然需要在 action 里面写很多的特判）。
- 正常情况下，遇到 `"` 进入 `<STRING>`；然后对需要特殊处理的部分编写规则：
  - `\` 转义：匹配两个字符，根据 `\` 后面的字符决定把什么东西写进最终的字符串常量里面，这个东西在 handout 的第4.3节和 manual 的10.2节有描述；需要注意的是，合法多行转义可以（在最后加一个 `\` 然后换行，后面没有其他空白符）在这里一并处理掉，只不过需要写成`(.|\n)`。
  - EOF：根据 manual，除了 `\0` 和 EOF，其他字符均可出现在字符串中。因此，需要单独处理这两玩意。EOF 和前面的注释类似，比较简单。
  - 0字符：0字符需要恢复。除了返回 ERROR Token，还要把状态切换到 `<STRINGREC>`，剩下的部分交给恢复规则。
  - 未转义换行：根据最长匹配，可以直接写 `<STRING>\n`，因为转义过的是2个字符。这个不需要进恢复状态，因为它直接返回 `<INITIAL>` 然后继续下一个字符就相当于处理了恢复的过程。
- 对于除了以上情况的所有字符（可以 exclude 或者把 `.` 写到最后），用 boilerplate 给我们定义的 `string_buf` 存放。这里需要处理一个字符串过长的问题，判断指针超限之后，进 `<STRINGREC` 并返回 ERROR Token，具体的边界情况可以对拍（对比标准实现）来获知。最后就是遇到了未转义的 `"`（根据最长匹配可以直接写 `<STRING>\"`），把 `string_buf` 塞进 `stringtable` 里面，回到 `<INITIAL>`。
- 对于恢复状态 `<STRINGREC>`，根据手册，从下一个未转义的换行符或者未转义的`"`（因为要求是 `closing "`）开始继续正常的词法分析。所以在前面需要把 `\\\n` 和 `\\\"` 给处理掉，然后遇到 `\"|\n` 这两个之后就直接返回 `<INITIAL>` 就行，也不需要 return 任何东西。期间也要记得维护行号。不过事实证明，不需要单独处理 EOF，因为恢复了也没东西了。
- 这里有一个细节问题，就是报错的行号必须等于出错的地方的行号，因此不能留到后面统一返回报错，而是一旦出错就立刻返回。

容易忘掉的点，一个是空白符匹配，需要一个 pattern（action 留空），不然遇到空白符 lexer 会行为异常，同时遇到换行还要维护行号；第二个是不合法字符，有些 ASCII 我们并没有用到，因此需要最后一个 `.` 来匹配并报错、跳过。同样的，`.` 和 `\n`（包含在空白符规则里面）的组合至少保证了所有的字符都会被匹配、处理。

不过其实，最离谱的是自己设计测试样例，这很考验对于手册的理解，不然就会漏掉点什么（虽然漏掉一些 edge case 对于后面也没啥影响就是了）。

在用了官方的评测脚本之后，又发现了一个问题：`\\0`（`0x5C 0x00`） 是不允许的。这个其实判定起来不难，但是会想不到，因为 PA2 的 handout 没写这一条，虽然 manual 里面 `A string may not contain the null` 的确是不允许这种情况（因为它转义完了还是 `\0`），但是谁会想到测这个呢？

### 基础设施说明

个人感觉，一开始做的比较迷惑的主要原因在于它的代码框架比较凌乱，文档也有点谜语人，读了好几遍文档才知道该写点啥。

写之前多读几遍文档，首先看一遍 PA2 的实验说明，了解一下需要干什么，里面也有一些读其他文档和代码的指导。然后是 `cool-manual.pdf` 看第10节（主要是写 pattern 的时候看）和第12节里面那个大的 BNF 文法（里面描述了所有用到的符号），以及 `cool-tour.pdf` 看第3节，在写 action 的时候会用到它来存字符串。最后看一下代码，主要是 `/include/cool-parse.h`，这里面定义了非 ASCII 符号的 TOKEN，还有 `YYSTYPE` 枚举，这个枚举是用来存 lexeme 的，`；lextest.cc` 也要看一下，至少知道输出的都是什么。

这个里面用到的基础设施主要是一个 String Table，用来存放所有遇到的字符串。里面有个 `add_string` 方法，会先查再加还带内存分配，所以直接调用就行。碰到需要存字符串值的东西，各种 identifier、String 常量和 Int 常量（不检查不转换，直接存），这三个每个都有一个单独的实例对象（`idtable` `stringtable` `inttable`），需要根据不同的 token 用。

另外一个就是 `YYSTYPE cool_yylval` 这个枚举。如果有需要存的信息（lexeme 或者错误信息，参考第5节 `Notes for the C++ Version of the Assignment`），每个 action 往里面写一个值，这个对应 PA2 文档第4节开头的部分，当时看了半天才理解到是要存这东西里面。用到里面的3个枚举值：`boolean`、`symbol`、`error_msg`。如果是解析出来 `Bool` 常量，那么直接写进去 `true/false`；如果出错了，给字符指针 `error_msg` 赋值，我猜需要用到 `strdup`，但是单就这个实验看不出来，总感觉要出内存 bug（也不知道后面会不会 free 掉）；剩下来那些需要存 lexeme 的就先 `add_string` 然后它会返回一个 Entry，这东西就是所谓的 Symbol 啦。

### flex 工具踩坑

最后记录一下 `flex` 这个工具的一些坑。
- **版本**：因为不知道在网上的什么地方看到了一个 blog 说是新版本的 `flex` 在处理 c++ 时的行为和旧版不一样，所以选择了旧版本，也就是 `apt install flex-old` 安装的版本（2.5.4）。问题在于我看的文档版本是最新的，遇到了一个不支持的语法，不过其他的倒是没有特别需要注意的地方。旧版本的 `flex` 似乎不支持 `(?i:xxx)` 这种写法，这个在关键字忽略大小写的时候会比较省事（因为只有关键字需要忽略大小写，所以不能开全局忽略），可惜的是旧版本只能像个傻子一样写成 `CLASS [Cc][Ll][Aa][Ss][Ss]`。
- **奇怪的缩进规则**：它居然和 python 一样，依靠缩进来分辨一些东西，每个 Section 的规则还稍有不同。比如在 Rule Section（就是实际写 action 的那一节），pattern 必须无缩进，然后其他的东西（比如注释）必须前面有空白符，不然就会被认为是一个 pattern。在看文档的时候要注意一下 `(un)indented` 这个词，可以避免很多奇怪的事情。  
  一开始没看到 `the pattern must be unindented and the action must begin on the same line` 这一句，没给注释加前面的空格，然后就调了半天的 `unrecognized rule`。
  还有就是 action 可以没有大括号但是不能没有分号，以及它其实是可以写多行的，只要前面有 indent 就行（不然会写出很长一串）。
- **EOF 符号**：`flex` 里面的 EOF 专门有一个 `<<EOF>>` 来表示，但是它不能和其他的 pattern 写在一起，也就是需要单独一行写一个这东西，不然报错。
- **0 字符（null character）**：这个本身没什么特别的，直接用就行，不过需要注意的是，`.` 通配符是包含 `\0` 的，也就是一不小心就用这玩意把 `\0` 给匹配进去了，单独写的那个 `\0` 规则就不生效，会导致一些费解的事情发生。
- **VSCode 插件**：搜到两个还算下载量比较多的，但是都不行，有一个着色有 bug（可能是因为它太老了，接口不兼容了），还有一个加了莫名其妙的、极具误导性的语法检查还关不掉（一点问题都没有的代码给我每行一个红色波浪线）。建议用第二个，然后大脑忽略他的错误提示。

剩下的就看两眼文档就会了，跟课上教的 Regular Expression 基本上大差不差。
