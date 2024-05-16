# Effective Go

[Effective Go 中文](https://learnku.com/docs/effective-go/2020/embedded/6248)

**不重复造了!!!**

## 嵌套（Embedding）

Go没有提供典型的、类型驱动的子类概念，但是可以通过在结构体和接口中嵌套类型来完成继承。

接口的嵌套很简单。比如`io.Reader`和`io.writer`接口的定义：

    type Reader interface {
        Read(p []byte) (n int, err error)
    }

    type Writer interface {
        Write(p []byte) (n int, err error)
    }

`io`包中还提供了很多其他的指定了可以实现多个这样的方法的对象的接口。比如`io.ReadWriter`是一个包含`Reader`和`Writer`的接口。可以通过显式地列出这两个方法来指定`io.ReadWriter`，但是更简单的方式是通过在新的接口中嵌套这两个接口：

    // ReadWriter is the interface that combines the Reader and Writer interfaces.
    type ReadWriter interface {
        Reader
        Writer
    }

显而易见，一个`ReadWriter`既可以做`Reader`的事情也可以做`Wirter`的事情。是两个被嵌套的接口的并集。  
注意：**接口中只能嵌套接口**。

类似的，结构体中也有相同的方式，单更具有深远的影响。`bufio`包中有两个结构体类型，`bufio.Reader`和`bufio.Writer`，两个都实现了和`io`包相似的接口。并且`bufio`还实现了一个缓冲的`Reader/Writer`，它通过嵌入`Reader`和`Writer`到一个结构中来实现：它列出了结构中的类型**但不给它们字段名称**。

    // ReadWriter stores pointers to a Reader and a Writer.
    // It implements io.ReadWriter.
    type ReadWriter struct {
        *Reader  // *bufio.Reader
        *Writer  // *bufio.Writer
    }
