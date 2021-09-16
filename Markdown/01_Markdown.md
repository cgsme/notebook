# Markdown

## 标题

## 段落格式

## 列表

## 区块

## 代码

## 链接

## 图片

语法格式：

    ![alt提示文本](图片链接地址)
    ![alt提示文本](图片链接地址 "可选标题title")

- 开头一个感叹号 !
- 接着一个方括号，里面放上图片的替代文字
- 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的 'title' 属性的文字。

使用示例：

    ![k8s组件](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

![微软Bing](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg 'k8s组件')

也可以像网址链接那样使用**变量**

    这个链接用 1 作为网址变量 [k8s组件][1].
    然后在文档的结尾为变量赋值（网址）

    [1]: https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg

## 表格

## 高级技巧
