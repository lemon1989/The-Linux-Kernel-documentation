# [Kconfig Language](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)

## Introduction
介绍

The configuration database is a collection of configuration options organized in a tree structure:\
配置数据库是组织成树结构的配置选项的集合：

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



