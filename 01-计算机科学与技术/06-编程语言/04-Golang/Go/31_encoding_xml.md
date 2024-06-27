# Xml

> [官方Demo](https://github.com/golang/go/blob/master/src/encoding/xml/example_test.go)

## 示例

### 序列号输出格式添加缩进（包含子元素）

#### 1. 通过 `xml.MarshalIndent(v, "  ", "    ")`

```go
func ExampleMarshalIndent() {
    type Address struct {
        City, State string
    }
    type Person struct {
        XMLName   xml.Name `xml:"person"`
        Id        int      `xml:"id,attr"`              // 当前父元素的 id 属性
        FirstName string   `xml:"name>first"`           // 子元素
        LastName  string   `xml:"name>last"`            // 子元素
        Age       int      `xml:"age"`
        Height    float32  `xml:"height,omitempty"`     // 忽略空值
        Married   bool
        Address
        Comment string `xml:",comment"`                 // 注释 <!-- -->
    }

    v := &Person{Id: 13, FirstName: "John", LastName: "Doe", Age: 42}
    // 添加注释内容
    v.Comment = " Need more details. "
    v.Address = Address{"Hanga Roa", "Easter Island"}

    output, err := xml.MarshalIndent(v, "  ", "    ")   // 输出时添加缩进，美化格式
    if err != nil {
        fmt.Printf("error: %v\n", err)
    }

    os.Stdout.Write(output)
    // Output:
    //   <person id="13">
    //       <name>
    //           <first>John</first>
    //           <last>Doe</last>
    //       </name>
    //       <age>42</age>
    //       <Married>false</Married>
    //       <City>Hanga Roa</City>
    //       <State>Easter Island</State>
    //       <!-- Need more details. -->
    //   </person>
}
```

#### 2. 通过 `Encoder`

```go
func ExampleEncoder() {
    type Address struct {
        City, State string
    }
    type Person struct {
        XMLName   xml.Name `xml:"person"`
        Id        int      `xml:"id,attr"`
        FirstName string   `xml:"name>first"`
        LastName  string   `xml:"name>last"`
        Age       int      `xml:"age"`
        Height    float32  `xml:"height,omitempty"`
        Married   bool
        Address
        Comment string `xml:",comment"`
    }

    v := &Person{Id: 13, FirstName: "John", LastName: "Doe", Age: 42}
    v.Comment = " Need more details. "
    v.Address = Address{"Hanga Roa", "Easter Island"}

    enc := xml.NewEncoder(os.Stdout)
    enc.Indent("  ", "    ")
    if err := enc.Encode(v); err != nil {
        fmt.Printf("error: %v\n", err)
    }

    // Output:
    //   <person id="13">
    //       <name>
    //           <first>John</first>
    //           <last>Doe</last>
    //       </name>
    //       <age>42</age>
    //       <Married>false</Married>
    //       <City>Hanga Roa</City>
    //       <State>Easter Island</State>
    //       <!-- Need more details. -->
    //   </person>
}
```

### 反序列化

注意：  
Phone字段没有修改过，xml字符串中不包含Phone相关内容，打印为 'none'。  
xml字符串中的 'Company' 被忽略了，结构体中没有定义对应的字段接收它。

```go
func ExampleUnmarshal() {
    type Email struct {
        Where string `xml:"where,attr"`
        Addr  string
    }
    type Address struct {
        City, State string
    }
    type Result struct {
        XMLName xml.Name `xml:"Person"`
        Name    string   `xml:"FullName"`
        Phone   string
        Email   []Email
        Groups  []string `xml:"Group>Value"`
        Address
    }
    v := Result{Name: "none", Phone: "none"}

    data := `
        <Person>
            <FullName>Grace R. Emlin</FullName>
            <Company>Example Inc.</Company>
            <Email where="home">
                <Addr>gre@example.com</Addr>
            </Email>
            <Email where='work'>
                <Addr>gre@work.com</Addr>
            </Email>
            <Group>
                <Value>Friends</Value>
                <Value>Squash</Value>
            </Group>
            <City>Hanga Roa</City>
            <State>Easter Island</State>
        </Person>
    `
    err := xml.Unmarshal([]byte(data), &v)
    if err != nil {
        fmt.Printf("error: %v", err)
        return
    }
    fmt.Printf("XMLName: %#v\n", v.XMLName)
    fmt.Printf("Name: %q\n", v.Name)
    fmt.Printf("Phone: %q\n", v.Phone)
    fmt.Printf("Email: %v\n", v.Email)
    fmt.Printf("Groups: %v\n", v.Groups)
    fmt.Printf("Address: %v\n", v.Address)
    // Output:
    // XMLName: xml.Name{Space:"", Local:"Person"}
    // Name: "Grace R. Emlin"
    // Phone: "none"
    // Email: [{home gre@example.com} {work gre@work.com}]
    // Groups: [Friends Squash]
    // Address: {Hanga Roa Easter Island}
}
```
