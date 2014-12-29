---
layout: post
title: Flex, A fast scanner generator
category: knowledge
tags: flex bison xx
---

> One size does not fit all

Flex is a tool for generating scanners: programs which recognized lexical patterns in text. flex reads the given input files, or its standard input if no file names are given, for a description of a scanner to generate.

The output of flex is the file `lex.yy.c`, which contains the scanning routine `yylex()`, a number of tables used by it for matching tokens, and a number of auxiliary routines and macros. By default, `yylex()` is declared as follows:

```c
int yylex()
{
    ... various definitions and the actions in here ...
}
```
<!-- more -->

(If your environment supports function prototypes, then it will be "int yylex( void  )".) This definition may be changed by defining the "YY_DECL" macro. For example, you could use:

```
#define YY_DECL int yylex(YYSTYPE *yylval)
```

Whenever `yylex()` is called, it scans tokens from the global input file `yyin` (which defaults to stdin). It continues until it either reaches an end-of-file (at which point it returns the value 0) or one of its actions executes a return statement.

When the scanner receives an end-of-file indication from YY_INPUT, it then checks the `yywrap()` function. If `yywrap()` returns false (zero), then it is assumed that the function has gone ahead and set up `yyin` to point to another input file, and scanning continues. If it returns true (non-zero), then the scanner terminates, returning 0 to its caller. Note that in either case, the start condition remains unchanged; it does not revert to INITIAL.

If you do not supply your own version of `yywrap()`, then you must either use `%option noyywrap` (in which case the scanner behaves as though `yywrap()` returned 1), or you must link with `-lfl` to obtain the default version of the routine, which always returns 1.)

### Values available to the user

* `char *yytext` holds the text of the current token. It may be modified but not lengthened (you cannot append characters to the end). If the special directive `%array` appears in the first section of the scanner description, then yytext is instead declared `char yytext[YYLMAX]`, where YYLMAX is a macro definition that you can redefine in the first section if you don't like the default value (generally 8KB). Using `%array` results in somewhat slower scanners, but the value of yytext becomes immune to calls to `input()` and `unput()`, which potentially destroy its value when yytext is a character pointer. The opposite of `%array` is `%pointer`, which is the default. You cannot use `%array` when generating C++ scanner classes (the `-+` flag).

* `int yyleng` holds the length of the current token.

* `FILE *yyin` is the file which by default flex reads from. It may be redefined but doing so only makes sense before scanning begins or after an EOF has been encountered. Changing it in the midst of scanning will have unexpected results since flex buffers its input; use `yyrestart()` instead. Once scanning terminates because an end-of-file has been seen, you can assign yyin at the new input file and then call the scanner again to continue scanning.

### Interfacing with YACC

One of the main uses of flex is as a companion to the yacc parser-generator. yacc parsers expect to call a routine named `yylex()` to find the next input token. The routine is supposed to return the type of the next token as well as putting any associated value in the global `yylval`.

Reference: [A fast scanner generator](http://dinosaur.compilertools.net/flex/index.html)

