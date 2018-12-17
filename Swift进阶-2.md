#Swift进阶-2

## 集合类型协议
	在之前，我们看到了 Array Dictionary 和 Set，它们并非空中楼阁，而是建立在一系列由 Swift 标准库提供的用于处理元素序列的抽象之上的。
  	Sequence 和 collection协议，它们构成了这套集合类型模型的基石。本章会研究这些协议是如何工作的，它们为什么要以这样的方式工作，以及如何写出自己的序列和集合类型等
  	
1. 序列
Sequence协议是集合序列的基础，代表的是一系列具有相同类型的值，你可以对这些值进行迭代。遍历一个序列最简单的方式是使用for循环 
 
```
for element in someSequence {
doSomething（with: element)
}
```

 Sequence协议提供了许多强大的功能，满足该协议的类型都可以直接使用这些功能。上面这样步进式地迭代元素的能力看起来十分简单，但它却是 Sequence可以提供这些强大功能的基础。在上一章中已经提到过不少这类功能了，每当你遇到一个能够针对元素序列进行的通用的操作，你都应该考虑将它实现在 Sequence层可能性。在接下来的部分，会看到许多这方面的例子。
满足 Sequence 协议非常简单 只需要提供一个返回迭代器`iterator`的 `makeIterator（）`方法:

```
 protocol Sequence {
		associatedtype Iterator: IteratorProtocol
 	  func makeIterator（）->Iterator
 }
 
```
对于迭代器，我们现在只能从 Sequence 的定于中看出他满足 `IteratorProtocol`协议的类型。所以首先来仔细看看迭代器是什么。

2. 迭代器
序列通过一个迭代器来访问元素 `IteratorProtocol` 协议中有一个`next()`方法返回序列中下一个元素的值直到序列耗尽返回 `nil`

```
protocol IteratorProtocol {
		associatedtype Element
		mutating func next() -> Element?
}
本质上for循环就是下面代码的简写 当然我们也可创造无限序列
var iterator = someSequence.makeIterator()
while let element = iterator.next() {
	doSomething(with: element)
}
```

next 被标记mutating 看起来是不需要的 但实践中迭代器本质是存在状态的

```
struct FibsIterator: IteratorProtocol {
		typealias Element = Int
    var state = (0, 1)
    mutating func next() -> Int? {
        let upcomingNumber = state.0
        state = (state.1, state.0 + state.1)
        return upcomingNumber
    }
}

```

基于函数的迭代器序列

```
func uniqueIntegerProvider() -> AnyIterator<Int> {
    var i = 0
    return AnyIterator {
        i += 1
        return i
    }
}

let prodiver = AnySequence(uniqueIntegerProvider)
Array(prodiver.prefix(10))
```

无限序列， sequence 的 next 闭包总是延时执行的 也就是说 下一个next 不会被自动计算  prodiver.prefix(10) 只会求前10个
如果序列主动计算 它就会溢出崩溃
对集合和序列来说 区别之一 就是 序列可以无限 而集合不行

不稳定序列 网络流 UI时间流 磁盘文件 都可以用序列建模
网络包这种序列将被遍历消耗再次遍历不能保证是同样的值，
sequence 明确指出了不保证可以被多次遍历

2. 集合类型 Collection 

是指那些稳定的序列 能够被多次遍历并保存一致，可以下标访问 有起始和终止索引。

Collection协议是基于Sequence来的除了继承所有方法外，它还可以获取指定位置的元素 获取 稳定的迭代的保证，count等新的特性

3. （1）实现一个队列 （2）遵守ExpressibleByArrayLiteral 协议 （3）关联类型

```
// 实现一个队列
protocol Queue {
    associatedtype Element // self持有的类型
    mutating func enquenue(_ element: Element) // 入队
    mutating func dequenue() -> Element? // 出队
}


struct FIFOQuenue<Element>: Queue {
    mutating func enquenue(_ element: Element) { // 入队
        right.append(element)
    }


    mutating func dequenue() -> Element? { // 出队
        if left.isEmpty {
            left = right.reversed()
            right.removeAll()
        }
        return left.popLast()
    }

    fileprivate var left: [Element] = []
    fileprivate var right: [Element] = []

}

// 实现了这些 就能满足Collection协议了
extension FIFOQuenue: Collection {
    func index(after i: Int) -> Int {
        precondition(i < endIndex)
        return i + 1
    }

    subscript(position: Int) -> Element {
        precondition((0..<endIndex).contains(position), "Index out of bounds")
        if position <  left.endIndex {
            return left[left.count - position - 1]
        } else {
            return right[position - left.count]
        }
    }

    // 我们要实现的有
    public var startIndex: Int { return 0 }
    public var endIndex: Int { return left.count + right.count }

}


var q = FIFOQuenue<String>()

for x in ["1", "2", "f00", "3"] {
    q.enquenue(x)
}

for s in q {
    print(s)
}

q.count

// 扩展 新的协议

extension FIFOQuenue: ExpressibleByArrayLiteral {
    init(arrayLiteral elements: Element...) {
        self.init(left: elements.reversed(), right: [])
    }
    typealias ArrayLiteralElement = Element
}

var qq = [1,2,3]

for ss in qq {
    print(ss)
}
```

4. 索引
startIndex 是第一个元素位置
endIndex 是最后一个元素之后的位置

通过索引访问 subscript 是Collection定义的 它总是返回非可选值
索引失效，可能是指向了另一个元素 或者本身失效了 访问可能导致崩溃

索引步进  index(after:) 

实现一个单向链表 （非整数索引）



## 结构体和类
* swift 中储存结构化的数据 可以用 结构体，枚举，类及使用闭包捕获变量。
* 类和结构体不同点： 1.结构体是值类型类是引用类型，编译器可以保证结构体的不变性，类要我们自己来保证 2.内存管理方式不同接头体可以直接持有及访问，类的实例只能通过地址间接访问，结构体是唯一的，类能有多个持有者。3.类可以代码共享，结构体不能被继承

1. 值类型

* 我们经常处理一些有生命周期的类型 `ViewController` 有init，delloc等等，而有的则不需要生命周期例如 `URL` 它一旦创建就不会被改变，它们结束时不需要额外操作。比较两个`URL`的值时 我们不关心是否指向同一地址，而是它们的属性是否一样。这种类型我们称之为值类型。`NSURL` 是一个不可变对象 而`URL` 是一个结构体。
* 值类型具有天然的线程安全性。不可变的东西是可以多线程共享的。、
* 结构体中也能定义 var 类型 但这个可变性只体现在变量自己身上。当我们改变结构体中的属性时 它总是生成一个全新的结构体来取代
* 结构体只能有一个持有者，当我们把它传递给一个函数是它被复制了一份 函数只能改变这个副本，这被叫做 值语义。而对象传递的是地址指针，称为引用语义。
* 结构体只有一个持有者所以不会造成循环引用，除非结构体包含类属性，否则就不需要考虑引用计数问题，let声明的结构体 一个字节也不会改变

2. 可变性

* 可变性是造成bug主要原因之一，而swift可以让我们写出安全代码的同时，保留可变代码的风格

```
var mutabelArray: NSMutableArray = [1,2,3]
for _ in mutabelArray {
    mutabelArray.removeLastObject()
}
```

* 我们知道数组的变化会破坏迭代器的内部结构 这个是不被允许的，我们知道这一点，不会犯这种错误。 然而我们不能保证 removeLastObject 不被其他地方调用。这种错误很难被发现
swift 就避免了这种错误

```
var mutabelArray = [1,2,3]
for _ in mutabelArray {
    mutabelArray.removeLast()
}
```

* 赋值问题 引用类型会改变赋值它的变量的属性 这是很强大的特性，但有时也会造成bug

```
var mutabelArray: NSMutableArray = [1,2,3]
var new = mutabelArray
new.add(4)
```

我们实现一个扫描器

```
class BinaryScanner {
    var position: Int
    let data: Data
    init(data: Data) {
        self.position = 0
        self.data = data
    }
}

extension BinaryScanner {
    func scanByte() -> UInt8? {
        guard position < data.endIndex else {
            return nil
        }
        position += 1
        return data[position - 1]
    }
}

正常可运行
func scanRemainingBytes(scanner: BinaryScanner) {
    while let byte = scanner.scanByte() {
        print(byte)
    }
}

有可能偶发线程不安全
for _ in 0..<Int.max {
    let newScanner = BinaryScanner(data: "hi".data(using: .utf8)!)
    DispatchQueue.global().async {
        scanRemainingBytes(scanner: newScanner)
    }
    scanRemainingBytes(scanner: newScanner)
}

```

3. 结构体

* 几乎在所有编程中 标量都是值类型
* 当我们把一个结构体赋值给另一个时，swift 自动对它进行赋值，听起来很昂贵， 但编译器会对这种赋值进行优化，称为写时复制。

```
struct Point {
    var x: Int
    var y: Int
}

let origin = Point(x: 0, y:0)
origin.x = 10 // ❎

var otherPoint = Point(x: 0, y:0)
otherPoint.x = 10 // ✅

当我们把一个结构体赋值给另一个时
var otherPoint = origin
otherPoint.x = 10 // ✅
otherPoint // x: 10, y: 0
origin // x: 0, y: 0

struct Size {
    var width: Int
    var height: Int
}

静态变量
extension Point {
    static let origin = Point(x: 0, y: 0)
}

struct Rectangle {
    var origin: Point
    var size: Size
}

写在扩展中的初始化方法，系统会保留原始的初始化方法。
extension Rectangle {
    init(x: Int = 0, y: Int = 0, width: Int, height: Int) {
        origin = Point(x: x, y: y)
        size = Size(width: width, height: height)
    }
}


var screen = Rectangle(width: 320, height: 480) {
    didSet {
        print("screen did Changed \(screen)")
    }
}

screen.origin.x = 10
我们只是改变了结构体的数量 但是它的didset会被触发

var array = [screen] {
    didSet {
        print("array did Changed")
    }
}

array[0].origin.x = 10
数组是结构体 数组内元素的变化 数组本身 也会触发 didset
如果Rectangle是类 那么 didset就不会被触发


func + (lhs: Point, rhs: Point) -> Point {
    return Point(x: lhs.x + rhs.x, y: lhs.y + rhs.y)
}

extension Rectangle {
    func translate(by offset: Point) {
        origin = origin + offset 
    // 这个方法系统会报错 self是不可变得 需要声明mutating
    }
}

extension Rectangle {
    mutating func translate(by offset: Point) {
        origin = origin + offset
    }
}

标记了mutating 意味着 self是可变的 表现的想一个 var
当然 let 声明的结构体 依然是不可变得

很多情况下 方法都有可变和不可变两种版本 ，sort() 原地排序 sorted() 返回新的数组

extension Rectangle {
    func translated(by offset: Point) {
    		var copy = self
				copy.translate(offset: offset)        
		    return copy
    }
}

事实上 mutating 方法只是结构体上 普通的方法 只是 self 被隐士的标记为 inout 了 &
```

我们再来看上面的scanner 问题 如果 BinaryScanner 是 结构体那么 每个方法调用的结构体都是一个 副本这样 就可以安全的迭代了而不用担心被其他线程更改。

4. 写时复制 copy-on-write

var x = [1,2,3]
var y = x
如果创建一个y 把并把x赋值它时 会发生复制
x 和 y 含有独立的结构体。
但是 数组内含有指向 元素位置的指针 在这时 x 和 y 共享了他们的部分储存 不过 当x改变时 这种共享 会被检测到，内存将被复制。
只有在 有一个 发生改变时 内存才被复制 成为写时复制

* 昂贵方式 

```
struct Mydata {
    fileprivate var _data: NSMutableData
    var _dataFOrWriting: NSMutableData {
        mutating get {
            _data = _data.mutableCopy() as! NSMutableData
            return _data
        }
    }

    init(data: NSData) {
        self._data = data.mutableCopy() as! NSMutableData
    }
}

```

* 高效方式

在 swift 中 可以用 `isKnownUniquelyReferenced(&<#T##object: T##T#>)`
来检查引用的唯一性。
获取时 可以通过 是否被唯一引用来决定是否 复制

得益于写时复制和相关联的编译器优化 大量不必要的操作被移除了。
如果我们写一个 结构体 而不能保证其中属性的不变性，我们可以考虑使用类来实现。


* 写时复制陷阱

我们使用array[0] 时 直接用下标访问 是不会发生写时复制的 因为它直接访问了内存中元素的位置。而字典和set 不同

5. 闭包和可变性
一个函数 每次生成一个唯一的整数直到Int.max可以将状态移动到函数外部，换句话说函数对i进行了闭合

```
var i = 0

func uniqueInteger() -> Int {
    i += 1
    return i
}
```

swift 函数也是引用类型 

let otherFunction = uniqueInteger
传递函数它会已引用方式存在 并共享了 其中的状态

```
func uniqueIntegerProvider() -> () -> Int {
    var i = 0
    return {
        i += 1
        return i
    }
}
返回一个从零开始的方法
也可以封装成 AnyIterator 一个整数发生器
func uniqueIntegerProvider() -> AnyIterator<Int> {
    var i = 0
    return AnyIterator {
        i += 1
        return i
    }
}
```

* 结构体 一般被储存在栈上，而非堆上这其实是一种优化，默认结构体是在堆上的，但大多数情况下 优化都会生效
* 当一个结构体被函数闭合了 它就在堆上就算函数退出了作用域，它仍然存在，同样结构体过大 也会放在堆上

6. 内存 swift 的强引用和循环引用类似于OC weak(必然是可选值) unowned  [weak self]












 




























