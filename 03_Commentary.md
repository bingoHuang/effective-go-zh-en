## Commentary

## 注释

Go provides C-style `/* */` block comments and C++-style `//` line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

Go 语言支持 C 风格的块注释 `/* */` 和 C++ 风格的行注释 `//`。 行注释更为常用，而块注释则主要用作包的注释，当然也可在禁用一大段代码时使用。

The program—and web server—godoc processes Go source files to extract documentation about the contents of the package. Comments that appear before top-level declarations, with no intervening newlines, are extracted along with the declaration to serve as explanatory text for the item. The nature and style of these comments determines the quality of the documentation godoc produces.

godoc 既是一个程序，又是一个 Web 服务器，它对 Go 的源码进行处理，并提取包中的文档内容。 出现在顶级声明之前，且与该声明之间没有空行的注释，将与该声明一起被提取出来，作为该条目的说明文档。 这些注释的类型和风格决定了 godoc 生成的文档质量。

Every package should have a package comment, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole. It will appear first on the godoc page and should set up the detailed documentation that follows.

每个包都应包含一段包注释，即放置在包子句前的一个**块注释**。对于包含多个文件的包， 包注释只需出现在其中的任一文件中即可。包注释应在整体上对该包进行介绍，并提供包的相关信息。 它将出现在 godoc 页面中的最上面，并为紧随其后的内容建立详细的文档。

```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

	regexp:
		concatenation { '|' concatenation }
	concatenation:
		{ closure }
	closure:
		term [ '*' | '+' | '?' ]
	term:
		'^'
		'$'
		'.'
		character
		'[' [ '^' ] character-ranges ']'
		'(' regexp ')'
*/
package regexp
```
```go
/*
	regexp 包为正则表达式实现了一个简单的库。

	该库接受的正则表达式语法为：

	正则表达式:
		串联 { '|' 串联 }
	串联:
		{ 闭包 }
	闭包:
		条目 [ '*' | '+' | '?' ]
	条目:
		'^'
		'$'
		'.'
		字符
		'[' [ '^' ] 字符遍历 ']'
		'(' 正则表达式 ')'
*/
package regexp
```
If the package is simple, the package comment can be brief.

若某个包比较简单，包注释同样可以简洁些。

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```
```go
// path 包实现了一些常用的工具，以便于操作用正斜杠分隔的路径.
```
Comments do not need extra formatting such as banners of stars. The generated output may not even be presented in a fixed-width font, so don't depend on spacing for alignment—godoc, like gofmt, takes care of that. The comments are uninterpreted plain text, so HTML and other annotations such as `_this_` will reproduce _verbatim_ and should not be used. One adjustment godoc does do is to display indented text in a fixed-width font, suitable for program snippets. The package comment for the [fmt package](https://go-zh.org/pkg/fmt/) uses this to good effect.

注释无需进行额外的格式化，如用星号来突出等。生成的输出甚至可能无法以等宽字体显示， 因此不要依赖于空格对齐，godoc 会像 gofmt 那样处理好这一切。 注释是不会被解析的纯文本，因此像 HTML 或其它类似于 `_这样_` 的东西将按照 _原样_ 输出，因此不应使用它们。godoc 所做的调整， 就是将已缩进的文本以等宽字体显示，来适应对应的程序片段。 [fmt 包](https://go-zh.org/pkg/fmt/) 的注释就用了这种不错的效果。

Depending on the context, godoc might not even reformat comments, so make sure they look good straight up: use correct spelling, punctuation, and sentence structure, fold long lines, and so on.

godoc 是否会重新格式化注释取决于上下文，因此必须确保它们看起来清晰易辨： 使用正确的拼写、标点和语句结构以及折叠长行等。

Inside a package, any comment immediately preceding a top-level declaration serves as a doc comment for that declaration. Every exported (capitalized) name in a program should have a doc comment.

在包中，任何顶级声明前面的注释都将作为该声明的**文档注释**。 在程序中，每个可导出（首字母大写）的名称都应该有文档注释。

Doc comments work best as complete sentences, which allow a wide variety of automated presentations. The first sentence should be a one-sentence summary that starts with the name being declared.

文档注释最好是完整的句子，这样它才能适应各种自动化的展示。 第一句应当以被声明的东西开头，并且是单句的摘要。

```go
// Compile parses a regular expression and returns, if successful, a Regexp
// object that can be used to match against text.
func Compile(str string) (regexp *Regexp, err error) {
```
```go
// Compile 用于解析正则表达式并返回，如果成功，则 Regexp 对象就可用于匹配所针对的文本。
func Compile(str string) (regexp *Regexp, err error) {
```
If the name always begins the comment, the output of godoc can usefully be run through grep. Imagine you couldn't remember the name"Compile" but were looking for the parsing function for regular expressions, so you ran the command,

若注释总是以名称开头，godoc 的输出就能通过 grep 变得更加有用。假如你记不住 “Compile” 这个名称，而又在找正则表达式的解析函数， 那就可以运行

```go
$ godoc regexp | grep parse
```
If all the doc comments in the package began, "This function...", grep wouldn't help you remember the name. But because the package starts each doc comment with the name, you'd see something like this, which recalls the word you're looking for.

若包中的所有文档注释都以 “This function...” 开头，grep 就无法帮你记住此名称。 但由于每个包的文档注释都以其名称开头，你就能看到这样的内容，它能显示你正在寻找的词语。

```go
$ godoc regexp | grep parse
	Compile parses a regular expression and returns, if successful, a Regexp
	parsed. It simplifies safe initialization of global variables holding
	cannot be parsed. It simplifies safe initialization of global variables
$
```

Go's declaration syntax allows grouping of declarations. A single doc comment can introduce a group of related constants or variables. Since the whole declaration is presented, such a comment can often be perfunctory.

Go的声明语法允许成组声明。单个文档注释应介绍一组相关的常量或变量。 由于是整体声明，这种注释往往较为笼统。

```go
// Error codes returned by failures to parse an expression.
var (
	ErrInternal      = errors.New("regexp: internal error")
	ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
	ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
	...
)
```
```go
// 表达式解析失败后返回错误代码。
var (
	ErrInternal      = errors.New("regexp: internal error")
	ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
	ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
	...
)
```
Grouping can also indicate relationships between items, such as the fact that a set of variables is protected by a mutex.

即便是对于私有名称，也可通过成组声明来表明各项间的关系，例如某一组由互斥体保护的变量。

```go
var (
	countLock   sync.Mutex
	inputCount  uint32
	outputCount uint32
	errorCount  uint32
)
```
