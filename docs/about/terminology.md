# 术语表

`ports` <-> `ports源码树`

:    Ports collections （又称 ports trees 或直接简称 ports）是一系列由 BSD 系列操作系统
    （比如 FreeBSD，NetBSD，和 OpenBSD）提供的一些 makefile 和 patch (Unix)，是一种简单的安装以及创建二进制包的方法。
    它们通常基于软件包管理系统，并带有 ports handling package 创建以及附加工具以对软件包删除、增添或进行其他操作。

`disk label` <-> `BSD 盘标`

：   Disk labels，disklabel（又称为 BSD 盘标）用于管理 OpenBSD 文件系统分区。
    它们包含有关磁盘的某些详细信息，例如 [disklabel(5)](https://man.openbsd.org/disklabel.5) 手册页中的详细描述的驱动器几何结构和文件系统信息。
    你可以使用 [disklabel(8)](https://man.openbsd.org/disklabel) 命令编辑标签。