#Swift进阶-函数
要了解swift的函数和闭包，首先要了解三件事
1. 函数可以像String 和 Int 那样可以赋值给变量，也可以作为另一个函数的参数和返回值。（函数是头等对象，这一点最重要）
2. 函数能捕获存在于其作用域之外的变量。
3. 有两种方式可以创建函数。一种是使用func关键字，另一种是 {}。在swift中后一种被称为闭包。

```
var str = "Hello, playground"
/// MARK: ---------------------变量
func printInt(i: Int) {
    print("you passed \(i)")
}

// 函数赋值 不加括号
let funVar = printInt

funVar(1)

// 一个接受函数作为参数的函数

func useFunction(function: (Int) -> Void) {
    function(2)
}

useFunction(function: printInt)

useFunction(function: funVar)

// 也可以返回一个 函数

func returnFunc() -> (Int) -> String {
    func innerFunc(i: Int) -> String {
        return "you passed \(i)"
    }
    return innerFunc
}

let myFunc = returnFunc()
myFunc(3)

/// MARK: ---------------------值捕获
// 当函数应用了函数作用域外部的变量时， 这个变量就被捕获了，他将在超出函数作用域后继续存在，而不是结束。

func counterFunc() -> (Int) -> String {
    var counter = 0
    func innerFunc(i: Int) -> String {
        counter += i
        return "running total: \(counter)"
    }
    return innerFunc
}
// counter 将会保存在堆上
let f = counterFunc()
f(3)
f(4)
// 会生成新的counter 不会影响f
let g = counterFunc()

g(2)
g(3)
// 在 编程中 一个函数和它捕获的环境变量组合起来称之为闭包

/// MARK: ---------------------闭包
// 函数可以用{}来声明闭包表达式 使用闭包来定义函数可以称之为函数的字面量 类似于 1 就是整数的字面量， “hello” 是字符串的字面量。
// 闭包与函数区别在于 它没有名字 只能直接赋值给一个变量
// 但是它和函数完全等价 甚至拥有同一个 命名空间
func doubler(i: Int) -> Int {
    return i*2
}

let doubilerAlt = { (i: Int) -> Int in return i*2 }

[1, 2, 3, 4].map(doubilerAlt)

// 闭包可以简写

[1, 2, 3, 4].map{ $0*2 }

// 闭包可以不声明变量类型 也可以事先指定 闭包使用一般已经有上下文所以第二种写法不重要


let isEven = {$0 % 2 == 0} ///会默认推定为 （Int） -> Bool

let isEvenAlt = {(i: Int) -> Bool in
    return i % 2 == 0
}

/// 总结：闭包是函数和捕获变量的组合，{}创建的函数是闭包表达试 我们称之为闭包但其实它与函数并无区别，它们都可以是函数也大都可以是闭包
```


##函数的灵活性



