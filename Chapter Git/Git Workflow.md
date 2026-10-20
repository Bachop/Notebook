# Git Workflow for Embedded 针对嵌入式工程的 Git 工作流

***[@Bachop](https://github.com/Bachop)***

<!--一套针对嵌入式工程的Git管理工作流。-->
本文档基于**嵌入式工程管理需求**编写，适用于 **Windows** 系统下基于 **Git** 嵌入式项目管理的快速初步使用。

---

## 目录
- **[1. Git 搭建](#1-git-搭建)**
  - [1.1 Git安装](#11-git-安装)
  - [1.2 Git 配置](#12-git-配置在-git-bash-中执行)
    - [1.2.1 用户信息](#121-用户信息)
    - [1.2.2 远程传输配置](#122-远程传输配置以ssh-密钥方式为例)
  - [1.3 Git LFS 大文件管理](#13-git-lfs-大文件管理)
    - [1.3.1 Git LFS 安装](#131-git-lfs-安装) 
    - [1.3.2 Git LFS 配置](#132-git-lfs-配置)
    - [1.3.3 可能遇到的问题](#133-可能遇到的问题)
- **[2. Git 工作流](#2-git-工作流)**
  - [2.1 嵌入式项目仓库结构 ](#21-嵌入式项目仓库结构)
  - [2.2 Git 仓库初始化](#22-git-仓库初始化)
    - [2.2.1 本地初始化](#221-本地初始化)
    - [2.2.2 远程初始化](#222-远程初始化)
  - [2.3 Git Bash 基本命令](#23-git-bash-基本命令)
    - [2.3.1 文件操作](#231-文件操作)
      - [2.3.1.1 添加文件](#2311-添加文件)
      - [2.3.1.2 移除文件](#2312-移除文件)
      - [2.3.1.3 撤销修改](#2313-撤销修改只针对本地仓库)
    - [2.3.2 分支操作](#232-分支操作)
      - [2.3.2.1 创建分支](#2321-创建分支)
      - [2.3.2.2 切换分支](#2322-切换分支)
      - [2.3.2.3 合并分支](#2323-合并分支)
      - [2.3.2.4 分支变基](#2324-分支变基)
      - [2.3.2.5 删除分支](#2325-删除分支)
    - [2.3.3 标签Tags](#233-标签tags)
  - [2.4 Git 工作流](#24-git-工作流)

---

## 1. Git 搭建
本节将介绍 Git 的安装、配置，以及 Git LFS 的相关内容。
### 1.1 Git 安装 
1. 前往[Git 官网](https://git-scm.com/install/windows)根据CPU架构获取安装程序（或使用[ winget tool ](https://learn.microsoft.com/en-us/windows/package-manager/winget/)命令）
```cmd
winget install --id Git.Git -e --source winget
```
> 使用 Setup 安装时，可勾选 Git LFS (Large File Storage) 以进行大文件管理，详见[Git LFS 大文件管理](#13-git-lfs-大文件管理)。
2. 验证安装：
```cmd
git version
```

### 1.2 Git 配置（在 Git Bash 中执行）
#### 1.2.1 用户信息
```Git Bash
# 全局配置
git config --global user.name "yourname"
git config --global user.email "youremail@example.com"

# 显示配置
git config --global --list

# 指定仓库局部配置
cd your_project_dir
git config user.name "yourname"
```

#### 1.2.2 远程传输配置（以SSH 密钥方式为例）
- 生成密钥（以 **ed25519** 数字签名算法为例），存放于 `用户/.ssh` 目录下：
```Git Bash
ssh-keygen -t ed25519 -C "youremail@example.com" -f ~/.ssh/id_ed25519
```
- 查看生成公钥：
```Git Bash
cd ~/.ssh
cat id_ed25519.pub
```
- 将公钥添加到[远端](http://10.10.100.100/-/user_settings/ssh_keys)（以 TaShan GitLab 服务器为例）
- 验证连接（以 TaShan GitLab 服务器为例）：
```Git Bash
# 此命令指定使用ed25519密钥
ssh -T -i ~/.ssh/id_ed25519 git@10.10.100.100

# 出现如下信息表示连接成功
Welcome to GitLab, @yourname!
```

### 1.3 Git LFS 大文件管理
> 无大文件管理需求可跳过此小节。

嵌入式项目中，PCB、原理图，docx文档，xlsx表格，甚至图片等**二进制文件**无法被原生Git有效处理，不能被比较差异，并且每次更改都会在Git仓库中存储几乎完整的Blob对象，易造成仓库体积膨胀，并拖慢`clone`或`pull`的速度。

**Git LFS (Large File Storage)** 使用指针文件替代仓库中的大文件本体，当进行`clone`或`pull`时能够按版本需求下载，而非下载所有版本的大文件。
#### 1.3.1 Git LFS 安装
1. 前往[Git LFS 官网](https://git-lfs.com/)下载安装包并安装，或在使用Git Setup 安装时勾选：
![[√] Git LFS (Large File Storage)](../Pics4Markdown/git-lfs.png)
2. 验证安装
```cmd
git lfs version
```

#### 1.3.2 Git LFS 配置
1. 初始化（仅需1次）
```Git Bash
# 全局初始化
git lfs install

# 指定仓库初始化，不影响其它仓库
git lfs install --local

# 检查Git LFS配置
git config --get-regexp filter.lfs

# 回复如下内容则表示配置正确
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required true
```

2. 追踪大文件
```Git Bash
# 每个有大文件管理需求的仓库都要执行以下命令
cd your_project_dir

# 例：追踪PCB、原理图、Excel、Word、图片等文件
git lfs track "*.PcbDoc"
git lfs track "*.SchDoc"
git lfs track "*.xlsx"
git lfs track "*.docx"
git lfs track "*.png"
git lfs track "*.jpg"
...
```
执行上述命令后，项目根目录通常会生成 `.gitattributes` 文件，内容如下（以前面track的文件为例）：
```.gitattributes
*.PcbDoc filter=lfs diff=lfs merge=lfs -text
*.SchDoc filter=lfs diff=lfs merge=lfs -text
*.xlsx filter=lfs diff=lfs merge=lfs -text
*.docx filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
...
```

> 也可以手动生成编辑 `.gitattributes` 文件。

**必须将`.gitattributes`文件提交到仓库中**，否则协作者拉取后不会生效：
```Git Bash
git add .gitattributes
git commit -m "chore: 配置Git LFS规则"

# 查看Git LFS追踪的文件
git lfs ls-files
```

#### 1.3.3 可能遇到的问题
- 当`clone`某仓库后发现大文件只是几行的文本指针，可以使用以下命令手动拉取：
```Git Bash
git lfs pull
```

---

## 2. Git 工作流
本节将介绍 Git 工作流，以Git Bash为例。
### 2.1 嵌入式项目仓库结构
以下是一个建议的工程目录结构：
```
YourEmbeddedProject/
├── PCB/                    // --------PCB/原理图
├── Code/                   // --------代码
│   ├── Libraries/              // --------芯片驱动
│   ├── Listings/               // --------映射文件 建议忽略
│   ├── Output/                 // --------构建输出 建议忽略
│   ├── Project/                // --------项目工程 建议仅收录工程文件
│   └── USER/                   // --------应用代码 建议全部添加
├── Document/               // --------文档
├── ...                     // --------其它目录
└── *files                  // --------根目录文件，如README.md, .gitattributes（大文件管理必需）
```
### 2.2 Git 仓库初始化
在初始化仓库之前，建立一个`.gitignore`文件是十分必要的，这允许在使用 Git 管理项目时忽略某些文件或目录。
通常`.gitignore`编写的原则有：
- **可复现**：协作者拉取后能够复现完全一致的编译结果；
- **环境无关**：提交的文件不要含有与个人PC绝对路径有关的信息，也不要提交个性化配置的文件（如Keil的个人配置文件`.uvoptx`。但**工程文件`.uvprojx`必须提交**，它定义了该工程的文件结构）；
- **忽略编译产物**：不要提交编译生成的日志、中间文件及最终产物（如`.hex` `.bin` `.map` `.o`等）。
```.gitignore
### ignore语法示例，其中相对路径以.gitignore所在目录为根目录
directory/sub_directory/        # 忽略该目录
directory/*.file_extension      # 忽略该目录下的指定扩展名的所有文件
directory/file.file_extension   # 忽略该目录下的指定文件

directory/*                     # 忽略该目录下的全部文件
!directory/file.file_extension  # 与上一条合用，保留该目录下的指定文件
```
#### 2.2.1 本地初始化
1. 本地新建仓库
```Git Bash
# 选取项目根目录进行仓库创建
cd D:/MyProject/Project_1
git init
#初始化成功后会看到目录路径后新增"(main)"，表示主分支已建立。
```

2. 克隆远程仓库
```Git Bash
# 克隆仓库会创建一个同名的目录，并把远程仓库的代码clone到该目录下
# 因此同样需要为此准备一个目录，但这个目录并非目标仓库的根目录，而是它的上级目录
cd D:/MyProject/

# 使用git clone指令，输入获取的远程仓库URL，以TaShan GitLab服务器、SSH为例
git clone git@10.10.100.100:username/Project_1.git

# 进入目标仓库的根目录，克隆仓库不再需要init，可以直接开始后续工作
cd D:/MyProject/Project_1
```

3. 可选——利用`.bashrc`（用于配置命令行环境变量、别名、提示符的文件）创建仓库路径的别名，能够更方便地进入仓库：
```Git Bash
# 检查.bashrc文件是否存在
ls -a ~ | grep .bashrc
# 创建/编辑.bashrc文件（以vim编辑器为例）
vim ~/.bashrc

# 为项目创建路径别名
alias project_1="cd D:/MyProject/Project_1"

# 保存并退出（编辑结束后，按ESC，然后输入以下命令）
:wq

# Git Bash期望由.bash_profile来加载.bashrc，因此需要创建这个文件
echo 'if [ -f ~/.bashrc ]; then source ~/.bashrc; fi' > ~/.bash_profile

# 否则会收到如下警告：
WARNING: Found ~/.bashrc but no ~/.bash_profile, ~/.bash_login or ~/.profile.
```

**上述操作在重启Git Bash窗口时生效**，后续就可以使用别名进入目标仓库了。
```Git Bash
# 使用别名快速进入目标仓库
cd project_1
```

#### 2.2.2 远程初始化
在服务器上创建对应仓库，并添加仓库URL到Git（以SSH为例），推送：
```Git Bash
# 先CD到你的仓库根目录，再执行添加远程仓库命令
git remote add origin git@10.10.100.100:yourname/Your_Project.git
# origin为自定义的远程仓库名，如本地仓库对应多个远程仓库时注意区分命名。

# 推送（-u 表示设置上游，每次新建本地分支第1次推送时使用即可）
git push -u origin main

# 如果想删除远程仓库
git remote rm origin
```

### 2.3 Git Bash 基本命令
现在项目仓库已经初始化完成，并成功进行了首次提交，以及远程仓库的首次推送。本小节将介绍一些常用的 Git Bash 命令。

> 如果你使用 **[VSCode](https://code.visualstudio.com/download)**，那么你可以在 Git 插件中直接管理操作文件、分支，以及远程仓库，从而避免记忆过多 Git Bash 命令。其它 Git 工具也一样。

#### 2.3.1 文件操作
##### 2.3.1.1 添加文件
当你建立并初始化仓库后，如果目录里面有文件，那么所有文件（没被Git忽视的）都将被请求追踪。
```Git Bash
# 查看当前Git状态
git status

# 通常会显示如下内容
Untracked files:
  (use "git add <file>..." to include in what will be committed)

# 添加所有新增文件
git add .

# 提交
git commit -m "init: 初始化提交"
# 以上步骤可以在初始化本地仓库时就执行
```

在每日工作中，当你修改了某些文件后：
```Git Bash
# 查看当前Git状态
git status
# 添加指定文件
git add directory/file.file_extension
# 提交（可以参考约定的提交规范编写提交信息）
git commit -m "feat: 新增功能"

# 如果你只修改了1个文件，或者修改的几个文件都服务于同一次提交更新
git add .
git commit -m "feat: message"
# git add .会添加所有已修改但未提交的文件
```

##### 2.3.1.2 移除文件
```Git Bash
# 取消git追踪，并删除本地文件
git rm directory/file.file_extension
# 取消git追踪，但保留本地文件
git rm --cached directory/file.file_extension
```

##### 2.3.1.3 撤销修改（只针对本地仓库）
```Git Bash
# 撤销上一次 git add 操作
git reset HEAD file.file_extension
# 撤销上1次 git commit 操作，修改 HEAD^i 以修改撤销提交数量。
# 参数--soft表示只撤销提交但保留文件修改，--hard表示撤销提交且放弃修改
git reset --soft HEAD^1
```

#### 2.3.2 分支操作
分支能够让多个开发人员并行工作，开发多个功能、修复 bug 或进行实验，而不会影响主代码库。一个分支代表一条独立的开发线，使用分支意味着你可以从开发主线上分离开来，然后在不影响主线的同时继续工作。
```Git Bash
# 查看所有分支
git branch

# 查看所有远程分支
git branch -r
```
##### 2.3.2.1 创建分支

```Git Bash
# 创建分支
git branch branch_name
```
##### 2.3.2.2 切换分支
```Git Bash
# 切换分支
git checkout branch_name

# 创建并切换分支，此后操作会在直接新分支下进行，推荐使用此命令创建分支
git checkout -b branch_name
```
##### 2.3.2.3 合并分支
```Git Bash
# 切换到待合并分支（如果当前分支为待合并分支，则忽略此步骤）
git checkout branch_name_1
# 合并分支
git merge branch_name_2
```
当两个分支对同一文件进行了不同修改时，在合并过程中可能会存在冲突，需要手动处理，按照Git的提示进行解决，解决冲突后：
```Git Bash
# 添加并提交冲突文件
git add file.file_extension
git commit -m "Merge: message"
```

##### 2.3.2.4 分支变基
在基于某分支创建的子分支开发时，如果父分支更新了提交，而子分支的开发需要基于最新改动进行，此时可以将父分支合并进子分支中：
D

也可以将子分支变基到新的父分支上。
分支变基操作可以获得**干净的线性提交历史**，**代价是重写所有被挪动提交的哈希值**。
:warning:**多人同时协作开发时慎用**:warning:
```Git Bash
# 切换到待变基分支
git checkout son_branch

git rebase father_branch
```
##### 2.3.2.5 删除分支
```Git Bash
# 删除分支（强制删除使用 -D ，删除远程分支不适用此项）
git branch -d branch_name
# 删除远程分支
git push origin -d branch_name
```

#### 2.3.3 标签Tags
标签用于对 Git 的提交进行标记，指向的是某个提交，与分支无关。标签通常用于版本发布。
```Git Bash
# 例：当完成提交后（通常是发布前的最后一次提交）
git tag -a vX.X.X -m "版本发布信息"

# 对指定提交打标签（需获取目标提交的hash值）
# 查看历史提交（完整40位hash）
git log
# 查看历史提交的短7位hash，通常使用此命令即可
git log --oneline
# 指定提交
git tag -a tag_name commit_hash -m "message"

# 推送标签
git push origin tag_name

# 查看标签
git tag

# 删除标签
git tag -d tag_name

# 删除远程标签
git push origin :refs/tags/tag_name
```

### 2.4 Git 工作流
在日常开发中，推荐使用以下工作流：
- 约定项目的目录、文件结构、协作管理内容，制定`.gitignore`；
- 使用多分支模式进行开发，保持`main`分支的干净、稳定；
- 避免提交编译不通过，或编写不完整的代码；
- To be continued...

---

***[Git Workflow Markdown](#git-workflow-for-embedded-针对嵌入式工程的-git-工作流)** Version 1.0, 2026.07.08*
