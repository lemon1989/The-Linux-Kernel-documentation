# [Kconfig Language](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)

## Introduction
介绍

The configuration database is a collection of configuration options organized in a tree structure:\
配置数据库是组织成树结构形式配置选项的集合：

    +- Code maturity level options
    |  +- Prompt for development and/or incomplete code/drivers
    +- General setup
    |  +- Networking support
    |  +- System V IPC
    |  +- BSD Process Accounting
    |  +- Sysctl support
    +- Loadable module support
    |  +- Enable loadable module support
    |     +- Set version information on all module symbols
    |     +- Kernel module loader
    +- ...

Every entry has its own dependencies. These dependencies are used to determine the visibility of an entry. Any child entry is only visible if its parent entry is also visible.\
每个条目都有自己的依赖项。这些依赖项用于确定条目的可见性。任何子条目仅在其父条目也可见时才可见。

## Menu entries
菜单条目

Most entries define a config option; all other entries help to organize them. A single configuration option is defined like this:\
大多数条目定义一个配置选项;所有其他条目都有助于组织它们。单个配置选项的定义是：

    config MODVERSIONS
          bool "Set version information on all module symbols"
          depends on MODULES
          help
            Usually, modules have to be recompiled whenever you switch to a new
            kernel.  ...
            
Every line starts with a key word and can be followed by multiple arguments. “config” starts a new config entry. The following lines define attributes for this config option. Attributes can be the type of the config option, input prompt, dependencies, help text and default values. A config option can be defined multiple times with the same name, but every definition can have only a single input prompt and the type must not conflict.\
每行都以一个关键词开头，后面可以跟多个参数。"配置"启动新的配置条目。以下行定义此配置选项的属性。属性可以是配置选项的类型、输入提示、依赖项、帮助文本和默认值。配置选项可以使用相同的名称多次定义，但每个定义只能有一个输入提示，并且类型不能冲突。

## Menu attributes
菜单属性

A menu entry can have a number of attributes. Not all of them are applicable everywhere (see syntax).\
菜单项可以具有多个属性。并非所有属性都适用于所有地方（请参阅语法）。

- type definition: “bool”/”tristate”/”string”/”hex”/”int”\
类型定义："布尔"/"三态"/"字符串"/"十六进制"/"整型"

  Every config option must have a type. There are only two basic types: tristate and string; the other types are based on these two. The type definition optionally accepts an input prompt, so these two examples are equivalent:\
  每个配置选项都必须具有类型。只有两种基本类型：三态和字符串;其他类型基于这两种类型。类型定义可选泽地接受输入提示，因此这两个示例等效：

    bool "Networking support"
    
  and:

    bool
    prompt "Networking support"

- input prompt: “prompt” <prompt> [“if” <expr>]\
输入提示："prompt"<prompt>"if"<expr>]
    
  Every menu entry can have at most one prompt, which is used to display to the user. Optionally dependencies only for this prompt can be added with “if”.\
  每个菜单项最多只能有一个提示，用于向用户显示。添加"if"可选择地为此提示添加依赖项。

- default value: “default” <expr> [“if” <expr>]\
默认值："默认"<expr>"如果"<expr>]
    
  A config option can have any number of default values. If multiple default values are visible, only the first defined one is active. Default values are not   limited to the menu entry where they are defined. This means the default can be defined somewhere else or be overridden by an earlier definition. The default value is only assigned to the config symbol if no other value was set by the user (via the input prompt above). If an input prompt is visible the default value is presented to the user and can be overridden by him. Optionally, dependencies only for this default value can be added with “if”.\
  配置选项可以具有任何数量的默认值。如果多个默认值可见，则只有第一个定义的默认值处于活动状态。默认值不限于定义默认值的菜单项。这意味着默认值可以在其他位置定义，或者由较早的定义覆盖。如果用户未设置其他值（通过上面的输入提示），默认值才分配给配置符号。如果输入提示可见，则默认值将呈现给用户，并可由用户覆盖。可选地，只能为此默认值添加依赖项，并添加"if"。

  The default value deliberately defaults to ‘n’ in order to avoid bloating the build. With few exceptions, new config options should not change this. The intent is for “make oldconfig” to add as little as possible to the config from release to release.\
  默认值有意地默认为"n"，以避免膨胀编译。除了少数例外，新的配置选项不应更改此选项。目的是"make oldconfig"尽可能少地添加配置从发布到发布。

  Note:\
  注意：

    Things that merit “default y/m” include:\
    值得"默认 y/m"的内容包括：

    a.A new Kconfig option for something that used to always be built should be “default y”.\
    对于以前始终构建的东西，新的 Kconfig 选项应该是"默认 y"。

    b.A new gatekeeping Kconfig option that hides/shows other Kconfig options (but does not generate any code of its own), should be “default y” so people will  see those other options.\
    隐藏/显示其他 Kconfig 选项（但不生成其自身任何代码）的新守门 Kconfig 选项应为"默认 y"，以便用户将看到这些其他选项。
    
    c.Sub-driver behavior or similar options for a driver that is “default n”. This allows you to provide sane defaults.\
    对于"默认 n"的驱动程序，子驱动程序行为或类似选项。这允许您提供理智的默认值。

    d.Hardware or infrastructure that everybody expects, such as CONFIG_NET or CONFIG_BLOCK. These are rare exceptions.\
    每个人都期望的硬件或基础架构，例如CONFIG_NET或CONFIG_BLOCK。这些都是罕见的例外。

    


