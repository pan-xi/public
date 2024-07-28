- LFS目标：从一个**完全**、**干净**的用户，从头开始构建、**编译**出一个系统（恍然大悟，一开始以为是手搓内核，准确来说是使用已有软件包和内核编译出一个系统 :doge:）
- 我的目标：更适合像我一样的参考的LFS( :doge: )
#### 待补
- [ ] [源码编译](https://moi.vonos.net/linux/beginners-installing-from-source/)
- [ ] [软件编译、安装](https://tldp.org/HOWTO/Software-Building-HOWTO.html)
- [ ] [FQA](https://www.linuxfromscratch.org/faq/)
- [ ] [疑难排查](https://www.linuxfromscratch.org/hints/downloads/files/errors.txt)
- [ ] [如何聪明的提问](http://catb.org/~esr/faqs/smart-questions.html)
- [ ] 没有运行2.2.2的检查脚本，我是手动检查并下载的，同时脚本的含义似乎是不低于就行了，所以没有完全按照提供版本，期待出现更多问题去解决
- [ ] 在第四章的4.2的创建目录有问题`case $(uname -m) in x86_64) mkdir -pv $LFS/lib64 ;; esac`，这行代码会将`lib64`文件夹建在`$LFS`，也就是LFS的根目录下，但是当前的(发行版)系统都在`/usr`下，只是符号链接到根目录下
- [x] 在“重要的提前阅读资料”的最后强调了“sh 是指向 bash 的符号链接。”，但是当前是`/bin/sh -> dash`，先不改变`/bin/sh`的指向，可能后续会存在问题，我先把这个问题记录在这里
	- 还是改正了
	- **兼容性:** LFS 的构建脚本可能使用了一些 `bash` 特有的语法或功能，如果 `/bin/sh` 指向 `dash`，这些脚本就可能无法正常运行。 为了避免兼容性问题，LFS 建议将 `/bin/sh` 指向 `bash`。
- [ ] 当前是使用服务器构建的，这是一个很大的问题，因为与当前的硬盘、设备进行了绑定，并不能直接通过拷贝或移植来安装到其他设备，要不以后趁没人的时间来继续做这件事情，要不尝试自己打包成ISO文件，使用虚拟机进行安装，可能需要更多的额外知识
#### 关于分阶段构建和宿主环境重启的注意事项
- ![[Pasted image 20240723195402.png]]
	- `export LFS=/mnt/lfs`![[Pasted image 20240724200520.png]]
	- 虽然但是，我重启了docker，这个环境变量还在
		- 是因为我把环境变量写入了`~/.bashrc`
- 5-6章![[Pasted image 20240723195440.png]]![[Pasted image 20240727041539.png]]
- 7-10章
  ![[Pasted image 20240723195542.png]]
  ![[Pasted image 20240728144528.png]]
#### 格式约定
- ![[Pasted image 20240722144847.png]]
- ![[Pasted image 20240722144944.png]]
- ![[Pasted image 20240722145006.png]]
- ![[Pasted image 20240722145018.png]]
- ![[Pasted image 20240722145154.png]]
#### 第三部分的编译术语约定
- ![[Pasted image 20240728114518.png]]
- ![[Pasted image 20240728114530.png]]
- 术语分为`build, host, target`，使用三个属于去描述一个阶段
	- build：指构建程序时使⽤的机器
		- 其实就是当前阶段使用的机器
	- host：指将来会运⾏被构建的程序的机器。
		- 其实就是这个编译器编译出来的代码给哪台机器运行的
	- target：编译器为这台机器产⽣代码。
		- （如果）这个编译器编译的是编译器，那么这个编译出来的编译器是编译谁的代码的
- 例子的语言性描述：我们仅在⼀台运⾏缓慢的机器上有编译器， 称这台机器为 A，这个编译器为 ccA。我们还有⼀台运⾏较快的机器 (B)，但它没有安装编译器，⽽我们 希望为另⼀台缓慢的机器 (C) ⽣成代码。如果要为 C 构建编译器，可以通过三个阶段完成：
	- 阶段1，AAB：在机器 A 上，使⽤ ccA 构建 交叉编译 器 cc1
		- 在A机器上，使用ccA，构建交叉编译器cc1，cc1是要运行在A机器上的，而cc1是为了编译B的代码
	- 阶段2，ABC：在机器 A 上，使⽤ cc1 构建 交叉编译 器 cc2
		- 在机器 A 上，使⽤ cc1 构建 交叉编译 器 cc2，这个cc2是要运行在B上，而cc2编译出来的代码是要在C上运行的
	- 阶段3，BCC：在机器 B 上，使⽤ cc2 构建 交叉编译 器 ccC
		- 在机器 B 上，使⽤ cc2 构建 交叉编译 器 ccC，这个ccC是要运行在C上，产生的代码是运行在C上的
	- **ccA和ccC被称为本地编译器，cc1和cc2是交叉编译器**
- 这样，我们可以为机器 C 使⽤ cc2 在快速的机器 B 上构建所有其他程序。
- 除⾮ B 能运⾏ 为 C 编译的程序，则在 C 上实际运⾏它们之前，⽆法测试它们的功能。例如，如果要测试 ccC，我们可 能需要增加第四个阶段：![[Pasted image 20240728115612.png]]
	- 描述，CCC：在机器 C 上，⽤ ccC 重新 构建它本 ⾝，并测 试
		- 在机器 C 上，⽤ ccC 重新 构建它本 ⾝（即ccC），ccC是运行在C上的，ccC产生的代码是运行在C上的
#### 宿主机设置和软件包一般构建过程说明
- ![[Pasted image 20240728152259.png]]
## 序言
- LFS 的结构尽可能遵循 Linux 的各项标准。主要的标准有：
	- POSIX.1-2008.
	- Filesystem Hierarchy Standard (FHS) Version 3.0
	- Linux Standard Base (LSB) Version 5.0 (2015)
- LSB[^1] 由四个独立的规范组成：Core、Desktop、Runtime Language 和 Imaging

[^1]:许多人不认同 LSB 的要求。定义 LSB 的主要目的是保证专有软件能够在满足 LSB 的系统上  正常安装并运行。然而 LFS 是基于源代码的，用戶完全知道自己需要什么软件包；您可以选  择不安装 LSB 要求的一些软件包

- LFS 的目标是构建一个完整且基本可用的系统 ⸺ 包括所有再次构建 LFS 系统本身所需的软件包
## 概述
## 准备工作
### 准备宿主系统
- ![[Pasted image 20240725133717.png]]
### 软件包和补丁
- 软件包的下载和工作（解压、编译）目录：`$LFS/sources`，同时执行`chmod -v a+wt $LFS/sources`
	- **`a`**: 表示所有用户 (all users)，包括文件所有者、文件所属组和其他用户。
	-  **`+`**: 表示添加权限。
	- **`w`**: 表示写入权限，用户可以修改文件或目录的内容。
	- **`t`**: 表示粘滞位 (sticky bit)，设置该位后，只有文件所有者和 root 用户可以删除或重命名该目录下的文件，即使其他用户拥有该目录的写入权限。
	- 由于 LFS 需要从源代码编译构建系统，因此需要将 `sources` 目录设置为可写，以便用户可以将下载的源代码文件放到该目录下。
- **尽量使用wget-list-sysv来减少重复工作**，使用[`wget-list-sysv`](https://www.linuxfromscratch.org/lfs/view/stable/wget-list-sysv) 文件来下载LFS的包，使用[md5sums](https://www.linuxfromscratch.org/lfs/view/stable/md5sums)来验证文件完整性
```bash
# need to set porxy firstly
# path: ~/   (is the '/root')
pwd

# need to copy the context of  web of 'wget-list-sysv' to the file which name is 'wget-list-sysv'
wget --input-file=wget-list-sysv --continue --directory-prefix=$LFS/sources

# need to copy the context of web of  'md5sums' to the file which name is 'md5sums'
# notice: there are some package can't be downloaded, because of some special reasons, they need to download by the device which can dirctly cross the GFW. Then execute the below instruction again.
pushd $LFS/sources
md5sum -c md5sums
popd
```
- ![[Pasted image 20240726223435.png]]
### 最后准备工作（last but far from least ( :doge: )）
#### 4.2 目录布局 
- 在 LFS ⽂件系统中创建有限⽬录布局
	- 使得在第 6 章中编译的程序 (以及第 5 章中的 glibc 和 libstdc++) 可以被安装到它们的最终位置
	- 在第 8 章中重新构建它们时，就能直接覆盖这些临时程序
	- `/etc`: 存放**系统范围的配置文件**，这些配置文件影响整个系统的行为，而不仅针对特定用户。(以下均为常见内容，并非所有)
	    - 用户帐户信息 (`passwd`, `shadow`, `group`)
	    - 网络配置 (`hosts`, `resolv.conf`, `network/interfaces`)
	    - 服务管理脚本 (`init.d`, `systemd`)
	    - 软件包配置
	- `/var`: 存放**系统运行时生成的可变数据**，这些数据的大小和内容可能会随着时间的推移而发生变化。
	    - 系统日志文件 (`/var/log`)
	    - 数据库文件 (`/var/lib/mysql`)
	    - 网站数据 (`/var/www`)
	    - 缓存文件 (`/var/cache`)
	    - 邮件队列 (`/var/spool/mail`)
	- `/usr`: 存放**与用户直接相关的程序和文件**，但不包括那些在系统启动和运行中必不可少的程序和文件（那些通常在`/bin`和`/sbin`中）
		- **`/usr/bin`**: 存放**所有用户都可以使用的可执行程序**。
		- **`/usr/lib`**: 存放**程序所需的库文件**，包括动态链接库 (.so 文件)。
		- **`/usr/sbin`**: 存放**系统管理员使用的可执行程序**, 用于系统管理和维护。
		- **`/usr/local`**: 存放**本地安装的软件**, 区别于通过系统包管理器安装的软件。
	- 准备`$LFS/tools`
```bash
# the usage can refer the above
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

# notice: there is path/to/lfs
# path: '/mnt/lfs'
pwd

# create the symbolic link to the dirs below of the usr(/mnt/lfs/usr), to make using comfortable
# this action can be found in the modern system
 for i in bin lib sbin; do
>  ln -sv usr/$i $LFS/$i
> done

# use the sentence of case to judge the architecture of the device, if it is the X86_64 then mkdir the '$LFS/lib64'
# the ';;' is the sign of the end of case
# why X86_64? some package need to install in the '/lib64' at the X86_64
case $(uname -m) in
>  x86_64) mkdir -pv $LFS/lib64 ;;
> esac

# prepare a dir for the chapter6, to be used to install the cross compiler, to separate with the other application
mkdir -pv $LFS/tools
```
#### 添加LFS用户
- 使用新用户`LFS`来操作编译软件包的事情，所以，新建用户、设置密码、更改文件的拥有者、登录lfs
```bash
# create a new group named lfs
groupadd lfs
# '-s': set the default shell is '/bin/bash'
# default: 不履行，违约；缺席，弃权；默认。"Default" 来自于古法语单词 "defaut"，意为 "缺乏"、"不足"，在法律领域，"default" 最初指的是 "未履行义务"，例如未按时偿还债务或未出庭。 这暗示着当事人没有采取行动，导致自动产生了某种后果，在计算机科学中，"default" 继承了 "未采取行动" 的含义。它表示在用户未进行特定选择的情况下，系统或程序将自动采取的预设选项或行为，随着计算机的普及，"default" 逐渐被广泛应用于日常生活中，表示任何预设的、标准的或自动的选择，"default" 表示默认的意思，因为它体现了在缺乏明确指令或选择的情况下，系统或程序将采取的预设行为。
# '-g': add the new user to the group of 'lfs'
# '-m': create a major directory for the new user
# '-k': mean to copy the config of the user from the exit's template, including the '.bashrc', '.bash_profile', 'profile' and so on. there use '/dev/null', the '/dev/null' is represented the 'null device', means nothing will be outputing, nothing will be inputting. in this work, '/dev/null' can be help build from scartch really.
useradd -s /bin/bash -g lfs -m -k /dev/null lfs

# set passwd: 123
passwd lfs

# change the owner of all directory
# this command only to change the owner, can use the '-R' to change the group of owned
chown -v lfs $LFS/{usr{,/*},lib,var,etc,bin,sbin,tools}
# this command will be print a fault about the '/mnt/lfs/etc/efi', is the "Operation not permitted". beacause the file system of 'efi' is the 'FAT32', it is to be stored the guidance file of 'UEFI BOOT', it not to support the user or ship of owed, so it neednot to change the owner
chown -v lfs $LFS/{boot{,/*},etc,home,opt,tmp,sources,tools,usr}

# 在某些宿主系统上，下⾯的 su 命令不会正确完成，⽽会将 lfs ⽤⼾的登录会话挂起到后台。如果提⽰符 “lfs:~$” 没有很快出现，输⼊ fg 命令以修复这个问题。
su - lfs
```
#### 配置环境（工作环境 .bash_profile && .bashrc）
##### .bash_profile
- 现在登录的shell的环境是之前root用户所启动的shell终端的环境，由于之前的设置，lfs用户直接登录shell时，没有配置文件，一片空白，所以现在要配置lfs的工作环境
- 执行完这个命令后，提示符就从`lfs@23fff41d56c2:~$`变成`lfs:~$ `
```bash
# output: /home/lfs
echo $HOME

# output: xterm
echo $TERM

# cat > ~/.bash_profile << "EOF"
# 额，由于<<重定向符号，所以被识别成了shell注释，就很难受，所以就加了个转义字符\，真正要执行这行命令的话，第一行应该是上面那样子才对
# exec: it is a instruction inside the shell, it will be instead the shell use the specified shell
# env: the instrction to set the values of environment
# -i: clean the all values of environment which inherited from the root before
# HOME=$HOME TERM=$TERM PS1='\u:\w\$ ': they are the values of the 'env' to set.
# HOME=$HOME: save the directory of the home of lfs
# TERM=$TERM: save the type of shell, like "xterm", to control the color of print, and the motivate of the cursor and so on, if not use this. the type may be the default like "dumb" or "linux", and some progresses will cannot be operated normally or displaed badly
# PS1='\u:\w\$ ': set the style of the shell print, '\u' means name of user, '\w' means the directory of the moment of the working, '\$' means the "command prompt", in general, '$' is used to be represent the common user, '#' is used to be represent root
# after this command, the prompt will be changing 'lfs:~$' instead of 'lfs@ContainerID_or_group:path$'
cat > ~/.bash_profile \<< "EOF"
> exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
> EOF
```
- 在以 lfs ⽤⼾登录或从其他⽤⼾使⽤带 “-” 选项的 su 命令切换到 lfs ⽤⼾时，初始的 shell 是⼀个登录 shell。它读取宿主系统的 /etc/profile ⽂件 (可能包含⼀些设置和环境变量)，然后读取 .bash_ profile。我们在 .bash_profile 中使⽤ exec env -i.../bin/bash 命令，新建⼀个除了 HOME, TERM 以及 PS1 外没有任何环境变量的 shell 并替换当前 shell。这可以防⽌宿主环境中不需要和有潜在⻛险的环境 变量进⼊构建环境。(可恶，执行完才告诉原因！)
##### .bashrc
- 新的 shell 实例是 ⾮登录 shell，它不会读取和执⾏ /etc/profile 或者 .bash_profile 的内容，⽽是读取并执⾏ .bashrc ⽂件。现在创建⼀个 .bashrc ⽂件
	- `set +h`
		- set +h 命令关闭 bash 的散列功能。
		- bash shell 的散列机制
			- bash shell 会维护一个内部的 **散列表 (hash table)**，用来记录之前执行过的命令的完整路径。
			- 当您再次执行相同的命令时，bash 会先查询散列表，如果找到匹配的记录，就会直接使用记录中的路径，而不需要再次搜索 `$PATH`。
		- ⼀般情况下，散列是很有⽤的 ⸺ bash 使⽤⼀个散列表维 护各个可执⾏⽂件的完整路径，这样就不⽤每次都在 PATH 指定的⽬录中搜索可执⾏⽂件。
		- 然⽽，在 构建 LFS 时，我们希望总是使⽤最新安装的⼯具。
		- 关闭散列功能强制 shell 在运⾏程序时总是搜索 PATH。
		- 这样，⼀旦$LFS/tools/bin 中有新的⼯具可⽤，shell 就能够找到它们，⽽不是使⽤之前记忆 在散列表中，由宿主发⾏版提供的 /usr/bin 或 /bin 中的⼯具。
	- `umask 022` 
		- 将⽤⼾的⽂件创建掩码 (umask) 设定为 022，保证只有⽂件所有者可以写新创建的⽂件和⽬录， 但任何⼈都可读取、执⾏它们。(如果系统调⽤使⽤默认模式，则新⽂件将具有权限模式 644，⽽新⽬录具有权限模式 755)。
	- `LFS=/mnt/lfs` 
		- LFS 环境变量必须被设定为之前选择的挂载点。
	- `LC_ALL=POSIX LC_ALL` 
		- 环境变量控制某些程序的本地化⾏为，使得它们以特定国家的语⾔和惯例输出消息。
		- 将 LC_ ALL 设置为 “POSIX” 或者 “C”(这两种设置是等价的) 可以保证在交叉编译环境中所有命令的⾏ 为完全符合预期，⽽与宿主的本地化设置⽆关。
			- "POSIX" 和 "C" 在 LC_ALL 环境变量中的效果 并不总是完全等价，但它们在很多情况下非常接近，足以在 LFS 构建中互换使用。
	- `LFS_TGT=$(uname -m)-lfs-linux-gnu`
		- LFS_ TGT变量设定了⼀个⾮默认，但与宿主系统兼容的机器描述符。该描述符被⽤于构建交叉编译器和交 叉编译临时⼯具链。**⼯具链技术**（后续章节）说明将提供关于这个描述符的更多信息。
		- **`-lfs-linux-gnu`**: 这是 LFS 项目约定的一个后缀，用于标识目标平台的操作系统类型和 C 标准库。
		- **`LFS_TGT`**: 这是一个环境变量，用于存储最终生成的 LFS 目标平台标识符。
			- "机器描述符" 并不是一个通用的术语，而是 LFS 项目内部使用的概念。
			- 用来识别目标平台的标识符，用于构建交叉编译工具链
			- 该标识符由宿主系统的硬件架构和 LFS 项目约定的后缀组成。
		- LFS 目标平台:**最终希望将 Linux 系统构建在哪个平台上**
			- 目标平台可以与宿主系统**相同**，也可以**不同**。
			- 明确目标平台后，LFS 构建系统才能选择正确的软件包、编译器选项和库文件，确保最终生成的系统能够在目标平台上正常运行。
			- 在交叉编译的情况下，目标平台的硬件架构、库版本等信息尤为重要，因为宿主系统无法直接运行目标平台的程序。
	- `PATH=/usr/bin `
		- 许多现代 Linux 发⾏版合并了 /bin 和 /usr/bin。在这种情况下，标准 PATH 变量应该被设定为 /usr/ bin，以满⾜第 6 章所需。否则，后续命令将会增加 /bin 到搜索路径中。
		- **总结**：像上面说的，现代发行版，一般都是软连接`/usr/bin`到根目录下使用，如果发现系统中存在`/bin`且没有软连接到`/usr/bin`，那么需要再添加一个`PATH`指向`/bin`
			- 早期的 Unix 系统
				- `/bin` 目录用于存放系统启动和运行所必需的 **最基本** 的可执行文件，例如 `ls`, `cp`, `mv` 等，这些文件通常比较小，并且会被频繁使用。
				- `/usr/bin` 目录则用于存放其他 **非系统核心** 的可执行文件，例如用户常用的命令行工具、应用程序等，这些文件通常比较大，并且使用频率相对较低
				- 在当时的技术条件下，将这两个目录分开可以 **优化系统启动速度和资源利用率**。
			- 现代linux发行版
				- 随着硬件技术的发展，磁盘空间和内存容量不再是主要限制因素。
				- 许多现代 Linux 发行版 **合并了 `/bin` 和 `/usr/bin` 目录**，将所有可执行文件都放在 `/usr/bin` 目录下，简化了目录结构。
				- 但是，一些旧的脚本或程序可能仍然依赖于 `/bin` 目录的存在。
		- 在 LFS (Linux From Scratch) 构建过程中，我们需要 **严格控制构建环境**，避免宿主系统的影响，确保使用的是我们自己构建的工具链
		- 为了避免潜在的冲突，LFS 构建脚本会 **将 `PATH` 环境变量初始值设置为 `/usr/bin`**。
		- 如果宿主系统已经合并了 /bin 和 /usr/bin 目录，那么 /usr/bin 就是正确的搜索路径。
		- `PATH=/usr/bin` 这条语句是为了在 LFS 构建环境中建立一个 **干净、可控的命令搜索路径**，避免宿主系统的影响。
		- 它考虑了现代 Linux 发行版和旧系统之间可能存在的差异，并根据实际情况进行调整，确保 LFS 构建过程的顺利进行。
	- `if [ ! -L /bin ]; then PATH=/bin:$PATH; fi `
		- 如果 /bin 不是符号链接，则它需要被添加到 PATH 变量中。
		- ***可恶！又是这样子，搞明白一件事，才发现下面可能有解释！***
	- `PATH=$LFS/tools/bin:$PATH`
		- 我们将 $LFS/tools/bin 附加在默认的 PATH 环境变量之前，这样在第 5 章中，我们⼀旦安装了新的程 序，shell 就能⽴刻使⽤它们。
		- **这与关闭散列功能相结合**，降低了在第 5 章环境中新程序可⽤时,错误地使⽤宿主系统中旧程序的⻛险。
		- 等于说优先搜索`/tools/bin`，然后才去搜索`/usr/bin`，保证优先使用自己构建的编译器等工具，真正从头开始构建
	- `CONFIG_SITE=$LFS/usr/share/config.site`
		- 在第 5 章和第 6 章中，如果没有设定这个变量，configure 脚本可能会从宿主系统的` /usr/share/ config.site` 加载⼀些发⾏版特有的配置信息。
		- 覆盖这⼀默认路径，**避免宿主系统可能造成的污染**。
		- `/usr/share/config.site` 文件通常用于存储**编译系统范围**的配置选项，这些选项会被 `configure` 脚本读取，用于定制软件包的编译过程。
			- **集中管理编译选项:** 将通用的编译选项放在 `/usr/share/config.site` 文件中，可以避免在编译每个软件包时都重复指定这些选项。
			- **定制系统级编译行为:** 可以通过修改 `/usr/share/config.site` 文件，来改变系统上所有软件包的默认编译行为，例如指定编译器优化级别、默认安装路径等。
			- **支持交叉编译:** 在交叉编译环境中，可以使用 `/usr/share/config.site` 文件传递目标平台的相关信息，例如目标架构、库路径等。
			- 当运行 `configure` 脚本时，它会自动读取 `/usr/share/config.site` 文件中定义的变量，并将这些变量应用到编译过程中。
			- 不是所有软件包都支持 `/usr/share/config.site` 文件，具体取决于软件包自身的配置脚本。
			- 修改 `/usr/share/config.site` 文件可能会影响系统上其他软件包的编译，因此需要谨慎操作。
			- 示例('\n'就代表一个回车)：`CC=gcc \n CXX=g++ \n CFLAGS="-O2 -march=native" \n CXXFLAGS="-O2 -march=native" \n PREFIX=/usr`
	- `export`导入（:doge）上述变量
```bash
# because of the shell also uses the '#' to express the commit, so the '>#' is not a problem, needn't to delete '>'
# ~/.bashrc << "EOF"
~/.bashrc \<< "EOF"
># shutdown the hash list
> set +h
># mask the limitation of other users
> umask 022
># set the value of environment
> LFS=/mnt/lfs
># set the standard of actions of cross compiling
> LC_ALL=POSIX
># set the identify of  the platform of LFS
> LFS_TGT=$(uname -m)-lfs-linux-gnu
># set the path of search to '/usr/bin'
> PATH=/usr/bin
> # add the '/bin' to the PATH, when the '/bin' of system is not a symbolic link  
> if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
> # set to the path of search to be searching the '/tools/bin' firstly, to guarantee/ensure/pledge/affrim/confirm using the newest tools which build from scratch(/skrætʃ/)
> PATH=$LFS/tools/bin:$PATH
> # set the default config for lfs's compiler to avoid use the config of host
> CONFIG_SITE=$LFS/usr/share/config.site
> # the above code are only define the value without adding to the system, so need to 'export' 
> # export 在 Linux 终端中的确是“导出”的意思，但它并不是把数据导出到文件或外部系统，而是将 shell 变量 导出到子进程的环境中。
> #在 shell 中定义的变量，默认情况下只在当前 shell 会话中有效。 当你打开一个新的终端窗口或者执行一个 shell 脚本时，实际上是创建了一个新的子进程。这个子进程会继承父进程的一些环境变量，但默认情况下不会继承父进程中定义的所有变量。
> #export 命令的作用就是将指定的 shell 变量 标记为可被子进程继承。 当你使用 export 导出一个变量后，再创建的子进程就能访问到这个变量的值。
> export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
> EOF
```
- 移走`/etc/bash.bashrc`![[Pasted image 20240727030435.png]]
	- 构建的 Bash 软件包时，**被特意配置为不读取或执行 `/etc/bash.bashrc` 文件。**
	- **避免依赖:** `/etc/bash.bashrc` 文件通常包含了系统管理员针对特定发行版或环境做的定制化配置。 LFS 希望避免对这些外部配置的依赖，保持系统的 "纯净" 和可移植性。
		- **系统级 Bash 配置失效:** `/etc/bash.bashrc` 文件中定义的别名、环境变量、shell 选项等设置，在 LFS 系统中将不会生效。
	- **自定义配置:** LFS 鼓励用户根据自己的需求，通过修改其他配置文件 (例如 `~/.bashrc`) 来定制 shell 环境，而不是依赖系统级的 `/etc/bash.bashrc` 文件。
		- **用户级配置不受影响:** 用户仍然可以通过修改 `~/.bashrc` 文件来自定义自己的 shell 环境。
```bash
# use the root!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# move the '/etc/bash.bashrc' to give a pure environment and to remain the transplantable
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```
- 根据CPU逻辑核心数，指定make最多生成的构建任务，并行make来减少时间
```bash
# find the numbers of logical core
# the 'CPU(s)' is the number of logical core, 'Thread(s) per core' is the ship of equal between physical and logical, 'Core(s) per socket' is the number of physical cores, 'Socket(s)' is the number of CPU
# my server has 2 CPUs, 22 physical cores everyCPU, beacause one pyhsical equal two logical, so my parameter of 'make' can be 88(起飞！)
lscpu

# set the maximum(/ˈmæksɪməm/) of the logical cores for making
# this command will product a fault, because it is used to build the project, and the project should have Makefile, so should execute the 'export MAKEFLAGS=-j32'（可恶！还好往下看了一眼，不然就要完全卡住了，这谁想得到噶）
# output: make: *** No targets specified and no makefile found.  Stop.
make -j88

# now, because of the MAKEFLAGS will be use many times, and the docker be restarted, it will be lost, so i decided to write it in the .bashrc
# output: ~/(/home/lfs)
pwd
vim .bashrc
<<'COMMIT'
# start
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
MAKEFLAGS=-j88
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE MAKEFLAGS
#end
COMMIT
```
- 啊！！！为什么又在下面有对应的教程，可恶！不过我的做法似乎更合适( :doge: )
```bash
# 'nproc' could print the useable logical cores(a!!!!!!!!!!!!) 
# 如果不希望使⽤全部逻辑 CPU 核⼼，将 $(nproc) 替换为希望使⽤的核⼼数。
cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```
- 最后，为了保证构建临时⼯具所需的环境准备就绪，**强制 bash shell 读取刚才创建的配置⽂件**：`source ~/.bash_profile`，执行完后就像上面说的，最明显的改变就是提示符了
#### SBU-也就是以第一轮构建时间为基数来算倍数，以make -j4为标准环境
- 许多⼈想在编译和安装各个软件包之前，了解这⼀过程⼤概需要多少时间。由于 Linux From Scratch 可以在许多不同系统上构建，我们⽆法直接给出估计时间。
- 例如，最⼤的软件包 (gcc) 在最快的系统上只 要⼤约 5 分钟就能构建好，⽽在⼀些较慢的系统上需要若⼲天！因此，我们不提供实际时间，⽽是以标准构建单位 (SBU) 衡量时间。
- （检验CPU的时候到了）
- 例如，考虑⼀个编译时间是 4.5 SBU 的软件包。如果在您的系统上，需要 10 分钟来编译和安装第⼀轮的 Binutils，那么⼤概需要 45 分钟才能构建这个软件包。幸运的是，多数软件包构建时间⽐ Binutils 少。
- SBU 不是完全准确的，这是由于它受到许多因素的影响，包括宿主系统的 GCC 版本。SBU 只能⽤来估 计安装⼀个软件包可能需要的时间，估计结果的误差在个别情况下可能达到⼏⼗分钟。
- ![[Pasted image 20240727041131.png]]
#### 测试套件
- 多数软件包提供测试套件，⼀般来说，为新构建的软件包运⾏测试套件是个好主意，这可以进⾏⼀次 “完整性检查”，从⽽确认所有东西编译正确。如果测试套件中的所有检验项⽬都能通过，⼀般就可以证明这个软件包像开发者期望的那样运⾏。然⽽，这并不保证软件包完全没有错误。
- 某些软件包的测试套件⽐其他的更为重要。
	- 例如，组成核⼼⼯具链的⼏个软件包 — GCC、Binutils 和 Glibc 的测试套件就最为重要，因为这些软件包在系统的正常⼯作中发挥中⼼作⽤。
	- GCC 和 Glibc 的测 试套件需要运⾏很⻓时间，特别是在较慢的硬件上，但我们仍然强烈推荐运⾏它们。
- 在第 5 章和第 6 章中运⾏测试套件没有意义，因为测试程序是使⽤交叉编译器编译的，可能⽆法在构建它们的宿主系统运⾏。（也许我可以试试？）
- 在运⾏ Binutils 和 GCC 的测试套件时，较常⻅的问题是伪终端 (PTY) 被耗尽。这会导致⼤量测试出现 失败结果。这种现象有多种可能原因，但最常⻅的原因是宿主系统没有正确设置 devpts ⽂件系统。
## 构建交叉工具链和临时工具
### 阅读资料
- 本书的这⼀部分被分为三个阶段
	- ⾸先，构建⼀个交叉编译器和与之相关的库
	- 然后，使⽤这个交叉⼯具链构建⼀些⼯具，并使⽤保证它们和宿主系统分离的构建⽅法；
	- 最后进⼊ chroot 环境 (它能够进⼀步 提⾼与宿主的隔离度)，并构建剩余的，在构建最终的系统时必须的⼯具。
- 第 5 章和第 6 章的总⽬标是构造⼀个临时环境，它包含⼀组可靠的，能够与宿主系统完全分离的⼯具。
- 这样，通过使⽤ chroot 命令，其余各章中执⾏的命令就被限制在这个临时环境中。这确保我们能够⼲ 净、顺利地构建 LFS 系统
- 构建过程是基于交叉编译过程的。交叉编译通常被⽤于为⼀台与本机完全不同的计算机构建编译器及其 ⼯具链。这对于 LFS 并不严格必要，因为新系统运⾏的机器就是构建它时使⽤的。但是，交叉编译拥有 ⼀项重要优势：任何交叉编译产⽣的程序都不可能依赖于宿主环境。
- **LFS ⼿册并不是 (也不包含) ⼀份通⽤的，构建交叉 (或本地) ⼯具链的指南。除⾮您完全明⽩⾃ ⼰在⼲什么，请勿使⽤⼿册中的命令构建交叉⼯具链并⽤于构建 LFS 以外的⽤途。**
#### LFS的交叉编译实现
- 本书中涉及交叉编译的软件包都使⽤基于 autoconf 的构建系统。
- 基于 autoconf 的构建系统使 ⽤形如 `CPU-供应商-内核-操作系统`，称为三元组的名称表⽰⽬标系统类型。由于供应商字段通 常⽆关紧要，autoconf 允许省略它。
	- **`vendor` (供应商):** 操作系统或发行版的供应商，例如 `pc` (表示 IBM PC 兼容机), `apple`, `redhat`, `debian` 等。
- 好奇的读者可能会问，为什么⼀个“三元组”却包含四个部分?
	- 这是由于内核和操作系统两个 字段起源于⼀个“系统”字段。⾄今，⼀些系统仍然⽤三字段的格式准确描述，例如，x86_64- unknown-freebsd。
	- 但是对于其他⼀些系统，即使两个系统使⽤相同的内核，它们也可能截然不同，以⾄ 于不能使⽤相同的三元组。
		- 例如，运⾏在智能⼿机的 Android 和运⾏在 ARM64 服务器的 Ubuntu 完全不同，尽管它们使⽤相同类型的 CPU (ARM64) 和相同的内核 (Linux)。
		- 在没有仿真中间层的情况下，显然不能在智能⼿机上运⾏⽤于服务器的可执⾏⽂件，反之亦 然。
	- 因此，“系统” 字段被拆分为内核和操作系统两部分，以准确区分这些系统。对于本 例，Android 被表⽰为 aarch64-unknown-linux-android，⽽ Ubuntu 被表⽰为 aarch64- unknown-linux-gnu。
- 上面所说的“机器描述符”就是这个三元组，应该是lfs项目约定的
	- `$(uname -m)-lfs-linux-gnu`
- 有⼀种简单⽅法可以获得您的机器的三元组，即运⾏许多软 件包附带的 config.guess 脚本。
	- `gcc -dumpmachine`
	- 解压软件包的源码，然后输⼊ ./config.guess 运⾏脚本
- 您还需要注意平台的动态链接器的名称，它⼜被称为动态加载器 (不要和 Binutils 中的普通链接 器 ld 混淆)。(动态链接与静态链接的最大区别就是是否提前链接好、内存里有几份、加载时机)
	- 动态链接器由 Glibc 提供，它寻找并加载程序所需的共享库，为程序运⾏做好准 备，然后运⾏程序。
	- 在 32 位 Intel 机器上动态链接器的名称是 ld-linux.so.2 (在 64 位系统上 是 ld-linux-x86-64.so.2)
```bash
# output: [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
readelf -l /usr/bin/bash | grep interpreter
```
- 在 LFS 的构建过程中，为了将本机伪装成交叉编译⽬标机器，我们在 LFS_TGT 变量中，将宿主系统三 元组的 "vendor" 域修改为 "lfs"。改。我们还会在构建交叉链接器和交叉编译器时使⽤ --with-sysroot 选项，指定查找所需的 host 系统⽂件的位置。
	- 为什么“将本机伪装成交叉编译目标机器”？（重点看最后的“效果部分”）
		- **让编译器和链接器 "误以为" 它们正在为目标系统编译代码，而实际上它们仍然运行在宿主系统上，并使用宿主系统的一些工具和库**
		- 这是一种在 LFS 构建初期，**为了避免宿主系统环境干扰，而采取的一种特殊技巧**，通常被称为 "伪装交叉编译" (simulated cross-compiling) 或 "加拿大交叉编译" (Canadian cross compiling)。
		- 在 LFS 构建初期，我们还没有完整的目标系统环境，包括 C 标准库 (glibc)。如果直接使用宿主系统的编译器和链接器，可能会链接到宿主系统的库文件，导致最终构建的 LFS 系统不完整或无法运行。
		- 为了构建 LFS 系统，我们需要一些基本的工具，例如编译器、链接器、汇编器等，这些工具被称为 "工具链" (toolchain)。 而 "伪装交叉编译" 可以帮助我们使用宿主系统的资源，快速构建出一套能够运行在目标系统上的基本工具链，用于后续的 LFS 构建过程。
			- **假设:**
				- 您正在一台运行 Ubuntu 22.04 (x86_64 架构) 的电脑上构建 LFS 系统。
				- 您希望构建的 LFS 系统也运行在 x86_64 架构上，但**使用的是不同的内核和库。**
					- 可以自己选择内核版本，不一定要与宿主机相同
			- **问题:**
				- 您需要一个交叉编译器，能够在 Ubuntu 系统上生成 LFS 系统可执行的程序。
				- 但是，在 LFS 构建的早期阶段，您还没有完整的 LFS 系统环境，包括 C 标准库 (glibc) 等关键组件。
			- **解决方案："伪装交叉编译"**：欺骗编译器衡哥隔离文件系统
			- **欺骗编译器:**
				- 您将 `LFS_TGT` 变量设置为 `x86_64-lfs-linux-gnu`，而不是 `x86_64-pc-linux-gnu`。
				- 当您使用这个 "伪造" 的目标系统标识符来编译程序时，编译器会认为它正在为一个名为 "lfs" 的特殊系统编译代码，而不是为您的 Ubuntu 系统编译代码。
			- **隔离文件系统:**
				- 您使用 `--with-sysroot=/mnt/lfs` 选项来编译交叉编译器，其中 `/mnt/lfs` 是您挂载 LFS 系统根目录的位置。
				- 这样做是为了告诉编译器，在查找头文件和库文件时，不要去 Ubuntu 系统的目录 (例如 `/usr/include` 或 `/usr/lib`)，而是去 `/mnt/lfs` 目录下查找。
			- **效果：**
				- 编译器会生成与 Ubuntu 系统二进制不兼容的代码，因为它认为目标系统是 "lfs"，而不是 "pc"。
				- 编译器会链接 `/mnt/lfs` 目录下的库文件，而不是 Ubuntu 系统目录下的库文件。
- 这保证在第 6 章中的其他程序在构建时不会链接到宿主 (build) 系统的库。前两个阶段是必要的，第三个阶段可以⽤于测试：
	- ![[Pasted image 20240728124005.png]]
	- 在 pc 上 使⽤ ccpc 构建 交叉编译器 cc1
	- 在 pc 上 使⽤ cc1 构建 cclfs
	- 在 lfs 上 使⽤ cclfs 重新构建并测 试它本⾝
	- “在 pc 上” 意味着命令在已经安装好的发⾏版中执⾏。“在 lfs 上” 意味着命令在 chroot 环境中执⾏。
- 现在，关于交叉编译，还有更多要处理的问题：C 语⾔并不仅仅是⼀个编译器；它还规定了⼀个标准 库。在本书中，我们使⽤ GNU C 运⾏库，即 glibc (除此之外，还有名为 "musl" 的另⼀种 C 运⾏库实 现)。它必须为 lfs ⽬标机器使⽤交叉编译器 cc1 编译。但是，编译器本⾝使⽤⼀个库，实现汇编指令集 并不⽀持的⼀些复杂指令。这个内部库称为 libgcc，它必须链接到 glibc 库才能实现完整功能。另外，C ++ 标准库 (libstdc++) 也必须链接到 glibc。为了解决这个”先有鸡还是先有蛋“的问题，只能先构建⼀ 个降级的 cc1，它的 libgcc 缺失线程和异常等功能，再⽤这个降级的编译器构建 glibc (这不会导致 glibc 缺失功能)，再构建 libstdc++。但是这种⽅法构建的 libstdc++ 会缺失⼀些依赖于 libgcc 的功能。
	- （以下内容均来自gemini-1.5-pro，赞！）
	- 这段话解释了在 LFS 构建过程中，交叉编译 C 标准库 (glibc) 和 C++ 标准库 (libstdc++) 时遇到的一个经典问题：**循环依赖**，或者说是"先有鸡还是先有蛋"的问题。让我们逐步拆解：
	- **问题背景:**
		- 目标： 我们需要为目标 LFS 系统构建 C 标准库 (glibc) 和 C++ 标准库 (libstdc++)。
		- 工具： 我们使用交叉编译器 cc1 来编译这些库。
		- 依赖关系：
		    - cc1 依赖于它自己的内部库 `libgcc` 来实现一些复杂操作。
		    - `libgcc` 又需要链接到 `glibc` 才能提供完整的功能。
		    - `libstdc++` 同样需要链接到 `glibc`。
		- 矛盾： 为了编译 `glibc`，我们需要完整的 `cc1`（包括 `libgcc`）；而为了拥有完整的 `libgcc`，我们需要先编译好 `glibc`。
	- **解决方案：分阶段构建**
		- **阶段 1：构建 "降级" 的 cc1:**
			- 编译一个功能受限的 `cc1`，它的 `libgcc` 缺少线程、异常处理等高级功能。
			- 由于这个 "降级" 的 `libgcc` 不依赖于 `glibc`，因此可以被编译出来。
		- **阶段 2：构建 glibc:**
			- 使用 "降级" 的 `cc1` 编译 `glibc`。
			- 尽管 `cc1` 功能受限，但这并不会影响 `glibc` 的功能完整性。
		- **阶段 3：构建 libstdc++:**
			- 使用 "降级" 的 `cc1` 编译 `libstdc++`。
			- 由于 `cc1` 缺少完整 `libgcc` 支持，因此最终的 `libstdc++` 也会缺少一些依赖于 `libgcc` 的功能。
		- **影响和后续步骤:**
			- 使用这种方法构建的 `libstdc++` 缺少一些功能，但这通常不会影响 LFS 系统的基本功能。
			- 在 LFS 构建的后期，我们会重新构建完整的 `cc1` (包括完整的 `libgcc`) 和 `libstdc++`， 以获得所有功能。
- 上⾯⼀段的结论是 cc1 ⽆法使⽤功能降级的 libgcc 构建功能完整的 libstdc++，但这是我们在阶段 2 构建 C/C++ 库时唯⼀可⽤的编译器。两项原因导致我们⽬前不能⽤第⼆阶段构建的编译器，cc-lfs，构建这些 库。
	- ⼀般来说，cc-lfs 不能在 pc (宿主系统) 上运⾏。尽管 pc 和 lfs 的三元组互相兼容，为 lfs 构建的可执 ⾏⽂件会依赖于 glibc-2.39；⽽宿主系统可能使⽤不同的 libc 实现 (例如，musl) 或较旧的 glibc 版 本 (例如，glibc-2.13)。 
	- 即使 cc-lfs 能在 pc 上运⾏，在 pc 上使⽤它可能产⽣链接到 pc (宿主系统) 库的⻛险，因为 cc-lfs 是 ⼀个本地编译器。
		- 我倒不这么认为，因为前面一传参数`--with-sysroot`来保证“其他程序在构建时不会链接到宿主 (build) 系统的库”
- 因此在第⼆阶段构建 gcc 时，我们指⽰构建系统使⽤ cc1 再次构建 libgcc 和 libstdc++，但是将 libstdc ++ 链接到刚刚重新构建的 libgcc，⽽不是旧的，功能降级的版本。这样重新构建的 libstdc++ 就会具有 完整的功能。
#### 构建过程的其他细节
- 交叉编译器会被安装在独⽴的 $LFS/tools ⽬录，因为它不属于最终构建的系统。
- 我们⾸先安装 Binutils。这是由于 GCC 和 Glibc 的 configure 脚本⾸先测试汇编器和链接 器的⼀些特性，以决定启⽤或禁⽤⼀些软件特性。初看起来这并不重要，但没有正确配置的 GCC 或者 Glibc 会导致⼯具链中潜伏的故障。这些故障可能到整个构建过程快要结束时才突然爆发，不过在花费⼤ 量⽆⽤功之前，测试套件的失败通常可以将这类错误暴露出来。
	- binutils(Binary Utilities) 是一套用于创建、管理和操作二进制程序的工具集，它在 Linux 和 Unix-like 系统中扮演着至关重要的角色。
		- 主要组件
			- **ld (链接器):** 将目标文件和库文件链接在一起，生成可执行文件或库文件。
			- **as (汇编器):** 将汇编语言代码转换为目标文件。
			- **ar (归档器):** 创建、修改和解压静态库文件。
			- **nm (符号表查看器):** 列出目标文件或库文件中的符号表信息。
			- **objdump (目标文件分析器):** 反汇编目标文件，显示其汇编代码、符号表等信息。
			- **objcopy (目标文件拷贝器):** 复制和修改目标文件，例如移除符号表、添加段等。
			- **readelf (ELF 文件查看器):** 显示 ELF 文件的详细信息，例如文件头、段表、符号表等。
			- **size (文件大小查看器):** 显示目标文件或库文件中各个段的大小。
			- **strings (字符串提取器):** 从二进制文件中提取可打印字符串。
			- **strip (符号表移除工具):** 从目标文件或库文件中移除符号表信息。
		- 主要功能
			- **程序编译和链接:** Binutils 提供了 `ld` 和 `as` 等工具，是程序编译和链接过程中不可或缺的组件。
			- **目标文件分析和调试:** `objdump`、`readelf`、`nm` 等工具可以帮助开发者分析目标文件的结构、符号表信息，辅助程序调试和优化。
			- **库文件管理:** `ar` 工具用于创建和管理静态库文件，方便代码复用。
			- **二进制文件操作:** `objcopy`、`strip` 等工具可以对二进制文件进行各种操作，例如移除调试信息、压缩文件大小等。
	- "决定启用或禁用一些软件特性" 是软件配置过程中的一个重要环节，它可以根据不同的需求和环境，灵活地调整软件的行为和功能，以达到最佳的使用效果。
		- 如：**GCC 编译器的优化选项**、**Glibc 的线程支持**
		- 由于 GCC 和 Glibc 的配置脚本都需要测试汇编器和链接器的功能，因此 **必须先安装 Binutils**，才能保证配置脚本能够正常运行。
		- 如果在没有安装 Binutils 的情况下运行配置脚本，可能会导致配置失败，或者构建出的 GCC 和 Glibc 无法正常工作。
- Binutils 将汇编器和链接器安装在两个位置，⼀个是 $LFS/tools/bin，另⼀个是 $LFS/tools/$LFS_TGT/ bin。这两个位置中的⼯具**互为硬链接**。
	- 链接器的⼀项重要属性是它搜索库的顺序，通过向 ld 命令加⼊ - -verbose 参数，可以得到关于搜索路径的详细信息。例如，ld --verbose | grep SEARCH 会输出当前 的搜索路径及其顺序。
- 下⼀步安装 GCC。在执⾏它的 configure 脚本时，您会看到类似下⾯这样的输出：
	```bash
	checking what assembler to use... /tools/i686-lfs-linux-gnu/bin/as
	checking what linker to use... /mnt/lfs/tools/i686-lfs-linux-gnu/bin/ld
	```
	- 基于我们上⾯论述的原因，这些输出⾮常重要。这也说明 gcc 的配置脚本没有在 PATH 变量指定的⽬录 中搜索⼯具。
		- 这两行信息表明，GCC 的配置脚本 **成功找到了** 我们之前安装在 `/tools/i686-lfs-linux-gnu/bin` 目录下的汇编器 (`as`) 和链接器 (`ld`)。
		- 证明了 GCC 的配置脚本 **没有** 从系统的默认路径 (`/usr/bin`, `/bin` 等) 中找到汇编器和链接器。
		- 表明我们设置的 `LFS_TGT` 变量和交叉编译环境 **生效了**，GCC 正在使用我们指定的工具链。
	- 然⽽，在 gcc 的实际运⾏中，未必会使⽤同样的搜索路径。
		- 这段话最后提醒我们，**GCC 在实际运行时，并不一定会使用与配置脚本相同的搜索路径**。
		- 这意味着，即使 GCC 的配置脚本找到了正确的工具链，我们仍然需要 **确保 GCC 在编译代码时，能够找到 LFS 系统中的库文件和头文件**。
	- 为了查询 gcc 会使⽤哪个链接 器，需要执⾏以下命令：`$LFS_TGT-gcc -print-prog-name=ld`。
		- 所有在已构建出的系统上的命令需要如上运行
		- 在以lfs用户身份执行的时候要去掉前面的‘$LFS_TGT-’
	- 通过向 gcc 传递 -v 参数，可以知道在编译程序时发⽣的细节。例如，$LFS_TGT-gcc -v example.c (如果 在复习这⾥的内容，可能需要移除 $LFS_TGT) 会输出预处理、编译和汇编阶段中的详细信息，包括 gcc 的包含⽂件搜索路径和顺序。
- 下⼀个步骤是：安装“净化的” (sanitized) Linux API 头⽂件。这些头⽂件允许 C 标准库 (glibc) 与 Linux 内核提供的各种特性交互。
- 下⼀步安装 Glibc。在构建 Glibc 时需要着重考虑编译器，⼆进制⼯具，以及内核头⽂件。
	- 编译器⼀般不 成问题，Glibc 总是使⽤传递给配置脚本的 --host 参数相关的编译器。
		- 例如，在我们的例⼦中，使⽤的 编译器是 \$LFS_TGT-gcc。
	- 但⼆进制⼯具和内核头⽂件的问题⽐较复杂。
		- 我们为了安全起⻅，使⽤配置 脚本提供的开关以确保正确选择。
		- 在 configure 脚本运⾏完成后，可以检查 build ⽬录中的 config.make ⽂件，了解全部重要的细节。
		- 注意参数 CC="$LFS_TGT-gcc" (其中 $LFS_TGT 会被展开) 控制构建系统使 ⽤正确的⼆进制⼯具，⽽参数 -nostdinc 和 -isystem 控制编译器的包含⽂件搜索路径。这些事项凸显了 Glibc 软件包的⼀个重要性质 ⸺ 它的构建机制是相当⾃给⾃⾜的，通常不依赖于⼯具链默认值。
			- "工具链的默认值" 指的是系统在编译和链接程序时，默认使用的编译器、链接器、库文件和其他工具的路径。
			- Glibc 的构建过程会显式地指定所需的编译器、链接器、头文件等，而不会依赖于系统的默认设置。
			- **`config.make` 文件的作用：**
				- 在运行完 Glibc 的配置脚本后，会在构建目录中生成一个 `config.make` 文件。
				- 这个文件记录了 Glibc 构建过程中的各种配置信息，包括使用的编译器、链接器、头文件目录等。
				- 您可以检查 `config.make` 文件，确认 Glibc 是否使用了正确的工具链和头文件。
- 正如前⽂所述，接下来构建 C++ 标准库，然后是第 6 章中的其他程序，必须交叉编译这些程序才能打破 构建时的循环依赖。
	- 在安装这些软件包时使⽤ DESTDIR 变量，以确保将它们安装到 LFS ⽂件系统中。
		- DESTDIR到现在还未设置
- 在第 6 章的末尾，构建 LFS 本地编译器。
	- ⾸先使⽤和其他程序相同的 DESTDIR 第⼆次构建 binutils
		- **这样做是为了确保 LFS 系统中的所有工具链组件都使用相同的编译器和配置选项构建，保持系统的一致性和稳定性。**
		- 第一次构建 Binutils 是在 LFS 构建过程的**第五章，也就是构建临时工具链的时候**。
			- **第一次构建 Binutils 的目的，就是使用宿主机的交叉编译器，构建出能够在 LFS 目标系统上运行的 Binutils 工具集。**
			- 这就像是在 "借用" 宿主机的资源，为 LFS 系统打造第一批可用的工具。
			- 之后，我们就可以使用这些 LFS 可用的工具，进一步构建 GCC、Glibc 等更复杂的组件，逐步摆脱对宿主机的依赖，最终构建出一个完整的、独立的 LFS 系统。
		- 为什么要二次构建？
			- 第一次构建的 Binutils 是用宿主机的交叉编译器构建的，它虽然可以在 LFS 目标系统上运行，但**它本身依赖于宿主系统的一些库文件。**
			- 为了构建一个完全独立的 LFS 系统，我们需要一个完全由 LFS 系统自身构建的工具链，包括 Binutils、GCC 和 Glibc。
			-  **第二次构建 Binutils，就是为了使用 LFS 系统中已经构建好的本地 GCC 编译器，生成完全不依赖于宿主系统的 Binutils 工具集。**
			- 第一次构建的 Binutils 是为通用目标系统构建的，它可能没有针对 LFS 系统进行特定的优化。
			- **第二次构建 Binutils 时，可以使用 LFS 系统的特定配置选项，例如 CPU 架构、优化级别等，生成性能更优的工具集。**
			- 虽然第二次构建 Binutils 看起来像是在 "重复造轮子"，但它对于构建一个完全独立、性能优化、并且内部一致的 LFS 系统至关重要。 这体现了 LFS 构建过程的精益求精和对系统底层的掌控力
	- 然 后第⼆次构建 GCC，构建时忽略⼀些不重要的库。
	- 由于 GCC 配置脚本的⼀些奇怪逻辑，CC_FOR_TARGET 变量在 host 系统和 target 相同，但与 build 不同时，被设定为 cc。因此我们必须显式地在配置选项中指定 CC_FOR_TARGET=$LFS_TGT-gcc。
- 在第 7 章中，进⼊ chroot 环境后，临时性地安装⼯具链的正常⼯作所必须的程序。此后，核⼼⼯具链 成为⾃包含的本地⼯具链。
- 在第 8 章中，构建，测试，并最终安装所有软件包，它们组成功能完整的系统。
- 注意事项：
	- 某些软件包在编译前需要打补丁，然⽽补丁只在绕过特定问题时才需要。补丁常常在本章和下⼀章都 要使⽤，但有时，对于多次构建的软件包，在初次构建时可能并不需要补丁。因此，如果发现本书 给出的步骤中没有使⽤某个下载好的补丁，这是正常的，不必担⼼。在应⽤补丁时可能会出现关于 offset 或者 fuzz 的警告信息。不⽤担⼼这些警告，补丁还是会成功应⽤到源码上的。
	- 在编译⼤多数软件包时，屏幕上都会出现⼀些警告。这是正常的，可以放⼼地忽略。这些警告就像它 们描述的那样，是关于⼀些过时的，但并不是错误的 C 或 C++ 语法。C 标准经常改变，⼀些软件包仍 然在使⽤旧的标准。这并不是⼀个严重的问题，但确实会触发警告。
	- 最后确认 LFS 环境变量是否配置正确：`echo $LFS`，确认上述命令输出 LFS 分区挂载点的路径，如果使⽤了本书的例⼦，就是 /mnt/lfs。
- 最后强调两个重要事项：
- 本书中的命令假设宿主系统需求中的所有内容，包括符号链接，都被正确设置
	- bash 是正在使⽤的 shell。
	- sh 是指向 bash 的符号链接。
		- `dash`是一个精简、快速、符合 POSIX 标准的 shell，相比 `bash` 更加轻量级。
			- **轻量级:** `dash` 的代码量和内存占用都比 `bash` 小很多，因此启动速度更快，也更节省系统资源。
			- **快速:** 由于代码精简，`dash` 在执行脚本时通常比 `bash` 更快。
			- **符合 POSIX 标准:** `dash` 严格遵循 POSIX shell 标准，因此编写的脚本具有更好的可移植性，可以在其他符合 POSIX 标准的系统上运行。
			- Ubuntu/Debian使用dash作为/bin/sh原因
				- **速度和效率:** `dash` 的轻量级和快速执行速度使其成为默认 `/bin/sh` 解释器的理想选择，尤其是在系统启动和执行系统脚本时，可以提高效率。
				- **安全性:** 由于 `dash` 功能精简，攻击面更小，相对更安全。
	- /usr/bin/awk 是指向 gawk 的符号链接。
	- /usr/bin/yacc 是指向 bison 的符号链接，或者⼀个执⾏ bison 的⼩脚本。
```bash
echo $SHELL
# /bin/bash

# there have a problem about the symbolic '/bin/sh' be pointted to the dash, not the bash
ls -l /bin/sh
# lrwxrwxrwx 1 root root 4 Nov  3  2021 /bin/sh -> dash

ls -l /usr/bin/awk
# lrwxrwxrwx 1 root root 21 Nov 22  2021 /usr/bin/awk -> /etc/alternatives/awk

ls -l /usr/bin/yacc
# lrwxrwxrwx 1 root root 22 Jul 22 08:38 /usr/bin/yacc -> /etc/alternatives/yacc

ls -l /etc/alternatives/awk
# lrwxrwxrwx 1 root root 13 Jul 22 08:46 /etc/alternatives/awk -> /usr/bin/gawk

ls -l /etc/alternatives/yacc
# lrwxrwxrwx 1 root root 19 Jul 22 08:38 /etc/alternatives/yacc -> /usr/bin/bison.yacc
```
- 下⾯给出软件包构建过程的概要。
	1. 把所有的源码包和补丁放在⼀个能够从 chroot 环境访问的⽬录，例如 /mnt/lfs/ sources/。
	2. 切换到 /mnt/lfs/sources/ ⽬录。 
	3. 对于每个软件包：
		- a. 使⽤ tar 程序，解压需要构建的软件包。在第 5 章和第 6 章中解压软件包时，确认您以 ⽤⼾ lfs 的⾝份登录。 除了使⽤ tar 命令解压源码包外，不要使⽤其他任何将源代码⽬录树置⼊⼯作⽬录的⽅ 法。特别需要注意的是，使⽤ cp -R 从其他位置复制源代码⽬录树会破坏其中的链接和 时间戳，并导致构建失败。
		- b. 切换到解压源码包时产⽣的⽬录。
		- c. 根据指⽰构建软件包。
		- d. 构建完成后，切换回包含所有源码包的⽬录。 e. 除⾮另有说明，删除解压出来的⽬录。
