# JFrog Artifactory

[JFrog 官网](https://jfrog.com/artifactory/)

## 基本介绍

Artifactory 是 JFrog 的一个产品，用作二进制存储库管理器。二进制存储库可以将所有这些二进制统一托管，从而使团队的管理更加高效和简单。  
类似Git 一样，Git 是用来管理代码的，Artifactory 是用来管理二进制文件的，通常是指 jar, war, pypi, DLL, EXE 等 build 文件。  
使用 Artifactory 的最大优势是创造了更好的持续集成环境，有助于其他持续集成任务去 Artifactory 里调用，再部署到不同的测试或开发环境，这对于实施 DevOps 至关重要。

## Artifactory仓库

Artifactory是一个通用管理仓库，支持所有主要的包格式的管理。不但可以管理二进制文件，也可以对市面上几乎所有的语言的包的依赖进行管理。

### 命名规范

官方推荐的规范：

    <team>-<technology>-<maturity>-<locator>

- team: 组织
- technology: 技术，如：java、golang
- maturity: 成熟度，一个仓库通常由四个级别的成熟度组成，从低到高这里分别是 dev, int（集成）, stage（预发布） 和 release（正式版）
- locator: 所在位置，cn、us等
