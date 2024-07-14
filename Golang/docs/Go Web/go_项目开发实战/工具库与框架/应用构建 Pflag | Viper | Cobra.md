## 1. 构建应用框架

一个应用框架通常需要包括 3 个部分：

1. **命令行参数解析**
2. **配置文件解析**：大型应用具有很多参数，通常会将参数放在一个配置文件中方便管理，并供程序读取与解析
3. **应用的命令行框架**：应用最终需要通过命令启动，这里有 3 个需求点：
    - 解析命令行参数和配置文件
    - 初始化业务代码
    - 具备 Help 功能，告诉使用者如何使用

对应于 3 个部分，通常会选取业界现有的成熟实践（全部出自 Go 团队成员 spf13 之手）

1. 命令行参数解析工具：[Pflag](https://github.com/spf13/pflag)
2. 配置解析：[Viper](https://github.com/spf13/viper)
3. 命令行架构：[Cobra](https://github.com/spf13/cobra)



## 2. 命令行解析工具 Pflag

Pflag 提供了很多强大的特性，非常适合用来构建大型项目

- 支持不同类型的参数（string、int、ip 等）
- 支持不同的使用方式（`--` 长选项，`-` 短选项）

- 使用 Pflag 构建的知名开源项目：Kubernetes、Istio、Helm、Docker、Etcd 等





## 3. 配置解析 Viper

Viper 可以从不同的位置读取配置，高优先级的配置会覆盖低优先级相同的配置，优先级**从高到低**如下：

1. 通过 `viper.Set` 函数显式设置的配置
2. 命令行参数
3. 环境变量
4. 配置文件
5. Key/Value 存储
6. 默认值

**Viper 的配置键不区分大小写**



### 3.1 读入配置









## Cobra 简介

CLI：Command-line interface，命令行界面

[Cobra](https://github.com/spf13/cobra)：创建强大现代 CLI 应用程序的库，可以生成应用和命令文件

- 如 Kubernetes、Docker、etcd、Rkt、Hugo 等都是基于 Cobra 构建应用程序的
- Cobra 提供了两种方式来创建命令：Cobra 命令、Cobra 库，Cobra 实际上是通过引用 Cobra 库构建的命令模板

- [Cobra 用户指南](https://github.com/spf13/cobra/blob/main/site/content/user_guide.md)



安装 Cobra 生成器和 Cobra 库

```bash
$ go get github.com/spf13/cobra/cobra
```



**Cobra 建立在 commands、arguments 和 flags 结构上**

- commands（命令）、arguments（非选项参数/必须参数）、flags（选项参数/标志）

```bash
# 命令结构1
appname verb noun --adjective
# verb 动词，noun 名词，adjective 形容词

# 命令结构2
appname command arg --flag
# e.g. git clone URL --depth=1，克隆仓库最新提交记录
```



## 使用 Cobra 库创建命令

### 创建命令模板

```go
// hugo/cmd/root.go
var rootCmd = &cobra.Command{	// cobra.Command 是命令结构体
	Use:   "hugo",	// 命令名称
    // 简短和完整描述
	Short: "Hugo is a very fast static site generator",
	Long: `A Fast and Flexible Static Site Generator built with
                love by spf13 and friends in Go.
                Complete documentation is available at https://gohugo.io`,
  
  	// Run 是执行命令时会调用的函数
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Println("run hugo...")
  	},
}

// 命令的执行入口
// 默认内部会解析 os.Args[1:] 参数列表（可通过 Command.SetArgs 设置参数）
func Execute() {
  	if err := rootCmd.Execute(); err != nil {
    	fmt.Println(err)
    	os.Exit(1)
  	}
}
```





















## 参考

[应用构建三剑客：Pflag、Viper、Cobra 核心功能介绍](https://time.geekbang.org/column/article/395705)

[Go 命令行参数解析工具 pflag 使用](https://jianghushinian.cn/2023/03/27/use-of-go-command-line-parameter-parsing-tool-pflag/)

[万字长文——Go 语言现代命令行框架 Cobra 详解](https://xie.infoq.cn/article/915006cf3760c99ad0028d895)（”中文文档“）

