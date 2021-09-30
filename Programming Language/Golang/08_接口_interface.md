# 接口（interface）

接口定义了一个对象的行为规范，只定义规范不实现，由具体的对象来实现行为细节。

## 接口

### 接口类型

在go语言中，接口（interface）是一种类型，一种抽象的类型。
接口是一组method的集合，是duck-type programming的一种体现。接口做的事情就像是定义一个协议（规则、规范），只要一台机器有洗衣服和甩干的功能，我就称它为洗衣机。不关心属性（数据），只关心行为（方法）。
记住：**interface是一种类型**，和其他数据类型的区别在于，接口是一种**抽象的类型**。

### 接口的定义

- 接口时一个或多个方法的集合
- 任何类型（结构体?）的方法集合中拥有了接口中所有的方法签名，则表示该类型实现了这个接口。这称为 **Structural Typing**。所谓的拥有接口所有方法签名是指具有相同方法名称、参数列表、以及返回值。当然，该类型还可以拥有其他方法。
- 接口只要方法声明，没有声明，没有数据字段。
- 接口可以匿名嵌入其他接口，或嵌入到结构中。
- 对象赋值给接口时，会发生拷贝，而接口内部存储的是指向这个复制品的指针，既无法修改复制品的状态，也无法获取指针。
- 只有当接口存储的类型和对象都为nil时，接口才等于nil。
- 接口调用不会做receiver的自动转换。
- 接口同样支持匿名字段方法。
- 接口也可以oop中的多态。
- 空接口可以作为任何类型数据的容器。
- 一个类型可以实现多个接口。
- 接口命名习惯以er结尾。

每个接口由数个方法组成，接口的定义格式如下：

    type 接口名称 interface {
        方法1(参数列表1) 返回值列表1
        方法2(参数列表2) 返回值列表2
        ...
    }

其中，

    1、接口名称：使用type将接口定义为自定义的类型名。Go语言的接口在命名时，习惯在单词后面加上er，如：有写操作的接口叫Writer，有操作字符串功能的接口叫Stringer等。接口命名最好能突出该接口的类型含义。
    2、方法名：当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问。
    3、参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。

举个列子：

    type Writer interface {
        Write([]byte) error
    }

### 实现接口的条件

一个对象只要实现了接口中的所有方法，那么就表示实现了这个接口。也就是说，接口就是一个需要实现的方法列表。

### 接口类型的变量

实现了接口有什么用呢？
接口类型的变量能够存储所有实现了该接口的实例。（我理解为：父类接口指向子类对象）

### 值接收者和指针接收者的区别

举例说明：

一个Mover接口、一个dog结构体。

    // 定义Mover接口
    type Mover interface {
        move()
    }

    // 定义dog接口体
    type dog struce {}

#### 值接收者实现接口

    // dog 值接收者
    func (d dog) move() {
        fmt.Println("dog can move!")
    }

此时实现接口的类型是dog类型：

    func main() {
        var x Mover
        var laifu = dog{}       // laifu是dog类型
        x = laifu               // x可以接收dog类型
        var fugui = &dog{}      // fugui是*dog类型（dog的指针类型）
        x = fugui               // x可以接收*dog类型
        x.move()
    }

可以发现，值接收者方式实现接口后，不管是`dog`结构体还是机构体指针`*dog`类型的变量都可以赋值给该接口变量`x`。因为go语言中有针对指针类型变量求值的语法糖，dog指针`fugui`内部会自动求值`*dog`。

#### 指针接收者实现接口

    func (d *dog) move() {
        fmt.Println("dog can move!")
    }

    func main() {
        var x Mover
        var laifu = dog{}   // laifu是dog类型
        x = laifu           // x不可接收dog类型
        var fugui = &dog{}  // fugui是*dog类型
        x = fugui           // x可以接受*dog类型
        x.move()
    }

此时Mover接口的实现者是`*dog`指针类型，所以laifu不能赋值给`x`，此时`x`只能存储`*dog`类型的值。

#### 多个类型实现一个接口

Go语言中一个类型可以实现多个接口，不同的类型也可以实现同一接口，并且一个接口的方法不一定需要由一个类型完全实现。接口的方法可以通过在类型中嵌入其他类型或结构体来实现。

#### 接口嵌套

接口与接口直接可以通过嵌套创造出新的接口。

    type Sayer interface {
        Say()
    }

    type Mover interface {
        Move()
    }

    // 嵌套接口
    type animal interface {
        Sayer
        Mover
    }

嵌套的接口与普通的接口使用方式一样。

## 空接口

### 空接口的定义

空接口是指没有定义任何方法的接口，因此任何类型都实现了空接口。
**空接口类型的变量可以存储任意类型的变量**

    func main() {
        // 空接口
        var x = interface{}
        s := "string"
        x = s
        i := 10
        x = i
        b := true
        x = b
    }

### 空接口的应用

- 空接口作为函数的参数

使用空接口作为参数的函数可以接收任意类型的参数。

    func enpty(any interface{}) {
        fmt.Println("type: %T", any)
    }

- 空接口作为map的值（value）

使用空接口实现可以保存任意值的字典。

    func main() {
        var info = make(map[string]interface{})
        info["name"] = "john"
        info["age"] = 30
        info["married"] = false
        fmt.Println(info)
    }

### 类型断言

#### 接口值

一个接口的值是由一个具体类和具体类型的值两部分组成。这两部分分别称为接口的**动态类型**和**动态值**。

想要判断空接口中的值，这个时候就可以使用类型断言，其语法格式为：

    x.(T)

其中：

    x：表示类型为interface{}的变量
    T：表示断言x是否可能为T类型

该语法返回两个参数，第一个为参数x转化为T类型后的变量。第二个值是布尔值，true表示断言成功，false表示断言失败。

举例：

    func main() {
        var x interface{}
        x = "xxxx"
        v, ok := x.(string)
        if of {
            fmt.Println(v)
        } else {
            fmt.Println("断言失败")
        }
    }

断言多次：

    func justify(x interface{}) {
        switch v := x.(type)
        case string:
            fmt.Printf("x is a string, value is %v\n")
        case int:
            fmt.Printf("x is a int, value is %v\n")
        case bool:
            fmt.Printf("x is a bool, value is %v\n")
        default:
            fmt.Println("断言失败！")
    }

由于空接口可以接收任意参数，所以在go中使用非常广泛。

## 最后

注意：只有当两个或者两个以上的具体类型必须以相同的方式进行处理时，才需要定义接口。不要为了接口而写接口，这样只会增加不必要的抽象，导致性能损耗。
