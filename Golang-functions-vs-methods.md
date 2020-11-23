# Golang functions vs methods

函数和方法在 Go 编程中被广泛使用，给使用者提供了一种抽象，并使我们的程序更容易阅读和判断。从表面上看，函数和方法看起来都很相似，但是也存在一些重要的语义差异，如果搞不清这其中的差异，这会对代码的可读性产生很大的影响。

下面一起来看看

#### **【函数的声明语法】**

通过指定参数、返回值和函数体的类型来声明函数:

```go
type Person struct {
	Name string
	Age  int
}

// This function returns a new instance of `Person`
func NewPerson(name string, age int) *Person {
	return &Person{
		Name: name,
		Age:  age,
	}
}
```

另一方面，一个方法是通过额外指定“接收者”(用面向对象的术语来说就是该方法所属的“类”)来声明的：

```go
// The `Person` pointer type is the receiver of the `isAdult` method
func (p *Person) isAdult() bool {
  return p.Age > 18
}
```

在上面的方法声明中，我们在 Person 类型上声明了 isAdult 方法。

#### 【执行语法 】

使用指定的参数独立调用函数，并根据其接收器的类型调用方法:

```go
p := NewPerson("John", 21)

fmt.Println(p.isAdult())
// true
```

#### 【转换】

函数和方法理论上可以互相转化。例如，我们可以将 isAdult 方法 转换成一个函数，并将 NewPerson 函数作为一个方法：

```go
type PersonFactory struct{}

// NewPerson 是 PersonFactory struct 的一个方法
func (p *PersonFactory) NewPerson(name string, age int) *Person {
	return &Person{
		Name: name,
		Age:  age,
	}
}

// 当我们以 Person 作为参数调用的时候 isAdult 就变成了一个方法
func isAdult(p *Person) bool {
	return p.Age > 18
}
```

执行结果看起来有点怪怪的。

```go
factory := &PersonFactory{}

p := factory.NewPerson("John", 21)

fmt.Println(isAdult(p))
// true
```

上面的代码看起来比实际需要的更加不必要和复杂。这向我们表明，方法和函数的区别主要是语法上的，您应该根据用例使用适当的抽象。

#### 用例 

让我们来看一下 go 应用程序中遇到的一些常见用例，以及为每个用例使用的适当抽象 ( 函数或方法 )：

**方法链**

方法的一个非常有用的特性是能够将它们链接在一起，同时保持代码的整洁。让我们举一个使用链接设置属性的例子:

```go
package main

import (
	"fmt"
)

type Person struct {
	Name string
	Age  int
}

func (p *Person) withName(name string) *Person {
	p.Name = name
	return p
}

func (p *Person) withAge(age int) *Person {
	p.Age = age
	return p
}

func main() {
	p := &Person{}
	p = p.withName("HRW").withAge(25)

	fmt.Println(*p)
  //print: {HRW 25}
}
```

如果我们用函数做同样的事情，那看起来会很可怕：

```go
p = withName(withAge(p, 18), "John")
```

#### 有状态与无状态执行 

在可交换性的例子中，我们看到了使用 PersonFactory 对象来创建一个新的person实例。事实证明，这是一种反模式，应该避免。 对于无状态执行，使用像前面声明的 NewPerson 函数这样的函数要好得多。 这里的“无状态”指的是对于相同的输入总是返回相同输出的任何一段代码 这里的推论是，如果你发现一个函数读取并修改了一个特定类型的很多值，那么它应该是一个该类型的方法。

#### 语义学 

语义指的是代码如何阅读。如果用口语大声朗读代码，什么更有意义？ 让我们看看 isAdult 的函数和方法实现

```go
customer := NewPerson("John", 21)

// Method
customer.isAdult()

// Function
isAdult(customer)
```

这里的 customer.isadult()  读起来更适合问 "顾客是成年人吗？

#### 结论 

虽然我们已经讨论了 Go 中函数和方法的一些关键区别和用例，但是总有例外！重要的是不要把这些规则当成一成不变的。 最后，函数和方法的区别在于结果代码的读取方式。如果你或你的团队觉得一种方法比另一种读起来更好，那么这就是正确的抽象！



Ref ： https://www.sohamkamani.com/golang/functions-vs-methods/