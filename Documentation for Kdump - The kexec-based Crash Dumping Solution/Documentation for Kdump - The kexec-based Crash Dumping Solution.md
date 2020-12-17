# Documentation for Kdump - The kexec-based Crash Dumping Solution
Kdump 的文档 - 一种基于 kexec 的崩溃转储解决方案

This document includes overview, setup and installation, and analysis information.\
本文档包括概述、设置和安装以及分析信息共四部分。

## Overview
概述

Kdump uses kexec to quickly boot to a dump-capture kernel whenever a dump of the system kernel’s memory needs to be taken (for example, when the system panics). The system kernel’s memory image is preserved across the reboot and is accessible to the dump-capture kernel.
每当需要转储系统内核的内存时(例如，当系统出现错误时)，Kdump 可以使用 kexec 快速启动到转储捕获内核。系统内核的内存映像在重新启动期间被保留，并且可由转储捕获内核访问。
