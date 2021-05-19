[TOC]

# 简介

Git LFS（Large File Storage, 大文件存储）是可以把音乐、图片、视频等指定的任意文件存在 Git 仓库之外，而在 Git 仓库中用一个占用空间 1KB 不到的文本指针来代替的小工具。通过把大文件存储在 Git 仓库之外，可以减小 Git 仓库本身的体积，使克隆 Git 仓库的速度加快，也使得 Git 不会因为仓库中充满大文件而损失性能。

使用 Git LFS，在默认情况下，只有当前签出的 commit 下的 LFS 对象的当前版本会被下载。此外，我们也可以做配置，只取由 Git LFS 管理的某些特定文件的实际内容，而对于其他由 Git LFS 管理的文件则只保留文件指针，从而节省带宽，加快克隆仓库的速度；也可以配置一次获取大文件的最近版本，从而能方便地检查大文件的近期变动。

# 下载与安装

## 下载

### Windows / Cygwin

https://git-lfs.github.com/

[(Cygwin 下使用 Git LFS 请参考后文相关注意事项）](https://zzz.buzz/zh/2016/04/19/the-guide-to-git-lfs/#tips-for-cygwin)

### macOS

```
brew install git-lfs
```

### Arch Linux

```
sudo pacman -S --noconfirm git-lfs
```

### ==Debian >= 9 / Ubuntu >= 18.04==

```
sudo apt-get update
sudo apt-get install git-lfs
```

### [其他系统或手动下载安装](https://github.com/github/git-lfs/releases)

## 安装

### Windows / Cygwin

根据安装向导安装下载的 .exe 包后，启动一个新的命令行窗口（以使包含 `git-lfs` 命令的目录能被加载进 PATH，并执行以下命令：

```
git lfs install
```

### macOS / Arch Linux / Debian / Ubuntu

执行以下命令即可：

```
git lfs install
```

### 其他系统或手动下载安装

下载完压缩包后，打开命令行进入下载文件所处的目录，并执行以下命令：

```
tar xf git-lfs-*.tar.gz
cd git-lfs-*
sudo ./install.sh
```

## 配置

使用 `git lfs track` 追踪需要使用 Git LFS 管理的文件。如：

```
git lfs track "*.psd"
```

也可以手动编辑 Git 仓库根目录下的 `.gitattributes` 文件，如：

```
*.psd filter=lfs diff=lfs merge=lfs -text
```

## 常用 Git LFS 命令

```
# 查看当前使用 Git LFS 管理的匹配列表
git lfs track

# 使用 Git LFS 管理指定的文件
git lfs track "*.psd"

# 不再使用 Git LFS 管理指定的文件
git lfs untrack "*.psd"

# 类似 `git status`，查看当前 Git LFS 对象的状态
git lfs status

# 枚举目前所有被 Git LFS 管理的具体文件
git lfs ls-files

# 检查当前所用 Git LFS 的版本
git lfs version

# 针对使用了 LFS 的仓库进行了特别优化的 clone 命令，显著提升获取
# LFS 对象的速度，接受和 `git clone` 一样的参数。 [1] [2]
git lfs clone https://github.com/user/repo.git
```

[1] `git lfs clone` 通过合并获取 LFS 对象的请求，减少了 LFS API 的调用，并行化 LFS 对象的下载，从而达到显著的速度提升。`git lfs clone` 命令同样也兼容没有使用 LFS 的仓库。即无论要克隆的仓库是否使用 LFS，都可以使用 `git lfs clone` 命令来进行克隆。
[2] 目前最新版本的 `git clone` 已经能够提供与 `git lfs clone` 一致的性能，因此自 Git LFS 2.3.0 版本起，`git lfs clone` 已不再推荐使用。

## 迁移已有的 git 仓库使用 git lfs 管理

```
# 重写 master 分支，将历史提交中的 *.zip 都用 git lfs 进行管理
git lfs migrate import --include-ref=master --include="*.zip"

# 重写所有分支及标签，将历史提交中的 *.rar,*.zip 都用 git lfs 进行管理
git lfs migrate import --everything --include="*.rar,*.zip"
```

**注意：** 重写历史后的提交需执行 `git commit --force`，请确认在本地的操作合适无误后再进行提交。
**注意：** 如有迁移至 git lfs 前的仓库的多份拷贝，其他拷贝可能需要执行 `git reset --hard origin/master` 来重置其本地的分支，注意执行 `git reset --hard` 命令将会丢失本地的改动。