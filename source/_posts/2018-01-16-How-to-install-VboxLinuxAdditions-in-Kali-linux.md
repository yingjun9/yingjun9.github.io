---
title: How to install VboxLinuxAdditions in Kali linux
date: 2018-01-16 15:29:01
tags: Kali Vbox
---

在Kali中直接安装Vbox的增强工具是会报错的。这里记录下我当前的解决方案


首先，我们在Vbox中挂载增强工具。然后，打开它的目录，将`VBoxLinuxAdditions.run`拷贝到一个地方，例如`download`下。
第二部，赋予755权限。
```bash
chmod 755 VBoxLinuxAdditions.run
```

然后执行这个文件。
```bash
./VBoxLinuxAdditions.run
```
就能安装成功了。

