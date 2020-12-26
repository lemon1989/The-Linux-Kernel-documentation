# [Kconfig Language](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)

## Introduction
介绍

The configuration database is a collection of configuration options organized in a tree structure:\
配置数据库是组织成树结构形式配置项的集合：

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



