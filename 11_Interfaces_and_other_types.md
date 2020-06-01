## Interfaces and other types

## 接口与其它类型

### 接口

Interfaces in Go provide a way to specify the behavior of an object: if something can do this, then it can be used here. We've seen a couple of simple examples already; custom printers can be implemented by a String method while Fprintf can generate output to anything with a Write method. Interfaces with only one or two methods are common in Go code, and are usually given a name derived from the method, such as io.Writer for something that implements Write.

Go 中的接口为指定对象的行为提供了一种方法：如果某样东西可以完成这个， 那么它就可以用在这里。我们已经见过许多简单的示例了；通过实现 String 方法，我们可以自定义打印函数，而通过 Write 方法，Fprintf 则能对任何对象产生输出。在 Go 代码中， 仅包含一两种方法的接口很常见，且其名称通常来自于实现它的方法， 如 io.Writer 就是实现了 Write 的一类对象。

A type can implement multiple interfaces. For instance, a collection can be sorted by the routines in package sort if it implements sort.Interface, which contains Len(), Less(i, j int) bool, and Swap(i, j int), and it could also have a custom formatter. In this contrived example Sequence satisfies both.

每种类型都能实现多个接口。例如一个实现了 sort.Interface 接口的集合就可通过 sort 包中的例程进行排序。该接口包括 Len()、Less(i, j int) bool 以及 Swap(i, j int)，另外，该集合仍然可以有一个自定义的格式化器。 以下特意构建的例子 Sequence 就同时满足这两种情况。

```go
type Sequence []int

// Methods required by sort.Interface.
// sort.Interface 所需的方法。
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Method for printing - sorts the elements before printing.
// 用于打印的方法 - 在打印前对元素进行排序。
func (s Sequence) String() string {
    sort.Sort(s)
    str := "["
    for i, elem := range s {
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```
### Conversions

### 类型转换

The String method of Sequence is recreating the work that Sprint already does for slices. We can share the effort if we convert the Sequence to a plain []int before calling Sprint.

Sequence 的 String 方法重新实现了 Sprint 为切片实现的功能。若我们在调用 Sprint 之前将 Sequence 转换为纯粹的 []int，就能共享已实现的功能。

```go
func (s Sequence) String() string {
	sort.Sort(s)
	return fmt.Sprint([]int(s))
}
```
This method is another example of the conversion technique for calling Sprintf safely from a String method. Because the two types (Sequence and []int) are the same if we ignore the type name, it's legal to convert between them. The conversion doesn't create a new value, it just temporarily acts as though the existing value has a new type. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

该方法是通过类型转换技术，在 String 方法中安全调用 Sprintf 的另个一例子。若我们忽略类型名的话，这两种类型（Sequence 和 []int）其实是相同的，因此在二者之间进行转换是合法的。 转换过程并不会创建新值，它只是暂时让现有的值看起来有个新类型而已。 （还有些合法转换则会创建新值，如从整数转换为浮点数等。）

It's an idiom in Go programs to convert the type of an expression to access a different set of methods. As an example, we could use the existing type sort.IntSlice to reduce the entire example to this:

在 Go 程序中，为访问不同的方法集而进行类型转换的情况非常常见。 例如，我们可使用现有的 sort.IntSlice 类型来简化整个示例：

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
	sort.IntSlice(s).Sort()
	return fmt.Sprint([]int(s))
}
```
```go
type Sequence []int

// // 用于打印的方法 - 在打印前对元素进行排序。
func (s Sequence) String() string {
	sort.IntSlice(s).Sort()
	return fmt.Sprint([]int(s))
}
```
Now, instead of having Sequence implement multiple interfaces (sorting and printing), we're using the ability of a data item to be converted to multiple types (Sequence, sort.IntSlice and []int), each of which does some part of the job. That's more unusual in practice but can be effective.

现在，不必让 Sequence 实现多个接口（排序和打印）， 我们可通过将数据条目转换为多种类型（Sequence、sort.IntSlice 和 []int）来使用相应的功能，每次转换都完成一部分工作。 这在实践中虽然有些不同寻常，但往往却很有效。

### Interface conversions and type assertions

### 接口转换与类型断言

[Type switches](https://go-zh.org/doc/effective_go.html#type_switch) are a form of conversion: they take an interface and, for each case in the switch, in a sense convert it to the type of that case. Here's a simplified version of how the code under fmt.Printf turns a value into a string using a type switch. If it's already a string, we want the actual string value held by the interface, while if it has a String method we want the result of calling the method.

[类型选择](https://go-zh.org/doc/effective_go.html#type_switch) 是类型转换的一种形式：它接受一个接口，在选择 （switch）中根据其判断选择对应的情况（case）， 并在某种意义上将其转换为该种类型。以下代码为 fmt.Printf 通过类型选择将值转换为字符串的简化版。若它已经为字符串，我们需要该接口中实际的字符串值； 若它有 String 方法，我们则需要调用该方法所得的结果。

```go
type Stringer interface {
	String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
	return str
case Stringer:
	return str.String()
}
```
```go
type Stringer interface {
	String() string
}

var value interface{} // 调用者提供的值。
switch str := value.(type) {
case string:
	return str
case Stringer:
	return str.String()
}
```
The first case finds a concrete value; the second converts the interface into another interface. It's perfectly fine to mix types this way.

第一种情况获取具体的值，第二种将该接口转换为另一个接口。这种方式对于混合类型来说非常完美。

What if there's only one type we care about? If we know the value holds a string and we just want to extract it? A one-case type switch would do, but so would a type assertion. A type assertion takes an interface value and extracts from it a value of the specified explicit type. The syntax borrows from the clause opening a type switch, but with an explicit type rather than the type keyword:

若我们只关心一种类型呢？若我们知道该值拥有一个 string 而想要提取它呢？ 只需一种情况的类型选择就行，但它需要类型断言。类型断言接受一个接口值， 并从中提取指定的明确类型的值。其语法借鉴自类型选择开头的子句，但它需要一个明确的类型， 而非 type 关键字：

```go
value.(typeName)
```
and the result is a new value with the static type typeName. That type must either be the concrete type held by the interface, or a second interface type that the value can be converted to. To extract the string we know is in the value, we could write:

而其结果则是拥有静态类型 typeName 的新值。该类型必须为该接口所拥有的具体类型， 或者该值可转换成的第二种接口类型。要提取我们知道在该值中的字符串，可以这样：

```go
str := value.(string)
```
But if it turns out that the value does not contain a string, the program will crash with a run-time error. To guard against that, use the "comma, ok" idiom to test, safely, whether the value is a string:

但若它所转换的值中不包含字符串，该程序就会以运行时错误崩溃。为避免这种情况， 需使用 “逗号, ok” 惯用法来测试它能安全地判断该值是否为字符串：

```go
str, ok := value.(string)
if ok {
	fmt.Printf("string value is: %q\n", str)
} else {
	fmt.Printf("value is not a string\n")
}
```
```go
str, ok := value.(string)
if ok {
	fmt.Printf("字符串值为 %q\n", str)
} else {
	fmt.Printf("该值非字符串\n")
}
```
If the type assertion fails, str will still exist and be of type string, but it will have the zero value, an empty string.

若类型断言失败，str 将继续存在且为字符串类型，但它将拥有零值，即空字符串。

As an illustration of the capability, here's an if-else statement that's equivalent to the type switch that opened this section.

作为对这种能力的说明，这里有个 if-else 语句，它等价于本节开头的类型选择。

```go
if str, ok := value.(string); ok {
	return str
} else if str, ok := value.(Stringer); ok {
	return str.String()
}
```
### 通用性

If a type exists only to implement an interface and has no exported methods beyond that interface, there is no need to export the type itself. Exporting just the interface makes it clear that it's the behavior that matters, not the implementation, and that other implementations with different properties can mirror the behavior of the original type. It also avoids the need to repeat the documentation on every instance of a common method.

若某种现有的类型仅实现了一个接口，且除此之外并无可导出的方法，则该类型本身就无需导出。 仅导出该接口能让我们更专注于其行为而非实现，其它属性不同的实现则能反映该原始类型的行为。 这也能够避免为每个通用接口的实例重复编写文档。

In such cases, the constructor should return an interface value rather than the implementing type. As an example, in the hash libraries both crc32.NewIEEE and adler32.New return the interface type hash.Hash32. Substituting the CRC-32 algorithm for Adler-32 in a Go program requires only changing the constructor call; the rest of the code is unaffected by the change of algorithm.

在这种情况下，构造函数应当返回一个接口值而非实现的类型。例如在 hash 库中，crc32.NewIEEE 和 adler32.New 都返回接口类型 hash.Hash32。要在 Go 程序中用 Adler-32 算法替代 CRC-32， 只需修改构造函数调用即可，其余代码则不受算法改变的影响。

A similar approach allows the streaming cipher algorithms in the various crypto packages to be separated from the block ciphers they chain together. The Block interface in the crypto/cipher package specifies the behavior of a block cipher, which provides encryption of a single block of data. Then, by analogy with the bufio package, cipher packages that implement this interface can be used to construct streaming ciphers, represented by the Stream interface, without knowing the details of the block encryption.

同样的方式能将 crypto 包中多种联系在一起的流密码算法与块密码算法分开。 crypto/cipher 包中的 Block 接口指定了块密码算法的行为， 它为单独的数据块提供加密。接着，和 bufio 包类似，任何实现了该接口的密码包都能被用于构造以 Stream 为接口表示的流密码，而无需知道块加密的细节。

The crypto/cipher interfaces look like this:

crypto/cipher 接口看其来就像这样：

```go
type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}

type Stream interface {
	XORKeyStream(dst, src []byte)
}
```
Here's the definition of the counter mode (CTR) stream, which turns a block cipher into a streaming cipher; notice that the block cipher's details are abstracted away:

这是计数器模式 CTR 流的定义，它将块加密改为流加密，注意块加密的细节已被抽象化了。

```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```
```go
// NewCTR 返回一个 Stream，其加密 / 解密使用计数器模式中给定的 Block 进行。
// iv 的长度必须与 Block 的块大小相同。
func NewCTR(block Block, iv []byte) Stream
```
NewCTR applies not just to one specific encryption algorithm and data source but to any implementation of the Block interface and any Stream. Because they return interface values, replacing CTR encryption with other encryption modes is a localized change. The constructor calls must be edited, but because the surrounding code must treat the result only as a Stream, it won't notice the difference.

NewCTR 的应用并不仅限于特定的加密算法和数据源，它适用于任何对 Block 接口和 Stream 的实现。因为它们返回接口值， 所以用其它加密模式来代替 CTR 只需做局部的更改。构造函数的调用过程必须被修改， 但由于其周围的代码只能将它看做 Stream，因此它们不会注意到其中的区别。

### Interfaces and methods

### 接口和方法

Since almost anything can have methods attached, almost anything can satisfy an interface. One illustrative example is in the http package, which defines the Handler interface. Any object that implements Handler can serve HTTP requests.

由于几乎任何类型都能添加方法，因此几乎任何类型都能满足一个接口。一个很直观的例子就是 http 包中定义的 Handler 接口。任何实现了 Handler 的对象都能够处理 HTTP 请求。

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```
ResponseWriter is itself an interface that provides access to the methods needed to return the response to the client. Those methods include the standard Write method, so an http.ResponseWriter can be used wherever an io.Writer can be used. Request is a struct containing a parsed representation of the request from the client.

ResponseWriter 接口提供了对方法的访问，这些方法需要响应客户端的请求。 由于这些方法包含了标准的 Write 方法，因此 http.ResponseWriter 可用于任何 io.Writer 适用的场景。Request 结构体包含已解析的客户端请求。

For brevity, let's ignore POSTs and assume HTTP requests are always GETs; that simplification does not affect the way the handlers are set up. Here's a trivial but complete implementation of a handler to count the number of times the page is visited.

为简单起见，我们假设所有的 HTTP 请求都是 GET 方法，而忽略 POST 方法， 这种简化不会影响处理程序的建立方式。这里有个短小却完整的处理程序实现， 它用于记录某个页面被访问的次数。

```go
// Simple counter server.
type Counter struct {
	n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	ctr.n++
	fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```
```go
// 简单的计数器服务。
type Counter struct {
	n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	ctr.n++
	fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```
(Keeping with our theme, note how Fprintf can print to an http.ResponseWriter.) For reference, here's how to attach such a server to a node on the URL tree.

（紧跟我们的主题，注意 Fprintf 如何能输出到 http.ResponseWriter。） 作为参考，这里演示了如何将这样一个服务器添加到 URL 树的一个节点上。

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```
But why make Counter a struct? An integer is all that's needed. (The receiver needs to be a pointer so the increment is visible to the caller.)

但为什么 Counter 要是结构体呢？一个整数就够了。（接收者必须为指针，增量操作对于调用者才可见。）

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	*ctr++
	fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```
```go
// 简单的计数器服务。
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	*ctr++
	fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```
What if your program has some internal state that needs to be notified that a page has been visited? Tie a channel to the web page.

当页面被访问时，怎样通知你的程序去更新一些内部状态呢？为 Web 页面绑定个信道吧。

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	ch <- req
	fmt.Fprint(w, "notification sent")
}
```
```go
// 每次浏览该信道都会发送一个提醒。
// （可能需要带缓冲的信道。）
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	ch <- req
	fmt.Fprint(w, "notification sent")
}
```
Finally, let's say we wanted to present on /args the arguments used when invoking the server binary. It's easy to write a function to print the arguments.

最后，假设我们需要输出调用服务器二进制程序时使用的实参 /args。 很简单，写个打印实参的函数就行了。

```go
func ArgServer() {
	fmt.Println(os.Args)
}
```
How do we turn that into an HTTP server? We could make ArgServer a method of some type whose value we ignore, but there's a cleaner way. Since we can define a method for any type except pointers and interfaces, we can write a method for a function. The http package contains this code:

我们如何将它转换为 HTTP 服务器呢？我们可以将 ArgServer 实现为某种可忽略值的方法，不过还有种更简单的方法。 既然我们可以为除指针和接口以外的任何类型定义方法，同样也能为一个函数写一个方法。 http 包包含以下代码：

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(c, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
	f(w, req)
}
```
```go
// HandlerFunc 类型是一个适配器，它允许将普通函数用做 HTTP 处理程序。
// 若 f 是个具有适当签名的函数，HandlerFunc(f) 就是个调用 f 的处理程序对象。
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(c, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
	f(w, req)
}
```
HandlerFunc is a type with a method, ServeHTTP, so values of that type can serve HTTP requests. Look at the implementation of the method: the receiver is a function, f, and the method calls f. That may seem odd but it's not that different from, say, the receiver being a channel and the method sending on the channel.

HandlerFunc 是个具有 ServeHTTP 方法的类型， 因此该类型的值就能处理 HTTP 请求。我们来看看该方法的实现：接收者是一个函数 f，而该方法调用 f。这看起来很奇怪，但是跟前面的例子没有区别：接收者是一个信道，（ServeHTTP）函数往该信道发消息。

To make ArgServer into an HTTP server, we first modify it to have the right signature.

为了将 ArgServer 实现成 HTTP 服务器，首先我们得让它拥有合适的签名。

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintln(w, os.Args)
}
```
```go
// 实参服务器。
func ArgServer(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintln(w, os.Args)
}
```
ArgServer now has same signature as HandlerFunc, so it can be converted to that type to access its methods, just as we converted Sequence to IntSlice to access IntSlice.Sort. The code to set it up is concise:

ArgServer 和 HandlerFunc 现在拥有了相同的签名， 因此我们可将其转换为这种类型以访问它的方法，就像我们将 Sequence 转换为 IntSlice 以访问 IntSlice.Sort 那样。 建立代码非常简单：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```
When someone visits the page /args, the handler installed at that page has value ArgServer and type HandlerFunc. The HTTP server will invoke the method ServeHTTP of that type, with ArgServer as the receiver, which will in turn call ArgServer (via the invocation f(c, req) inside HandlerFunc.ServeHTTP). The arguments will then be displayed.

当有人访问 /args 页面时，安装到该页面的处理程序就有了值 ArgServer 和类型 HandlerFunc。 HTTP 服务器会以 ArgServer 为接收者，调用该类型的 ServeHTTP 方法，它会反过来调用 ArgServer（通过 f(c, req)），接着实参就会被显示出来。

In this section we have made an HTTP server from a struct, an integer, a channel, and a function, all because interfaces are just sets of methods, which can be defined for (almost) any type.

在本节中，我们通过一个结构体，一个整数，一个信道和一个函数，建立了一个 HTTP 服务器， 这一切都是因为接口只是方法的集合，而几乎任何类型都能定义方法。
