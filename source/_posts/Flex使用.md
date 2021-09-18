---
title: Flex使用
categories:
  - 学习之路
date: 2021-09-18 14:05:25
tags:
- 编译
---

### Flex介绍

Flex,通过使用正则表达式来生成匹配相应字符串的C代码

例：

~~~flex
/* scanner for a toy Pascal-like language */

%{
/* need this for the call to atof() below */
#include <math.h>
%}

DIGIT    [0-9]
ID       [a-z][a-z0-9]*

%%

{DIGIT}+    {
            printf( "An integer: %s (%d)\n", yytext,
                    atoi( yytext ) );
            }

{DIGIT}+"."{DIGIT}*        {
            printf( "A float: %s (%g)\n", yytext,
                    atof( yytext ) );
            }

if|then|begin|end|procedure|function        {
            printf( "A keyword: %s\n", yytext );
            }

{ID}        printf( "An identifier: %s\n", yytext );

"+"|"-"|"*"|"/"   printf( "An operator: %s\n", yytext );

"{"[^}\n]*"}"     /* eat up one-line comments */

[ \t\n]+          /* eat up whitespace */

.           printf( "Unrecognized character: %s\n", yytext );

%%

main( argc, argv )
int argc;
char **argv;
    {
    ++argv, --argc;  /* skip over program name */
    if ( argc > 0 )
            yyin = fopen( argv[0], "r" );
    else
            yyin = stdin;

    yylex();
    }
~~~

***

### 输入文件格式

~~~
definitions
%%
rules
%%
user code
--------------
- %{ … %} 是直接拷贝到 C 文件开头
- %% … %% 是模式匹配的代码区域, 左边是正则表达式, 右边是匹配的 C 代码
- yytext 代表匹配正则表达式的字符串
- flex 的匹配默认是从最长匹配开始, 如果有多个匹配的正则表达式, 从最早的那个开始匹配, 所以上面的模式匹配, 首先是按照单词 -> 行尾符 -> 剩余字符串的顺序进行匹配的, 不会产生重复统计的问题
- yylex 是调用 flex 的词法分析函数 yylex 进行计算
- Linux 系统上用 -lfl 选项编译, Mac 的编译选项是 -ll
~~~

若要将Flex和Bison联合起来工作，需要将Bison生成的头文件包含进来。

如果要更改默认的lex.yy.c名字,加上-o lexname

* yytext char * 当前匹配的字符串

* yyleng in 当前匹配的字符串长度

* yyin FILE * lex当前的解析文件，默认为标准输入

* yyout FILE * lex解析后的输出文件，默认为标准输出

* yylineno in 当前的行数信息
* yylex(void) 调用lex进行语法分析
* yywrap(void) 在文件(或输入)的末尾调用，如果函数的返回值是1，就停止解析。因此它可以用来解析多个文件。
