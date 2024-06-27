# YAML （Yet Another Markup Language"）

## 数组

yaml配置文件中，数组的表示形式： 数组以 `-` 开头

    - A
    - B
    - C

一行内表示：

    key: [A, B, C]

多维数组：

    - 
     - A
     - B
     - C
     - D

对象数组：

    # 意思是 companies 属性是一个数组，每一个数组元素又是由 id、name、price 三个属性构成。
    companies:
        -
            id: 1
            name: company1
            price: 200W
        -
            id: 2
            name: company2
            price: 500W    

数组也可以使用流式(flow)的方式表示：

    companies: [{id: 1,name: company1,price: 200W}, {id: 2,name: company2,price: 500W}]

## 复合结构

数组和对象可以构成复合结构，例：

    languages:
      - Ruby
      - Perl
      - Python 
    websites:
      YAML: yaml.org 
      Ruby: ruby-lang.org 
      Python: python.org 
      Perl: use.perl.org

转换为 json 为：

    { 
        languages: [ 'Ruby', 'Perl', 'Python'],
        websites: {
            YAML: 'yaml.org',
            Ruby: 'ruby-lang.org',
            Python: 'python.org',
            Perl: 'use.perl.org' 
        } 
    }

## 参考

- [YAML](https://yaml.org/)

## FAQ

[FAQ](https://yaml.org/faq.html)

> Is there an official extension for YAML files?

    Please use ".yaml" when possible.

> Why does YAML forbid tabs?

    Tabs have been outlawed since they are treated differently by different editors and tools. And since indentation is so critical to proper interpretation of YAML, this issue is just too tricky to even attempt. Indeed Guido van Rossum of Python has acknowledged that allowing TABs in Python source is a headache for many people and that were he to design Python again, he would forbid them.
