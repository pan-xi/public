### 问题/待确认/可优化
- （bash脚本的定义均为第一次出现的地方）
- [ ] as_echo_n_body定义处的疑问：为什么要打印两次？
	- 目前认为是“谜之操作”，就很鬼，可能下文会有用？或者是维护不好（应该不可能）？
- [ ] 关于PATH_SEPARATOR的定义的优化
	- 这里逻辑可以优化，将判断冒号的给优化掉，因为既然都已经判断处用的是分号，就没必要再判断冒号了呀
- [ ] 在定义脚本路径的地方，case分支的第二个分支
	- 每次执行for循环分支都要执行`IFS=$as_save_IFS `，其实整体逻辑可以调整一下，先分隔，再用for去处理，分隔后的结果可以存储在一个字符串中，并用空格进行分隔（保证对老旧版本的兼容性，老版本可能不支持数组），然后再执行`IFS=$as_save_IFS `和for循环
	- 在for循环结束后再次执行`IFS=$as_save_IFS `，这一步也可以优化掉
	- [ ] 在“一大段if的开始”这一部分，这部分的工作是假如当前shell不能满足`as_required`或`as_suggested`(需要和建议)，同样也是在每次执行for循环时要执行`IFS=$as_save_IFS `，其实可以优化一下
- [ ] 在`BASH_ENV=/dev/null  `这里，搞不懂为什么前面的for循环已经unset了，这里还要再设置成/dev/null后再一次unset
	- 有可能是为了清空重新启动指定的shell的环境变量，但是他也没有显式地使用`BASH_ENV`，也没有export
	- gemini解释地非常牵强！！！
- [ ] `as_fn_exit`没被定义就直接使用了？？
- [ ] 目前卡在了这段代码的分隔，太坑了，在服务器运行都运行不明白
	- [ ] 现在分行是搞明白了，但是还是调试不明白
```bash
  as_suggested="  as_lineno_1=";
  
  as_suggested=$as_suggested$LINENO;
  
  as_suggested=$as_suggested" as_lineno_1a=\$LINENO   as_lineno_2=";
  
  as_suggested=$as_suggested$LINENO;
  
  as_suggested=$as_suggested" as_lineno_2a=\$LINENO  
  eval 'test \"x\$as_lineno_1'\$as_run'\" != \"x\$as_lineno_2'\$as_run'\" &&  
  test \"x\`expr \$as_lineno_1'\$as_run' + 1\`\" = \"x\$as_lineno_2'\$as_run'\"' || exit 1  test \$(( 1 + 1 )) = 2 || exit 1"  

```
- [ ] 逻辑有问题，在寻找可用的shell的时候（设置CONFIG_SHELL，很长一段if），如果一开始测试as_required和as_suggested不成功，且没有搜索成功，还将换回当前shell
	- [ ] 这里有个问题：之前测试当前shell不成功，这里又设置回去了
- [ ] 在测试mkdir的时候（as_fn_mkdir_p ()  内） 逻辑有误，空执行 eval $as_mkdir_p，这肯定是false啊
- [ ] 阅读策略有点问题，但是不知道如何解决这个问题，比如前面一直在定义环境和函数，定义环境没啥问题，但是定义函数，函数是后面要执行的，所以函数内涉及的变量可能是后面定义的，就导致很烦
### 总览
- Guess values for system-dependent variables and create Makefiles.
	- 这个文件看起来像是用于配置软件构建系统的脚本， 它会检查系统环境和软件依赖， 然后生成用于编译软件的 Makefile 文件
	- **检查系统环境**: 例如操作系统类型、 系统架构、 重要的系统头文件和库文件是否存在等。
	- **查找依赖的软件**: 例如编译器、 链接器、 特定版本的库文件等， 并检查版本是否满足要求。
	- **设置编译选项**: 根据系统环境和用户指定的选项， 设置编译器和链接器的参数， 例如优化级别、 调试信息等。
	- **生成 Makefile 文件**: 根据前面的检查和设置， 生成 Makefile 文件， 这个文件包含了编译软件所需的规则和依赖关系。
- 目标，写下输出信息，并与打印比对
- `case $arg in #(  *"$as_nl"*)  `
	- 巨坑！！！
	- 在case语句中`#(`代表开启扩展正则的模式匹配，在这个模式下\*代表通配符，就像在shell终端里的`rm -rf *`一样，不再是普通正则里的`a*`代表“任意个”的意思
- 哈哈哈哈哈，吐槽sed语法，好！很有精神！
	- ` # Blame Lee E. McMahon (1931-1989) for sed's syntax.  :-) `

### 变量记录
#### 没用的
- `DUALCASE=1; export DUALCASE`
	- 当 `DUALCASE` 被设置为 `1` 并导出为环境变量时， `MKS sh` 就能识别大小写字母。
- `alias -g '${1+"$@"}'='"$@"'`
	- 解决Zsh的兼容性
- `NULLCMD=:  `设置空命令
- `setopt NO_GLOB_SUBST`
	- 禁止Zsh自动扩展参数或变量
- 打印的变量
	- 不是bash或Zsh的终端，但是内置print
		- as_echo='print -r --'  
		- as_echo_n='print -rn --' 
	- 没有内置打印函数的，使用expr
		- as_echo_n='sh -c $as_echo_n_body as_echo'  
		- as_echo='sh -c $as_echo_body as_echo'  
- windows的系统环境
	- PATH_SEPARATOR=；
- `CONFIG_SHELL`，使用指定的路径下的shell重新运行脚本
- `as_opt`，与CONFIG_SHELL搭配使用，用于指定shell的操作选项
- `_as_can_reexec`，指明脚本是否存在递归调用，影响了脚本是否可以被CONFIG_SHELL重新执行
- `as_bourne_compatible`，让当前shell兼容Bourne shell的语法，主要针对Zsh，与上面操作类似，解决传参兼容性和POSIX兼容模式
#### 与我环境相关的
- 打印调用的函数
	- as_echo='printf %s\\n'  
	- as_echo_n='printf %s'
	- printf函数内置在（大多发行版）shell里
- 系统环境(分隔符)：`PATH_SEPARATOR=:`
- `IFS=" ""    $as_nl"  `：指定参数分隔符
```bash 
# 主提示符
PS1='$ '  
# 次提示符，多行时候使用
PS2='> '  
# 调试提示符，在 shell 跟踪调试模式下， 在每个命令执行前显示 + 和一个空格
PS4='+ '  
```
- `as_myself='/mnt/lfs/sources/binutils-2.42/configure'`存储的是脚本自身完整路径
- 取消`CDPATH`，尽管我就没有这个变量，但是执行这个操作
- `LC_ALL=C; LANGUAGE=C`
- 解决Zsh兼容性问题（用不到）
```bash
  as_bourne_compatible="if test -n \"\${ZSH_VERSION+set}\" && (emulate sh) >/dev/null 2>&1; then :  
  emulate sh  
  NULLCMD=:  
  # Pre-4.2 versions of Zsh do word splitting on \${1+\"\$@\"}, which  
  # is contrary to our usage.  Disable this feature.  
  alias -g '\${1+\"\$@\"}'='\"\$@\"'  
  setopt NO_GLOB_SUBST  
else  
  case \`(set -o) 2>/dev/null\` in #(  
  *posix*) :  
  # 确保 POSIX 兼容模式被明确启用
    set -o posix ;; #(  
  *) :  
     ;;  
esac  
fi  
"  
```
- `as_required`定义了一些必要的shell函数，主要包括函数调用返回值、根据函数执行状态做返回值
```bash
  as_required="as_fn_return () { (exit \$1); }  
as_fn_success () { as_fn_return 0; }  
as_fn_failure () { as_fn_return 1; }  
as_fn_ret_success () { return 0; }  
as_fn_ret_failure () { return 1; }  
  
exitcode=0  
# 注意此处的逻辑运算的逻辑，当前面的命令运行成功，退出码为0，但是是“执行成功”，所以用 || 表示不执行右边的命令
as_fn_success || { exitcode=1; echo as_fn_success failed.; }  
as_fn_failure && { exitcode=1; echo as_fn_failure succeeded.; }  
as_fn_ret_success || { exitcode=1; echo as_fn_ret_success failed.; }  
as_fn_ret_failure && { exitcode=1; echo as_fn_ret_failure succeeded.; }  
# 测试shell的保存位置参数的能力是否正常（虽然我感觉对现在来说没什么作用，就没遇到过shell不能正确保存参数的）
	# 测试保存位置参数的能力对于 shell 命令的执行至关重要，因为它关系到脚本能否正确处理和传递参数，从而影响到脚本的正确性和功能。
if ( set x; as_fn_ret_success y && test x = \"\$1\" ); then :  
  
else  
  exitcode=1; echo positional parameters were not saved.  
fi  
test x\$exitcode = x0 || exit 1  
# 检查根目录 / 是否存在并且可执行。如果根目录不存在或不可执行，则表示系统环境存在问题，脚本会退出并返回退出码 1
test -x / || exit 1"  
```
- `as_suggested`定义了一些shell功能测试，包括是否支持基本的行号变量LINENO、算术表达式、是否正确执行eval命令
- `CLICOLOR_FORCE= GREP_OPTIONS=  `
	- `CLICOLOR_FORCE`  变量通常用于强制在终端输出彩色文本。
	- `GREP_OPTIONS` 变量用于设置 grep 命令的默认选项。
- `as_unset=as_fn_unset `，`as_fn_unset()`：以一种可移植的方式取消设置变量 
- `as_fn_set_status () `：将函数的第一个参数（`$1`）作为返回值返回
- `as_fn_exit ()  `:返回错误码，即使是在trap 0或set -e的情况下也可以生效，方便处理错误码
- `as_fn_mkdir_p () `：实现`mkdir -p`的效果
- `as_fn_executable_p ()  `：测试指定文件是否为可执行的常规文件
- `as_fn_append () `：字符串追加操作定义
- `as_fn_arith () `: 算术运算定义
- `as_fn_error () `：将错误信息写入指定的日志文件，并使用指定的状态码调用`as_fn_exit`来退出脚本
- `as_expr=expr ; as_basename=basename; as_dirname=dirname `：expr（正则表达匹配）是否可用、basename（用于shell脚本中提取自身文件名）是否可用、dirname（获取父目录）是否可用
- `as_me`：存储当前脚本名称
### 代码
```bash
#! /bin/sh  
# Guess values for system-dependent variables and create Makefiles.  
# Generated by GNU Autoconf 2.69.  
#  
#  
# Copyright (C) 1992-1996, 1998-2012 Free Software Foundation, Inc.  
#  
#  
# This configure script is free software; the Free Software Foundation  
# gives unlimited permission to copy, distribute and modify it.  
## -------------------- ##  
## M4sh Initialization. ##  
## -------------------- ##  

# 增强脚本对不同 shell 的兼容性， 特别是让它在 Bourne shell 和其兼容 shell 中能够正常运行
# Be more Bourne compatible  
#  MKS sh 是一种在 Windows 系统上模拟 Unix 环境的 shell，这两句是为了兼容 MKS sh。
#  MKS sh 在处理 ${1+"$@"} 这种语法时，与 Bourne shell 的行为不一致。为了避免这种不一致导致脚本运行错误，需要针对 MKS sh 做特殊处理。
# 当 DUALCASE 被设置为 1 并导出为环境变量时， MKS sh 就能识别大小写字母。
DUALCASE=1; export DUALCASE # for MKS sh 


# 判断当前 shell 是否为 Zsh 并进行相应处理
# Zsh 是一款强大的 Unix shell，它既可以作为交互式登录 shell，也可以作为脚本解释器。 兼容 Bash
# (emulate sh)  尝试在 Zsh 中启用 sh 的仿真模式
# “:” 是一个空命令， 表示如果条件成立， 则不执行任何操作。
# 一般情况下then 后面是不加":"的，但是“：”可以增加代码可读性，显示地说明，其实这个分支不执行任何操作
# 无输出
if test -n "${ZSH_VERSION+set}" && (emulate sh) >/dev/null 2>&1; then :  
	# 在 Zsh 中启用 `sh` 的仿真模式， 以确保脚本中的语法和行为与 Bourne shell 保持一致。
	  emulate sh  
	  # 定义一个名为 NULLCMD 的变量， 并将其值设为 :, 这是为了在 Zsh 中定义一个空命令（就是什么都不执行地一个命令，跟上面的then：相同）的别名。
	  # 每个bash或shell都应该有自己的空命令
	  NULLCMD=:  
	  # Pre-4.2 versions of Zsh do word splitting on ${1+"$@"}, which  
	  # is contrary to our usage.  Disable this feature.  
	  # 处理早期的zsh中的问题，还是兼容性问题
		  # $@：会获取所有参数，并将每个参数用双引号括起来（只有一个str）。
		  # ${1+"$@"}：早期的zsh会将参数拆成单个参数（有好多个str，即便是一个str也会拆成多个），同时如果第一个参数为空，整个${1+"$@"}将什么都不输出
		  # 将所有参数作为一个整体的话，就允许参数内有特殊符号，如文件名允许存在空格，同时作为一个整体，方便传输个另外一个命令或函数
	  alias -g '${1+"$@"}'='"$@"'  
	  # 禁用 Zsh 的文件名扩展功能， 即禁止将 *、 ? 等通配符替换为匹配的文件名， 以确保脚本的行为与 Bourne shell 一致。
	  # 在一些情况下， 我们希望 shell 不要自动进行通配符替换， 而是将通配符原样传递给程序处理
		  # 传递参数给程序: 如果要将通配符作为参数传递给某个程序， 就需要禁止通配符替换， 否则程序接收到的参数就不是我们想要的。
		  # 进行字符串比较: 如果要对包含通配符的字符串进行比较， 就需要禁止通配符替换， 否则比较结果可能会不准确。
	  setopt NO_GLOB_SUBST  
else  
	# 判断当前 shell 是否支持 posix 模式， 如果是， 则启用 posix 模式
	# 无输出，``会捕获命令的输出，$() 是 `` 的 现代版本，结果会被捕获而不会打印在终端
		# $()在 POSIX 标准中被推荐使用，可读性更强、不用转义、可移植性更好
	case `(set -o) 2>/dev/null` in #(  
	# 如果输出中包含 `posix` 字符串， 则表示当前 shell 支持 `posix` 模式
	  *posix*) :  
	  # 启用 `posix` 模式
    set -o posix ;; #(  
# 如果不支持，什么都不做
	  *) :  
     ;;  
esac  
fi  
  



# 解决在不同 shell 和操作系统中， 打印输出的兼容性问题， 特别是针对 Solaris 7 系统上 /usr/bin/printf 命令的bug。
# 设置一个换行符
as_nl='  
'  
export as_nl  
# 在 Solaris 7 系统上， 使用 /usr/bin/printf 命令打印长字符串会导致程序崩溃。
# Printing a long string crashes Solaris 7 /usr/bin/printf.  
# 定义了一个非常长的字符串， 并将其赋值给 as_echo 变量。 这个长字符串是由多个连续的反斜杠字符组成的， 其目的是为了测试 /usr/bin/printf 命令的崩溃问题。(这么多斜杠，调皮hhh)
as_echo=\'\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\'  
as_echo=$as_echo$as_echo$as_echo$as_echo$as_echo  
as_echo=$as_echo$as_echo$as_echo$as_echo$as_echo$as_echo  
# Prefer a ksh shell builtin over an external printf program on Solaris,  
# but without wasting forks for bash or zsh.  
# 为了避免在 Solaris 系统上使用 /usr/bin/printf 命令导致程序崩溃， 这段代码会优先使用 ksh shell 内置的命令来进行输出， 同时避免在 bash 或 zsh 中创建不必要的子进程
# 首先判断当前 shell 是否为 ksh， 如果是， 则使用 ksh 内置的 print 命令来进行输出
# test -z: 测试字符串是否为空
# 如果这两个变量($BASH_VERSION$ZSH_VERSION)都没有定义， 则认为当前 shell 不是 bash 或 zsh
# 尝试使用 print -r -- 命令输出 as_echo 变量的值， 并将其与 X$as_echo 进行比较。 如果两者相等， 则说明 print -r --` 命令可以正常工作
if test -z "$BASH_VERSION$ZSH_VERSION" \  
    && (test "X`print -r -- $as_echo`" = "X$as_echo") 2>/dev/null; then  
    #如果条件成立， 则将 `as_echo` 设置为 `print -r --`， 将 `as_echo_n` 设置为 `print -rn --`
	as_echo='print -r --'  
	as_echo_n='print -rn --'  
# 如果当前 shell 不是 ksh， 则尝试使用 printf 命令进行输出
# 尝试使用 printf %s 命令输出 as_echo 变量的值， 并将其与 X$as_echo 进行比较。 如果两者相等， 则说明 printf %s` 命令可以正常工作
elif (test "X`printf %s $as_echo`" = "X$as_echo") 2>/dev/null; then  
	as_echo='printf %s\n'  
	as_echo_n='printf %s'  
# 如果 print -r -- 和 printf %s 都不能正常工作， 则使用 /usr/ucb/echo 命令进行输出
# /usr/ucb 目录在一些Unix系统中是存在的，主要包含BSD系列的Unix的软件和工具
else  
# 首先测试 /usr/ucb/echo -n -n 命令是否可以正常工作
# 某些版本的 echo 命令， 特别是早期的 Unix 系统和一些 BSD 系统， 会将第一个 -n 选项解释为禁止输出结尾的换行符， 而将第二个 -n 选项解释为要输出的字符串的一部分。
  if test "X`(/usr/ucb/echo -n -n $as_echo) 2>/dev/null`" = "X-n $as_echo"; then  
	  # eval：读取参数， 将其组合成一个新的命令， 然后执行该命令
    as_echo_body='eval /usr/ucb/echo -n "$1$as_nl"'  
    as_echo_n='/usr/ucb/echo -n'  
  else  
	  # 如果 `/usr/ucb/echo -n -n` 也不能正常工作， 则使用 `expr` 命令来实现 `echo` 的功能
	  # expr 是一个命令行工具， 用于求表达式的值。 它可以处理数字和字符串表达式， 并支持各种运算符，如算术运算、关系运算、字符串运算等
	  # 去除字符串开头和结尾的换行符
	  # expr 命令只会将处理结果“发送到终端”以供管道或其它变量使用，默认情况下不执行打印操作
	  # 命令分类
		  # 先说明一下发送终端
			  # 终端 terminal： 用户与计算机系统进行交互的设备， 通常指的是命令行界面。"发送到终端" 指的是将数据发送到 控制终端设备
			  # 1、命令将输出数据写入到标准输出 stdout 文件描述符。2、操作系统内核将 stdout 重定向到终端设备。3、终端设备将接收到的数据显示在屏幕上。
			  # 所以shell或bash不能说是终端
		# 不发送终端：不发送终端的一定不答应，比如重定向、管道传递命令、发送到网络
		# 发送终端：（呈现结果来划分，并无系统概念）分为打印和不打印
			# 打印：ls、echo之类的
			# 不打印指的是不输出到终端，如cd（发送新位置到终端，改变提示符，不打印），export（发送环境变量到终端，不打印）
	  # 匹配任意字符零次或多次， 并将匹配到的内容保存到一个捕获组中
    as_echo_body='eval expr "X$1" : "X\\(.*\\)"'  
    
    as_echo_n_body='eval  
      arg=$1;  
      # 巨坑啊！！！
      case $arg in #(  
      *"$as_nl"*)  
    expr "X$arg" : "X\\(.*\\)$as_nl";  
    arg=`expr "X$arg" : ".*$as_nl\\(.*\\)"`;;  
      esac;  
      expr "X$arg" : "X\\(.*\\)" | tr -d "$as_nl"  
    '  
    # 这里有两点没搞明白：1、为什么as_echo_n的参数是固定的？2、为什么只export as_echo_n_body和as_echo_body
    # 第二个问题的解答：因为as_echo_n_body和as_echo_body都是采用"sh-c"的方式执行，sh -c会创建新的shell，子进程继承父进程环境变量而非普通变量，所以需要export
    # 第一个问题的尝试解答：根据搜索到的下文来看，如果走这个分支，每次打印的情况是先答应一串“/////”，再打印其他内容，同时as_echo_n（as_echo_n_body）会打印信息打印两次，第一次是expr，第二次是expr… | tr，就很奇怪这个设计，为什么要打印两次
    export as_echo_n_body  
    as_echo_n='sh -c $as_echo_n_body as_echo'  
  fi  
  export as_echo_body  
  as_echo='sh -c $as_echo_body as_echo'  
fi  




# 好！哈哈哈
# The user is always right.  
# 如果PATH_SEPARATOR没有，就设置一下
# ${PATH_SEPARATOR+set} 是一个特殊的变量扩展语法， 如果 PATH_SEPARATOR 变量已经被设置， 则它的值会被替换为 set， 否则会被替换为空字符串
# 使用 ${PATH_SEPARATOR+set} 这种方式的主要目的是为了保持代码的严谨性和可移植性。
	# 区分未设置和空值，避免语法错误（老的shell可能不支持直接使用未设置的变量）
# 这里的逻辑有点绕
	# 一般情况下，类Unix用冒号，windows用分号
	# 先是将PATH_SEPARATOR设置为 :冒号，默认为类Unix，然后尝试windows行不行，行的话设置成分号，因此我认为这里逻辑可以优化，将判断分号的给优化掉
if test "${PATH_SEPARATOR+set}" != set; then  
  PATH_SEPARATOR=:  
  (PATH='/bin;/bin'; FPATH=$PATH; sh -c :) >/dev/null 2>&1 && {  
    (PATH='/bin:/bin'; FPATH=$PATH; sh -c :) >/dev/null 2>&1 ||  
      PATH_SEPARATOR=';'  
  }  
fi  
  
  
# IFS  
# We need space, tab and new line, in precisely that order.  Quoting is  
# there to prevent editors from complaining about space-tab.  
# (If _AS_PATH_WALK were called with IFS unset, it would disable word  
# splitting by setting IFS to empty value.)  
# IFS InternalFieldSeparator 是 shell 中的一个特殊变量，用于指定单词分隔符。
# 将 IFS 设置为空格、制表符和换行符，以便在后续的代码中能够正确地处理路径中的空格等特殊字符。
# 应该是规定好的格式，是用字符串传参，将空格和制表符分开，制表符代表4个空格，不分开的话不能区分
IFS=" ""    $as_nl"  

# 找到脚本自身所在的完整路径并存储在 as_myself 变量中，如果找不到则会报错并退出
# Find who we are.  Look in the path if we contain no directory separator.  
as_myself=  
# 判断脚本名 $0 中是否包含路径分隔符（/ 或 \\）
case $0 in #((  
  *[\\/]* ) as_myself=$0 ;;  
  # 保存当前的 IFS 值，并将 IFS 设置为路径分隔符，以便正确地分割 PATH 变量。
  *) as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
# 与上面的IFS=$PATH_SEPARATOR结合起来，for语句只对PATH分隔一次，分割出所有的“对象”
for as_dir in $PATH  
do  
# 将IFS改回来，但是注意这里的IFS有作用域的问题！！！
  IFS=$as_save_IFS  
  #  如果目录名为空， 则设置为当前目录 .
  test -z "$as_dir" && as_dir=.  
  # 如果在当前目录下找到了脚本文件 (`-r` 表示文件可读)， 则将完整路径赋值给 `as_myself`， 并跳出循环。
    test -r "$as_dir/$0" && as_myself=$as_dir/$0 && break  
  done  
IFS=$as_save_IFS  
  
     ;;  
esac  
# We did not find ourselves, most probably we were run as `sh COMMAND'  
# in which case we are not to be found in the path.  
# 如果在 PATH 中没有找到脚本， 则 as_myself 仍然为空字符串。 这时将 as_myself 设置为脚本名 $0， 这在某些情况下可能是相对路径。
if test "x$as_myself" = x; then  
  as_myself=$0  
fi  
# 确定脚本自身的完整路径，以便脚本可以引用自身或其所在的目录，不存在就报错，退出
if test ! -f "$as_myself"; then  
  $as_echo "$as_myself: error: cannot find myself; rerun with an absolute file name" >&2  
  exit 1  
fi  




# 为了避免潜在问题， 尝试取消设置一些可能导致错误的环境变量， 并设置 shell 提示符
# 某些环境变量（如 BASH_ENV， ENV， MAIL， MAILPATH ）在某些旧版本的 ksh 中会导致问题。
# Unset variables that we do not need and which cause bugs (e.g. in  
# pre-3.0 UWIN ksh).  But do not cause bugs in bash 2.01; the "|| exit 1"  
# suppresses any "Segmentation fault" message there.  '((' could  
# trigger a bug in pdksh 5.2.14.  
for as_var in BASH_ENV ENV MAIL MAILPATH  
# eval test x\\${$as_var+set} = xset 这部分检查变量是否已设置。
do eval test x\${$as_var+set} = xset \  
#  如果变量已设置， 则尝试取消设置它， 如果取消设置失败则退出脚本。 输出被重定向到 /dev/null， 以保持脚本输出的干净。
# 在某些 shell 环境下， 第一次 unset 可能只是禁用了变量， 而没有真正删除它， 因此需要执行第二次 unset 来确保变量被完全删除。
# 如果变量没被设置，或第二次取消设置失败就执行空命令
  && ( (unset $as_var) || exit 1) >/dev/null 2>&1 && unset $as_var || :  
done  
# 主提示符
PS1='$ '  
# 次提示符，多行时候使用
PS2='> '  
# 调试提示符，在 shell 跟踪调试模式下， 在每个命令执行前显示 + 和一个空格
PS4='+ '  





#  NLS（National Language Support）是指软件对不同语言和地区的字符、日期、时间等格式的支持
# NLS nuisances.  
# 设置为 C 会强制使用默认的 POSIX 本地化设置， 这通常是英语， 并且不进行任何特定于语言环境的转换。
LC_ALL=C  
export LC_ALL  
# LANGUAGE 变量用于指定软件的语言环境， 例如用于选择消息的翻译。 将其设置为 C 通常意味着使用默认语言， 通常是英语。
LANGUAGE=C  
export LANGUAGE  



# CDPATH.  
# CDPATH 是一个用于指定 cd 命令搜索路径的环境变量， 如果设置不当， 可能会导致脚本在错误的目录中执行命令。
# 使用 cd 命令切换目录时，如果目标目录不是当前目录的子目录，Shell 就会在 CDPATH 环境变量指定的路径列表中搜索目标目录
# 为什么取消CDPATH
	# 取消CDPATH就限定cd仅在当前目录下搜索
	# CDPATH 可能会引入安全风险。 如果 CDPATH 包含用户主目录以外的目录， 攻击者可能会利用 CDPATH 诱导脚本在非预期目录中执行命令。 例如， 攻击者可能会在 CDPATH 指定的目录中创建一个与常用命令同名的恶意程序， 当脚本使用相对路径执行该命令时， 就会执行恶意程序。
	# 在编写可移植和安全的脚本时， 尽量减少对环境变量的依赖是一种最佳实践。 CDPATH 并不是一个标准的 POSIX 环境变量， 在某些 Shell 中可能不被支持。 取消 CDPATH 可以减少脚本对特定环境的依赖， 提高脚本的可移植性。
# 取消设置 CDPATH 环境变量
(unset CDPATH) >/dev/null 2>&1 && unset CDPATH  



# 这段不会执行，因为我没有指定CONFIG_SHELL
# 这段不会执行，因为我没有指定CONFIG_SHELL
# 这段不会执行，因为我没有指定CONFIG_SHELL
# Use a proper internal environment variable to ensure we don't fall  
  # into an infinite loop, continuously re-executing ourselves.  
# 使用 CONFIG_SHELL 指定的 shell 重新执行当前脚本，同时避免无限递归执行
# test x"${_as_can_reexec}" != xno: 检查 _as_can_reexec 环境变量的值是否不等于 no。 当 `_as_can_reexec` 的值不等于 `no` 时， 脚本会认为自身可以被重新执行
# test "x$CONFIG_SHELL" != x: 检查 CONFIG_SHELL 环境变量是否被设置。 CONFIG_SHELL 应该包含了用于重新执行脚本的 shell 路径。
# 使用指定的 shell 重新执行脚本是为了提高脚本的兼容性、 可移植性和可配置性。这表明脚本的设计者希望用户能够自定义运行脚本的 shell 环境。
  if test x"${_as_can_reexec}" != xno && test "x$CONFIG_SHELL" != x; then  
	  # 将 _as_can_reexec 环境变量设置为 no， 并将其导出。 这样一来， 如果脚本再次被递归调用， 这个条件判断语句就会失败， 从而避免了无限递归执行。
    _as_can_reexec=no; export _as_can_reexec;  
    # We cannot yet assume a decent shell, so we have to provide a  
# neutralization value for shells without unset; and this also  
# works around shells that cannot unset nonexistent variables.  
# Preserve -v and -x to the replacement shell.  
# 将 BASH_ENV 和 ENV 环境变量都设置为 /dev/null。 这两个环境变量都用于指定 shell 启动时要加载的配置文件， 将它们设置为空设备可以防止加载任何配置文件。
# （前面已经unset了）unset 命令的行为可能与预期不符。 因此， 再次将它们设置为 /dev/null 可以确保它们被清空， 避免潜在的问题。
BASH_ENV=/dev/null  
ENV=/dev/null  
# 不同的 shell 对环境变量的处理方式可能存在差异。 有些 shell 可能会忽略 /dev/null 设置， 而有些 shell 可能会将其解释为空字符串。 为了确保脚本在不同的 shell 环境中都能正常工作， 最好同时使用 unset 命令和 /dev/null 设置来清除环境变量。
# 这段代码可能是从早期版本演变而来的。 在早期版本中， 可能只使用了 unset 命令来清除环境变量， 而没有设置 /dev/null。 后来为了提高兼容性， 添加了 /dev/null 设置， 但 unset 命令被保留了下来。
(unset BASH_ENV) >/dev/null 2>&1 && unset BASH_ENV ENV  
# 根据当前 shell 的选项设置， 构造一个用于重新执行脚本的选项字符串 as_opts
# $- 是一个特殊的 shell 变量， 包含了当前 shell 的选项设置。
# -v verbose: 开启详细模式。当 Shell 执行脚本时，会将读取到的每一行命令都打印到标准输出，方便用户跟踪脚本的执行过程。
# -x xtrace: 开启调试模式。当 Shell 执行脚本时，除了打印每一行命令，还会将命令展开后的结果也打印出来，方便用户调试脚本中的错误。
case $- in # ((((  
# as_opts 只考虑 v 和 x 是为了在新 shell 中保留脚本调试相关的选项，方便进行调试
  *v*x* | *x*v* ) as_opts=-vx ;;  
  *v* ) as_opts=-v ;;  
  *x* ) as_opts=-x ;;  
  * ) as_opts= ;;  
esac  
# 使用 exec 命令启动新的 shell， 并执行当前脚本
# exec命令会用新的进程替换当前进程， 因此不会创建新的子 shell。$CONFIG_SHELL 是新的 shell 的路径。$as_opts 是传递给新 shell 的选项。"$as_myself" 是当前脚本的路径。{1+"$@"} 是传递给当前脚本的参数。
exec $CONFIG_SHELL $as_opts "$as_myself" ${1+"$@"}  
# Admittedly, this is quite paranoid, since all the known shells bail  
# out after a failed `exec'.  
# 处理 exec 命令执行失败的情况。
$as_echo "$0: could not re-execute with $CONFIG_SHELL" >&2  
as_fn_exit 255  
  fi  





# 运行本段代码
# 检查当前 shell 环境是否满足脚本运行的最低要求， 并设置一些辅助函数和变量， 用于后续的配置和编译过程
  # We don't want this to propagate to other subprocesses.  
  # 取消_as_can_reexec 环境变量防止脚本重复执行
  # 前面没有设置 _as_can_reexec变量，甚至目前这个脚本也没涉及这个变量，所以这是一种通用的写法，与本脚本关联不大
          { _as_can_reexec=; unset _as_can_reexec;}  
# CONFIG_SHELL是指定shell的地址，用此shell来重新运行脚本
if test "x$CONFIG_SHELL" = x; then  
# 尝试让当前 shell 兼容 Bourne shell 的语法。主要针对 Zsh，尝试模拟 sh 的行为，并禁用一些与 Bourne shell 不兼容的特性。
# 与最前面解决Zsh兼容性的操作相同
  as_bourne_compatible="if test -n \"\${ZSH_VERSION+set}\" && (emulate sh) >/dev/null 2>&1; then :  
  emulate sh  
  NULLCMD=:  
  # Pre-4.2 versions of Zsh do word splitting on \${1+\"\$@\"}, which  
  # is contrary to our usage.  Disable this feature.  
  alias -g '\${1+\"\$@\"}'='\"\$@\"'  
  setopt NO_GLOB_SUBST  
else  
  case \`(set -o) 2>/dev/null\` in #(  
  *posix*) :  
  # 确保 POSIX 兼容模式被明确启用
    set -o posix ;; #(  
  *) :  
     ;;  
esac  
fi  
"  
# 定义了一些必要的 shell 函数，并测试这些函数
  as_required="as_fn_return () { (exit \$1); }  
as_fn_success () { as_fn_return 0; }  
as_fn_failure () { as_fn_return 1; }  
as_fn_ret_success () { return 0; }  
as_fn_ret_failure () { return 1; }  
  
exitcode=0  
# 注意此处的逻辑运算的逻辑，当前面的命令运行成功，退出码为0，但是是“执行成功”，所以用 || 表示不执行右边的命令
as_fn_success || { exitcode=1; echo as_fn_success failed.; }  
as_fn_failure && { exitcode=1; echo as_fn_failure succeeded.; }  
as_fn_ret_success || { exitcode=1; echo as_fn_ret_success failed.; }  
as_fn_ret_failure && { exitcode=1; echo as_fn_ret_failure succeeded.; }  
# 测试shell的保存位置参数的能力是否正常（虽然我感觉对现在来说没什么作用，就没遇到过shell不能正确保存参数的）
	# 测试保存位置参数的能力对于 shell 命令的执行至关重要，因为它关系到脚本能否正确处理和传递参数，从而影响到脚本的正确性和功能。
if ( set x; as_fn_ret_success y && test x = \"\$1\" ); then :  
  
else  
  exitcode=1; echo positional parameters were not saved.  
fi  
test x\$exitcode = x0 || exit 1  
# 检查根目录 / 是否存在并且可执行。如果根目录不存在或不可执行，则表示系统环境存在问题，脚本会退出并返回退出码 1
test -x / || exit 1"  

# 定义了一些建议的 shell 功能测试，测试当前 shell 是否支持基本的行号 (LINENO) 变量和算术表达式，以及是否能够正确执行 eval 命令
  as_suggested="  as_lineno_1=";
  # 直接执行echo $LINENO，打印出来是当前shell的行号
  as_suggested=$as_suggested$LINENO;
  # 注意这里使用了 \$LINENO，这是为了防止 $LINENO 被立即展开，而是将其作为字符串传递给 eval 命令
  as_suggested=$as_suggested" as_lineno_1a=\$LINENO  
  as_lineno_2=";
  as_suggested=$as_suggested$LINENO;
  # eval的工作：判断as_lineno_1
  # 一些建议的 shell 功能测试，例如检查行号和简单的算术运算
  as_suggested=$as_suggested" as_lineno_2a=\$LINENO  
  eval 'test \"x\$as_lineno_1'\$as_run'\" != \"x\$as_lineno_2'\$as_run'\" &&  
  test \"x\`expr \$as_lineno_1'\$as_run' + 1\`\" = \"x\$as_lineno_2'\$as_run'\"' || exit 1  
test \$(( 1 + 1 )) = 2 || exit 1"  


# 开始测试as_required
  if (eval "$as_required") 2>/dev/null; then :  
  as_have_required=yes  
else  
  as_have_required=no  
fi  




# 一大段if的开始
# 如果 as_have_required=yes，则执行 as_suggested 中定义的测试
if test x$as_have_required = xyes && (eval "$as_suggested") 2>/dev/null; then :  

# 如果 as_required 测试失败或者 as_suggested 测试失败，则执行后续代码块，尝试寻找合适的 shell（设置CONFIG_SHELL）
# 整体的逻辑是通过PATH_SEPARATOR做分隔符去在PATH里搜索可用的shell，找到一个就开始测试shell可执行文件是否存在，且as_required或as_suggested是否可以测试成功
# 如果找到CONFIG_SHELL并测试成功，就重新运行当前脚本，不成功就打印帮助信息并退出
else  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
as_found=false  
# 双层for循环，外层处理PATH，PATH=/bin:/usr/bin:$PATH
# 内层去寻找具体的可执行文件
for as_dir in /bin$PATH_SEPARATOR/usr/bin$PATH_SEPARATOR$PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
  as_found=:  
  case $as_dir in #(  
     /*)  
       for as_base in sh bash ksh sh5; do  
         # Try only shells that exist, to save several forks.  
         as_shell=$as_dir/$as_base  
         # 检查当前 shell 的可执行文件是否存在
         if { test -f "$as_shell" || test -f "$as_shell.exe"; } &&  
            { $as_echo "$as_bourne_compatible""$as_required" | as_run=a "$as_shell"; } 2>/dev/null; then :  
  CONFIG_SHELL=$as_shell as_have_required=yes  
           if { $as_echo "$as_bourne_compatible""$as_suggested" | as_run=a "$as_shell"; } 2>/dev/null; then :  
  break 2  
fi  
fi  
       done;;  
       esac  
  as_found=false  
done  
# 如果测试不成功的话，重新用当前shell来开启兼容性并测试as_required，将CONFIG_SHELL设置为当前shell变量（不一定，也可能CONFIG_SHELL为空，毕竟as_required不一定执行成功，所以有可能为空或者退出脚本）
$as_found || { if { test -f "$SHELL" || test -f "$SHELL.exe"; } &&  
          { $as_echo "$as_bourne_compatible""$as_required" | as_run=a "$SHELL"; } 2>/dev/null; then :  
  CONFIG_SHELL=$SHELL as_have_required=yes  
fi; }  
IFS=$as_save_IFS  
  
  # 如果找到了合适的shell，CONFIG_SHELL不为空，则重新用找到的shell使用exec命令来用新的shell执行当前脚本（此时还是在else分支内，与我自己正常执行过程无关）
      if test "x$CONFIG_SHELL" != x; then :  
  export CONFIG_SHELL  
             # We cannot yet assume a decent shell, so we have to provide a  
# neutralization value for shells without unset; and this also  
# works around shells that cannot unset nonexistent variables.  
# Preserve -v and -x to the replacement shell.  
BASH_ENV=/dev/null  
ENV=/dev/null  
(unset BASH_ENV) >/dev/null 2>&1 && unset BASH_ENV ENV  
case $- in # ((((  
  *v*x* | *x*v* ) as_opts=-vx ;;  
  *v* ) as_opts=-v ;;  
  *x* ) as_opts=-x ;;  
  * ) as_opts= ;;  
esac  
# 上面的一些动作跟前面是一样的，取消一些环境变量，添加shell操作描述符、处理传参的兼容性问题，同时使用exec执行新的shell
exec $CONFIG_SHELL $as_opts "$as_myself" ${1+"$@"}  
# Admittedly, this is quite paranoid, since all the known shells bail  
# out after a failed `exec'.  
# exec可能执行失败，失败后会返回当前shell脚本，继续执行后续代码，这里的处理是让它报错并退出
$as_echo "$0: could not re-execute with $CONFIG_SHELL" >&2  
exit 255  
fi  
  # as_have_required=no的时候，CONFIG_SHELL一定为空，打印信息，表明shell版本太落后了
    if test x$as_have_required = xno; then :  
  $as_echo "$0: This script requires a shell more modern than all"  
  $as_echo "$0: the shells that I found on your system."  
  # 如果设置了ZSH_VERSION，打印zsh有bug，而且应该使用4.3.4以后的版本
  if test x${ZSH_VERSION+set} = xset ; then  
    $as_echo "$0: In particular, zsh $ZSH_VERSION has bugs and should"  
    $as_echo "$0: be upgraded to zsh 4.3.4 or later."  
# 如果ZSH_VERSION没设置就让联系gnu
  else  
    $as_echo "$0: Please tell bug-autoconf@gnu.org about your system,  
$0: including any error possibly output before this  
$0: message. Then install a modern shell, or manually run  
$0: the script under such a shell if you do have one."  
  fi  
  exit 1  
fi  
fi  
fi  # 分隔的一大段if到这里才结束，好长一段if
SHELL=${CONFIG_SHELL-/bin/sh}  
export SHELL  





# 将 `CLICOLOR_FORCE` 和 `GREP_OPTIONS` 这两个变量设置为空值，并取消两个变量
# Unset more variables known to interfere with behavior of common tools.  
CLICOLOR_FORCE= GREP_OPTIONS=  
unset CLICOLOR_FORCE GREP_OPTIONS  

# 这里定义了一些 M4sh 的 shell 函数
# M4sh 并不是一个独立的 shell，而是一种在 Autoconf 环境下使用的 shell 脚本语言的约定俗成的称呼，它主要用于编写可移植的 configure 脚本，并最终生成 Makefile。
# Autoconf是一个用于生成 shell 脚本的工具，这些脚本可以自动配置软件源代码包，使其能够在各种不同的类 Unix 系统上进行编译和安装。
# M4是一个功能强大的宏处理器，它可以用于各种文本处理和代码生成任务,主要用于将输入文本转换为输出文本，过程中可以使用宏来进行文本替换、文件包含、条件判断等操作。由于其灵活性和可扩展性，M4被广泛应用于软件开发、系统管理等领域。
## --------------------- ##  
## M4sh Shell Functions. ##  
## --------------------- ##  
# as_fn_unset VAR  
# ---------------  
# Portably unset VAR.  
# 以一种可移植的方式取消设置变量 
as_fn_unset ()  
{  
  { eval $1=; unset $1;}  
}  
as_unset=as_fn_unset  

# 定义shell的返回函数
# as_fn_set_status STATUS  
# -----------------------  
# Set $? to STATUS, without forking. 
# 将函数的第一个参数（`$1`）作为返回值返回
as_fn_set_status ()  
{  
  return $1  
} # as_fn_set_status  

# 定义shell的退出函数
# as_fn_exit STATUS  
# -----------------  
# Exit the shell with STATUS, even in a "trap 0" or "set -e" context.  
# 使用 `STATUS` 作为退出码退出 shell，即使在 "trap 0" 或 "set -e" 的情况下也能生效。
as_fn_exit ()  
{  
# 这行命令关闭了 `errexit` 选项（`set -e`）。 `errexit` 选项会在任何命令返回非零退出码时立即退出脚本。关闭 `errexit` 选项可以确保即使 `as_fn_set_status` 返回非零值，脚本也能继续执行 `exit` 命令。
  set +e  
  # 将 shell 的返回值设置为函数的第一个参数（`$1`），返回错误码
  as_fn_set_status $1  
  exit $1  
} # as_fn_exit  
  
# as_fn_mkdir_p  
# -------------  
# Create "$as_dir" as a directory, including parents if necessary.  
# 实现mkdir-p的操作
as_fn_mkdir_p ()  
{  
  
  case $as_dir in #(
  # 判断文件名是否以-开头，以-开头可能被误解为参数，则需要进行处理
  -*) as_dir=./$as_dir;;  
  esac  
  # 这行代码检查 `as_dir` 是否已经存在且是一个目录。如果是，则直接跳过后续操作（后续操作是指的这一长串的所有操作）
  # 逻辑有误，如果as_mkdir_p已被设置为mkdir -p的话，还是需要跟操作符（具体的文件夹名字）
  test -d "$as_dir" || eval $as_mkdir_p || {  
    as_dirs=  
    # 处理文件路径内可能含有“'”单引号的情况，将所有单引号替换成“\'”
    while :; do  
      case $as_dir in #(  
      *\'*) as_qdir=`$as_echo "$as_dir" | sed "s/'/'\\\\\\\\''/g"`;; #'(  
      *) as_qdir=$as_dir;;  
      esac  
      # 将当前目录（`$as_qdir`）添加到 `as_dirs` 列表的开头。
      as_dirs="'$as_qdir' $as_dirs"  
      # 这部分代码使用了多种方法来获取 `as_dir` 的父目录，并将结果赋值给 `as_dir` 变量。
      # “$as_expr X"$as_dir" :”带上后面的几行，这个是一个正则表达式，而"\|"在表达逻辑或
      #  接着使用 `sed` 命令处理 `as_dir` 变量，目的是获取其父目录，q表示退出sed命令
      as_dir=`$as_dirname -- "$as_dir" ||  
$as_expr X"$as_dir" : 'X\(.*[^/]\)//*[^/][^/]*/*$' \| \  
     X"$as_dir" : 'X\(//\)[^/]' \| \  
     X"$as_dir" : 'X\(//\)$' \| \  
     X"$as_dir" : 'X\(/\)' \| . 2>/dev/null ||
$as_echo X"$as_dir" |  
    sed '/^X\(.*[^/]\)\/\/*[^/][^/]*\/*$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)[^/].*/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\).*/{  
        s//\1/  
        q  
      }  
      s/.*/./; q'`  
      test -d "$as_dir" && break  
    done  
    # 如果 `as_dirs` 列表不为空，则使用 `mkdir` 命令创建所有父目录。
    test -z "$as_dirs" || eval "mkdir $as_dirs"  
  }  ||
  # 最后再次检查 `as_dir` 是否已经存在且是一个目录。如果仍然失败，则调用 `as_fn_error` 函数报告错误。 
 test -d "$as_dir" || as_fn_error $? "cannot create directory $as_dir"  
  
} # as_fn_mkdir_p  



# as_fn_executable_p FILE  
# -----------------------  
# Test if FILE is an executable regular file.  
# 用于测试指定的文件是否为可执行的常规文件
as_fn_executable_p ()  
{  
  test -f "$1" && test -x "$1"  
} # as_fn_executable_p  


# as_fn_append VAR VALUE  
# ----------------------  
# Append the text in VALUE to the end of the definition contained in VAR. Take  
# advantage of any shell optimizations that allow amortized linear growth over  
# repeated appends, instead of the typical quadratic growth present in naive 
# 将一个字符串追加到一个变量的值的末尾。它会根据 shell 的特性选择不同的实现方式，以优化性能（shell支持+=的话可以提高性能）。
# implementations.  
# 测试当前 shell 是否支持 `+=` 运算符进行字符串追加操作
if (eval "as_var=1; as_var+=2; test x\$as_var = x12") 2>/dev/null; then :  
  eval 'as_fn_append ()  
  {  
    eval $1+=\$2  
  }'  
else  
  as_fn_append ()  
  {  
    eval $1=\$$1\$2  
  }  
fi # as_fn_append  



# as_fn_arith ARG...  
# ------------------  
# Perform arithmetic evaluation on the ARGs, and store the result in the  
# global $as_val. Take advantage of shells that can avoid forks. The arguments  
# must be portable across $(()) and expr.  
# 实现执行算术运算的功能
# 测试当前 shell 是否支持 `$(( ... ))` 语法进行算术运算。
# 在shell终端$(())是执行算数运算，$()是将括号内的当作命令执行
if (eval "test \$(( 1 + 1 )) = 2") 2>/dev/null; then :  
  eval 'as_fn_arith ()  
  {  
    as_val=$(( $* ))  
  }'  
else  
  as_fn_arith ()  
  {  
  # 使用 `expr` 命令计算所有参数的算术运算结果，并将结果赋值给全局变量
  # 如果 `expr` 命令执行失败，则测试其返回值是否为 `1`。如果返回值为 `1`，则表示 `expr` 命令执行失败是因为参数不是有效的数字，而不是因为其他错误。
    as_val=`expr "$@" || test $? -eq 1`  
  }  
fi # as_fn_arith  
  
  
# as_fn_error STATUS ERROR [LINENO LOG_FD]  
# ----------------------------------------  
# Output "`basename $0`: error: ERROR" to stderr. If LINENO and LOG_FD are  
# provided, also output the error to LOG_FD, referencing LINENO. Then exit the  
# script with STATUS, using 1 if that was 0.  
# 输出错误信息到指定日志文件和stderr，并退出脚本，接收四个参数：
# STAUS：退出状态码，如果为 0，则会被设置为 1。（为了防止脚本执行失败，但是仍退出码为0而忽略错误，更明确指示执行失败）
# ERROR : 错误信息。
# LINENO： 行号信息
# LOG_FD： 日志文件的文件描述符
# 输出错误信息到标准错误输出（stderr），如果提供了 `LINENO` 和 `LOG_FD`，还会将错误信息输出到指定的日志文件，并使用指定的退出状态码退出脚本。
as_fn_error ()  
{  
  as_status=$1; test $as_status -eq 0 && as_status=1  
  # `LOG_FD`不为空
  if test "$4"; then  
  # 设置 `as_lineno` 变量为第三个参数（`LINENO`）的值，如果 `LINENO` 为空，则使用默认值。
    as_lineno=${as_lineno-"$3"} as_lineno_stack=as_lineno_stack=$as_lineno_stack  
    # 将错误信息输出到指定的日志文件描述符 
    $as_echo "$as_me:${as_lineno-$LINENO}: error: $2" >&$4  
  fi  
  # 将错误信息输出到标准错误输出（stderr）
  $as_echo "$as_me: error: $2" >&2  
  # 使用as_status退出码，退出脚本
  as_fn_exit $as_status  
} # as_fn_error  



# 分别检查了 `expr`, `basename` 和 `dirname` 这三个命令是否可用，并将结果分别存储在变量 `as_expr`, `as_basename` 和 `as_dirname` 中。如果命令可用，则变量的值为命令本身；否则，变量的值为 `false`
# 测试 `expr` 命令是否可用。
#  使用 `expr` 命令进行正则表达式匹配，判断 `a` 是否匹配 `a`
if expr a : '\(a\)' >/dev/null 2>&1 &&  
   test "X`expr 00001 : '.*\(...\)'`" = X001; then  
  as_expr=expr  
else  
  as_expr=false  
fi  
 #  测试 `basename` 命令是否可用。
if (basename -- /) >/dev/null 2>&1 && test "X`basename -- / 2>&1`" = "X/"; then  
  as_basename=basename  
else  
  as_basename=false  
fi  
# dirname -- /``: 使用 `dirname` 命令获取 `/` 的父目录，并将结果存储在 `as_dir` 变量中，判断 `as_dir` 的值是否为 `/`
if (as_dir=`dirname -- /` && test "X$as_dir" = X/) >/dev/null 2>&1; then  
  as_dirname=dirname  
else  
  as_dirname=false  
fi  
  
as_me=`$as_basename -- "$0" ||  
$as_expr X/"$0" : '.*/\([^/][^/]*\)/*$' \| \  
     X"$0" : 'X\(//\)$' \| \  
     X"$0" : 'X\(/\)' \| . 2>/dev/null ||  
$as_echo X/"$0" |  
    sed '/^.*\/\([^/][^/]*\)\/*$/{  
        s//\1/  
        q  
      }  
      /^X\/\(\/\/\)$/{  
        s//\1/  
        q  
      }  
      /^X\/\(\/\).*/{  
        s//\1/  
        q  
      }  
      s/.*/./; q'`  
  
# Avoid depending upon Character Ranges.  
# 定义了一组变量，用于表示不同类型的字符，包括小写字母、大写字母、字母（大小写）、数字和字母数字字符。它的目的是避免依赖于字符范围，从而提高脚本的可移植性。
# 在不同的系统和字符编码中，字符范围可能会有所不同。例如，在 ASCII 编码中，字母 `a` 到 `z` 的范围是 97 到 122，但在 EBCDIC 编码中，它们的范围是 129 到 169。如果脚本依赖于特定的字符范围，那么在不同的系统上运行时可能会出现问题。
as_cr_letters='abcdefghijklmnopqrstuvwxyz'  
as_cr_LETTERS='ABCDEFGHIJKLMNOPQRSTUVWXYZ'  
as_cr_Letters=$as_cr_letters$as_cr_LETTERS  
as_cr_digits='0123456789'  
as_cr_alnum=$as_cr_Letters$as_cr_digits    
  
  
  as_lineno_1=$LINENO as_lineno_1a=$LINENO  
  as_lineno_2=$LINENO as_lineno_2a=$LINENO  
  eval 'test "x$as_lineno_1'$as_run'" != "x$as_lineno_2'$as_run'" &&  
  test "x`expr $as_lineno_1'$as_run' + 1`" = "x$as_lineno_2'$as_run'"' || {  
  # Blame Lee E. McMahon (1931-1989) for sed's syntax.  :-)  
  sed -n '  
    p  
    /[$]LINENO/=  
  ' <$as_myself |  
    sed '  
      s/[$]LINENO.*/&-/  
      t lineno  
      b  
      :lineno  
      N  
      :loop  
      s/[$]LINENO\([^'$as_cr_alnum'_].*\n\)\(.*\)/\2\1\2/  
      t loop  
      s/-\n.*//  
    ' >$as_me.lineno &&  
  chmod +x "$as_me.lineno" ||  
    { $as_echo "$as_me: error: cannot create $as_me.lineno; rerun with a POSIX shell" >&2; as_fn_exit 1; }  
  
  # If we had to re-execute with $CONFIG_SHELL, we're ensured to have  
  # already done that, so ensure we don't try to do so again and fall  
  # in an infinite loop.  This has already happened in practice.  
  _as_can_reexec=no; export _as_can_reexec  
  # Don't try to exec as it changes $[0], causing all sort of problems  
  # (the dirname of $[0] is not the place where we might find the  
  # original and so on.  Autoconf is especially sensitive to this).  
  . "./$as_me.lineno"  
  # Exit status is that of the last command.  
  exit  
}  
  
ECHO_C= ECHO_N= ECHO_T=  
case `echo -n x` in #(((((  
-n*)  
  case `echo 'xy\c'` in  
  *c*) ECHO_T='    ';;    # ECHO_T is single tab character.  
  xy)  ECHO_C='\c';;  
  *)   echo `echo ksh88 bug on AIX 6.1` > /dev/null  
       ECHO_T='    ';;  
  esac;;  
*)  
  ECHO_N='-n';;  
esac  
  
rm -f conf$$ conf$$.exe conf$$.file  
if test -d conf$$.dir; then  
  rm -f conf$$.dir/conf$$.file  
else  
  rm -f conf$$.dir  
  mkdir conf$$.dir 2>/dev/null  
fi  
if (echo >conf$$.file) 2>/dev/null; then  
  if ln -s conf$$.file conf$$ 2>/dev/null; then  
    as_ln_s='ln -s'  
    # ... but there are two gotchas:  
    # 1) On MSYS, both `ln -s file dir' and `ln file dir' fail.  
    # 2) DJGPP < 2.04 has no symlinks; `ln -s' creates a wrapper executable.  
    # In both cases, we have to default to `cp -pR'.  
    ln -s conf$$.file conf$$.dir 2>/dev/null && test ! -f conf$$.exe ||  
      as_ln_s='cp -pR'  
  elif ln conf$$.file conf$$ 2>/dev/null; then  
    as_ln_s=ln  
  else  
    as_ln_s='cp -pR'  
  fi  
else  
  as_ln_s='cp -pR'  
fi  
rm -f conf$$ conf$$.exe conf$$.dir/conf$$.file conf$$.file  
rmdir conf$$.dir 2>/dev/null  
  
if mkdir -p . 2>/dev/null; then  
  as_mkdir_p='mkdir -p "$as_dir"'  
else  
  test -d ./-p && rmdir ./-p  
  as_mkdir_p=false  
fi  
  
as_test_x='test -x'  
as_executable_p=as_fn_executable_p  
  
# Sed expression to map a string onto a valid CPP name.  
as_tr_cpp="eval sed 'y%*$as_cr_letters%P$as_cr_LETTERS%;s%[^_$as_cr_alnum]%_%g'"  
  
# Sed expression to map a string onto a valid variable name.  
as_tr_sh="eval sed 'y%*+%pp%;s%[^_$as_cr_alnum]%_%g'"  
  
  
test -n "$DJDIR" || exec 7<&0 </dev/null  
exec 6>&1  
  
# Name of the host.  
# hostname on some systems (SVR3.2, old GNU/Linux) returns a bogus exit status,  
# so uname gets run too.  
ac_hostname=`(hostname || uname -n) 2>/dev/null | sed 1q`  
  
#  
# Initializations.  
#  
ac_default_prefix=/usr/local  
ac_clean_files=  
ac_config_libobj_dir=.  
LIBOBJS=  
cross_compiling=no  
subdirs=  
MFLAGS=  
MAKEFLAGS=  
  
# Identity of this package.  
PACKAGE_NAME=  
PACKAGE_TARNAME=  
PACKAGE_VERSION=  
PACKAGE_STRING=  
PACKAGE_BUGREPORT=  
PACKAGE_URL=  
  
ac_unique_file="move-if-change"  
enable_option_checking=no  
ac_subst_vars='LTLIBOBJS  
LIBOBJS  
compare_exclusions  
stage2_werror_flag  
stage1_checking  
stage1_cflags  
MAINT  
MAINTAINER_MODE_FALSE  
MAINTAINER_MODE_TRUE  
COMPILER_NM_FOR_TARGET  
COMPILER_LD_FOR_TARGET  
COMPILER_AS_FOR_TARGET  
FLAGS_FOR_TARGET  
RAW_CXX_FOR_TARGET  
WINDMC_FOR_TARGET  
WINDRES_FOR_TARGET  
STRIP_FOR_TARGET  
READELF_FOR_TARGET  
RANLIB_FOR_TARGET  
OTOOL_FOR_TARGET  
OBJDUMP_FOR_TARGET  
OBJCOPY_FOR_TARGET  
NM_FOR_TARGET  
LIPO_FOR_TARGET  
LD_FOR_TARGET  
DSYMUTIL_FOR_TARGET  
DLLTOOL_FOR_TARGET  
AS_FOR_TARGET  
AR_FOR_TARGET  
GM2_FOR_TARGET  
GDC_FOR_TARGET  
GOC_FOR_TARGET  
GFORTRAN_FOR_TARGET  
GCC_FOR_TARGET  
CXX_FOR_TARGET  
CC_FOR_TARGET  
RANLIB_PLUGIN_OPTION  
AR_PLUGIN_OPTION  
PKG_CONFIG_PATH  
GDCFLAGS  
READELF  
OTOOL  
OBJDUMP  
OBJCOPY  
WINDMC  
WINDRES  
STRIP  
RANLIB  
NM  
LIPO  
LD  
DSYMUTIL  
DLLTOOL  
AS  
AR  
RUNTEST  
EXPECT  
MAKEINFO  
FLEX  
LEX  
M4  
BISON  
YACC  
WINDRES_FOR_BUILD  
WINDMC_FOR_BUILD  
RANLIB_FOR_BUILD  
NM_FOR_BUILD  
LD_FOR_BUILD  
LDFLAGS_FOR_BUILD  
GDC_FOR_BUILD  
GOC_FOR_BUILD  
GFORTRAN_FOR_BUILD  
DSYMUTIL_FOR_BUILD  
DLLTOOL_FOR_BUILD  
CXX_FOR_BUILD  
CXXFLAGS_FOR_BUILD  
CPPFLAGS_FOR_BUILD  
CPP_FOR_BUILD  
CFLAGS_FOR_BUILD  
CC_FOR_BUILD  
AS_FOR_BUILD  
AR_FOR_BUILD  
target_configdirs  
configdirs  
build_configdirs  
INSTALL_GDB_TK  
GDB_TK  
CONFIGURE_GDB_TK  
build_tooldir  
tooldir  
GCC_SHLIB_SUBDIR  
RPATH_ENVVAR  
target_configargs  
host_configargs  
build_configargs  
BUILD_CONFIG  
LDFLAGS_FOR_TARGET  
CXXFLAGS_FOR_TARGET  
CFLAGS_FOR_TARGET  
DEBUG_PREFIX_CFLAGS_FOR_TARGET  
SYSROOT_CFLAGS_FOR_TARGET  
get_gcc_base_ver  
extra_host_zlib_configure_flags  
extra_host_libiberty_configure_flags  
stage1_languages  
host_libs_picflag  
PICFLAG  
host_shared  
gcc_host_pie  
host_pie  
extra_linker_plugin_flags  
extra_linker_plugin_configure_flags  
islinc  
isllibs  
poststage1_ldflags  
poststage1_libs  
stage1_ldflags  
stage1_libs  
extra_isl_gmp_configure_flags  
extra_mpc_mpfr_configure_flags  
extra_mpc_gmp_configure_flags  
extra_mpfr_configure_flags  
gmpinc  
gmplibs  
PGO_BUILD_LTO_CFLAGS  
PGO_BUILD_USE_CFLAGS  
PGO_BUILD_GEN_CFLAGS  
HAVE_CXX11_FOR_BUILD  
HAVE_CXX11  
do_compare  
GDC  
GNATMAKE  
GNATBIND  
ac_ct_CXX  
CXXFLAGS  
CXX  
OBJEXT  
EXEEXT  
ac_ct_CC  
CPPFLAGS  
LDFLAGS  
CFLAGS  
CC  
target_subdir  
host_subdir  
build_subdir  
build_libsubdir  
AWK  
SED  
LN_S  
LN  
INSTALL_DATA  
INSTALL_SCRIPT  
INSTALL_PROGRAM  
target_os  
target_vendor  
target_cpu  
target  
host_os  
host_vendor  
host_cpu  
host  
target_noncanonical  
host_noncanonical  
build_noncanonical  
build_os  
build_vendor  
build_cpu  
build  
TOPLEVEL_CONFIGURE_ARGUMENTS  
target_alias  
host_alias  
build_alias  
LIBS  
ECHO_T  
ECHO_N  
ECHO_C  
DEFS  
mandir  
localedir  
libdir  
psdir  
pdfdir  
dvidir  
htmldir  
infodir  
docdir  
oldincludedir  
includedir  
localstatedir  
sharedstatedir  
sysconfdir  
datadir  
datarootdir  
libexecdir  
sbindir  
bindir  
program_transform_name  
prefix  
exec_prefix  
PACKAGE_URL  
PACKAGE_BUGREPORT  
PACKAGE_STRING  
PACKAGE_VERSION  
PACKAGE_TARNAME  
PACKAGE_NAME  
PATH_SEPARATOR  
SHELL'  
ac_subst_files='serialization_dependencies  
host_makefile_frag  
target_makefile_frag  
alphaieee_frag  
ospace_frag'  
ac_user_opts='  
enable_option_checking  
with_build_libsubdir  
with_system_zlib  
with_zstd  
enable_as_accelerator_for  
enable_offload_targets  
enable_offload_defaulted  
enable_gold  
enable_ld  
enable_gprofng  
enable_compressed_debug_sections  
enable_default_compressed_debug_sections_algorithm  
enable_year2038  
enable_libquadmath  
enable_libquadmath_support  
enable_libada  
enable_libgm2  
enable_libssp  
enable_libstdcxx  
enable_bootstrap  
enable_pgo_build  
with_mpc  
with_mpc_include  
with_mpc_lib  
with_mpfr  
with_mpfr_include  
with_mpfr_lib  
with_gmp  
with_gmp_include  
with_gmp_lib  
with_stage1_libs  
with_static_standard_libraries  
with_stage1_ldflags  
with_boot_libs  
with_boot_ldflags  
with_isl  
with_isl_include  
with_isl_lib  
enable_isl_version_check  
enable_lto  
enable_linker_plugin_configure_flags  
enable_linker_plugin_flags  
enable_host_pie  
enable_host_shared  
enable_stage1_languages  
enable_objc_gc  
with_target_bdw_gc  
with_target_bdw_gc_include  
with_target_bdw_gc_lib  
with_gcc_major_version_only  
with_build_sysroot  
with_debug_prefix_map  
with_build_config  
enable_vtable_verify  
enable_serial_configure  
with_build_time_tools  
enable_maintainer_mode  
enable_stage1_checking  
enable_werror  
'  
      ac_precious_vars='build_alias  
host_alias  
target_alias  
CC  
CFLAGS  
LDFLAGS  
LIBS  
CPPFLAGS  
CXX  
CXXFLAGS  
CCC  
build_configargs  
host_configargs  
target_configargs  
AR  
AS  
DLLTOOL  
DSYMUTIL  
LD  
LIPO  
NM  
RANLIB  
STRIP  
WINDRES  
WINDMC  
OBJCOPY  
OBJDUMP  
OTOOL  
READELF  
CC_FOR_TARGET  
CXX_FOR_TARGET  
GCC_FOR_TARGET  
GFORTRAN_FOR_TARGET  
GOC_FOR_TARGET  
GDC_FOR_TARGET  
GM2_FOR_TARGET  
AR_FOR_TARGET  
AS_FOR_TARGET  
DLLTOOL_FOR_TARGET  
DSYMUTIL_FOR_TARGET  
LD_FOR_TARGET  
LIPO_FOR_TARGET  
NM_FOR_TARGET  
OBJCOPY_FOR_TARGET  
OBJDUMP_FOR_TARGET  
OTOOL_FOR_TARGET  
RANLIB_FOR_TARGET  
READELF_FOR_TARGET  
STRIP_FOR_TARGET  
WINDRES_FOR_TARGET  
WINDMC_FOR_TARGET'  
  
  
# Initialize some variables set by options.  
ac_init_help=  
ac_init_version=false  
ac_unrecognized_opts=  
ac_unrecognized_sep=  
# The variables have the same names as the options, with  
# dashes changed to underlines.  
cache_file=/dev/null  
exec_prefix=NONE  
no_create=  
no_recursion=  
prefix=NONE  
program_prefix=NONE  
program_suffix=NONE  
program_transform_name=s,x,x,  
silent=  
site=  
srcdir=  
verbose=  
x_includes=NONE  
x_libraries=NONE  
  
# Installation directory options.  
# These are left unexpanded so users can "make install exec_prefix=/foo"  
# and all the variables that are supposed to be based on exec_prefix  
# by default will actually change.  
# Use braces instead of parens because sh, perl, etc. also accept them.  
# (The list follows the same order as the GNU Coding Standards.)  
bindir='${exec_prefix}/bin'  
sbindir='${exec_prefix}/sbin'  
libexecdir='${exec_prefix}/libexec'  
datarootdir='${prefix}/share'  
datadir='${datarootdir}'  
sysconfdir='${prefix}/etc'  
sharedstatedir='${prefix}/com'  
localstatedir='${prefix}/var'  
includedir='${prefix}/include'  
oldincludedir='/usr/include'  
docdir='${datarootdir}/doc/${PACKAGE}'  
infodir='${datarootdir}/info'  
htmldir='${docdir}'  
dvidir='${docdir}'  
pdfdir='${docdir}'  
psdir='${docdir}'  
libdir='${exec_prefix}/lib'  
localedir='${datarootdir}/locale'  
mandir='${datarootdir}/man'  
  
ac_prev=  
ac_dashdash=  
for ac_option  
do  
  # If the previous option needs an argument, assign it.  
  if test -n "$ac_prev"; then  
    eval $ac_prev=\$ac_option  
    ac_prev=  
    continue  
  fi  
  
  case $ac_option in  
  *=?*) ac_optarg=`expr "X$ac_option" : '[^=]*=\(.*\)'` ;;  
  *=)   ac_optarg= ;;  
  *)    ac_optarg=yes ;;  
  esac  
  
  # Accept the important Cygnus configure options, so we can diagnose typos.  
  
  case $ac_dashdash$ac_option in  
  --)  
    ac_dashdash=yes ;;  
  
  -bindir | --bindir | --bindi | --bind | --bin | --bi)  
    ac_prev=bindir ;;  
  -bindir=* | --bindir=* | --bindi=* | --bind=* | --bin=* | --bi=*)  
    bindir=$ac_optarg ;;  
  
  -build | --build | --buil | --bui | --bu)  
    ac_prev=build_alias ;;  
  -build=* | --build=* | --buil=* | --bui=* | --bu=*)  
    build_alias=$ac_optarg ;;  
  
  -cache-file | --cache-file | --cache-fil | --cache-fi \  
  | --cache-f | --cache- | --cache | --cach | --cac | --ca | --c)  
    ac_prev=cache_file ;;  
  -cache-file=* | --cache-file=* | --cache-fil=* | --cache-fi=* \  
  | --cache-f=* | --cache-=* | --cache=* | --cach=* | --cac=* | --ca=* | --c=*)  
    cache_file=$ac_optarg ;;  
  
  --config-cache | -C)  
    cache_file=config.cache ;;  
  
  -datadir | --datadir | --datadi | --datad)  
    ac_prev=datadir ;;  
  -datadir=* | --datadir=* | --datadi=* | --datad=*)  
    datadir=$ac_optarg ;;  
  
  -datarootdir | --datarootdir | --datarootdi | --datarootd | --dataroot \  
  | --dataroo | --dataro | --datar)  
    ac_prev=datarootdir ;;  
  -datarootdir=* | --datarootdir=* | --datarootdi=* | --datarootd=* \  
  | --dataroot=* | --dataroo=* | --dataro=* | --datar=*)  
    datarootdir=$ac_optarg ;;  
  
  -disable-* | --disable-*)  
    ac_useropt=`expr "x$ac_option" : 'x-*disable-\(.*\)'`  
    # Reject names that are not valid shell variable names.  
    expr "x$ac_useropt" : ".*[^-+._$as_cr_alnum]" >/dev/null &&  
      as_fn_error $? "invalid feature name: $ac_useropt"  
    ac_useropt_orig=$ac_useropt  
    ac_useropt=`$as_echo "$ac_useropt" | sed 's/[-+.]/_/g'`  
    case $ac_user_opts in  
      *"  
"enable_$ac_useropt"  
"*) ;;  
      *) ac_unrecognized_opts="$ac_unrecognized_opts$ac_unrecognized_sep--disable-$ac_useropt_orig"  
     ac_unrecognized_sep=', ';;  
    esac  
    eval enable_$ac_useropt=no ;;  
  
  -docdir | --docdir | --docdi | --doc | --do)  
    ac_prev=docdir ;;  
  -docdir=* | --docdir=* | --docdi=* | --doc=* | --do=*)  
    docdir=$ac_optarg ;;  
  
  -dvidir | --dvidir | --dvidi | --dvid | --dvi | --dv)  
    ac_prev=dvidir ;;  
  -dvidir=* | --dvidir=* | --dvidi=* | --dvid=* | --dvi=* | --dv=*)  
    dvidir=$ac_optarg ;;  
  
  -enable-* | --enable-*)  
    ac_useropt=`expr "x$ac_option" : 'x-*enable-\([^=]*\)'`  
    # Reject names that are not valid shell variable names.  
    expr "x$ac_useropt" : ".*[^-+._$as_cr_alnum]" >/dev/null &&  
      as_fn_error $? "invalid feature name: $ac_useropt"  
    ac_useropt_orig=$ac_useropt  
    ac_useropt=`$as_echo "$ac_useropt" | sed 's/[-+.]/_/g'`  
    case $ac_user_opts in  
      *"  
"enable_$ac_useropt"  
"*) ;;  
      *) ac_unrecognized_opts="$ac_unrecognized_opts$ac_unrecognized_sep--enable-$ac_useropt_orig"  
     ac_unrecognized_sep=', ';;  
    esac  
    eval enable_$ac_useropt=\$ac_optarg ;;  
  
  -exec-prefix | --exec_prefix | --exec-prefix | --exec-prefi \  
  | --exec-pref | --exec-pre | --exec-pr | --exec-p | --exec- \  
  | --exec | --exe | --ex)  
    ac_prev=exec_prefix ;;  
  -exec-prefix=* | --exec_prefix=* | --exec-prefix=* | --exec-prefi=* \  
  | --exec-pref=* | --exec-pre=* | --exec-pr=* | --exec-p=* | --exec-=* \  
  | --exec=* | --exe=* | --ex=*)  
    exec_prefix=$ac_optarg ;;  
  
  -gas | --gas | --ga | --g)  
    # Obsolete; use --with-gas.  
    with_gas=yes ;;  
  
  -help | --help | --hel | --he | -h)  
    ac_init_help=long ;;  
  -help=r* | --help=r* | --hel=r* | --he=r* | -hr*)  
    ac_init_help=recursive ;;  
  -help=s* | --help=s* | --hel=s* | --he=s* | -hs*)  
    ac_init_help=short ;;  
  
  -host | --host | --hos | --ho)  
    ac_prev=host_alias ;;  
  -host=* | --host=* | --hos=* | --ho=*)  
    host_alias=$ac_optarg ;;  
  
  -htmldir | --htmldir | --htmldi | --htmld | --html | --htm | --ht)  
    ac_prev=htmldir ;;  
  -htmldir=* | --htmldir=* | --htmldi=* | --htmld=* | --html=* | --htm=* \  
  | --ht=*)  
    htmldir=$ac_optarg ;;  
  
  -includedir | --includedir | --includedi | --included | --include \  
  | --includ | --inclu | --incl | --inc)  
    ac_prev=includedir ;;  
  -includedir=* | --includedir=* | --includedi=* | --included=* | --include=* \  
  | --includ=* | --inclu=* | --incl=* | --inc=*)  
    includedir=$ac_optarg ;;  
  
  -infodir | --infodir | --infodi | --infod | --info | --inf)  
    ac_prev=infodir ;;  
  -infodir=* | --infodir=* | --infodi=* | --infod=* | --info=* | --inf=*)  
    infodir=$ac_optarg ;;  
  
  -libdir | --libdir | --libdi | --libd)  
    ac_prev=libdir ;;  
  -libdir=* | --libdir=* | --libdi=* | --libd=*)  
    libdir=$ac_optarg ;;  
  
  -libexecdir | --libexecdir | --libexecdi | --libexecd | --libexec \  
  | --libexe | --libex | --libe)  
    ac_prev=libexecdir ;;  
  -libexecdir=* | --libexecdir=* | --libexecdi=* | --libexecd=* | --libexec=* \  
  | --libexe=* | --libex=* | --libe=*)  
    libexecdir=$ac_optarg ;;  
  
  -localedir | --localedir | --localedi | --localed | --locale)  
    ac_prev=localedir ;;  
  -localedir=* | --localedir=* | --localedi=* | --localed=* | --locale=*)  
    localedir=$ac_optarg ;;  
  
  -localstatedir | --localstatedir | --localstatedi | --localstated \  
  | --localstate | --localstat | --localsta | --localst | --locals)  
    ac_prev=localstatedir ;;  
  -localstatedir=* | --localstatedir=* | --localstatedi=* | --localstated=* \  
  | --localstate=* | --localstat=* | --localsta=* | --localst=* | --locals=*)  
    localstatedir=$ac_optarg ;;  
  
  -mandir | --mandir | --mandi | --mand | --man | --ma | --m)  
    ac_prev=mandir ;;  
  -mandir=* | --mandir=* | --mandi=* | --mand=* | --man=* | --ma=* | --m=*)  
    mandir=$ac_optarg ;;  
  
  -nfp | --nfp | --nf)  
    # Obsolete; use --without-fp.  
    with_fp=no ;;  
  
  -no-create | --no-create | --no-creat | --no-crea | --no-cre \  
  | --no-cr | --no-c | -n)  
    no_create=yes ;;  
  
  -no-recursion | --no-recursion | --no-recursio | --no-recursi \  
  | --no-recurs | --no-recur | --no-recu | --no-rec | --no-re | --no-r)  
    no_recursion=yes ;;  
  
  -oldincludedir | --oldincludedir | --oldincludedi | --oldincluded \  
  | --oldinclude | --oldinclud | --oldinclu | --oldincl | --oldinc \  
  | --oldin | --oldi | --old | --ol | --o)  
    ac_prev=oldincludedir ;;  
  -oldincludedir=* | --oldincludedir=* | --oldincludedi=* | --oldincluded=* \  
  | --oldinclude=* | --oldinclud=* | --oldinclu=* | --oldincl=* | --oldinc=* \  
  | --oldin=* | --oldi=* | --old=* | --ol=* | --o=*)  
    oldincludedir=$ac_optarg ;;  
  
  -prefix | --prefix | --prefi | --pref | --pre | --pr | --p)  
    ac_prev=prefix ;;  
  -prefix=* | --prefix=* | --prefi=* | --pref=* | --pre=* | --pr=* | --p=*)  
    prefix=$ac_optarg ;;  
  
  -program-prefix | --program-prefix | --program-prefi | --program-pref \  
  | --program-pre | --program-pr | --program-p)  
    ac_prev=program_prefix ;;  
  -program-prefix=* | --program-prefix=* | --program-prefi=* \  
  | --program-pref=* | --program-pre=* | --program-pr=* | --program-p=*)  
    program_prefix=$ac_optarg ;;  
  
  -program-suffix | --program-suffix | --program-suffi | --program-suff \  
  | --program-suf | --program-su | --program-s)  
    ac_prev=program_suffix ;;  
  -program-suffix=* | --program-suffix=* | --program-suffi=* \  
  | --program-suff=* | --program-suf=* | --program-su=* | --program-s=*)  
    program_suffix=$ac_optarg ;;  
  
  -program-transform-name | --program-transform-name \  
  | --program-transform-nam | --program-transform-na \  
  | --program-transform-n | --program-transform- \  
  | --program-transform | --program-transfor \  
  | --program-transfo | --program-transf \  
  | --program-trans | --program-tran \  
  | --progr-tra | --program-tr | --program-t)  
    ac_prev=program_transform_name ;;  
  -program-transform-name=* | --program-transform-name=* \  
  | --program-transform-nam=* | --program-transform-na=* \  
  | --program-transform-n=* | --program-transform-=* \  
  | --program-transform=* | --program-transfor=* \  
  | --program-transfo=* | --program-transf=* \  
  | --program-trans=* | --program-tran=* \  
  | --progr-tra=* | --program-tr=* | --program-t=*)  
    program_transform_name=$ac_optarg ;;  
  
  -pdfdir | --pdfdir | --pdfdi | --pdfd | --pdf | --pd)  
    ac_prev=pdfdir ;;  
  -pdfdir=* | --pdfdir=* | --pdfdi=* | --pdfd=* | --pdf=* | --pd=*)  
    pdfdir=$ac_optarg ;;  
  
  -psdir | --psdir | --psdi | --psd | --ps)  
    ac_prev=psdir ;;  
  -psdir=* | --psdir=* | --psdi=* | --psd=* | --ps=*)  
    psdir=$ac_optarg ;;  
  
  -q | -quiet | --quiet | --quie | --qui | --qu | --q \  
  | -silent | --silent | --silen | --sile | --sil)  
    silent=yes ;;  
  
  -sbindir | --sbindir | --sbindi | --sbind | --sbin | --sbi | --sb)  
    ac_prev=sbindir ;;  
  -sbindir=* | --sbindir=* | --sbindi=* | --sbind=* | --sbin=* \  
  | --sbi=* | --sb=*)  
    sbindir=$ac_optarg ;;  
  
  -sharedstatedir | --sharedstatedir | --sharedstatedi \  
  | --sharedstated | --sharedstate | --sharedstat | --sharedsta \  
  | --sharedst | --shareds | --shared | --share | --shar \  
  | --sha | --sh)  
    ac_prev=sharedstatedir ;;  
  -sharedstatedir=* | --sharedstatedir=* | --sharedstatedi=* \  
  | --sharedstated=* | --sharedstate=* | --sharedstat=* | --sharedsta=* \  
  | --sharedst=* | --shareds=* | --shared=* | --share=* | --shar=* \  
  | --sha=* | --sh=*)  
    sharedstatedir=$ac_optarg ;;  
  
  -site | --site | --sit)  
    ac_prev=site ;;  
  -site=* | --site=* | --sit=*)  
    site=$ac_optarg ;;  
  
  -srcdir | --srcdir | --srcdi | --srcd | --src | --sr)  
    ac_prev=srcdir ;;  
  -srcdir=* | --srcdir=* | --srcdi=* | --srcd=* | --src=* | --sr=*)  
    srcdir=$ac_optarg ;;  
  
  -sysconfdir | --sysconfdir | --sysconfdi | --sysconfd | --sysconf \  
  | --syscon | --sysco | --sysc | --sys | --sy)  
    ac_prev=sysconfdir ;;  
  -sysconfdir=* | --sysconfdir=* | --sysconfdi=* | --sysconfd=* | --sysconf=* \  
  | --syscon=* | --sysco=* | --sysc=* | --sys=* | --sy=*)  
    sysconfdir=$ac_optarg ;;  
  
  -target | --target | --targe | --targ | --tar | --ta | --t)  
    ac_prev=target_alias ;;  
  -target=* | --target=* | --targe=* | --targ=* | --tar=* | --ta=* | --t=*)  
    target_alias=$ac_optarg ;;  
  
  -v | -verbose | --verbose | --verbos | --verbo | --verb)  
    verbose=yes ;;  
  
  -version | --version | --versio | --versi | --vers | -V)  
    ac_init_version=: ;;  
  
  -with-* | --with-*)  
    ac_useropt=`expr "x$ac_option" : 'x-*with-\([^=]*\)'`  
    # Reject names that are not valid shell variable names.  
    expr "x$ac_useropt" : ".*[^-+._$as_cr_alnum]" >/dev/null &&  
      as_fn_error $? "invalid package name: $ac_useropt"  
    ac_useropt_orig=$ac_useropt  
    ac_useropt=`$as_echo "$ac_useropt" | sed 's/[-+.]/_/g'`  
    case $ac_user_opts in  
      *"  
"with_$ac_useropt"  
"*) ;;  
      *) ac_unrecognized_opts="$ac_unrecognized_opts$ac_unrecognized_sep--with-$ac_useropt_orig"  
     ac_unrecognized_sep=', ';;  
    esac  
    eval with_$ac_useropt=\$ac_optarg ;;  
  
  -without-* | --without-*)  
    ac_useropt=`expr "x$ac_option" : 'x-*without-\(.*\)'`  
    # Reject names that are not valid shell variable names.  
    expr "x$ac_useropt" : ".*[^-+._$as_cr_alnum]" >/dev/null &&  
      as_fn_error $? "invalid package name: $ac_useropt"  
    ac_useropt_orig=$ac_useropt  
    ac_useropt=`$as_echo "$ac_useropt" | sed 's/[-+.]/_/g'`  
    case $ac_user_opts in  
      *"  
"with_$ac_useropt"  
"*) ;;  
      *) ac_unrecognized_opts="$ac_unrecognized_opts$ac_unrecognized_sep--without-$ac_useropt_orig"  
     ac_unrecognized_sep=', ';;  
    esac  
    eval with_$ac_useropt=no ;;  
  
  --x)  
    # Obsolete; use --with-x.  
    with_x=yes ;;  
  
  -x-includes | --x-includes | --x-include | --x-includ | --x-inclu \  
  | --x-incl | --x-inc | --x-in | --x-i)  
    ac_prev=x_includes ;;  
  -x-includes=* | --x-includes=* | --x-include=* | --x-includ=* | --x-inclu=* \  
  | --x-incl=* | --x-inc=* | --x-in=* | --x-i=*)  
    x_includes=$ac_optarg ;;  
  
  -x-libraries | --x-libraries | --x-librarie | --x-librari \  
  | --x-librar | --x-libra | --x-libr | --x-lib | --x-li | --x-l)  
    ac_prev=x_libraries ;;  
  -x-libraries=* | --x-libraries=* | --x-librarie=* | --x-librari=* \  
  | --x-librar=* | --x-libra=* | --x-libr=* | --x-lib=* | --x-li=* | --x-l=*)  
    x_libraries=$ac_optarg ;;  
  
  -*) as_fn_error $? "unrecognized option: \`$ac_option'  
Try \`$0 --help' for more information"  
    ;;  
  
  *=*)  
    ac_envvar=`expr "x$ac_option" : 'x\([^=]*\)='`  
    # Reject names that are not valid shell variable names.  
    case $ac_envvar in #(  
      '' | [0-9]* | *[!_$as_cr_alnum]* )  
      as_fn_error $? "invalid variable name: \`$ac_envvar'" ;;  
    esac  
    eval $ac_envvar=\$ac_optarg  
    export $ac_envvar ;;  
  
  *)  
    # FIXME: should be removed in autoconf 3.0.  
    $as_echo "$as_me: WARNING: you should use --build, --host, --target" >&2  
    expr "x$ac_option" : ".*[^-._$as_cr_alnum]" >/dev/null &&  
      $as_echo "$as_me: WARNING: invalid host type: $ac_option" >&2  
    : "${build_alias=$ac_option} ${host_alias=$ac_option} ${target_alias=$ac_option}"  
    ;;  
  
  esac  
done  
  
if test -n "$ac_prev"; then  
  ac_option=--`echo $ac_prev | sed 's/_/-/g'`  
  as_fn_error $? "missing argument to $ac_option"  
fi  
  
if test -n "$ac_unrecognized_opts"; then  
  case $enable_option_checking in  
    no) ;;  
    fatal) as_fn_error $? "unrecognized options: $ac_unrecognized_opts" ;;  
    *)     $as_echo "$as_me: WARNING: unrecognized options: $ac_unrecognized_opts" >&2 ;;  
  esac  
fi  
  
# Check all directory arguments for consistency.  
for ac_var in    exec_prefix prefix bindir sbindir libexecdir datarootdir \  
        datadir sysconfdir sharedstatedir localstatedir includedir \  
        oldincludedir docdir infodir htmldir dvidir pdfdir psdir \  
        libdir localedir mandir  
do  
  eval ac_val=\$$ac_var  
  # Remove trailing slashes.  
  case $ac_val in  
    */ )  
      ac_val=`expr "X$ac_val" : 'X\(.*[^/]\)' \| "X$ac_val" : 'X\(.*\)'`  
      eval $ac_var=\$ac_val;;  
  esac  
  # Be sure to have absolute directory names.  
  case $ac_val in  
    [\\/$]* | ?:[\\/]* )  continue;;  
    NONE | '' ) case $ac_var in *prefix ) continue;; esac;;  
  esac  
  as_fn_error $? "expected an absolute directory name for --$ac_var: $ac_val"  
done  
  
# There might be people who depend on the old broken behavior: `$host'  
# used to hold the argument of --host etc.  
# FIXME: To remove some day.  
build=$build_alias  
host=$host_alias  
target=$target_alias  
  
# FIXME: To remove some day.  
if test "x$host_alias" != x; then  
  if test "x$build_alias" = x; then  
    cross_compiling=maybe  
  elif test "x$build_alias" != "x$host_alias"; then  
    cross_compiling=yes  
  fi  
fi  
  
ac_tool_prefix=  
test -n "$host_alias" && ac_tool_prefix=$host_alias-  
  
test "$silent" = yes && exec 6>/dev/null  
  
  
ac_pwd=`pwd` && test -n "$ac_pwd" &&  
ac_ls_di=`ls -di .` &&  
ac_pwd_ls_di=`cd "$ac_pwd" && ls -di .` ||  
  as_fn_error $? "working directory cannot be determined"  
test "X$ac_ls_di" = "X$ac_pwd_ls_di" ||  
  as_fn_error $? "pwd does not report name of working directory"  
  
  
# Find the source files, if location was not specified.  
if test -z "$srcdir"; then  
  ac_srcdir_defaulted=yes  
  # Try the directory containing this script, then the parent directory.  
  ac_confdir=`$as_dirname -- "$as_myself" ||  
$as_expr X"$as_myself" : 'X\(.*[^/]\)//*[^/][^/]*/*$' \| \  
     X"$as_myself" : 'X\(//\)[^/]' \| \  
     X"$as_myself" : 'X\(//\)$' \| \  
     X"$as_myself" : 'X\(/\)' \| . 2>/dev/null ||  
$as_echo X"$as_myself" |  
    sed '/^X\(.*[^/]\)\/\/*[^/][^/]*\/*$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)[^/].*/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\).*/{  
        s//\1/  
        q  
      }  
      s/.*/./; q'`  
  srcdir=$ac_confdir  
  if test ! -r "$srcdir/$ac_unique_file"; then  
    srcdir=..  
  fi  
else  
  ac_srcdir_defaulted=no  
fi  
if test ! -r "$srcdir/$ac_unique_file"; then  
  test "$ac_srcdir_defaulted" = yes && srcdir="$ac_confdir or .."  
  as_fn_error $? "cannot find sources ($ac_unique_file) in $srcdir"  
fi  
ac_msg="sources are in $srcdir, but \`cd $srcdir' does not work"  
ac_abs_confdir=`(  
    cd "$srcdir" && test -r "./$ac_unique_file" || as_fn_error $? "$ac_msg"  
    pwd)`  
# When building in place, set srcdir=.  
if test "$ac_abs_confdir" = "$ac_pwd"; then  
  srcdir=.  
fi  
# Remove unnecessary trailing slashes from srcdir.  
# Double slashes in file names in object file debugging info  
# mess up M-x gdb in Emacs.  
case $srcdir in  
*/) srcdir=`expr "X$srcdir" : 'X\(.*[^/]\)' \| "X$srcdir" : 'X\(.*\)'`;;  
esac  
case $srcdir in  
  *" "*)  
    as_fn_error $? "path to source, $srcdir, contains spaces"  
    ;;  
esac  
ac_subdirs_all=`cd $srcdir && echo */configure | sed 's,/configure,,g'`  
  
for ac_var in $ac_precious_vars; do  
  eval ac_env_${ac_var}_set=\${${ac_var}+set}  
  eval ac_env_${ac_var}_value=\$${ac_var}  
  eval ac_cv_env_${ac_var}_set=\${${ac_var}+set}  
  eval ac_cv_env_${ac_var}_value=\$${ac_var}  
done  
  
#  
# Report the --help message.  
#  
if test "$ac_init_help" = "long"; then  
  # Omit some internal or obsolete options to make the list less imposing.  
  # This message is too long to be a string in the A/UX 3.1 sh.  
  cat \<<_ACEOF  
\`configure' configures this package to adapt to many kinds of systems.  
  
Usage: $0 [OPTION]... [VAR=VALUE]...  
  
To assign environment variables (e.g., CC, CFLAGS...), specify them as  
VAR=VALUE.  See below for descriptions of some of the useful variables.  
  
Defaults for the options are specified in brackets.  
  
Configuration:  
  -h, --help              display this help and exit  
      --help=short        display options specific to this package  
      --help=recursive    display the short help of all the included packages  
  -V, --version           display version information and exit  
  -q, --quiet, --silent   do not print \`checking ...' messages  
      --cache-file=FILE   cache test results in FILE [disabled]  
  -C, --config-cache      alias for \`--cache-file=config.cache'  
  -n, --no-create         do not create output files  
      --srcdir=DIR        find the sources in DIR [configure dir or \`..']  
  
Installation directories:  
  --prefix=PREFIX         install architecture-independent files in PREFIX  
                          [$ac_default_prefix]  
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX  
                          [PREFIX]  
  
By default, \`make install' will install all the files in  
\`$ac_default_prefix/bin', \`$ac_default_prefix/lib' etc.  You can specify  
an installation prefix other than \`$ac_default_prefix' using \`--prefix',  
for instance \`--prefix=\$HOME'.  
  
For better control, use the options below.  
  
Fine tuning of the installation directories:  
  --bindir=DIR            user executables [EPREFIX/bin]  
  --sbindir=DIR           system admin executables [EPREFIX/sbin]  
  --libexecdir=DIR        program executables [EPREFIX/libexec]  
  --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]  
  --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]  
  --localstatedir=DIR     modifiable single-machine data [PREFIX/var]  
  --libdir=DIR            object code libraries [EPREFIX/lib]  
  --includedir=DIR        C header files [PREFIX/include]  
  --oldincludedir=DIR     C header files for non-gcc [/usr/include]  
  --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]  
  --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]  
  --infodir=DIR           info documentation [DATAROOTDIR/info]  
  --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]  
  --mandir=DIR            man documentation [DATAROOTDIR/man]  
  --docdir=DIR            documentation root [DATAROOTDIR/doc/PACKAGE]  
  --htmldir=DIR           html documentation [DOCDIR]  
  --dvidir=DIR            dvi documentation [DOCDIR]  
  --pdfdir=DIR            pdf documentation [DOCDIR]  
  --psdir=DIR             ps documentation [DOCDIR]  
_ACEOF  
  
  cat \<<\_ACEOF  
  
Program names:  
  --program-prefix=PREFIX            prepend PREFIX to installed program names  
  --program-suffix=SUFFIX            append SUFFIX to installed program names  
  --program-transform-name=PROGRAM   run sed PROGRAM on installed program names  
  
System types:  
  --build=BUILD     configure for building on BUILD [guessed]  
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]  
  --target=TARGET   configure for building compilers for TARGET [HOST]  
_ACEOF  
fi  
  
if test -n "$ac_init_help"; then  
  
  cat \<<\_ACEOF  
  
Optional Features:  
  --disable-option-checking  ignore unrecognized --enable/--with options  
  --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)  
  --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]  
  --enable-as-accelerator-for=ARG  
                          build as offload target compiler. Specify offload  
                          host triple by ARG  
  --enable-offload-targets=LIST  
                          enable offloading to devices from comma-separated  
                          LIST of TARGET[=DIR]. Use optional path to find  
                          offload target compiler during the build  
  --enable-offload-defaulted  
        If enabled, configured but not installed offload compilers and  
        libgomp plugins are silently ignored.  Useful for distribution  
        compilers where those are in separate optional packages.  
  
  --enable-gold[=ARG]     build gold [ARG={default,yes,no}]  
  --enable-ld[=ARG]       build ld [ARG={default,yes,no}]  
  --enable-gprofng[=ARG]  build gprofng [ARG={yes,no}]  
  --enable-compressed-debug-sections={all,gas,gold,ld,none}  
                          Enable compressed debug sections for gas, gold or ld  
                          by default  
  --enable-default-compressed-debug-sections-algorithm={zlib,zstd}  
                          Default compression algorithm for  
                          --enable-compressed-debug-sections.  
  --enable-year2038       enable support for timestamps past the year 2038  
  --disable-libquadmath   do not build libquadmath directory  
  --disable-libquadmath-support  
                          disable libquadmath support for Fortran  
  --enable-libada         build libada directory  
  --enable-libgm2         build libgm2 directory  
  --enable-libssp         build libssp directory  
  --disable-libstdcxx     do not build libstdc++-v3 directory  
  --enable-bootstrap      enable bootstrapping [yes if native build]  
  --enable-pgo-build[=lto]  
                          enable the PGO build  
  --disable-isl-version-check  
                          disable check for isl version  
  --enable-lto            enable link time optimization support  
  --enable-linker-plugin-configure-flags=FLAGS  
                          additional flags for configuring linker plugins  
                          [none]  
  --enable-linker-plugin-flags=FLAGS  
                          additional flags for configuring and building linker  
                          plugins [none]  
  --enable-host-pie       build position independent host executables  
  --enable-host-shared    build host code as shared libraries  
  --enable-stage1-languages[=all]  
                          choose additional languages to build during stage1.  
                          Mostly useful for compiler development  
  --enable-objc-gc        enable use of Boehm's garbage collector with the GNU  
                          Objective-C runtime  
  --enable-vtable-verify  Enable vtable verification feature  
  --enable-serial-[{host,target,build}-]configure  
                          force sequential configuration of sub-packages for  
                          the host, target or build machine, or all  
                          sub-packages  
  --enable-maintainer-mode  
                          enable make rules and dependencies not useful (and  
                          sometimes confusing) to the casual installer  
  --enable-stage1-checking[=all]  
                          choose additional checking for stage1 of the  
                          compiler  
  --enable-werror         enable -Werror in bootstrap stage2 and later  
  
Optional Packages:  
  --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]  
  --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)  
  --with-build-libsubdir=DIR  Directory where to find libraries for build system  
  --with-system-zlib      use installed libz  
  --with-zstd             Support zstd compressed debug sections  
                          (default=auto)  
  --with-mpc=PATH         specify prefix directory for installed MPC package.  
                          Equivalent to --with-mpc-include=PATH/include plus  
                          --with-mpc-lib=PATH/lib  
  --with-mpc-include=PATH specify directory for installed MPC include files  
  --with-mpc-lib=PATH     specify directory for the installed MPC library  
  --with-mpfr=PATH        specify prefix directory for installed MPFR package.  
                          Equivalent to --with-mpfr-include=PATH/include plus  
                          --with-mpfr-lib=PATH/lib  
  --with-mpfr-include=PATH  
                          specify directory for installed MPFR include files  
  --with-mpfr-lib=PATH    specify directory for the installed MPFR library  
  --with-gmp=PATH         specify prefix directory for the installed GMP  
                          package. Equivalent to  
                          --with-gmp-include=PATH/include plus  
                          --with-gmp-lib=PATH/lib  
  --with-gmp-include=PATH specify directory for installed GMP include files  
  --with-gmp-lib=PATH     specify directory for the installed GMP library  
  --with-stage1-libs=LIBS libraries for stage1  
  --with-static-standard-libraries  
                          use -static-libstdc++ and -static-libgcc  
                          (default=auto)  
  --with-stage1-ldflags=FLAGS  
                          linker flags for stage1  
  --with-boot-libs=LIBS   libraries for stage2 and later  
  --with-boot-ldflags=FLAGS  
                          linker flags for stage2 and later  
  --with-isl=PATH         Specify prefix directory for the installed isl  
                          package. Equivalent to  
                          --with-isl-include=PATH/include plus  
                          --with-isl-lib=PATH/lib  
  --with-isl-include=PATH Specify directory for installed isl include files  
  --with-isl-lib=PATH     Specify the directory for the installed isl library  
  --with-target-bdw-gc=PATHLIST  
                          specify prefix directory for installed bdw-gc  
                          package. Equivalent to  
                          --with-target-bdw-gc-include=PATH/include plus  
                          --with-target-bdw-gc-lib=PATH/lib  
  --with-target-bdw-gc-include=PATHLIST  
                          specify directories for installed bdw-gc include  
                          files  
  --with-target-bdw-gc-lib=PATHLIST  
                          specify directories for installed bdw-gc library  
  --with-gcc-major-version-only  
                          use only GCC major number in filesystem paths  
  --with-build-sysroot=SYSROOT  
                          use sysroot as the system root during the build  
  --with-debug-prefix-map='A=B C=D ...'  
                          map A to B, C to D ... in debug information  
  --with-build-config='NAME NAME2...'  
                          use config/NAME.mk build configuration  
  --with-build-time-tools=PATH  
                          use given path to find target tools during the build  
  
Some influential environment variables:  
  CC          C compiler command  
  CFLAGS      C compiler flags  
  LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a  
              nonstandard directory <lib dir>  
  LIBS        libraries to pass to the linker, e.g. -l<library>  
  CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if  
              you have headers in a nonstandard directory <include dir>  
  CXX         C++ compiler command  
  CXXFLAGS    C++ compiler flags  
  build_configargs  
              additional configure arguments for build directories  
  host_configargs  
              additional configure arguments for host directories  
  target_configargs  
              additional configure arguments for target directories  
  AR          AR for the host  
  AS          AS for the host  
  DLLTOOL     DLLTOOL for the host  
  DSYMUTIL    DSYMUTIL for the host  
  LD          LD for the host  
  LIPO        LIPO for the host  
  NM          NM for the host  
  RANLIB      RANLIB for the host  
  STRIP       STRIP for the host  
  WINDRES     WINDRES for the host  
  WINDMC      WINDMC for the host  
  OBJCOPY     OBJCOPY for the host  
  OBJDUMP     OBJDUMP for the host  
  OTOOL       OTOOL for the host  
  READELF     READELF for the host  
  CC_FOR_TARGET  
              CC for the target  
  CXX_FOR_TARGET  
              CXX for the target  
  GCC_FOR_TARGET  
              GCC for the target  
  GFORTRAN_FOR_TARGET  
              GFORTRAN for the target  
  GOC_FOR_TARGET  
              GOC for the target  
  GDC_FOR_TARGET  
              GDC for the target  
  GM2_FOR_TARGET  
              GM2 for the target  
  AR_FOR_TARGET  
              AR for the target  
  AS_FOR_TARGET  
              AS for the target  
  DLLTOOL_FOR_TARGET  
              DLLTOOL for the target  
  DSYMUTIL_FOR_TARGET  
              DSYMUTIL for the target  
  LD_FOR_TARGET  
              LD for the target  
  LIPO_FOR_TARGET  
              LIPO for the target  
  NM_FOR_TARGET  
              NM for the target  
  OBJCOPY_FOR_TARGET  
              OBJCOPY for the target  
  OBJDUMP_FOR_TARGET  
              OBJDUMP for the target  
  OTOOL_FOR_TARGET  
              OTOOL for the target  
  RANLIB_FOR_TARGET  
              RANLIB for the target  
  READELF_FOR_TARGET  
              READELF for the target  
  STRIP_FOR_TARGET  
              STRIP for the target  
  WINDRES_FOR_TARGET  
              WINDRES for the target  
  WINDMC_FOR_TARGET  
              WINDMC for the target  
  
Use these variables to override the choices made by `configure' or to help  
it to find libraries and programs with nonstandard names/locations.  
  
Report bugs to the package provider.  
_ACEOF  
ac_status=$?  
fi  
  
if test "$ac_init_help" = "recursive"; then  
  # If there are subdirs, report their specific --help.  
  for ac_dir in : $ac_subdirs_all; do test "x$ac_dir" = x: && continue  
    test -d "$ac_dir" ||  
      { cd "$srcdir" && ac_pwd=`pwd` && srcdir=. && test -d "$ac_dir"; } ||  
      continue  
    ac_builddir=.  
  
case "$ac_dir" in  
.) ac_dir_suffix= ac_top_builddir_sub=. ac_top_build_prefix= ;;  
*)  
  ac_dir_suffix=/`$as_echo "$ac_dir" | sed 's|^\.[\\/]||'`  
  # A ".." for each directory in $ac_dir_suffix.  
  ac_top_builddir_sub=`$as_echo "$ac_dir_suffix" | sed 's|/[^\\/]*|/..|g;s|/||'`  
  case $ac_top_builddir_sub in  
  "") ac_top_builddir_sub=. ac_top_build_prefix= ;;  
  *)  ac_top_build_prefix=$ac_top_builddir_sub/ ;;  
  esac ;;  
esac  
ac_abs_top_builddir=$ac_pwd  
ac_abs_builddir=$ac_pwd$ac_dir_suffix  
# for backward compatibility:  
ac_top_builddir=$ac_top_build_prefix  
  
case $srcdir in  
  .)  # We are building in place.  
    ac_srcdir=.  
    ac_top_srcdir=$ac_top_builddir_sub  
    ac_abs_top_srcdir=$ac_pwd ;;  
  [\\/]* | ?:[\\/]* )  # Absolute name.  
    ac_srcdir=$srcdir$ac_dir_suffix;  
    ac_top_srcdir=$srcdir  
    ac_abs_top_srcdir=$srcdir ;;  
  *) # Relative name.  
    ac_srcdir=$ac_top_build_prefix$srcdir$ac_dir_suffix  
    ac_top_srcdir=$ac_top_build_prefix$srcdir  
    ac_abs_top_srcdir=$ac_pwd/$srcdir ;;  
esac  
ac_abs_srcdir=$ac_abs_top_srcdir$ac_dir_suffix  
  
    cd "$ac_dir" || { ac_status=$?; continue; }  
    # Check for guested configure.  
    if test -f "$ac_srcdir/configure.gnu"; then  
      echo &&  
      $SHELL "$ac_srcdir/configure.gnu" --help=recursive  
    elif test -f "$ac_srcdir/configure"; then  
      echo &&  
      $SHELL "$ac_srcdir/configure" --help=recursive  
    else  
      $as_echo "$as_me: WARNING: no configuration information is in $ac_dir" >&2  
    fi || ac_status=$?  
    cd "$ac_pwd" || { ac_status=$?; break; }  
  done  
fi  
  
test -n "$ac_init_help" && exit $ac_status  
if $ac_init_version; then  
  cat \<<\_ACEOF  
configure  
generated by GNU Autoconf 2.69  
  
Copyright (C) 2012 Free Software Foundation, Inc.  
This configure script is free software; the Free Software Foundation  
gives unlimited permission to copy, distribute and modify it.  
_ACEOF  
  exit  
fi  
  
## ------------------------ ##  
## Autoconf initialization. ##  
## ------------------------ ##  
  
# ac_fn_c_try_compile LINENO  
# --------------------------  
# Try to compile conftest.$ac_ext, and return whether this succeeded.  
ac_fn_c_try_compile ()  
{  
  as_lineno=${as_lineno-"$1"} as_lineno_stack=as_lineno_stack=$as_lineno_stack  
  rm -f conftest.$ac_objext  
  if { { ac_try="$ac_compile"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_compile") 2>conftest.err  
  ac_status=$?  
  if test -s conftest.err; then  
    grep -v '^ *+' conftest.err >conftest.er1  
    cat conftest.er1 >&5  
    mv -f conftest.er1 conftest.err  
  fi  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; } && {  
     test -z "$ac_c_werror_flag" ||  
     test ! -s conftest.err  
       } && test -s conftest.$ac_objext; then :  
  ac_retval=0  
else  
  $as_echo "$as_me: failed program was:" >&5  
sed 's/^/| /' conftest.$ac_ext >&5  
  
    ac_retval=1  
fi  
  eval $as_lineno_stack; ${as_lineno_stack:+:} unset as_lineno  
  as_fn_set_status $ac_retval  
  
} # ac_fn_c_try_compile  
  
# ac_fn_cxx_try_compile LINENO  
# ----------------------------  
# Try to compile conftest.$ac_ext, and return whether this succeeded.  
ac_fn_cxx_try_compile ()  
{  
  as_lineno=${as_lineno-"$1"} as_lineno_stack=as_lineno_stack=$as_lineno_stack  
  rm -f conftest.$ac_objext  
  if { { ac_try="$ac_compile"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_compile") 2>conftest.err  
  ac_status=$?  
  if test -s conftest.err; then  
    grep -v '^ *+' conftest.err >conftest.er1  
    cat conftest.er1 >&5  
    mv -f conftest.er1 conftest.err  
  fi  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; } && {  
     test -z "$ac_cxx_werror_flag" ||  
     test ! -s conftest.err  
       } && test -s conftest.$ac_objext; then :  
  ac_retval=0  
else  
  $as_echo "$as_me: failed program was:" >&5  
sed 's/^/| /' conftest.$ac_ext >&5  
  
    ac_retval=1  
fi  
  eval $as_lineno_stack; ${as_lineno_stack:+:} unset as_lineno  
  as_fn_set_status $ac_retval  
  
} # ac_fn_cxx_try_compile  
  
# ac_fn_cxx_try_link LINENO  
# -------------------------  
# Try to link conftest.$ac_ext, and return whether this succeeded.  
ac_fn_cxx_try_link ()  
{  
  as_lineno=${as_lineno-"$1"} as_lineno_stack=as_lineno_stack=$as_lineno_stack  
  rm -f conftest.$ac_objext conftest$ac_exeext  
  if { { ac_try="$ac_link"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_link") 2>conftest.err  
  ac_status=$?  
  if test -s conftest.err; then  
    grep -v '^ *+' conftest.err >conftest.er1  
    cat conftest.er1 >&5  
    mv -f conftest.er1 conftest.err  
  fi  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; } && {  
     test -z "$ac_cxx_werror_flag" ||  
     test ! -s conftest.err  
       } && test -s conftest$ac_exeext && {  
     test "$cross_compiling" = yes ||  
     test -x conftest$ac_exeext  
       }; then :  
  ac_retval=0  
else  
  $as_echo "$as_me: failed program was:" >&5  
sed 's/^/| /' conftest.$ac_ext >&5  
  
    ac_retval=1  
fi  
  # Delete the IPA/IPO (Inter Procedural Analysis/Optimization) information  
  # created by the PGI compiler (conftest_ipa8_conftest.oo), as it would  
  # interfere with the next link command; also delete a directory that is  
  # left behind by Apple's compiler.  We do this before executing the actions.  
  rm -rf conftest.dSYM conftest_ipa8_conftest.oo  
  eval $as_lineno_stack; ${as_lineno_stack:+:} unset as_lineno  
  as_fn_set_status $ac_retval  
  
} # ac_fn_cxx_try_link  
  
# ac_fn_c_try_link LINENO  
# -----------------------  
# Try to link conftest.$ac_ext, and return whether this succeeded.  
ac_fn_c_try_link ()  
{  
  as_lineno=${as_lineno-"$1"} as_lineno_stack=as_lineno_stack=$as_lineno_stack  
  rm -f conftest.$ac_objext conftest$ac_exeext  
  if { { ac_try="$ac_link"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_link") 2>conftest.err  
  ac_status=$?  
  if test -s conftest.err; then  
    grep -v '^ *+' conftest.err >conftest.er1  
    cat conftest.er1 >&5  
    mv -f conftest.er1 conftest.err  
  fi  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; } && {  
     test -z "$ac_c_werror_flag" ||  
     test ! -s conftest.err  
       } && test -s conftest$ac_exeext && {  
     test "$cross_compiling" = yes ||  
     test -x conftest$ac_exeext  
       }; then :  
  ac_retval=0  
else  
  $as_echo "$as_me: failed program was:" >&5  
sed 's/^/| /' conftest.$ac_ext >&5  
  
    ac_retval=1  
fi  
  # Delete the IPA/IPO (Inter Procedural Analysis/Optimization) information  
  # created by the PGI compiler (conftest_ipa8_conftest.oo), as it would  
  # interfere with the next link command; also delete a directory that is  
  # left behind by Apple's compiler.  We do this before executing the actions.  
  rm -rf conftest.dSYM conftest_ipa8_conftest.oo  
  eval $as_lineno_stack; ${as_lineno_stack:+:} unset as_lineno  
  as_fn_set_status $ac_retval  
  
} # ac_fn_c_try_link  
cat >config.log \<<_ACEOF  
This file contains any messages produced by compilers while  
running configure, to aid debugging if configure makes a mistake.  
  
It was created by $as_me, which was  
generated by GNU Autoconf 2.69.  Invocation command line was  
  
  $ $0 $@  
  
_ACEOF  
exec 5>>config.log  
{  
cat \<<_ASUNAME  
## --------- ##  
## Platform. ##  
## --------- ##  
  
hostname = `(hostname || uname -n) 2>/dev/null | sed 1q`  
uname -m = `(uname -m) 2>/dev/null || echo unknown`  
uname -r = `(uname -r) 2>/dev/null || echo unknown`  
uname -s = `(uname -s) 2>/dev/null || echo unknown`  
uname -v = `(uname -v) 2>/dev/null || echo unknown`  
  
/usr/bin/uname -p = `(/usr/bin/uname -p) 2>/dev/null || echo unknown`  
/bin/uname -X     = `(/bin/uname -X) 2>/dev/null     || echo unknown`  
  
/bin/arch              = `(/bin/arch) 2>/dev/null              || echo unknown`  
/usr/bin/arch -k       = `(/usr/bin/arch -k) 2>/dev/null       || echo unknown`  
/usr/convex/getsysinfo = `(/usr/convex/getsysinfo) 2>/dev/null || echo unknown`  
/usr/bin/hostinfo      = `(/usr/bin/hostinfo) 2>/dev/null      || echo unknown`  
/bin/machine           = `(/bin/machine) 2>/dev/null           || echo unknown`  
/usr/bin/oslevel       = `(/usr/bin/oslevel) 2>/dev/null       || echo unknown`  
/bin/universe          = `(/bin/universe) 2>/dev/null          || echo unknown`  
  
_ASUNAME  
  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    $as_echo "PATH: $as_dir"  
  done  
IFS=$as_save_IFS  
  
} >&5  
  
cat >&5 \<<_ACEOF  
  
  
## ----------- ##  
## Core tests. ##  
## ----------- ##  
  
_ACEOF  
  
  
# Keep a trace of the command line.  
# Strip out --no-create and --no-recursion so they do not pile up.  
# Strip out --silent because we don't want to record it for future runs.  
# Also quote any args containing shell meta-characters.  
# Make two passes to allow for proper duplicate-argument suppression.  
ac_configure_args=  
ac_configure_args0=  
ac_configure_args1=  
ac_must_keep_next=false  
for ac_pass in 1 2  
do  
  for ac_arg  
  do  
    case $ac_arg in  
    -no-create | --no-c* | -n | -no-recursion | --no-r*) continue ;;  
    -q | -quiet | --quiet | --quie | --qui | --qu | --q \  
    | -silent | --silent | --silen | --sile | --sil)  
      continue ;;  
    *\'*)  
      ac_arg=`$as_echo "$ac_arg" | sed "s/'/'\\\\\\\\''/g"` ;;  
    esac  
    case $ac_pass in  
    1) as_fn_append ac_configure_args0 " '$ac_arg'" ;;  
    2)  
      as_fn_append ac_configure_args1 " '$ac_arg'"  
      if test $ac_must_keep_next = true; then  
    ac_must_keep_next=false # Got value, back to normal.  
      else  
    case $ac_arg in  
      *=* | --config-cache | -C | -disable-* | --disable-* \  
      | -enable-* | --enable-* | -gas | --g* | -nfp | --nf* \  
      | -q | -quiet | --q* | -silent | --sil* | -v | -verb* \  
      | -with-* | --with-* | -without-* | --without-* | --x)  
        case "$ac_configure_args0 " in  
          "$ac_configure_args1"*" '$ac_arg' "* ) continue ;;  
        esac  
        ;;  
      -* ) ac_must_keep_next=true ;;  
    esac  
      fi  
      as_fn_append ac_configure_args " '$ac_arg'"  
      ;;  
    esac  
  done  
done  
{ ac_configure_args0=; unset ac_configure_args0;}  
{ ac_configure_args1=; unset ac_configure_args1;}  
  
# When interrupted or exit'd, cleanup temporary files, and complete  
# config.log.  We remove comments because anyway the quotes in there  
# would cause problems or look ugly.  
# WARNING: Use '\'' to represent an apostrophe within the trap.  
# WARNING: Do not start the trap code with a newline, due to a FreeBSD 4.0 bug.  
trap 'exit_status=$?  
  # Save into config.log some information that might help in debugging.  
  {  
    echo  
  
    $as_echo "## ---------------- ##  
## Cache variables. ##  
## ---------------- ##"  
    echo  
    # The following way of writing the cache mishandles newlines in values,  
(  
  for ac_var in `(set) 2>&1 | sed -n '\''s/^\([a-zA-Z_][a-zA-Z0-9_]*\)=.*/\1/p'\''`; do  
    eval ac_val=\$$ac_var  
    case $ac_val in #(  
    *${as_nl}*)  
      case $ac_var in #(  
      *_cv_*) { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: cache variable $ac_var contains a newline" >&5  
$as_echo "$as_me: WARNING: cache variable $ac_var contains a newline" >&2;} ;;  
      esac  
      case $ac_var in #(  
      _ | IFS | as_nl) ;; #(  
      BASH_ARGV | BASH_SOURCE) eval $ac_var= ;; #(  
      *) { eval $ac_var=; unset $ac_var;} ;;  
      esac ;;  
    esac  
  done  
  (set) 2>&1 |  
    case $as_nl`(ac_space='\'' '\''; set) 2>&1` in #(  
    *${as_nl}ac_space=\ *)  
      sed -n \  
    "s/'\''/'\''\\\\'\'''\''/g;  
      s/^\\([_$as_cr_alnum]*_cv_[_$as_cr_alnum]*\\)=\\(.*\\)/\\1='\''\\2'\''/p"  
      ;; #(  
    *)  
      sed -n "/^[_$as_cr_alnum]*_cv_[_$as_cr_alnum]*=/p"  
      ;;  
    esac |  
    sort  
)  
    echo  
  
    $as_echo "## ----------------- ##  
## Output variables. ##  
## ----------------- ##"  
    echo  
    for ac_var in $ac_subst_vars  
    do  
      eval ac_val=\$$ac_var  
      case $ac_val in  
      *\'\''*) ac_val=`$as_echo "$ac_val" | sed "s/'\''/'\''\\\\\\\\'\'''\''/g"`;;  
      esac  
      $as_echo "$ac_var='\''$ac_val'\''"  
    done | sort  
    echo  
  
    if test -n "$ac_subst_files"; then  
      $as_echo "## ------------------- ##  
## File substitutions. ##  
## ------------------- ##"  
      echo  
      for ac_var in $ac_subst_files  
      do  
    eval ac_val=\$$ac_var  
    case $ac_val in  
    *\'\''*) ac_val=`$as_echo "$ac_val" | sed "s/'\''/'\''\\\\\\\\'\'''\''/g"`;;  
    esac  
    $as_echo "$ac_var='\''$ac_val'\''"  
      done | sort  
      echo  
    fi  
  
    if test -s confdefs.h; then  
      $as_echo "## ----------- ##  
## confdefs.h. ##  
## ----------- ##"  
      echo  
      cat confdefs.h  
      echo  
    fi  
    test "$ac_signal" != 0 &&  
      $as_echo "$as_me: caught signal $ac_signal"  
    $as_echo "$as_me: exit $exit_status"  
  } >&5  
  rm -f core *.core core.conftest.* &&  
    rm -f -r conftest* confdefs* conf$$* $ac_clean_files &&  
    exit $exit_status  
' 0  
for ac_signal in 1 2 13 15; do  
  trap 'ac_signal='$ac_signal'; as_fn_exit 1' $ac_signal  
done  
ac_signal=0  
  
# confdefs.h avoids OS command line length limits that DEFS can exceed.  
rm -f -r conftest* confdefs.h  
  
$as_echo "/* confdefs.h */" > confdefs.h  
  
# Predefined preprocessor variables.  
  
cat >>confdefs.h \<<_ACEOF  
#define PACKAGE_NAME "$PACKAGE_NAME"  
_ACEOF  
  
cat >>confdefs.h \<<_ACEOF  
#define PACKAGE_TARNAME "$PACKAGE_TARNAME"  
_ACEOF  
  
cat >>confdefs.h \<<_ACEOF  
#define PACKAGE_VERSION "$PACKAGE_VERSION"  
_ACEOF  
  
cat >>confdefs.h \<<_ACEOF  
#define PACKAGE_STRING "$PACKAGE_STRING"  
_ACEOF  
  
cat >>confdefs.h \<<_ACEOF  
#define PACKAGE_BUGREPORT "$PACKAGE_BUGREPORT"  
_ACEOF  
  
cat >>confdefs.h \<<_ACEOF  
#define PACKAGE_URL "$PACKAGE_URL"  
_ACEOF  
  
  
# Let the site file select an alternate cache file if it wants to.  
# Prefer an explicitly selected file to automatically selected ones.  
ac_site_file1=NONE  
ac_site_file2=NONE  
if test -n "$CONFIG_SITE"; then  
  # We do not want a PATH search for config.site.  
  case $CONFIG_SITE in #((  
    -*)  ac_site_file1=./$CONFIG_SITE;;  
    */*) ac_site_file1=$CONFIG_SITE;;  
    *)   ac_site_file1=./$CONFIG_SITE;;  
  esac  
elif test "x$prefix" != xNONE; then  
  ac_site_file1=$prefix/share/config.site  
  ac_site_file2=$prefix/etc/config.site  
else  
  ac_site_file1=$ac_default_prefix/share/config.site  
  ac_site_file2=$ac_default_prefix/etc/config.site  
fi  
for ac_site_file in "$ac_site_file1" "$ac_site_file2"  
do  
  test "x$ac_site_file" = xNONE && continue  
  if test /dev/null != "$ac_site_file" && test -r "$ac_site_file"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: loading site script $ac_site_file" >&5  
$as_echo "$as_me: loading site script $ac_site_file" >&6;}  
    sed 's/^/| /' "$ac_site_file" >&5  
    . "$ac_site_file" \  
      || { { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
as_fn_error $? "failed to load site script $ac_site_file  
See \`config.log' for more details" "$LINENO" 5; }  
  fi  
done  
  
if test -r "$cache_file"; then  
  # Some versions of bash will fail to source /dev/null (special files  
  # actually), so we avoid doing that.  DJGPP emulates it as a regular file.  
  if test /dev/null != "$cache_file" && test -f "$cache_file"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: loading cache $cache_file" >&5  
$as_echo "$as_me: loading cache $cache_file" >&6;}  
    case $cache_file in  
      [\\/]* | ?:[\\/]* ) . "$cache_file";;  
      *)                      . "./$cache_file";;  
    esac  
  fi  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: creating cache $cache_file" >&5  
$as_echo "$as_me: creating cache $cache_file" >&6;}  
  >$cache_file  
fi  
  
# Check that the precious variables saved in the cache have kept the same  
# value.  
ac_cache_corrupted=false  
for ac_var in $ac_precious_vars; do  
  eval ac_old_set=\$ac_cv_env_${ac_var}_set  
  eval ac_new_set=\$ac_env_${ac_var}_set  
  eval ac_old_val=\$ac_cv_env_${ac_var}_value  
  eval ac_new_val=\$ac_env_${ac_var}_value  
  case $ac_old_set,$ac_new_set in  
    set,)  
      { $as_echo "$as_me:${as_lineno-$LINENO}: error: \`$ac_var' was set to \`$ac_old_val' in the previous run" >&5  
$as_echo "$as_me: error: \`$ac_var' was set to \`$ac_old_val' in the previous run" >&2;}  
      ac_cache_corrupted=: ;;  
    ,set)  
      { $as_echo "$as_me:${as_lineno-$LINENO}: error: \`$ac_var' was not set in the previous run" >&5  
$as_echo "$as_me: error: \`$ac_var' was not set in the previous run" >&2;}  
      ac_cache_corrupted=: ;;  
    ,);;  
    *)  
      if test "x$ac_old_val" != "x$ac_new_val"; then  
    # differences in whitespace do not lead to failure.  
    ac_old_val_w=`echo x $ac_old_val`  
    ac_new_val_w=`echo x $ac_new_val`  
    if test "$ac_old_val_w" != "$ac_new_val_w"; then  
      { $as_echo "$as_me:${as_lineno-$LINENO}: error: \`$ac_var' has changed since the previous run:" >&5  
$as_echo "$as_me: error: \`$ac_var' has changed since the previous run:" >&2;}  
      ac_cache_corrupted=:  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: warning: ignoring whitespace changes in \`$ac_var' since the previous run:" >&5  
$as_echo "$as_me: warning: ignoring whitespace changes in \`$ac_var' since the previous run:" >&2;}  
      eval $ac_var=\$ac_old_val  
    fi  
    { $as_echo "$as_me:${as_lineno-$LINENO}:   former value:  \`$ac_old_val'" >&5  
$as_echo "$as_me:   former value:  \`$ac_old_val'" >&2;}  
    { $as_echo "$as_me:${as_lineno-$LINENO}:   current value: \`$ac_new_val'" >&5  
$as_echo "$as_me:   current value: \`$ac_new_val'" >&2;}  
      fi;;  
  esac  
  # Pass precious variables to config.status.  
  if test "$ac_new_set" = set; then  
    case $ac_new_val in  
    *\'*) ac_arg=$ac_var=`$as_echo "$ac_new_val" | sed "s/'/'\\\\\\\\''/g"` ;;  
    *) ac_arg=$ac_var=$ac_new_val ;;  
    esac  
    case " $ac_configure_args " in  
      *" '$ac_arg' "*) ;; # Avoid dups.  Use of quotes ensures accuracy.  
      *) as_fn_append ac_configure_args " '$ac_arg'" ;;  
    esac  
  fi  
done  
if $ac_cache_corrupted; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
  { $as_echo "$as_me:${as_lineno-$LINENO}: error: changes in the environment can compromise the build" >&5  
$as_echo "$as_me: error: changes in the environment can compromise the build" >&2;}  
  as_fn_error $? "run \`make distclean' and/or \`rm $cache_file' and start over" "$LINENO" 5  
fi  
## -------------------- ##  
## Main body of script. ##  
## -------------------- ##  
  
ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
  
  
  
  
  
  
  
  
progname=$0  
# if PWD already has a value, it is probably wrong.  
if test -n "$PWD" ; then PWD=`${PWDCMD-pwd}`; fi  
  
# Export original configure arguments for use by sub-configures.  
# Quote arguments with shell meta charatcers.  
TOPLEVEL_CONFIGURE_ARGUMENTS=  
set -- "$progname" "$@"  
for ac_arg  
do  
  case "$ac_arg" in  
  *" "*|*"    "*|*[\[\]\~\#\$\^\&\*\(\)\{\}\\\|\;\<\>\?\']*)  
    ac_arg=`echo "$ac_arg" | sed "s/'/'\\\\\\\\''/g"`  
    # if the argument is of the form -foo=baz, quote the baz part only  
    ac_arg=`echo "'$ac_arg'" | sed "s/^'\([-a-zA-Z0-9]*=\)/\\1'/"` ;;  
  *) ;;  
  esac  
  # Add the quoted argument to the list.  
  TOPLEVEL_CONFIGURE_ARGUMENTS="$TOPLEVEL_CONFIGURE_ARGUMENTS $ac_arg"  
done  
if test "$silent" = yes; then  
  TOPLEVEL_CONFIGURE_ARGUMENTS="$TOPLEVEL_CONFIGURE_ARGUMENTS --silent"  
fi  
# Remove the initial space we just introduced and, as these will be  
# expanded by make, quote '$'.  
TOPLEVEL_CONFIGURE_ARGUMENTS=`echo "x$TOPLEVEL_CONFIGURE_ARGUMENTS" | sed -e 's/^x *//' -e 's,\\$,$$,g'`  
  
  
# Find the build, host, and target systems.  
ac_aux_dir=  
for ac_dir in "$srcdir" "$srcdir/.." "$srcdir/../.."; do  
  if test -f "$ac_dir/install-sh"; then  
    ac_aux_dir=$ac_dir  
    ac_install_sh="$ac_aux_dir/install-sh -c"  
    break  
  elif test -f "$ac_dir/install.sh"; then  
    ac_aux_dir=$ac_dir  
    ac_install_sh="$ac_aux_dir/install.sh -c"  
    break  
  elif test -f "$ac_dir/shtool"; then  
    ac_aux_dir=$ac_dir  
    ac_install_sh="$ac_aux_dir/shtool install -c"  
    break  
  fi  
done  
if test -z "$ac_aux_dir"; then  
  as_fn_error $? "cannot find install-sh, install.sh, or shtool in \"$srcdir\" \"$srcdir/..\" \"$srcdir/../..\"" "$LINENO" 5  
fi  
  
# These three variables are undocumented and unsupported,  
# and are intended to be withdrawn in a future Autoconf release.  
# They can cause serious problems if a builder's source tree is in a directory  
# whose full name contains unusual characters.  
ac_config_guess="$SHELL $ac_aux_dir/config.guess"  # Please don't use this var.  
ac_config_sub="$SHELL $ac_aux_dir/config.sub"  # Please don't use this var.  
ac_configure="$SHELL $ac_aux_dir/configure"  # Please don't use this var.  
  
  
# Make sure we can run config.sub.  
$SHELL "$ac_aux_dir/config.sub" sun4 >/dev/null 2>&1 ||  
  as_fn_error $? "cannot run $SHELL $ac_aux_dir/config.sub" "$LINENO" 5  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking build system type" >&5  
$as_echo_n "checking build system type... " >&6; }  
if ${ac_cv_build+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_build_alias=$build_alias  
test "x$ac_build_alias" = x &&  
  ac_build_alias=`$SHELL "$ac_aux_dir/config.guess"`  
test "x$ac_build_alias" = x &&  
  as_fn_error $? "cannot guess build type; you must specify one" "$LINENO" 5  
ac_cv_build=`$SHELL "$ac_aux_dir/config.sub" $ac_build_alias` ||  
  as_fn_error $? "$SHELL $ac_aux_dir/config.sub $ac_build_alias failed" "$LINENO" 5  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_build" >&5  
$as_echo "$ac_cv_build" >&6; }  
case $ac_cv_build in  
*-*-*) ;;  
*) as_fn_error $? "invalid value of canonical build" "$LINENO" 5;;  
esac  
build=$ac_cv_build  
ac_save_IFS=$IFS; IFS='-'  
set x $ac_cv_build  
shift  
build_cpu=$1  
build_vendor=$2  
shift; shift  
# Remember, the first character of IFS is used to create $*,  
# except with old shells:  
build_os=$*  
IFS=$ac_save_IFS  
case $build_os in *\ *) build_os=`echo "$build_os" | sed 's/ /-/g'`;; esac  
  
  
 case ${build_alias} in  
  "") build_noncanonical=${build} ;;  
  *) build_noncanonical=${build_alias} ;;  
esac  
  
  
  
 case ${host_alias} in  
  "") host_noncanonical=${build_noncanonical} ;;  
  *) host_noncanonical=${host_alias} ;;  
esac  
  
  
  
 case ${target_alias} in  
  "") target_noncanonical=${host_noncanonical} ;;  
  *) target_noncanonical=${target_alias} ;;  
esac  
  
  
  
  
test "$host_noncanonical" = "$target_noncanonical" &&  
  test "$program_prefix$program_suffix$program_transform_name" = \  
    NONENONEs,x,x, &&  
  program_transform_name=s,y,y,  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking host system type" >&5  
$as_echo_n "checking host system type... " >&6; }  
if ${ac_cv_host+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test "x$host_alias" = x; then  
  ac_cv_host=$ac_cv_build  
else  
  ac_cv_host=`$SHELL "$ac_aux_dir/config.sub" $host_alias` ||  
    as_fn_error $? "$SHELL $ac_aux_dir/config.sub $host_alias failed" "$LINENO" 5  
fi  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_host" >&5  
$as_echo "$ac_cv_host" >&6; }  
case $ac_cv_host in  
*-*-*) ;;  
*) as_fn_error $? "invalid value of canonical host" "$LINENO" 5;;  
esac  
host=$ac_cv_host  
ac_save_IFS=$IFS; IFS='-'  
set x $ac_cv_host  
shift  
host_cpu=$1  
host_vendor=$2  
shift; shift  
# Remember, the first character of IFS is used to create $*,  
# except with old shells:  
host_os=$*  
IFS=$ac_save_IFS  
case $host_os in *\ *) host_os=`echo "$host_os" | sed 's/ /-/g'`;; esac  
  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking target system type" >&5  
$as_echo_n "checking target system type... " >&6; }  
if ${ac_cv_target+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test "x$target_alias" = x; then  
  ac_cv_target=$ac_cv_host  
else  
  ac_cv_target=`$SHELL "$ac_aux_dir/config.sub" $target_alias` ||  
    as_fn_error $? "$SHELL $ac_aux_dir/config.sub $target_alias failed" "$LINENO" 5  
fi  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_target" >&5  
$as_echo "$ac_cv_target" >&6; }  
case $ac_cv_target in  
*-*-*) ;;  
*) as_fn_error $? "invalid value of canonical target" "$LINENO" 5;;  
esac  
target=$ac_cv_target  
ac_save_IFS=$IFS; IFS='-'  
set x $ac_cv_target  
shift  
target_cpu=$1  
target_vendor=$2  
shift; shift  
# Remember, the first character of IFS is used to create $*,  
# except with old shells:  
target_os=$*  
IFS=$ac_save_IFS  
case $target_os in *\ *) target_os=`echo "$target_os" | sed 's/ /-/g'`;; esac  
  
  
# The aliases save the names the user supplied, while $host etc.  
# will get canonicalized.  
test -n "$target_alias" &&  
  test "$program_prefix$program_suffix$program_transform_name" = \  
    NONENONEs,x,x, &&  
  program_prefix=${target_alias}-  
  
test "$program_prefix" != NONE &&  
  program_transform_name="s&^&$program_prefix&;$program_transform_name"  
# Use a double $ so make ignores it.  
test "$program_suffix" != NONE &&  
  program_transform_name="s&\$&$program_suffix&;$program_transform_name"  
# Double any \ or $.  
# By default was `s,x,x', remove it if useless.  
ac_script='s/[\\$]/&&/g;s/;s,x,x,$//'  
program_transform_name=`$as_echo "$program_transform_name" | sed "$ac_script"`  
  
  
  
# Get 'install' or 'install-sh' and its variants.  
# Find a good install program.  We prefer a C program (faster),  
# so one script is as good as another.  But avoid the broken or  
# incompatible versions:  
# SysV /etc/install, /usr/sbin/install  
# SunOS /usr/etc/install  
# IRIX /sbin/install  
# AIX /bin/install  
# AmigaOS /C/install, which installs bootblocks on floppy discs  
# AIX 4 /usr/bin/installbsd, which doesn't work without a -g flag  
# AFS /usr/afsws/bin/install, which mishandles nonexistent args  
# SVR4 /usr/ucb/install, which tries to use the nonexistent group "staff"  
# OS/2's system install, which has a completely different semantic  
# ./install, which can be erroneously created by make from ./install.sh.  
# Reject install programs that cannot install multiple files.  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for a BSD-compatible install" >&5  
$as_echo_n "checking for a BSD-compatible install... " >&6; }  
if test -z "$INSTALL"; then  
if ${ac_cv_path_install+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    # Account for people who put trailing slashes in PATH elements.  
case $as_dir/ in #((  
  ./ | .// | /[cC]/* | \  
  /etc/* | /usr/sbin/* | /usr/etc/* | /sbin/* | /usr/afsws/bin/* | \  
  ?:[\\/]os2[\\/]install[\\/]* | ?:[\\/]OS2[\\/]INSTALL[\\/]* | \  
  /usr/ucb/* ) ;;  
  *)  
    # OSF1 and SCO ODT 3.0 have their own names for install.  
    # Don't use installbsd from OSF since it installs stuff as root  
    # by default.  
    for ac_prog in ginstall scoinst install; do  
      for ac_exec_ext in '' $ac_executable_extensions; do  
    if as_fn_executable_p "$as_dir/$ac_prog$ac_exec_ext"; then  
      if test $ac_prog = install &&  
        grep dspmsg "$as_dir/$ac_prog$ac_exec_ext" >/dev/null 2>&1; then  
        # AIX install.  It has an incompatible calling convention.  
        :  
      elif test $ac_prog = install &&  
        grep pwplus "$as_dir/$ac_prog$ac_exec_ext" >/dev/null 2>&1; then  
        # program-specific install script used by HP pwplus--don't use.  
        :  
      else  
        rm -rf conftest.one conftest.two conftest.dir  
        echo one > conftest.one  
        echo two > conftest.two  
        mkdir conftest.dir  
        if "$as_dir/$ac_prog$ac_exec_ext" -c conftest.one conftest.two "`pwd`/conftest.dir" &&  
          test -s conftest.one && test -s conftest.two &&  
          test -s conftest.dir/conftest.one &&  
          test -s conftest.dir/conftest.two  
        then  
          ac_cv_path_install="$as_dir/$ac_prog$ac_exec_ext -c"  
          break 3  
        fi  
      fi  
    fi  
      done  
    done  
    ;;  
esac  
  
  done  
IFS=$as_save_IFS  
  
rm -rf conftest.one conftest.two conftest.dir  
  
fi  
  if test "${ac_cv_path_install+set}" = set; then  
    INSTALL=$ac_cv_path_install  
  else  
    # As a last resort, use the slow shell script.  Don't cache a  
    # value for INSTALL within a source directory, because that will  
    # break other packages using the cache if that directory is  
    # removed, or if the value is a relative name.  
    INSTALL=$ac_install_sh  
  fi  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $INSTALL" >&5  
$as_echo "$INSTALL" >&6; }  
  
# Use test -z because SunOS4 sh mishandles braces in ${var-val}.  
# It thinks the first close brace ends the variable substitution.  
test -z "$INSTALL_PROGRAM" && INSTALL_PROGRAM='${INSTALL}'  
  
test -z "$INSTALL_SCRIPT" && INSTALL_SCRIPT='${INSTALL}'  
  
test -z "$INSTALL_DATA" && INSTALL_DATA='${INSTALL} -m 644'  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether ln works" >&5  
$as_echo_n "checking whether ln works... " >&6; }  
if ${acx_cv_prog_LN+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  rm -f conftestdata_t  
echo >conftestdata_f  
if ln conftestdata_f conftestdata_t 2>/dev/null  
then  
  acx_cv_prog_LN=ln  
else  
  acx_cv_prog_LN=no  
fi  
rm -f conftestdata_f conftestdata_t  
  
fi  
if test $acx_cv_prog_LN = no; then  
  LN="cp"  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no, using $LN" >&5  
$as_echo "no, using $LN" >&6; }  
else  
  LN="$acx_cv_prog_LN"  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether ln -s works" >&5  
$as_echo_n "checking whether ln -s works... " >&6; }  
LN_S=$as_ln_s  
if test "$LN_S" = "ln -s"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no, using $LN_S" >&5  
$as_echo "no, using $LN_S" >&6; }  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for a sed that does not truncate output" >&5  
$as_echo_n "checking for a sed that does not truncate output... " >&6; }  
if ${ac_cv_path_SED+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
            ac_script=s/aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa/bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb/  
     for ac_i in 1 2 3 4 5 6 7; do  
       ac_script="$ac_script$as_nl$ac_script"  
     done  
     echo "$ac_script" 2>/dev/null | sed 99q >conftest.sed  
     { ac_script=; unset ac_script;}  
     if test -z "$SED"; then  
  ac_path_SED_found=false  
  # Loop through the user's path and test for each of PROGNAME-LIST  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_prog in sed gsed; do  
    for ac_exec_ext in '' $ac_executable_extensions; do  
      ac_path_SED="$as_dir/$ac_prog$ac_exec_ext"  
      as_fn_executable_p "$ac_path_SED" || continue  
# Check for GNU ac_path_SED and select it if it is found.  
  # Check for GNU $ac_path_SED  
case `"$ac_path_SED" --version 2>&1` in  
*GNU*)  
  ac_cv_path_SED="$ac_path_SED" ac_path_SED_found=:;;  
*)  
  ac_count=0  
  $as_echo_n 0123456789 >"conftest.in"  
  while :  
  do  
    cat "conftest.in" "conftest.in" >"conftest.tmp"  
    mv "conftest.tmp" "conftest.in"  
    cp "conftest.in" "conftest.nl"  
    $as_echo '' >> "conftest.nl"  
    "$ac_path_SED" -f conftest.sed < "conftest.nl" >"conftest.out" 2>/dev/null || break  
    diff "conftest.out" "conftest.nl" >/dev/null 2>&1 || break  
    as_fn_arith $ac_count + 1 && ac_count=$as_val  
    if test $ac_count -gt ${ac_path_SED_max-0}; then  
      # Best one so far, save it but keep looking for a better one  
      ac_cv_path_SED="$ac_path_SED"  
      ac_path_SED_max=$ac_count  
    fi  
    # 10*(2^10) chars as input seems more than enough  
    test $ac_count -gt 10 && break  
  done  
  rm -f conftest.in conftest.tmp conftest.nl conftest.out;;  
esac  
  
      $ac_path_SED_found && break 3  
    done  
  done  
  done  
IFS=$as_save_IFS  
  if test -z "$ac_cv_path_SED"; then  
    as_fn_error $? "no acceptable sed could be found in \$PATH" "$LINENO" 5  
  fi  
else  
  ac_cv_path_SED=$SED  
fi  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_SED" >&5  
$as_echo "$ac_cv_path_SED" >&6; }  
 SED="$ac_cv_path_SED"  
  rm -f conftest.sed  
  
for ac_prog in gawk mawk nawk awk  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AWK+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AWK"; then  
  ac_cv_prog_AWK="$AWK" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AWK="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AWK=$ac_cv_prog_AWK  
if test -n "$AWK"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AWK" >&5  
$as_echo "$AWK" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$AWK" && break  
done  
  
  
srcpwd=`cd ${srcdir} ; ${PWDCMD-pwd}`  
  
# We pass INSTALL explicitly to sub-makes.  Make sure that it is not  
# a relative path.  
if test "$INSTALL" = "${srcdir}/install-sh -c"; then  
  INSTALL="${srcpwd}/install-sh -c"  
fi  
  
# Set srcdir to "." if that's what it is.  
# This is important for multilib support.  
pwd=`${PWDCMD-pwd}`  
if test "${pwd}" = "${srcpwd}" ; then  
  srcdir=.  
fi  
  
topsrcdir=$srcpwd  
  
extra_host_args=  
  
### To add a new directory to the tree, first choose whether it is a target  
### or a host dependent tool.  Then put it into the appropriate list  
### (library or tools, host or target), doing a dependency sort.  
  
# Subdirs will be configured in the order listed in build_configdirs,  
# configdirs, or target_configdirs; see the serialization section below.  
  
# Dependency sorting is only needed when *configuration* must be done in  
# a particular order.  In all cases a dependency should be specified in  
# the Makefile, whether or not it's implicitly specified here.  
  
# Double entries in build_configdirs, configdirs, or target_configdirs may  
# cause circular dependencies and break everything horribly.  
  
# these library is used by various programs built for the build  
# environment  
#  
build_libs="build-libiberty build-libcpp"  
  
# these tools are built for the build environment  
build_tools="build-texinfo build-flex build-bison build-m4 build-fixincludes"  
  
# these libraries are used by various programs built for the host environment  
#f  
host_libs="gettext libiberty opcodes bfd readline tcl tk itcl libgui zlib libbacktrace libcpp libcody libdecnumber gmp mpfr mpc isl libiconv libctf libsframe libgrust "  
  
# these tools are built for the host environment  
# Note, the powerpc-eabi build depends on sim occurring before gdb in order to  
# know that we are building the simulator.  
# binutils, gas and ld appear in that order because it makes sense to run  
# "make check" in that particular order.  
# If --enable-gold is used, "gold" may replace "ld".  
host_tools="texinfo flex bison binutils gas ld fixincludes gcc cgen sid sim gdb gdbserver gprof etc expect dejagnu m4 utils guile fastjar gnattools libcc1 gm2tools gotools c++tools"  
  
# these libraries are built for the target environment, and are built after  
# the host libraries and the host tools (which may be a cross compiler)  
# Note that libiberty is not a target library.  
target_libraries="target-libgcc \  
        target-libbacktrace \  
        target-libgloss \  
        target-newlib \  
        target-libgomp \  
        target-libatomic \  
        target-libitm \  
        target-libstdc++-v3 \  
        target-libsanitizer \  
        target-libvtv \  
        target-libssp \  
        target-libquadmath \  
        target-libgfortran \  
        target-libffi \  
        target-libobjc \  
        target-libada \  
        target-libgm2 \  
        target-libgo \  
        target-libgrust \  
        target-libphobos \  
        target-zlib"  
  
# these tools are built using the target libraries, and are intended to  
# run only in the target environment  
#  
# note: any program that *uses* libraries that are in the "target_libraries"  
# list belongs in this list.  
#  
target_tools="target-rda"  
  
################################################################################  
  
## All tools belong in one of the four categories, and are assigned above  
## We assign ${configdirs} this way to remove all embedded newlines.  This  
## is important because configure will choke if they ever get through.  
## ${configdirs} is directories we build using the host tools.  
## ${target_configdirs} is directories we build using the target tools.  
configdirs=`echo ${host_libs} ${host_tools}`  
target_configdirs=`echo ${target_libraries} ${target_tools}`  
build_configdirs=`echo ${build_libs} ${build_tools}`  
  
  
  
################################################################################  
  
srcname="gnu development package"  
  
# This gets set non-empty for some net releases of packages.  
appdirs=""  
  
# Define is_cross_compiler to save on calls to 'test'.  
is_cross_compiler=  
if test x"${host}" = x"${target}" ; then  
  is_cross_compiler=no  
else  
  is_cross_compiler=yes  
fi  
  
# Find the build and target subdir names.  
  
# post-stage1 host modules use a different CC_FOR_BUILD so, in order to  
# have matching libraries, they should use host libraries: Makefile.tpl  
# arranges to pass --with-build-libsubdir=$(HOST_SUBDIR).  
# However, they still use the build modules, because the corresponding  
# host modules (e.g. bison) are only built for the host when bootstrap  
# finishes. So:  
# - build_subdir is where we find build modules, and never changes.  
# - build_libsubdir is where we find build libraries, and can be overridden.  
  
# Prefix 'build-' so this never conflicts with target_subdir.  
build_subdir="build-${build_noncanonical}"  
  
# Check whether --with-build-libsubdir was given.  
if test "${with_build_libsubdir+set}" = set; then :  
  withval=$with_build_libsubdir; build_libsubdir="$withval"  
else  
  build_libsubdir="$build_subdir"  
fi  
  
# --srcdir=. covers the toplevel, while "test -d" covers the subdirectories  
if ( test $srcdir = . && test -d gcc ) \  
   || test -d $srcdir/../host-${host_noncanonical}; then  
  host_subdir="host-${host_noncanonical}"  
else  
  host_subdir=.  
fi  
# No prefix.  
target_subdir=${target_noncanonical}  
  
# Be sure to cover against remnants of an in-tree build.  
if test $srcdir != .  && test -d $srcdir/host-${host_noncanonical}; then  
  as_fn_error $? "building out of tree but $srcdir contains host-${host_noncanonical}.  
Use a pristine source tree when building in a separate tree" "$LINENO" 5  
fi  
  
# Skipdirs are removed silently.  
skipdirs=  
# Noconfigdirs are removed loudly.  
noconfigdirs=""  
  
use_gnu_ld=  
# Make sure we don't let GNU ld be added if we didn't want it.  
if test x$with_gnu_ld = xno ; then  
  use_gnu_ld=no  
  noconfigdirs="$noconfigdirs ld gold"  
fi  
  
use_gnu_as=  
# Make sure we don't let GNU as be added if we didn't want it.  
if test x$with_gnu_as = xno ; then  
  use_gnu_as=no  
  noconfigdirs="$noconfigdirs gas"  
fi  
  
use_included_zlib=  
  
# Check whether --with-system-zlib was given.  
if test "${with_system_zlib+set}" = set; then :  
  withval=$with_system_zlib;  
fi  
  
# Make sure we don't let ZLIB be added if we didn't want it.  
if test x$with_system_zlib = xyes ; then  
  use_included_zlib=no  
  noconfigdirs="$noconfigdirs zlib"  
fi  
  
# Don't compile the bundled readline/libreadline.a if --with-system-readline  
# is provided.  
if test x$with_system_readline = xyes ; then  
  noconfigdirs="$noconfigdirs readline"  
fi  
  
  
# Check whether --with-zstd was given.  
if test "${with_zstd+set}" = set; then :  
  withval=$with_zstd;  
fi  
  
  
# some tools are so dependent upon X11 that if we're not building with X,  
# it's not even worth trying to configure, much less build, that tool.  
  
case ${with_x} in  
  yes | "") ;; # the default value for this tree is that X11 is available  
  no)  
    skipdirs="${skipdirs} tk itcl libgui"  
    # We won't be able to build gdbtk without X.  
    enable_gdbtk=no  
    ;;  
  *)  echo "*** bad value \"${with_x}\" for -with-x flag; ignored" 1>&2 ;;  
esac  
  
# Some are only suitable for cross toolchains.  
# Remove these if host=target.  
cross_only="target-libgloss target-newlib target-opcodes"  
  
case $is_cross_compiler in  
  no) skipdirs="${skipdirs} ${cross_only}" ;;  
esac  
  
# If both --with-headers and --with-libs are specified, default to  
# --without-newlib.  
if test x"${with_headers}" != x && test x"${with_headers}" != xno \  
   && test x"${with_libs}" != x && test x"${with_libs}" != xno ; then  
  if test x"${with_newlib}" = x ; then  
    with_newlib=no  
  fi  
fi  
  
# Recognize --with-newlib/--without-newlib.  
case ${with_newlib} in  
  no) skipdirs="${skipdirs} target-newlib" ;;  
  yes) skipdirs=`echo " ${skipdirs} " | sed -e 's/ target-newlib / /'` ;;  
esac  
  
# Check whether --enable-as-accelerator-for was given.  
if test "${enable_as_accelerator_for+set}" = set; then :  
  enableval=$enable_as_accelerator_for;  
fi  
  
  
# Check whether --enable-offload-targets was given.  
if test "${enable_offload_targets+set}" = set; then :  
  enableval=$enable_offload_targets;  
  if test x"$enable_offload_targets" = x; then  
    as_fn_error $? "no offload targets specified" "$LINENO" 5  
  fi  
  
else  
  enable_offload_targets=  
fi  
  
  
# Check whether --enable-offload-defaulted was given.  
if test "${enable_offload_defaulted+set}" = set; then :  
  enableval=$enable_offload_defaulted; enable_offload_defaulted=$enableval  
else  
  enable_offload_defaulted=  
fi  
  
  
# Handle --enable-gold, --enable-ld.  
# --disable-gold [--enable-ld]  
#     Build only ld.  Default option.  
# --enable-gold [--enable-ld]  
#     Build both gold and ld.  Install gold as "ld.gold", install ld  
#     as "ld.bfd" and "ld".  
# --enable-gold=default [--enable-ld]  
#     Build both gold and ld.  Install gold as "ld.gold" and "ld",  
#     install ld as "ld.bfd".  
# --enable-gold[=default] --disable-ld  
#     Build only gold, which is then installed as both "ld.gold" and "ld".  
# --enable-gold --enable-ld=default  
#     Build both gold (installed as "ld.gold") and ld (installed as "ld"  
#     and ld.bfd).  
#     In other words, ld is default  
# --enable-gold=default --enable-ld=default  
#     Error.  
  
default_ld=  
# Check whether --enable-gold was given.  
if test "${enable_gold+set}" = set; then :  
  enableval=$enable_gold; ENABLE_GOLD=$enableval  
else  
  ENABLE_GOLD=no  
fi  
  
case "${ENABLE_GOLD}" in  
  yes|default)  
    # Check for ELF target.  
    is_elf=no  
    case "${target}" in  
      *-*-elf* | *-*-sysv4* | *-*-unixware* | *-*-eabi* | hppa*64*-*-hpux* \  
      | *-*-linux* | *-*-gnu* | frv-*-uclinux* | *-*-irix5* | *-*-irix6* \  
      | *-*-netbsd* | *-*-openbsd* | *-*-freebsd* | *-*-dragonfly* \  
      | *-*-solaris2* | *-*-nto* | *-*-nacl* | *-*-haiku*)  
        case "${target}" in  
          *-*-linux*aout* | *-*-linux*oldld*)  
            ;;  
          *)  
            is_elf=yes  
            ;;  
        esac  
    esac  
  
    if test "$is_elf" = "yes"; then  
      # Check for target supported by gold.  
      case "${target}" in  
        i?86-*-* | x86_64-*-* | sparc*-*-* | powerpc*-*-* | arm*-*-* \  
        | aarch64*-*-* | tilegx*-*-* | mips*-*-* | s390*-*-* | loongarch*-*-*)  
      configdirs="$configdirs gold"  
      if test x${ENABLE_GOLD} = xdefault; then  
        default_ld=gold  
      fi  
      ENABLE_GOLD=yes  
          ;;  
      esac  
    fi  
    ;;  
  no)  
    ;;  
  *)  
    as_fn_error $? "invalid --enable-gold argument" "$LINENO" 5  
    ;;  
esac  
  
# Check whether --enable-ld was given.  
if test "${enable_ld+set}" = set; then :  
  enableval=$enable_ld; ENABLE_LD=$enableval  
else  
  ENABLE_LD=yes  
fi  
  
  
case "${ENABLE_LD}" in  
  default)  
    if test x${default_ld} != x; then  
      as_fn_error $? "either gold or ld can be the default ld" "$LINENO" 5  
    fi  
    ;;  
  yes)  
    ;;  
  no)  
    if test x${ENABLE_GOLD} != xyes; then  
      { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: neither ld nor gold are enabled" >&5  
$as_echo "$as_me: WARNING: neither ld nor gold are enabled" >&2;}  
    fi  
    configdirs=`echo " ${configdirs} " | sed -e 's/ ld / /'`  
    ;;  
  *)  
    as_fn_error $? "invalid --enable-ld argument" "$LINENO" 5  
    ;;  
esac  
  
# Check whether --enable-gprofng was given.  
if test "${enable_gprofng+set}" = set; then :  
  enableval=$enable_gprofng; enable_gprofng=$enableval  
else  
  enable_gprofng=yes  
fi  
  
if test "$enable_gprofng" = "yes"; then  
  case "${target}" in  
    x86_64-*-linux* | i?86-*-linux* | aarch64-*-linux*)  
    configdirs="$configdirs gprofng"  
    ;;  
  esac  
fi  
  
  
# PR gas/19109  
# Decide the default method for compressing debug sections.  
# Provide a configure time option to override our default.  
# Check whether --enable-compressed_debug_sections was given.  
if test "${enable_compressed_debug_sections+set}" = set; then :  
  enableval=$enable_compressed_debug_sections;  
  if test x"$enable_compressed_debug_sections" = xyes; then  
    as_fn_error $? "no program with compressed debug sections specified" "$LINENO" 5  
  fi  
  
else  
  enable_compressed_debug_sections=  
fi  
  
  
# Select default compression algorithm.  
# Check whether --enable-default_compressed_debug_sections_algorithm was given.  
if test "${enable_default_compressed_debug_sections_algorithm+set}" = set; then :  
  enableval=$enable_default_compressed_debug_sections_algorithm;  
else  
  default_compressed_debug_sections_algorithm=  
fi  
  
  
# Configure extra directories which are host specific  
  
case "${host}" in  
  *-cygwin*)  
    configdirs="$configdirs libtermcap" ;;  
esac  
  
# A target can indicate whether a language isn't supported for some reason.  
# Only spaces may be used in this macro; not newlines or tabs.  
unsupported_languages=  
  
# Remove more programs from consideration, based on the host or  
# target this usually means that a port of the program doesn't  
# exist yet.  
  
case "${host}" in  
  i[3456789]86-*-msdosdjgpp*)  
    noconfigdirs="$noconfigdirs tcl tk itcl"  
    ;;  
esac  
  
# Default to --disable-year2038 until we can handle differences between  
# projects that use gnulib (which understands year 2038) and projects that  
# do not (like BFD).  
# Check whether --enable-year2038 was given.  
if test "${enable_year2038+set}" = set; then :  
  enableval=$enable_year2038; ENABLE_YEAR2038=$enableval  
else  
  ENABLE_YEAR2038=no  
fi  
  
enable_year2038=  
if test "${ENABLE_YEAR2038}" = "no" ; then  
  enable_year2038=no  
fi  
  
# Check whether --enable-libquadmath was given.  
if test "${enable_libquadmath+set}" = set; then :  
  enableval=$enable_libquadmath; ENABLE_LIBQUADMATH=$enableval  
else  
  ENABLE_LIBQUADMATH=yes  
fi  
  
if test "${ENABLE_LIBQUADMATH}" = "no" ; then  
  noconfigdirs="$noconfigdirs target-libquadmath"  
fi  
  
  
# Check whether --enable-libquadmath-support was given.  
if test "${enable_libquadmath_support+set}" = set; then :  
  enableval=$enable_libquadmath_support; ENABLE_LIBQUADMATH_SUPPORT=$enableval  
else  
  ENABLE_LIBQUADMATH_SUPPORT=yes  
fi  
  
enable_libquadmath_support=  
if test "${ENABLE_LIBQUADMATH_SUPPORT}" = "no" ; then  
  enable_libquadmath_support=no  
fi  
  
  
# Check whether --enable-libada was given.  
if test "${enable_libada+set}" = set; then :  
  enableval=$enable_libada; ENABLE_LIBADA=$enableval  
else  
  ENABLE_LIBADA=yes  
fi  
  
if test "${ENABLE_LIBADA}" != "yes" ; then  
  noconfigdirs="$noconfigdirs gnattools"  
fi  
  
# Check whether --enable-libgm2 was given.  
if test "${enable_libgm2+set}" = set; then :  
  enableval=$enable_libgm2; ENABLE_LIBGM2=$enableval  
else  
  ENABLE_LIBGM2=no  
fi  
  
if test "${ENABLE_LIBGM2}" != "yes" ; then  
  noconfigdirs="$noconfigdirs gm2tools"  
fi  
  
# Check whether --enable-libssp was given.  
if test "${enable_libssp+set}" = set; then :  
  enableval=$enable_libssp; ENABLE_LIBSSP=$enableval  
else  
  ENABLE_LIBSSP=yes  
fi  
  
  
# Check whether --enable-libstdcxx was given.  
if test "${enable_libstdcxx+set}" = set; then :  
  enableval=$enable_libstdcxx; ENABLE_LIBSTDCXX=$enableval  
else  
  ENABLE_LIBSTDCXX=default  
fi  
  
if test "${ENABLE_LIBSTDCXX}" = "no" ; then  
  noconfigdirs="$noconfigdirs target-libstdc++-v3"  
fi  
  
# Enable libgomp by default on hosted POSIX systems, and a few others.  
if test x$enable_libgomp = x ; then  
    case "${target}" in  
    *-*-linux* | *-*-gnu* | *-*-k*bsd*-gnu | *-*-kopensolaris*-gnu)  
    ;;  
    *-*-netbsd* | *-*-freebsd* | *-*-openbsd* | *-*-dragonfly*)  
    ;;  
    *-*-solaris2* | *-*-hpux11*)  
    ;;  
    *-*-darwin* | *-*-aix*)  
    ;;  
    nvptx*-*-* | amdgcn*-*-*)  
    ;;  
    *)  
    noconfigdirs="$noconfigdirs target-libgomp"  
    ;;  
    esac  
fi  
  
# Disable libatomic on unsupported systems.  
if test -d ${srcdir}/libatomic; then  
    if test x$enable_libatomic = x; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for libatomic support" >&5  
$as_echo_n "checking for libatomic support... " >&6; }  
    if (srcdir=${srcdir}/libatomic; \  
        . ${srcdir}/configure.tgt; \  
        test -n "$UNSUPPORTED")  
    then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
        noconfigdirs="$noconfigdirs target-libatomic"  
    else  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    fi  
    fi  
fi  
  
# Disable libitm on unsupported systems.  
if test -d ${srcdir}/libitm; then  
    if test x$enable_libitm = x; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for libitm support" >&5  
$as_echo_n "checking for libitm support... " >&6; }  
    if (srcdir=${srcdir}/libitm; \  
        . ${srcdir}/configure.tgt; \  
        test -n "$UNSUPPORTED")  
    then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
        noconfigdirs="$noconfigdirs target-libitm"  
    else  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    fi  
    fi  
fi  
  
# Disable libsanitizer on unsupported systems.  
if test -d ${srcdir}/libsanitizer; then  
    if test x$enable_libsanitizer = x; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for libsanitizer support" >&5  
$as_echo_n "checking for libsanitizer support... " >&6; }  
    if (srcdir=${srcdir}/libsanitizer; \  
        . ${srcdir}/configure.tgt; \  
        test -n "$UNSUPPORTED")  
    then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
        noconfigdirs="$noconfigdirs target-libsanitizer"  
    else  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    fi  
    fi  
fi  
  
# Disable libvtv on unsupported systems.  
if test -d ${srcdir}/libvtv; then  
    if test x$enable_libvtv = x; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for libvtv support" >&5  
$as_echo_n "checking for libvtv support... " >&6; }  
    if (srcdir=${srcdir}/libvtv; \  
        . ${srcdir}/configure.tgt; \  
        test "$VTV_SUPPORTED" != "yes")  
    then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
        noconfigdirs="$noconfigdirs target-libvtv"  
    else  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    fi  
    fi  
fi  
  
# Disable libquadmath for some systems.  
case "${target}" in  
  avr-*-*)  
    noconfigdirs="$noconfigdirs target-libquadmath"  
    ;;  
  # libquadmath is unused on AIX and libquadmath build process use of  
  # LD_LIBRARY_PATH can break AIX bootstrap.  
  powerpc-*-aix* | rs6000-*-aix*)  
    noconfigdirs="$noconfigdirs target-libquadmath"  
    ;;  
esac  
  
# Disable libssp for some systems.  
case "${target}" in  
  avr-*-*)  
    # No hosted I/O support.  
    noconfigdirs="$noconfigdirs target-libssp"  
    ;;  
  bpf-*-*)  
    noconfigdirs="$noconfigdirs target-libssp"  
    ;;  
  powerpc-*-aix* | rs6000-*-aix*)  
    noconfigdirs="$noconfigdirs target-libssp"  
    ;;  
  pru-*-*)  
    # No hosted I/O support.  
    noconfigdirs="$noconfigdirs target-libssp"  
    ;;  
  rl78-*-*)  
    # libssp uses a misaligned load to trigger a fault, but the RL78  
    # doesn't fault for those - instead, it gives a build-time error  
    # for explicit misaligned loads.  
    noconfigdirs="$noconfigdirs target-libssp"  
    ;;  
  visium-*-*)  
    # No hosted I/O support.  
    noconfigdirs="$noconfigdirs target-libssp"  
    ;;  
esac  
  
# Disable libstdc++-v3 for some systems.  
# Allow user to override this if they pass --enable-libstdcxx  
if test "${ENABLE_LIBSTDCXX}" = "default" ; then  
  case "${target}" in  
    *-*-vxworks*)  
      # VxWorks uses the Dinkumware C++ library.  
      noconfigdirs="$noconfigdirs target-libstdc++-v3"  
      ;;  
    amdgcn*-*-*)  
      # Not ported/fails to build when using newlib.  
      noconfigdirs="$noconfigdirs target-libstdc++-v3"  
      ;;  
    arm*-wince-pe*)  
      # the C++ libraries don't build on top of CE's C libraries  
      noconfigdirs="$noconfigdirs target-libstdc++-v3"  
      ;;  
    avr-*-*)  
      noconfigdirs="$noconfigdirs target-libstdc++-v3"  
      ;;  
    bpf-*-*)  
      noconfigdirs="$noconfigdirs target-libstdc++-v3"  
      ;;  
    ft32-*-*)  
      noconfigdirs="$noconfigdirs target-libstdc++-v3"  
      ;;  
  esac  
fi  
  
# Disable C++ on systems where it is known to not work.  
# For testing, you can override this with --enable-languages=c++.  
case ,${enable_languages}, in  
  *,c++,*)  
    ;;  
  *)  
      case "${target}" in  
        bpf-*-*)  
          unsupported_languages="$unsupported_languages c++"  
          ;;  
      esac  
      ;;  
esac  
  
# Disable Objc on systems where it is known to not work.  
# For testing, you can override this with --enable-languages=objc.  
case ,${enable_languages}, in  
  *,objc,*)  
    ;;  
  *)  
      case "${target}" in  
        bpf-*-*)  
          unsupported_languages="$unsupported_languages objc"  
          ;;  
      esac  
      ;;  
esac  
  
# Disable D on systems where it is known to not work.  
# For testing, you can override this with --enable-languages=d.  
case ,${enable_languages}, in  
  *,d,*)  
    ;;  
  *)  
    case "${target}" in  
      bpf-*-*)  
    unsupported_languages="$unsupported_languages d"  
    ;;  
    esac  
    ;;  
esac  
  
# Disable libphobos on unsupported systems.  
# For testing, you can override this with --enable-libphobos.  
if test -d ${srcdir}/libphobos; then  
    if test x$enable_libphobos = x; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for libphobos support" >&5  
$as_echo_n "checking for libphobos support... " >&6; }  
    if (srcdir=${srcdir}/libphobos; \  
        . ${srcdir}/configure.tgt; \  
        test "$LIBPHOBOS_SUPPORTED" != "yes")  
    then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
        noconfigdirs="$noconfigdirs target-libphobos"  
    else  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    fi  
    fi  
fi  
  
# Disable Fortran for some systems.  
case "${target}" in  
  mmix-*-*)  
    # See <http://gcc.gnu.org/ml/gcc-patches/2004-11/msg00572.html>.  
    unsupported_languages="$unsupported_languages fortran"  
    ;;  
  bpf-*-*)  
    unsupported_languages="$unsupported_languages fortran"  
    ;;  
esac  
  
# Disable libffi for some systems.  
case "${target}" in  
  powerpc-*-darwin*)  
    ;;  
  i[3456789]86-*-darwin*)  
    ;;  
  x86_64-*-darwin[912]*)  
    ;;  
  *-*-darwin*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-netware*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-phoenix*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-rtems*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-tpf*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-uclinux*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-vxworks*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  aarch64*-*-freebsd*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  alpha*-*-*vms*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  arm*-*-freebsd*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  arm-wince-pe)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  arm*-*-symbianelf*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  bpf-*-*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  cris-*-* | crisv32-*-*)  
    case "${target}" in  
      *-*-linux*)  
    ;;  
      *) # See PR46792 regarding target-libffi.  
    noconfigdirs="$noconfigdirs target-libffi";;  
    esac  
    ;;  
  hppa*64*-*-hpux*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  hppa*-hp-hpux11*)  
    ;;  
  hppa*-*-hpux*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  ia64*-*-*vms*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  i[3456789]86-w64-mingw*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  i[3456789]86-*-mingw*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  x86_64-*-mingw*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  mmix-*-*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  powerpc-*-aix*)  
    ;;  
  rs6000-*-aix*)  
    ;;  
  ft32-*-*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
  *-*-lynxos*)  
    noconfigdirs="$noconfigdirs target-libffi"  
    ;;  
esac  
  
# Disable the go frontend on systems where it is known to not work. Please keep  
# this in sync with contrib/config-list.mk.  
case "${target}" in  
*-*-darwin* | *-*-cygwin* | *-*-mingw* | bpf-* )  
    unsupported_languages="$unsupported_languages go"  
    ;;  
esac  
  
# Only allow gdbserver on some systems.  
if test -d ${srcdir}/gdbserver; then  
    if test x$enable_gdbserver = x; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for gdbserver support" >&5  
$as_echo_n "checking for gdbserver support... " >&6; }  
    if (srcdir=${srcdir}/gdbserver; \  
        . ${srcdir}/configure.srv; \  
        test -n "$UNSUPPORTED")  
    then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
        noconfigdirs="$noconfigdirs gdbserver"  
    else  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    fi  
    fi  
fi  
  
# Disable libgo for some systems where it is known to not work.  
# For testing, you can easily override this with --enable-libgo.  
if test x$enable_libgo = x; then  
    case "${target}" in  
    *-*-darwin*)  
    # PR 46986  
    noconfigdirs="$noconfigdirs target-libgo"  
    ;;  
    *-*-cygwin* | *-*-mingw*)  
    noconfigdirs="$noconfigdirs target-libgo"  
    ;;  
    bpf-*-*)  
        noconfigdirs="$noconfigdirs target-libgo"  
        ;;  
    esac  
fi  
  
# Default libgloss CPU subdirectory.  
libgloss_dir="$target_cpu"  
  
case "${target}" in  
  sh*-*-pe|mips*-*-pe|*arm-wince-pe)  
    libgloss_dir=wince  
    ;;  
  aarch64*-*-* )  
    libgloss_dir=aarch64  
    ;;  
  arm*-*-*)  
    libgloss_dir=arm  
    ;;  
  cris-*-* | crisv32-*-*)  
    libgloss_dir=cris  
    ;;  
  kvx-*-elf)  
    libgloss_dir=kvx-elf  
    ;;  
  kvx-*-mbr)  
    libgloss_dir=kvx-mbr  
    ;;  
  kvx-*-cos)  
    libgloss_dir=kvx-cos  
    ;;  
  hppa*-*-*)  
    libgloss_dir=pa  
    ;;  
  i[3456789]86-*-*)  
    libgloss_dir=i386  
    ;;  
  loongarch*-*-*)  
    libgloss_dir=loongarch  
    ;;  
  m68hc11-*-*|m6811-*-*|m68hc12-*-*|m6812-*-*)  
    libgloss_dir=m68hc11  
    ;;  
  m68*-*-* | fido-*-*)  
    libgloss_dir=m68k  
    ;;  
  mips*-*-*)  
    libgloss_dir=mips  
    ;;  
  powerpc*-*-*)  
    libgloss_dir=rs6000  
    ;;  
  pru-*-*)  
    libgloss_dir=pru  
    ;;  
  sparc*-*-*)  
    libgloss_dir=sparc  
    ;;  
esac  
  
# Disable newlib and libgloss for various target OSes.  
case "${target}" in  
  alpha*-dec-osf*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  i[3456789]86-*-linux*)  
    # This section makes it possible to build newlib natively on linux.  
    # If we are using a cross compiler then don't configure newlib.  
    if test x${is_cross_compiler} != xno ; then  
      noconfigdirs="$noconfigdirs target-newlib"  
    fi  
    noconfigdirs="$noconfigdirs target-libgloss"  
    # If we are not using a cross compiler, do configure newlib.  
    # Note however, that newlib will only be configured in this situation  
    # if the --with-newlib option has been given, because otherwise  
    # 'target-newlib' will appear in skipdirs.  
    ;;  
  i[3456789]86-*-rdos*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  sh*-*-pe|mips*-*-pe|arm-wince-pe)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  sparc-*-sunos4*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  bpf-*-*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-aix*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-beos*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-chorusos)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-dragonfly*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-freebsd*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-linux* | *-*-gnu* | *-*-k*bsd*-gnu | *-*-kopensolaris*-gnu)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-lynxos*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-mingw*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-netbsd*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-netware*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-tpf*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-uclinux*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  *-*-vxworks*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
esac  
  
case "${target}" in  
  *-*-chorusos)  
    ;;  
  aarch64-*-darwin*)  
    noconfigdirs="$noconfigdirs ld gas gdb gprof"  
    noconfigdirs="$noconfigdirs sim target-rda"  
    ;;  
  amdgcn*-*-*)  
    ;;  
  arm-*-darwin*)  
    noconfigdirs="$noconfigdirs ld gas gdb gprof"  
    noconfigdirs="$noconfigdirs sim target-rda"  
    ;;  
  powerpc-*-darwin*)  
    noconfigdirs="$noconfigdirs ld gas gdb gprof"  
    noconfigdirs="$noconfigdirs sim target-rda"  
    ;;  
  i[3456789]86-*-darwin*)  
    noconfigdirs="$noconfigdirs ld gprof"  
    noconfigdirs="$noconfigdirs sim target-rda"  
    ;;  
  x86_64-*-darwin[912]*)  
    noconfigdirs="$noconfigdirs ld gas gprof"  
    noconfigdirs="$noconfigdirs sim target-rda"  
    ;;  
  *-*-darwin*)  
    noconfigdirs="$noconfigdirs ld gas gdb gprof"  
    noconfigdirs="$noconfigdirs sim target-rda"  
    ;;  
  *-*-dragonfly*)  
    ;;  
  *-*-freebsd*)  
    if test "x$with_gmp" = x \  
    && ! test -d ${srcdir}/gmp \  
    && test -f /usr/local/include/gmp.h; then  
      with_gmp=/usr/local  
    fi  
    ;;  
  *-*-kaos*)  
    # Remove unsupported stuff on all kaOS configurations.  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  *-*-netbsd*)  
    ;;  
  *-*-netware*)  
    ;;  
  *-*-phoenix*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  *-*-rtems*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
    # The tpf target doesn't support gdb yet.  
  *-*-tpf*)  
    noconfigdirs="$noconfigdirs gdb tcl tk libgui itcl"  
    ;;  
  *-*-uclinux*)  
    noconfigdirs="$noconfigdirs target-rda"  
    ;;  
  *-*-vxworks*)  
    ;;  
  alpha*-dec-osf*)  
    # ld works, but does not support shared libraries.  
    # gas doesn't generate exception information.  
    noconfigdirs="$noconfigdirs gas ld"  
    ;;  
  alpha*-*-*vms*)  
    noconfigdirs="$noconfigdirs gdb target-newlib target-libgloss"  
    ;;  
  alpha*-*-*)  
    # newlib is not 64 bit ready  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  bpf-*-*)  
    noconfigdirs="$noconfigdirs target-libobjc target-libbacktrace"  
    ;;  
  sh*-*-pe|mips*-*-pe|*arm-wince-pe)  
    noconfigdirs="$noconfigdirs tcl tk itcl libgui sim"  
    ;;  
  arc*-*-*)  
    noconfigdirs="$noconfigdirs sim"  
    ;;  
  arm-*-pe*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  arm-*-riscix*)  
    noconfigdirs="$noconfigdirs ld target-libgloss"  
    ;;  
  avr-*-*)  
    if test x${with_avrlibc} != xno; then  
      noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    fi  
    ;;  
  c4x-*-* | tic4x-*-*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  tic54x-*-*)  
    noconfigdirs="$noconfigdirs target-libgloss gdb"  
    ;;  
  d10v-*-*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  d30v-*-*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  fr30-*-elf*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  ft32-*-*)  
    noconfigdirs="$noconfigdirs target-rda gprof"  
    ;;  
  moxie-*-*)  
    noconfigdirs="$noconfigdirs"  
    ;;  
  h8300*-*-*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  h8500-*-*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  hppa1.1-*-osf* | hppa1.1-*-bsd* )  
    ;;  
  hppa*64*-*-hpux*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  hppa*-*-hpux11*)  
    noconfigdirs="$noconfigdirs gdb ld"  
    ;;  
  hppa*64*-*-linux*)  
    ;;  
  hppa*-*-linux*)  
    ;;  
  hppa*-*-*elf* | \  
  hppa*-*-lites* | \  
  hppa*-*-openbsd* | \  
  hppa*64*-*-*)  
    ;;  
  hppa*-*-pro*)  
    ;;  
  hppa*-*-*)  
    noconfigdirs="$noconfigdirs ld"  
    ;;  
  i960-*-*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  ia64*-*-elf*)  
    # No gdb support yet.  
    noconfigdirs="$noconfigdirs readline libgui itcl gdb"  
    ;;  
  ia64*-**-hpux*)  
    # No ld support yet.  
    noconfigdirs="$noconfigdirs gdb libgui itcl ld"  
    ;;  
  ia64*-*-*vms*)  
    # No ld support yet.  
    noconfigdirs="$noconfigdirs libgui itcl ld"  
    ;;  
  i[3456789]86-w64-mingw*)  
    ;;  
  i[3456789]86-*-mingw*)  
    target_configdirs="$target_configdirs target-winsup"  
    ;;  
  *-*-cygwin*)  
    target_configdirs="$target_configdirs target-libtermcap target-winsup"  
    noconfigdirs="$noconfigdirs target-libgloss"  
    # always build newlib if winsup directory is present.  
    if test -d "$srcdir/winsup/cygwin"; then  
      skipdirs=`echo " ${skipdirs} " | sed -e 's/ target-newlib / /'`  
    elif test -d "$srcdir/newlib"; then  
      echo "Warning: winsup/cygwin is missing so newlib can't be built."  
    fi  
    ;;  
  i[3456789]86-*-pe)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  i[3456789]86-*-sco3.2v5*)  
    # The linker does not yet know about weak symbols in COFF,  
    # and is not configured to handle mixed ELF and COFF.  
    noconfigdirs="$noconfigdirs ld target-libgloss"  
    ;;  
  i[3456789]86-*-sco*)  
    noconfigdirs="$noconfigdirs gprof target-libgloss"  
    ;;  
  i[3456789]86-*-solaris2* | x86_64-*-solaris2.1[0-9]*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  i[3456789]86-*-sysv4*)  
    noconfigdirs="$noconfigdirs target-libgloss"  
    ;;  
  i[3456789]86-*-beos*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  i[3456789]86-*-rdos*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  kvx-*-*)  
    noconfigdirs="$noconfigdirs gdb gdbserver sim"  
    ;;  
  mmix-*-*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  mt-*-*)  
    noconfigdirs="$noconfigdirs sim"  
    ;;  
  nfp-*-*)  
    noconfigdirs="$noconfigdirs ld gas gdb gprof sim"  
    noconfigdirs="$noconfigdirs $target_libraries"  
    ;;  
  pdp11-*-*)  
    noconfigdirs="$noconfigdirs gdb gprof"  
    ;;  
  powerpc-*-aix*)  
    # copied from rs6000-*-* entry  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
  powerpc*-*-winnt* | powerpc*-*-pe*)  
    target_configdirs="$target_configdirs target-winsup"  
    noconfigdirs="$noconfigdirs gdb tcl tk target-libgloss itcl"  
    # always build newlib.  
    skipdirs=`echo " ${skipdirs} " | sed -e 's/ target-newlib / /'`  
    ;;  
    # This is temporary until we can link against shared libraries  
  powerpcle-*-solaris*)  
    noconfigdirs="$noconfigdirs gdb sim tcl tk itcl"  
    ;;  
  powerpc-*-beos*)  
    noconfigdirs="$noconfigdirs gdb"  
    ;;  
  rs6000-*-lynxos*)  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
  rs6000-*-aix*)  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
  rs6000-*-*)  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
  m68k-apollo-*)  
    noconfigdirs="$noconfigdirs ld binutils gprof target-libgloss"  
    ;;  
  microblaze*)  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
  mips*-sde-elf* | mips*-mti-elf* | mips*-img-elf*)  
    if test x$with_newlib = xyes; then  
      noconfigdirs="$noconfigdirs gprof"  
    fi  
    ;;  
  mips*-*-irix5*)  
    noconfigdirs="$noconfigdirs gprof target-libgloss"  
    ;;  
  mips*-*-irix6*)  
    noconfigdirs="$noconfigdirs gprof target-libgloss"  
    ;;  
  mips*-*-bsd*)  
    noconfigdirs="$noconfigdirs ld gas gprof target-libgloss"  
    ;;  
  mips*-*-linux*)  
    ;;  
  mips*-*-ultrix* | mips*-*-osf* | mips*-*-ecoff* | mips*-*-pe* \  
  | mips*-*-irix* | mips*-*-lnews* | mips*-*-riscos*)  
    noconfigdirs="$noconfigdirs ld gas gprof"  
    ;;  
  mips*-*-*)  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
  nvptx*-*-*)  
    noconfigdirs="$noconfigdirs target-libssp target-libstdc++-v3 target-libobjc"  
    ;;  
  sh-*-*)  
    case "${target}" in  
      sh*-*-elf)  
         ;;  
      *)  
         noconfigdirs="$noconfigdirs target-libgloss" ;;  
    esac  
    ;;  
  sparc-*-sunos4*)  
    if test x${is_cross_compiler} = xno ; then  
           use_gnu_ld=no  
    fi  
    ;;  
  tic6x-*-*)  
    noconfigdirs="$noconfigdirs sim"  
    ;;  
  tilepro*-*-* | tilegx*-*-*)  
    noconfigdirs="$noconfigdirs sim"  
    ;;  
  v810-*-*)  
    noconfigdirs="$noconfigdirs bfd binutils gas gdb ld opcodes target-libgloss"  
    ;;  
  vax-*-*)  
    noconfigdirs="$noconfigdirs target-newlib target-libgloss"  
    ;;  
  wasm32-*-*)  
    noconfigdirs="$noconfigdirs ld"  
    ;;  
  loongarch*-*-linux*)  
    ;;  
  loongarch*-*-*)  
    noconfigdirs="$noconfigdirs gprof"  
    ;;  
esac  
  
# If we aren't building newlib, then don't build libgloss, since libgloss  
# depends upon some newlib header files.  
case "${noconfigdirs}" in  
  *target-libgloss*) ;;  
  *target-newlib*) noconfigdirs="$noconfigdirs target-libgloss" ;;  
esac  
  
# Work in distributions that contain no compiler tools, like Autoconf.  
host_makefile_frag=/dev/null  
if test -d ${srcdir}/config ; then  
case "${host}" in  
  i[3456789]86-*-msdosdjgpp*)  
    host_makefile_frag="config/mh-djgpp"  
    ;;  
  *-cygwin*)  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking to see if cat works as expected" >&5  
$as_echo_n "checking to see if cat works as expected... " >&6; }  
echo a >cygwin-cat-check  
if test `cat cygwin-cat-check` = a ; then  
  rm cygwin-cat-check  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  rm cygwin-cat-check  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
  as_fn_error $? "The cat command does not ignore carriage return characters.  
  Please either mount the build directory in binary mode or run the following  
  commands before running any configure script:  
set -o igncr  
export SHELLOPTS  
  " "$LINENO" 5  
fi  
  
    host_makefile_frag="config/mh-cygwin"  
    ;;  
  *-mingw*)  
    host_makefile_frag="config/mh-mingw"  
    ;;  
  alpha*-linux*)  
    host_makefile_frag="config/mh-alpha-linux"  
    ;;  
  hppa*-hp-hpux*)  
    host_makefile_frag="config/mh-pa"  
    ;;  
  hppa*-*)  
    host_makefile_frag="config/mh-pa"  
    ;;  
  i?86-*-darwin[89]* | i?86-*-darwin1[0-7]* | powerpc*-*-darwin*)  
    host_makefile_frag="config/mh-darwin"  
    ;;  
  powerpc-*-aix*)  
    host_makefile_frag="config/mh-ppc-aix"  
    ;;  
  rs6000-*-aix*)  
    host_makefile_frag="config/mh-ppc-aix"  
    ;;  
esac  
fi  
  
if test "${build}" != "${host}" ; then  
  AR_FOR_BUILD=${AR_FOR_BUILD-ar}  
  AS_FOR_BUILD=${AS_FOR_BUILD-as}  
  CC_FOR_BUILD=${CC_FOR_BUILD-gcc}  
  CPP_FOR_BUILD="${CPP_FOR_BUILD-\$(CC_FOR_BUILD) -E}"  
  CXX_FOR_BUILD=${CXX_FOR_BUILD-g++}  
  DSYMUTIL_FOR_BUILD=${DSYMUTIL_FOR_BUILD-dsymutil}  
  GFORTRAN_FOR_BUILD=${GFORTRAN_FOR_BUILD-gfortran}  
  GOC_FOR_BUILD=${GOC_FOR_BUILD-gccgo}  
  GDC_FOR_BUILD=${GDC_FOR_BUILD-gdc}  
  DLLTOOL_FOR_BUILD=${DLLTOOL_FOR_BUILD-dlltool}  
  LD_FOR_BUILD=${LD_FOR_BUILD-ld}  
  NM_FOR_BUILD=${NM_FOR_BUILD-nm}  
  RANLIB_FOR_BUILD=${RANLIB_FOR_BUILD-ranlib}  
  WINDRES_FOR_BUILD=${WINDRES_FOR_BUILD-windres}  
  WINDMC_FOR_BUILD=${WINDMC_FOR_BUILD-windmc}  
else  
  AR_FOR_BUILD="\$(AR)"  
  AS_FOR_BUILD="\$(AS)"  
  CC_FOR_BUILD="\$(CC)"  
  CXX_FOR_BUILD="\$(CXX)"  
  DSYMUTIL_FOR_BUILD="\$(DSYMUTIL)"  
  GFORTRAN_FOR_BUILD="\$(GFORTRAN)"  
  GOC_FOR_BUILD="\$(GOC)"  
  GDC_FOR_BUILD="\$(GDC)"  
  DLLTOOL_FOR_BUILD="\$(DLLTOOL)"  
  LD_FOR_BUILD="\$(LD)"  
  NM_FOR_BUILD="\$(NM)"  
  RANLIB_FOR_BUILD="\$(RANLIB)"  
  WINDRES_FOR_BUILD="\$(WINDRES)"  
  WINDMC_FOR_BUILD="\$(WINDMC)"  
fi  
  
ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
if test -n "$ac_tool_prefix"; then  
  # Extract the first word of "${ac_tool_prefix}gcc", so it can be a program name with args.  
set dummy ${ac_tool_prefix}gcc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC"; then  
  ac_cv_prog_CC="$CC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CC="${ac_tool_prefix}gcc"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CC=$ac_cv_prog_CC  
if test -n "$CC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC" >&5  
$as_echo "$CC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_prog_CC"; then  
  ac_ct_CC=$CC  
  # Extract the first word of "gcc", so it can be a program name with args.  
set dummy gcc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_CC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_CC"; then  
  ac_cv_prog_ac_ct_CC="$ac_ct_CC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_CC="gcc"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_CC=$ac_cv_prog_ac_ct_CC  
if test -n "$ac_ct_CC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_CC" >&5  
$as_echo "$ac_ct_CC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  if test "x$ac_ct_CC" = x; then  
    CC=""  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    CC=$ac_ct_CC  
  fi  
else  
  CC="$ac_cv_prog_CC"  
fi  
  
if test -z "$CC"; then  
          if test -n "$ac_tool_prefix"; then  
    # Extract the first word of "${ac_tool_prefix}cc", so it can be a program name with args.  
set dummy ${ac_tool_prefix}cc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC"; then  
  ac_cv_prog_CC="$CC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CC="${ac_tool_prefix}cc"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CC=$ac_cv_prog_CC  
if test -n "$CC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC" >&5  
$as_echo "$CC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
fi  
if test -z "$CC"; then  
  # Extract the first word of "cc", so it can be a program name with args.  
set dummy cc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC"; then  
  ac_cv_prog_CC="$CC" # Let the user override the test.  
else  
  ac_prog_rejected=no  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    if test "$as_dir/$ac_word$ac_exec_ext" = "/usr/ucb/cc"; then  
       ac_prog_rejected=yes  
       continue  
     fi  
    ac_cv_prog_CC="cc"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
if test $ac_prog_rejected = yes; then  
  # We found a bogon in the path, so make sure we never use it.  
  set dummy $ac_cv_prog_CC  
  shift  
  if test $# != 0; then  
    # We chose a different compiler from the bogus one.  
    # However, it has the same basename, so the bogon will be chosen  
    # first if we set CC to just the basename; use the full file name.  
    shift  
    ac_cv_prog_CC="$as_dir/$ac_word${1+' '}$@"  
  fi  
fi  
fi  
fi  
CC=$ac_cv_prog_CC  
if test -n "$CC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC" >&5  
$as_echo "$CC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$CC"; then  
  if test -n "$ac_tool_prefix"; then  
  for ac_prog in cl.exe  
  do  
    # Extract the first word of "$ac_tool_prefix$ac_prog", so it can be a program name with args.  
set dummy $ac_tool_prefix$ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC"; then  
  ac_cv_prog_CC="$CC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CC="$ac_tool_prefix$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CC=$ac_cv_prog_CC  
if test -n "$CC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC" >&5  
$as_echo "$CC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    test -n "$CC" && break  
  done  
fi  
if test -z "$CC"; then  
  ac_ct_CC=$CC  
  for ac_prog in cl.exe  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_CC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_CC"; then  
  ac_cv_prog_ac_ct_CC="$ac_ct_CC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_CC="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_CC=$ac_cv_prog_ac_ct_CC  
if test -n "$ac_ct_CC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_CC" >&5  
$as_echo "$ac_ct_CC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$ac_ct_CC" && break  
done  
  
  if test "x$ac_ct_CC" = x; then  
    CC=""  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    CC=$ac_ct_CC  
  fi  
fi  
  
fi  
  
  
test -z "$CC" && { { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
as_fn_error $? "no acceptable C compiler found in \$PATH  
See \`config.log' for more details" "$LINENO" 5; }  
  
# Provide some information about the compiler.  
$as_echo "$as_me:${as_lineno-$LINENO}: checking for C compiler version" >&5  
set X $ac_compile  
ac_compiler=$2  
for ac_option in --version -v -V -qversion; do  
  { { ac_try="$ac_compiler $ac_option >&5"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_compiler $ac_option >&5") 2>conftest.err  
  ac_status=$?  
  if test -s conftest.err; then  
    sed '10a\  
... rest of stderr output deleted ...  
         10q' conftest.err >conftest.er1  
    cat conftest.er1 >&5  
  fi  
  rm -f conftest.er1 conftest.err  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }  
done  
  
cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
ac_clean_files_save=$ac_clean_files  
ac_clean_files="$ac_clean_files a.out a.out.dSYM a.exe b.out"  
# Try to create an executable without -o first, disregard a.out.  
# It will help us diagnose broken compilers, and finding out an intuition  
# of exeext.  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether the C compiler works" >&5  
$as_echo_n "checking whether the C compiler works... " >&6; }  
ac_link_default=`$as_echo "$ac_link" | sed 's/ -o *conftest[^ ]*//'`  
  
# The possible output files:  
ac_files="a.out conftest.exe conftest a.exe a_out.exe b.out conftest.*"  
  
ac_rmfiles=  
for ac_file in $ac_files  
do  
  case $ac_file in  
    *.$ac_ext | *.xcoff | *.tds | *.d | *.pdb | *.xSYM | *.bb | *.bbg | *.map | *.inf | *.dSYM | *.o | *.obj ) ;;  
    * ) ac_rmfiles="$ac_rmfiles $ac_file";;  
  esac  
done  
rm -f $ac_rmfiles  
  
if { { ac_try="$ac_link_default"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_link_default") 2>&5  
  ac_status=$?  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }; then :  
  # Autoconf-2.13 could set the ac_cv_exeext variable to `no'.  
# So ignore a value of `no', otherwise this would lead to `EXEEXT = no'  
# in a Makefile.  We should not override ac_cv_exeext if it was cached,  
# so that the user can short-circuit this test for compilers unknown to  
# Autoconf.  
for ac_file in $ac_files ''  
do  
  test -f "$ac_file" || continue  
  case $ac_file in  
    *.$ac_ext | *.xcoff | *.tds | *.d | *.pdb | *.xSYM | *.bb | *.bbg | *.map | *.inf | *.dSYM | *.o | *.obj )  
    ;;  
    [ab].out )  
    # We found the default executable, but exeext='' is most  
    # certainly right.  
    break;;  
    *.* )  
    if test "${ac_cv_exeext+set}" = set && test "$ac_cv_exeext" != no;  
    then :; else  
       ac_cv_exeext=`expr "$ac_file" : '[^.]*\(\..*\)'`  
    fi  
    # We set ac_cv_exeext here because the later test for it is not  
    # safe: cross compilers may not add the suffix if given an `-o'  
    # argument, so we may need to know it at that point already.  
    # Even if this section looks crufty: it has the advantage of  
    # actually working.  
    break;;  
    * )  
    break;;  
  esac  
done  
test "$ac_cv_exeext" = no && ac_cv_exeext=  
  
else  
  ac_file=''  
fi  
if test -z "$ac_file"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
$as_echo "$as_me: failed program was:" >&5  
sed 's/^/| /' conftest.$ac_ext >&5  
  
{ { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
as_fn_error 77 "C compiler cannot create executables  
See \`config.log' for more details" "$LINENO" 5; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for C compiler default output file name" >&5  
$as_echo_n "checking for C compiler default output file name... " >&6; }  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_file" >&5  
$as_echo "$ac_file" >&6; }  
ac_exeext=$ac_cv_exeext  
  
rm -f -r a.out a.out.dSYM a.exe conftest$ac_cv_exeext b.out  
ac_clean_files=$ac_clean_files_save  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for suffix of executables" >&5  
$as_echo_n "checking for suffix of executables... " >&6; }  
if { { ac_try="$ac_link"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_link") 2>&5  
  ac_status=$?  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }; then :  
  # If both `conftest.exe' and `conftest' are `present' (well, observable)  
# catch `conftest.exe'.  For instance with Cygwin, `ls conftest' will  
# work properly (i.e., refer to `conftest.exe'), while it won't with  
# `rm'.  
for ac_file in conftest.exe conftest conftest.*; do  
  test -f "$ac_file" || continue  
  case $ac_file in  
    *.$ac_ext | *.xcoff | *.tds | *.d | *.pdb | *.xSYM | *.bb | *.bbg | *.map | *.inf | *.dSYM | *.o | *.obj ) ;;  
    *.* ) ac_cv_exeext=`expr "$ac_file" : '[^.]*\(\..*\)'`  
      break;;  
    * ) break;;  
  esac  
done  
else  
  { { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
as_fn_error $? "cannot compute suffix of executables: cannot compile and link  
See \`config.log' for more details" "$LINENO" 5; }  
fi  
rm -f conftest conftest$ac_cv_exeext  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_exeext" >&5  
$as_echo "$ac_cv_exeext" >&6; }  
  
rm -f conftest.$ac_ext  
EXEEXT=$ac_cv_exeext  
ac_exeext=$EXEEXT  
cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <stdio.h>  
int  
main ()  
{  
FILE *f = fopen ("conftest.out", "w");  
 return ferror (f) || fclose (f) != 0;  
  
  ;  
  return 0;  
}  
_ACEOF  
ac_clean_files="$ac_clean_files conftest.out"  
# Check that the compiler produces executables we can run.  If not, either  
# the compiler is broken, or we cross compile.  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether we are cross compiling" >&5  
$as_echo_n "checking whether we are cross compiling... " >&6; }  
if test "$cross_compiling" != yes; then  
  { { ac_try="$ac_link"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_link") 2>&5  
  ac_status=$?  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }  
  if { ac_try='./conftest$ac_cv_exeext'  
  { { case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_try") 2>&5  
  ac_status=$?  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }; }; then  
    cross_compiling=no  
  else  
    if test "$cross_compiling" = maybe; then  
    cross_compiling=yes  
    else  
    { { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
as_fn_error $? "cannot run C compiled programs.  
If you meant to cross compile, use \`--host'.  
See \`config.log' for more details" "$LINENO" 5; }  
    fi  
  fi  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $cross_compiling" >&5  
$as_echo "$cross_compiling" >&6; }  
  
rm -f conftest.$ac_ext conftest$ac_cv_exeext conftest.out  
ac_clean_files=$ac_clean_files_save  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for suffix of object files" >&5  
$as_echo_n "checking for suffix of object files... " >&6; }  
if ${ac_cv_objext+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
rm -f conftest.o conftest.obj  
if { { ac_try="$ac_compile"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_compile") 2>&5  
  ac_status=$?  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }; then :  
  for ac_file in conftest.o conftest.obj conftest.*; do  
  test -f "$ac_file" || continue;  
  case $ac_file in  
    *.$ac_ext | *.xcoff | *.tds | *.d | *.pdb | *.xSYM | *.bb | *.bbg | *.map | *.inf | *.dSYM ) ;;  
    *) ac_cv_objext=`expr "$ac_file" : '.*\.\(.*\)'`  
       break;;  
  esac  
done  
else  
  $as_echo "$as_me: failed program was:" >&5  
sed 's/^/| /' conftest.$ac_ext >&5  
  
{ { $as_echo "$as_me:${as_lineno-$LINENO}: error: in \`$ac_pwd':" >&5  
$as_echo "$as_me: error: in \`$ac_pwd':" >&2;}  
as_fn_error $? "cannot compute suffix of object files: cannot compile  
See \`config.log' for more details" "$LINENO" 5; }  
fi  
rm -f conftest.$ac_cv_objext conftest.$ac_ext  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_objext" >&5  
$as_echo "$ac_cv_objext" >&6; }  
OBJEXT=$ac_cv_objext  
ac_objext=$OBJEXT  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether we are using the GNU C compiler" >&5  
$as_echo_n "checking whether we are using the GNU C compiler... " >&6; }  
if ${ac_cv_c_compiler_gnu+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
#ifndef __GNUC__  
       choke me  
#endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  ac_compiler_gnu=yes  
else  
  ac_compiler_gnu=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
ac_cv_c_compiler_gnu=$ac_compiler_gnu  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_c_compiler_gnu" >&5  
$as_echo "$ac_cv_c_compiler_gnu" >&6; }  
if test $ac_compiler_gnu = yes; then  
  GCC=yes  
else  
  GCC=  
fi  
ac_test_CFLAGS=${CFLAGS+set}  
ac_save_CFLAGS=$CFLAGS  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CC accepts -g" >&5  
$as_echo_n "checking whether $CC accepts -g... " >&6; }  
if ${ac_cv_prog_cc_g+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_save_c_werror_flag=$ac_c_werror_flag  
   ac_c_werror_flag=yes  
   ac_cv_prog_cc_g=no  
   CFLAGS="-g"  
   cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  ac_cv_prog_cc_g=yes  
else  
  CFLAGS=""  
      cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  
else  
  ac_c_werror_flag=$ac_save_c_werror_flag  
     CFLAGS="-g"  
     cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  ac_cv_prog_cc_g=yes  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
   ac_c_werror_flag=$ac_save_c_werror_flag  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_prog_cc_g" >&5  
$as_echo "$ac_cv_prog_cc_g" >&6; }  
if test "$ac_test_CFLAGS" = set; then  
  CFLAGS=$ac_save_CFLAGS  
elif test $ac_cv_prog_cc_g = yes; then  
  if test "$GCC" = yes; then  
    CFLAGS="-g -O2"  
  else  
    CFLAGS="-g"  
  fi  
else  
  if test "$GCC" = yes; then  
    CFLAGS="-O2"  
  else  
    CFLAGS=  
  fi  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $CC option to accept ISO C89" >&5  
$as_echo_n "checking for $CC option to accept ISO C89... " >&6; }  
if ${ac_cv_prog_cc_c89+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_cv_prog_cc_c89=no  
ac_save_CC=$CC  
cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <stdarg.h>  
#include <stdio.h>  
struct stat;  
/* Most of the following tests are stolen from RCS 5.7's src/conf.sh.  */  
struct buf { int x; };  
FILE * (*rcsopen) (struct buf *, struct stat *, int);  
static char *e (p, i)  
     char **p;  
     int i;  
{  
  return p[i];  
}  
static char *f (char * (*g) (char **, int), char **p, ...)  
{  
  char *s;  
  va_list v;  
  va_start (v,p);  
  s = g (p, va_arg (v,int));  
  va_end (v);  
  return s;  
}  
  
/* OSF 4.0 Compaq cc is some sort of almost-ANSI by default.  It has  
   function prototypes and stuff, but not '\xHH' hex character constants.  
   These don't provoke an error unfortunately, instead are silently treated  
   as 'x'.  The following induces an error, until -std is added to get  
   proper ANSI mode.  Curiously '\x00'!='x' always comes out true, for an  
   array size at least.  It's necessary to write '\x00'==0 to get something  
   that's true only with -std.  */  
int osf4_cc_array ['\x00' == 0 ? 1 : -1];  
  
/* IBM C 6 for AIX is almost-ANSI by default, but it replaces macro parameters  
   inside strings and character constants.  */  
#define FOO(x) 'x'  
int xlc6_cc_array[FOO(a) == 'x' ? 1 : -1];  
  
int test (int i, double x);  
struct s1 {int (*f) (int a);};  
struct s2 {int (*f) (double a);};  
int pairnames (int, char **, FILE *(*)(struct buf *, struct stat *, int), int, int);  
int argc;  
char **argv;  
int  
main ()  
{  
return f (e, argv, 0) != argv[0]  ||  f (e, argv, 1) != argv[1];  
  ;  
  return 0;  
}  
_ACEOF  
for ac_arg in '' -qlanglvl=extc89 -qlanglvl=ansi -std \  
    -Ae "-Aa -D_HPUX_SOURCE" "-Xc -D__EXTENSIONS__"  
do  
  CC="$ac_save_CC $ac_arg"  
  if ac_fn_c_try_compile "$LINENO"; then :  
  ac_cv_prog_cc_c89=$ac_arg  
fi  
rm -f core conftest.err conftest.$ac_objext  
  test "x$ac_cv_prog_cc_c89" != "xno" && break  
done  
rm -f conftest.$ac_ext  
CC=$ac_save_CC  
  
fi  
# AC_CACHE_VAL  
case "x$ac_cv_prog_cc_c89" in  
  x)  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: none needed" >&5  
$as_echo "none needed" >&6; } ;;  
  xno)  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: unsupported" >&5  
$as_echo "unsupported" >&6; } ;;  
  *)  
    CC="$CC $ac_cv_prog_cc_c89"  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_prog_cc_c89" >&5  
$as_echo "$ac_cv_prog_cc_c89" >&6; } ;;  
esac  
if test "x$ac_cv_prog_cc_c89" != xno; then :  
  
fi  
  
ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
  
   { $as_echo "$as_me:${as_lineno-$LINENO}: checking for $CC option to accept ISO C99" >&5  
$as_echo_n "checking for $CC option to accept ISO C99... " >&6; }  
if ${ac_cv_prog_cc_c99+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_cv_prog_cc_c99=no  
ac_save_CC=$CC  
cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <stdarg.h>  
#include <stdbool.h>  
#include <stdlib.h>  
#include <wchar.h>  
#include <stdio.h>  
  
// Check varargs macros.  These examples are taken from C99 6.10.3.5.  
#define debug(...) fprintf (stderr, __VA_ARGS__)  
#define showlist(...) puts (#__VA_ARGS__)  
#define report(test,...) ((test) ? puts (#test) : printf (__VA_ARGS__))  
static void  
test_varargs_macros (void)  
{  
  int x = 1234;  
  int y = 5678;  
  debug ("Flag");  
  debug ("X = %d\n", x);  
  showlist (The first, second, and third items.);  
  report (x>y, "x is %d but y is %d", x, y);  
}  
  
// Check long long types.  
#define BIG64 18446744073709551615ull  
#define BIG32 4294967295ul  
#define BIG_OK (BIG64 / BIG32 == 4294967297ull && BIG64 % BIG32 == 0)  
#if !BIG_OK  
  your preprocessor is broken;  
#endif  
#if BIG_OK  
#else  
  your preprocessor is broken;  
#endif  
static long long int bignum = -9223372036854775807LL;  
static unsigned long long int ubignum = BIG64;  
  
struct incomplete_array  
{  
  int datasize;  
  double data[];  
};  
  
struct named_init {  
  int number;  
  const wchar_t *name;  
  double average;  
};  
  
typedef const char *ccp;  
  
static inline int  
test_restrict (ccp restrict text)  
{  
  // See if C++-style comments work.  
  // Iterate through items via the restricted pointer.  
  // Also check for declarations in for loops.  
  for (unsigned int i = 0; *(text+i) != '\0'; ++i)  
    continue;  
  return 0;  
}  
  
// Check varargs and va_copy.  
static void  
test_varargs (const char *format, ...)  
{  
  va_list args;  
  va_start (args, format);  
  va_list args_copy;  
  va_copy (args_copy, args);  
  
  const char *str;  
  int number;  
  float fnumber;  
  
  while (*format)  
    {  
      switch (*format++)  
    {  
    case 's': // string  
      str = va_arg (args_copy, const char *);  
      break;  
    case 'd': // int  
      number = va_arg (args_copy, int);  
      break;  
    case 'f': // float  
      fnumber = va_arg (args_copy, double);  
      break;  
    default:  
      break;  
    }  
    }  
  va_end (args_copy);  
  va_end (args);  
}  
  
int  
main ()  
{  
  
  // Check bool.  
  _Bool success = false;  
  
  // Check restrict.  
  if (test_restrict ("String literal") == 0)  
    success = true;  
  char *restrict newvar = "Another string";  
  
  // Check varargs.  
  test_varargs ("s, d' f .", "string", 65, 34.234);  
  test_varargs_macros ();  
  
  // Check flexible array members.  
  struct incomplete_array *ia =  
    malloc (sizeof (struct incomplete_array) + (sizeof (double) * 10));  
  ia->datasize = 10;  
  for (int i = 0; i < ia->datasize; ++i)  
    ia->data[i] = i * 1.234;  
  
  // Check named initializers.  
  struct named_init ni = {  
    .number = 34,  
    .name = L"Test wide string",  
    .average = 543.34343,  
  };  
  
  ni.number = 58;  
  
  int dynamic_array[ni.number];  
  dynamic_array[ni.number - 1] = 543;  
  
  // work around unused variable warnings  
  return (!success || bignum == 0LL || ubignum == 0uLL || newvar[0] == 'x'  
      || dynamic_array[ni.number - 1] != 543);  
  
  ;  
  return 0;  
}  
_ACEOF  
for ac_arg in '' -std=gnu99 -std=c99 -c99 -AC99 -D_STDC_C99= -qlanglvl=extc99  
do  
  CC="$ac_save_CC $ac_arg"  
  if ac_fn_c_try_compile "$LINENO"; then :  
  ac_cv_prog_cc_c99=$ac_arg  
fi  
rm -f core conftest.err conftest.$ac_objext  
  test "x$ac_cv_prog_cc_c99" != "xno" && break  
done  
rm -f conftest.$ac_ext  
CC=$ac_save_CC  
  
fi  
# AC_CACHE_VAL  
case "x$ac_cv_prog_cc_c99" in  
  x)  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: none needed" >&5  
$as_echo "none needed" >&6; } ;;  
  xno)  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: unsupported" >&5  
$as_echo "unsupported" >&6; } ;;  
  *)  
    CC="$CC $ac_cv_prog_cc_c99"  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_prog_cc_c99" >&5  
$as_echo "$ac_cv_prog_cc_c99" >&6; } ;;  
esac  
if test "x$ac_cv_prog_cc_c99" != xno; then :  
  
fi  
  
  
ac_ext=cpp  
ac_cpp='$CXXCPP $CPPFLAGS'  
ac_compile='$CXX -c $CXXFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CXX -o conftest$ac_exeext $CXXFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_cxx_compiler_gnu  
if test -z "$CXX"; then  
  if test -n "$CCC"; then  
    CXX=$CCC  
  else  
    if test -n "$ac_tool_prefix"; then  
  for ac_prog in g++ c++ gpp aCC CC cxx cc++ cl.exe FCC KCC RCC xlC_r xlC  
  do  
    # Extract the first word of "$ac_tool_prefix$ac_prog", so it can be a program name with args.  
set dummy $ac_tool_prefix$ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CXX+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CXX"; then  
  ac_cv_prog_CXX="$CXX" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CXX="$ac_tool_prefix$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CXX=$ac_cv_prog_CXX  
if test -n "$CXX"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CXX" >&5  
$as_echo "$CXX" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    test -n "$CXX" && break  
  done  
fi  
if test -z "$CXX"; then  
  ac_ct_CXX=$CXX  
  for ac_prog in g++ c++ gpp aCC CC cxx cc++ cl.exe FCC KCC RCC xlC_r xlC  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_CXX+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_CXX"; then  
  ac_cv_prog_ac_ct_CXX="$ac_ct_CXX" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_CXX="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_CXX=$ac_cv_prog_ac_ct_CXX  
if test -n "$ac_ct_CXX"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_CXX" >&5  
$as_echo "$ac_ct_CXX" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$ac_ct_CXX" && break  
done  
  
  if test "x$ac_ct_CXX" = x; then  
    CXX="g++"  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    CXX=$ac_ct_CXX  
  fi  
fi  
  
  fi  
fi  
# Provide some information about the compiler.  
$as_echo "$as_me:${as_lineno-$LINENO}: checking for C++ compiler version" >&5  
set X $ac_compile  
ac_compiler=$2  
for ac_option in --version -v -V -qversion; do  
  { { ac_try="$ac_compiler $ac_option >&5"  
case "(($ac_try" in  
  *\"* | *\`* | *\\*) ac_try_echo=\$ac_try;;  
  *) ac_try_echo=$ac_try;;  
esac  
eval ac_try_echo="\"\$as_me:${as_lineno-$LINENO}: $ac_try_echo\""  
$as_echo "$ac_try_echo"; } >&5  
  (eval "$ac_compiler $ac_option >&5") 2>conftest.err  
  ac_status=$?  
  if test -s conftest.err; then  
    sed '10a\  
... rest of stderr output deleted ...  
         10q' conftest.err >conftest.er1  
    cat conftest.er1 >&5  
  fi  
  rm -f conftest.er1 conftest.err  
  $as_echo "$as_me:${as_lineno-$LINENO}: \$? = $ac_status" >&5  
  test $ac_status = 0; }  
done  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether we are using the GNU C++ compiler" >&5  
$as_echo_n "checking whether we are using the GNU C++ compiler... " >&6; }  
if ${ac_cv_cxx_compiler_gnu+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
#ifndef __GNUC__  
       choke me  
#endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  ac_compiler_gnu=yes  
else  
  ac_compiler_gnu=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
ac_cv_cxx_compiler_gnu=$ac_compiler_gnu  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_cxx_compiler_gnu" >&5  
$as_echo "$ac_cv_cxx_compiler_gnu" >&6; }  
if test $ac_compiler_gnu = yes; then  
  GXX=yes  
else  
  GXX=  
fi  
ac_test_CXXFLAGS=${CXXFLAGS+set}  
ac_save_CXXFLAGS=$CXXFLAGS  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX accepts -g" >&5  
$as_echo_n "checking whether $CXX accepts -g... " >&6; }  
if ${ac_cv_prog_cxx_g+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_save_cxx_werror_flag=$ac_cxx_werror_flag  
   ac_cxx_werror_flag=yes  
   ac_cv_prog_cxx_g=no  
   CXXFLAGS="-g"  
   cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  ac_cv_prog_cxx_g=yes  
else  
  CXXFLAGS=""  
      cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  
else  
  ac_cxx_werror_flag=$ac_save_cxx_werror_flag  
     CXXFLAGS="-g"  
     cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
int  
main ()  
{  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  ac_cv_prog_cxx_g=yes  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
   ac_cxx_werror_flag=$ac_save_cxx_werror_flag  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_prog_cxx_g" >&5  
$as_echo "$ac_cv_prog_cxx_g" >&6; }  
if test "$ac_test_CXXFLAGS" = set; then  
  CXXFLAGS=$ac_save_CXXFLAGS  
elif test $ac_cv_prog_cxx_g = yes; then  
  if test "$GXX" = yes; then  
    CXXFLAGS="-g -O2"  
  else  
    CXXFLAGS="-g"  
  fi  
else  
  if test "$GXX" = yes; then  
    CXXFLAGS="-O2"  
  else  
    CXXFLAGS=  
  fi  
fi  
ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
  
  
# We must set the default linker to the linker used by gcc for the correct  
# operation of libtool.  If LD is not defined and we are using gcc, try to  
# set the LD default to the ld used by gcc.  
if test -z "$LD"; then  
  if test "$GCC" = yes; then  
    case $build in  
    *-*-mingw*)  
      gcc_prog_ld=`$CC -print-prog-name=ld 2>&1 | tr -d '\015'` ;;  
    *)  
      gcc_prog_ld=`$CC -print-prog-name=ld 2>&1` ;;  
    esac  
    case $gcc_prog_ld in  
    # Accept absolute paths.  
    [\\/]* | [A-Za-z]:[\\/]*)  
      LD="$gcc_prog_ld" ;;  
    esac  
  fi  
fi  
  
# Check whether -static-libstdc++ -static-libgcc is supported.  
have_static_libs=no  
if test "$GCC" = yes; then  
  saved_LDFLAGS="$LDFLAGS"  
  
  LDFLAGS="$LDFLAGS -static-libstdc++ -static-libgcc"  
  { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether g++ accepts -static-libstdc++ -static-libgcc" >&5  
$as_echo_n "checking whether g++ accepts -static-libstdc++ -static-libgcc... " >&6; }  
  ac_ext=cpp  
ac_cpp='$CXXCPP $CPPFLAGS'  
ac_compile='$CXX -c $CXXFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CXX -o conftest$ac_exeext $CXXFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_cxx_compiler_gnu  
  
  
cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
#if (__GNUC__ < 4) || (__GNUC__ == 4 && __GNUC_MINOR__ < 5)  
#error -static-libstdc++ not implemented  
#endif  
int main() {}  
_ACEOF  
if ac_fn_cxx_try_link "$LINENO"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }; have_static_libs=yes  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
rm -f core conftest.err conftest.$ac_objext \  
    conftest$ac_exeext conftest.$ac_ext  
  ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
  
  
  LDFLAGS="$saved_LDFLAGS"  
fi  
  
  
  
  
if test -n "$ac_tool_prefix"; then  
  # Extract the first word of "${ac_tool_prefix}gnatbind", so it can be a program name with args.  
set dummy ${ac_tool_prefix}gnatbind; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GNATBIND+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GNATBIND"; then  
  ac_cv_prog_GNATBIND="$GNATBIND" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GNATBIND="${ac_tool_prefix}gnatbind"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GNATBIND=$ac_cv_prog_GNATBIND  
if test -n "$GNATBIND"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GNATBIND" >&5  
$as_echo "$GNATBIND" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_prog_GNATBIND"; then  
  ac_ct_GNATBIND=$GNATBIND  
  # Extract the first word of "gnatbind", so it can be a program name with args.  
set dummy gnatbind; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_GNATBIND+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_GNATBIND"; then  
  ac_cv_prog_ac_ct_GNATBIND="$ac_ct_GNATBIND" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_GNATBIND="gnatbind"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_GNATBIND=$ac_cv_prog_ac_ct_GNATBIND  
if test -n "$ac_ct_GNATBIND"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_GNATBIND" >&5  
$as_echo "$ac_ct_GNATBIND" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  if test "x$ac_ct_GNATBIND" = x; then  
    GNATBIND="no"  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    GNATBIND=$ac_ct_GNATBIND  
  fi  
else  
  GNATBIND="$ac_cv_prog_GNATBIND"  
fi  
  
if test -n "$ac_tool_prefix"; then  
  # Extract the first word of "${ac_tool_prefix}gnatmake", so it can be a program name with args.  
set dummy ${ac_tool_prefix}gnatmake; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GNATMAKE+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GNATMAKE"; then  
  ac_cv_prog_GNATMAKE="$GNATMAKE" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GNATMAKE="${ac_tool_prefix}gnatmake"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GNATMAKE=$ac_cv_prog_GNATMAKE  
if test -n "$GNATMAKE"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GNATMAKE" >&5  
$as_echo "$GNATMAKE" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_prog_GNATMAKE"; then  
  ac_ct_GNATMAKE=$GNATMAKE  
  # Extract the first word of "gnatmake", so it can be a program name with args.  
set dummy gnatmake; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_GNATMAKE+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_GNATMAKE"; then  
  ac_cv_prog_ac_ct_GNATMAKE="$ac_ct_GNATMAKE" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_GNATMAKE="gnatmake"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_GNATMAKE=$ac_cv_prog_ac_ct_GNATMAKE  
if test -n "$ac_ct_GNATMAKE"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_GNATMAKE" >&5  
$as_echo "$ac_ct_GNATMAKE" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  if test "x$ac_ct_GNATMAKE" = x; then  
    GNATMAKE="no"  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    GNATMAKE=$ac_ct_GNATMAKE  
  fi  
else  
  GNATMAKE="$ac_cv_prog_GNATMAKE"  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether compiler driver understands Ada and is recent enough" >&5  
$as_echo_n "checking whether compiler driver understands Ada and is recent enough... " >&6; }  
if ${acx_cv_cc_gcc_supports_ada+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat >conftest.adb \<<EOF  
pragma Warnings (Off);  
with System.CRTL;  
pragma Warnings (On);  
use type System.CRTL.int64;  
procedure conftest is begin null; end conftest;  
EOF  
acx_cv_cc_gcc_supports_ada=no  
# There is a bug in old released versions of GCC which causes the  
# driver to exit successfully when the appropriate language module  
# has not been installed.  This is fixed in 2.95.4, 3.0.2, and 3.1.  
# Therefore we must check for the error message as well as an  
# unsuccessful exit.  
# Other compilers, like HP Tru64 UNIX cc, exit successfully when  
# given a .adb file, but produce no object file.  So we must check  
# if an object file was really produced to guard against this.  
errors=`(${CC} -c conftest.adb) 2>&1 || echo failure`  
if test x"$errors" = x && test -f conftest.$ac_objext; then  
  acx_cv_cc_gcc_supports_ada=yes  
fi  
rm -f conftest.*  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $acx_cv_cc_gcc_supports_ada" >&5  
$as_echo "$acx_cv_cc_gcc_supports_ada" >&6; }  
  
if test "x$GNATBIND" != xno && test "x$GNATMAKE" != xno && test x$acx_cv_cc_gcc_supports_ada != xno; then  
  have_gnat=yes  
else  
  have_gnat=no  
fi  
  
  
  
if test -n "$ac_tool_prefix"; then  
  # Extract the first word of "${ac_tool_prefix}gdc", so it can be a program name with args.  
set dummy ${ac_tool_prefix}gdc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GDC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GDC"; then  
  ac_cv_prog_GDC="$GDC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GDC="${ac_tool_prefix}gdc"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GDC=$ac_cv_prog_GDC  
if test -n "$GDC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GDC" >&5  
$as_echo "$GDC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_prog_GDC"; then  
  ac_ct_GDC=$GDC  
  # Extract the first word of "gdc", so it can be a program name with args.  
set dummy gdc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_GDC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_GDC"; then  
  ac_cv_prog_ac_ct_GDC="$ac_ct_GDC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_GDC="gdc"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_GDC=$ac_cv_prog_ac_ct_GDC  
if test -n "$ac_ct_GDC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_GDC" >&5  
$as_echo "$ac_ct_GDC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  if test "x$ac_ct_GDC" = x; then  
    GDC="no"  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    GDC=$ac_ct_GDC  
  fi  
else  
  GDC="$ac_cv_prog_GDC"  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether the D compiler works" >&5  
$as_echo_n "checking whether the D compiler works... " >&6; }  
if ${acx_cv_d_compiler_works+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat >conftest.d \<<EOF  
module conftest; int main() { return 0; }  
EOF  
acx_cv_d_compiler_works=no  
if test "x$GDC" != xno; then  
  errors=`(${GDC} -c conftest.d) 2>&1 || echo failure`  
  if test x"$errors" = x && test -f conftest.$ac_objext; then  
    acx_cv_d_compiler_works=yes  
  fi  
  rm -f conftest.*  
fi  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $acx_cv_d_compiler_works" >&5  
$as_echo "$acx_cv_d_compiler_works" >&6; }  
if test "x$GDC" != xno && test x$acx_cv_d_compiler_works != xno; then  
  have_gdc=yes  
else  
  have_gdc=no  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking how to compare bootstrapped objects" >&5  
$as_echo_n "checking how to compare bootstrapped objects... " >&6; }  
if ${gcc_cv_prog_cmp_skip+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
   echo abfoo >t1  
  echo cdfoo >t2  
  gcc_cv_prog_cmp_skip='tail -c +17 $$f1 > tmp-foo1; tail -c +17 $$f2 > tmp-foo2; cmp tmp-foo1 tmp-foo2'  
  if cmp t1 t2 2 2 > /dev/null 2>&1; then  
    if cmp t1 t2 1 1 > /dev/null 2>&1; then  
      :  
    else  
      gcc_cv_prog_cmp_skip='cmp $$f1 $$f2 16 16'  
    fi  
  fi  
  if cmp --ignore-initial=2 t1 t2 > /dev/null 2>&1; then  
    if cmp --ignore-initial=1 t1 t2 > /dev/null 2>&1; then  
      :  
    else  
      gcc_cv_prog_cmp_skip='cmp --ignore-initial=16 $$f1 $$f2'  
    fi  
  fi  
  rm t1 t2  
  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $gcc_cv_prog_cmp_skip" >&5  
$as_echo "$gcc_cv_prog_cmp_skip" >&6; }  
do_compare="$gcc_cv_prog_cmp_skip"  
  
  
  
# Check whether --enable-bootstrap was given.  
if test "${enable_bootstrap+set}" = set; then :  
  enableval=$enable_bootstrap;  
else  
  enable_bootstrap=default  
fi  
  
  
# Issue errors and warnings for invalid/strange bootstrap combinations.  
if test -r $srcdir/gcc/configure; then  
  have_compiler=yes  
else  
  have_compiler=no  
fi  
  
case "$have_compiler:$host:$target:$enable_bootstrap" in  
  *:*:*:no) ;;  
  
  # Default behavior.  Enable bootstrap if we have a compiler  
  # and we are in a native configuration.  
  yes:$build:$build:default)  
    enable_bootstrap=yes ;;  
  
  *:*:*:default)  
    enable_bootstrap=no ;;  
  
  # We have a compiler and we are in a native configuration, bootstrap is ok  
  yes:$build:$build:yes)  
    ;;  
  
  # Other configurations, but we have a compiler.  Assume the user knows  
  # what he's doing.  
  yes:*:*:yes)  
    { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: trying to bootstrap a cross compiler" >&5  
$as_echo "$as_me: WARNING: trying to bootstrap a cross compiler" >&2;}  
    ;;  
  
  # No compiler: if they passed --enable-bootstrap explicitly, fail  
  no:*:*:yes)  
    as_fn_error $? "cannot bootstrap without a compiler" "$LINENO" 5 ;;  
  
  # Fail if wrong command line  
  *)  
    as_fn_error $? "invalid option for --enable-bootstrap" "$LINENO" 5  
    ;;  
esac  
  
# When bootstrapping with GCC, build stage 1 in C++11 mode to ensure that a  
# C++11 compiler can still start the bootstrap.  Otherwise, if building GCC,  
# require C++11 (or higher).  
if test "$enable_bootstrap:$GXX" = "yes:yes"; then  
  CXX="$CXX -std=c++11"  
elif test "$have_compiler" = yes; then  
    ax_cxx_compile_alternatives="11 0x"    ax_cxx_compile_cxx11_required=true  
  ac_ext=cpp  
ac_cpp='$CXXCPP $CPPFLAGS'  
ac_compile='$CXX -c $CXXFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CXX -o conftest$ac_exeext $CXXFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_cxx_compiler_gnu  
  ac_success=no  
  
      { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX supports C++11 features by default" >&5  
$as_echo_n "checking whether $CXX supports C++11 features by default... " >&6; }  
if ${ax_cv_cxx_compile_cxx11+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
  
// If the compiler admits that it is not ready for C++11, why torture it?  
// Hopefully, this will speed up the test.  
  
#ifndef __cplusplus  
  
#error "This is not a C++ compiler"  
  
#elif __cplusplus < 201103L  
  
#error "This is not a C++11 compiler"  
  
#else  
  
namespace cxx11  
{  
  
  namespace test_static_assert  
  {  
  
    template <typename T>  
    struct check  
    {  
      static_assert(sizeof(int) <= sizeof(T), "not big enough");  
    };  
  
  }  
  
  namespace test_final_override  
  {  
  
    struct Base  
    {  
      virtual ~Base() {}  
      virtual void f() {}  
    };  
  
    struct Derived : public Base  
    {  
      virtual ~Derived() override {}  
      virtual void f() override {}  
    };  
  
  }  
  
  namespace test_double_right_angle_brackets  
  {  
  
    template < typename T >  
    struct check {};  
  
    typedef check<void> single_type;  
    typedef check<check<void>> double_type;  
    typedef check<check<check<void>>> triple_type;  
    typedef check<check<check<check<void>>>> quadruple_type;  
  
  }  
  
  namespace test_decltype  
  {  
  
    int  
    f()  
    {  
      int a = 1;  
      decltype(a) b = 2;  
      return a + b;  
    }  
  
  }  
  
  namespace test_type_deduction  
  {  
  
    template < typename T1, typename T2 >  
    struct is_same  
    {  
      static const bool value = false;  
    };  
  
    template < typename T >  
    struct is_same<T, T>  
    {  
      static const bool value = true;  
    };  
  
    template < typename T1, typename T2 >  
    auto  
    add(T1 a1, T2 a2) -> decltype(a1 + a2)  
    {  
      return a1 + a2;  
    }  
  
    int  
    test(const int c, volatile int v)  
    {  
      static_assert(is_same<int, decltype(0)>::value == true, "");  
      static_assert(is_same<int, decltype(c)>::value == false, "");  
      static_assert(is_same<int, decltype(v)>::value == false, "");  
      auto ac = c;  
      auto av = v;  
      auto sumi = ac + av + 'x';  
      auto sumf = ac + av + 1.0;  
      static_assert(is_same<int, decltype(ac)>::value == true, "");  
      static_assert(is_same<int, decltype(av)>::value == true, "");  
      static_assert(is_same<int, decltype(sumi)>::value == true, "");  
      static_assert(is_same<int, decltype(sumf)>::value == false, "");  
      static_assert(is_same<int, decltype(add(c, v))>::value == true, "");  
      return (sumf > 0.0) ? sumi : add(c, v);  
    }  
  
  }  
  
  namespace test_noexcept  
  {  
  
    int f() { return 0; }  
    int g() noexcept { return 0; }  
  
    static_assert(noexcept(f()) == false, "");  
    static_assert(noexcept(g()) == true, "");  
  
  }  
  
  namespace test_constexpr  
  {  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c_r(const CharT *const s, const unsigned long acc) noexcept  
    {  
      return *s ? strlen_c_r(s + 1, acc + 1) : acc;  
    }  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c(const CharT *const s) noexcept  
    {  
      return strlen_c_r(s, 0UL);  
    }  
  
    static_assert(strlen_c("") == 0UL, "");  
    static_assert(strlen_c("1") == 1UL, "");  
    static_assert(strlen_c("example") == 7UL, "");  
    static_assert(strlen_c("another\0example") == 7UL, "");  
  
  }  
  
  namespace test_rvalue_references  
  {  
  
    template < int N >  
    struct answer  
    {  
      static constexpr int value = N;  
    };  
  
    answer<1> f(int&)       { return answer<1>(); }  
    answer<2> f(const int&) { return answer<2>(); }  
    answer<3> f(int&&)      { return answer<3>(); }  
  
    void  
    test()  
    {  
      int i = 0;  
      const int c = 0;  
      static_assert(decltype(f(i))::value == 1, "");  
      static_assert(decltype(f(c))::value == 2, "");  
      static_assert(decltype(f(0))::value == 3, "");  
    }  
  
  }  
  
  namespace test_uniform_initialization  
  {  
  
    struct test  
    {  
      static const int zero {};  
      static const int one {1};  
    };  
  
    static_assert(test::zero == 0, "");  
    static_assert(test::one == 1, "");  
  
  }  
  
  namespace test_lambdas  
  {  
  
    void  
    test1()  
    {  
      auto lambda1 = [](){};  
      auto lambda2 = lambda1;  
      lambda1();  
      lambda2();  
    }  
  
    int  
    test2()  
    {  
      auto a = [](int i, int j){ return i + j; }(1, 2);  
      auto b = []() -> int { return '0'; }();  
      auto c = [=](){ return a + b; }();  
      auto d = [&](){ return c; }();  
      auto e = [a, &b](int x) mutable {  
        const auto identity = [](int y){ return y; };  
        for (auto i = 0; i < a; ++i)  
          a += b--;  
        return x + identity(a + b);  
      }(0);  
      return a + b + c + d + e;  
    }  
  
    int  
    test3()  
    {  
      const auto nullary = [](){ return 0; };  
      const auto unary = [](int x){ return x; };  
      using nullary_t = decltype(nullary);  
      using unary_t = decltype(unary);  
      const auto higher1st = [](nullary_t f){ return f(); };  
      const auto higher2nd = [unary](nullary_t f1){  
        return [unary, f1](unary_t f2){ return f2(unary(f1())); };  
      };  
      return higher1st(nullary) + higher2nd(nullary)(unary);  
    }  
  
  }  
  
  namespace test_variadic_templates  
  {  
  
    template <int...>  
    struct sum;  
  
    template <int N0, int... N1toN>  
    struct sum<N0, N1toN...>  
    {  
      static constexpr auto value = N0 + sum<N1toN...>::value;  
    };  
  
    template <>  
    struct sum<>  
    {  
      static constexpr auto value = 0;  
    };  
  
    static_assert(sum<>::value == 0, "");  
    static_assert(sum<1>::value == 1, "");  
    static_assert(sum<23>::value == 23, "");  
    static_assert(sum<1, 2>::value == 3, "");  
    static_assert(sum<5, 5, 11>::value == 21, "");  
    static_assert(sum<2, 3, 5, 7, 11, 13>::value == 41, "");  
  
  }  
  
  // http://stackoverflow.com/questions/13728184/template-aliases-and-sfinae  
  // Clang 3.1 fails with headers of libstd++ 4.8.3 when using std::function  
  // because of this.  
  namespace test_template_alias_sfinae  
  {  
  
    struct foo {};  
  
    template<typename T>  
    using member = typename T::member_type;  
  
    template<typename T>  
    void func(...) {}  
  
    template<typename T>  
    void func(member<T>*) {}  
  
    void test();  
  
    void test() { func<foo>(0); }  
  
  }  
  
}  // namespace cxx11  
  
#endif  // __cplusplus >= 201103L  
  
  
  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  ax_cv_cxx_compile_cxx11=yes  
else  
  ax_cv_cxx_compile_cxx11=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ax_cv_cxx_compile_cxx11" >&5  
$as_echo "$ax_cv_cxx_compile_cxx11" >&6; }  
    if test x$ax_cv_cxx_compile_cxx11 = xyes; then  
      ac_success=yes  
    fi  
  
    if test x$ac_success = xno; then  
    for alternative in ${ax_cxx_compile_alternatives}; do  
      switch="-std=gnu++${alternative}"  
      cachevar=`$as_echo "ax_cv_cxx_compile_cxx11_$switch" | $as_tr_sh`  
      { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX supports C++11 features with $switch" >&5  
$as_echo_n "checking whether $CXX supports C++11 features with $switch... " >&6; }  
if eval \${$cachevar+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_save_CXX="$CXX"  
         CXX="$CXX $switch"  
         cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
  
// If the compiler admits that it is not ready for C++11, why torture it?  
// Hopefully, this will speed up the test.  
  
#ifndef __cplusplus  
  
#error "This is not a C++ compiler"  
  
#elif __cplusplus < 201103L  
  
#error "This is not a C++11 compiler"  
  
#else  
  
namespace cxx11  
{  
  
  namespace test_static_assert  
  {  
  
    template <typename T>  
    struct check  
    {  
      static_assert(sizeof(int) <= sizeof(T), "not big enough");  
    };  
  
  }  
  
  namespace test_final_override  
  {  
  
    struct Base  
    {  
      virtual ~Base() {}  
      virtual void f() {}  
    };  
  
    struct Derived : public Base  
    {  
      virtual ~Derived() override {}  
      virtual void f() override {}  
    };  
  
  }  
  
  namespace test_double_right_angle_brackets  
  {  
  
    template < typename T >  
    struct check {};  
  
    typedef check<void> single_type;  
    typedef check<check<void>> double_type;  
    typedef check<check<check<void>>> triple_type;  
    typedef check<check<check<check<void>>>> quadruple_type;  
  
  }  
  
  namespace test_decltype  
  {  
  
    int  
    f()  
    {  
      int a = 1;  
      decltype(a) b = 2;  
      return a + b;  
    }  
  
  }  
  
  namespace test_type_deduction  
  {  
  
    template < typename T1, typename T2 >  
    struct is_same  
    {  
      static const bool value = false;  
    };  
  
    template < typename T >  
    struct is_same<T, T>  
    {  
      static const bool value = true;  
    };  
  
    template < typename T1, typename T2 >  
    auto  
    add(T1 a1, T2 a2) -> decltype(a1 + a2)  
    {  
      return a1 + a2;  
    }  
  
    int  
    test(const int c, volatile int v)  
    {  
      static_assert(is_same<int, decltype(0)>::value == true, "");  
      static_assert(is_same<int, decltype(c)>::value == false, "");  
      static_assert(is_same<int, decltype(v)>::value == false, "");  
      auto ac = c;  
      auto av = v;  
      auto sumi = ac + av + 'x';  
      auto sumf = ac + av + 1.0;  
      static_assert(is_same<int, decltype(ac)>::value == true, "");  
      static_assert(is_same<int, decltype(av)>::value == true, "");  
      static_assert(is_same<int, decltype(sumi)>::value == true, "");  
      static_assert(is_same<int, decltype(sumf)>::value == false, "");  
      static_assert(is_same<int, decltype(add(c, v))>::value == true, "");  
      return (sumf > 0.0) ? sumi : add(c, v);  
    }  
  
  }  
  
  namespace test_noexcept  
  {  
  
    int f() { return 0; }  
    int g() noexcept { return 0; }  
  
    static_assert(noexcept(f()) == false, "");  
    static_assert(noexcept(g()) == true, "");  
  
  }  
  
  namespace test_constexpr  
  {  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c_r(const CharT *const s, const unsigned long acc) noexcept  
    {  
      return *s ? strlen_c_r(s + 1, acc + 1) : acc;  
    }  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c(const CharT *const s) noexcept  
    {  
      return strlen_c_r(s, 0UL);  
    }  
  
    static_assert(strlen_c("") == 0UL, "");  
    static_assert(strlen_c("1") == 1UL, "");  
    static_assert(strlen_c("example") == 7UL, "");  
    static_assert(strlen_c("another\0example") == 7UL, "");  
  
  }  
  
  namespace test_rvalue_references  
  {  
  
    template < int N >  
    struct answer  
    {  
      static constexpr int value = N;  
    };  
  
    answer<1> f(int&)       { return answer<1>(); }  
    answer<2> f(const int&) { return answer<2>(); }  
    answer<3> f(int&&)      { return answer<3>(); }  
  
    void  
    test()  
    {  
      int i = 0;  
      const int c = 0;  
      static_assert(decltype(f(i))::value == 1, "");  
      static_assert(decltype(f(c))::value == 2, "");  
      static_assert(decltype(f(0))::value == 3, "");  
    }  
  
  }  
  
  namespace test_uniform_initialization  
  {  
  
    struct test  
    {  
      static const int zero {};  
      static const int one {1};  
    };  
  
    static_assert(test::zero == 0, "");  
    static_assert(test::one == 1, "");  
  
  }  
  
  namespace test_lambdas  
  {  
  
    void  
    test1()  
    {  
      auto lambda1 = [](){};  
      auto lambda2 = lambda1;  
      lambda1();  
      lambda2();  
    }  
  
    int  
    test2()  
    {  
      auto a = [](int i, int j){ return i + j; }(1, 2);  
      auto b = []() -> int { return '0'; }();  
      auto c = [=](){ return a + b; }();  
      auto d = [&](){ return c; }();  
      auto e = [a, &b](int x) mutable {  
        const auto identity = [](int y){ return y; };  
        for (auto i = 0; i < a; ++i)  
          a += b--;  
        return x + identity(a + b);  
      }(0);  
      return a + b + c + d + e;  
    }  
  
    int  
    test3()  
    {  
      const auto nullary = [](){ return 0; };  
      const auto unary = [](int x){ return x; };  
      using nullary_t = decltype(nullary);  
      using unary_t = decltype(unary);  
      const auto higher1st = [](nullary_t f){ return f(); };  
      const auto higher2nd = [unary](nullary_t f1){  
        return [unary, f1](unary_t f2){ return f2(unary(f1())); };  
      };  
      return higher1st(nullary) + higher2nd(nullary)(unary);  
    }  
  
  }  
  
  namespace test_variadic_templates  
  {  
  
    template <int...>  
    struct sum;  
  
    template <int N0, int... N1toN>  
    struct sum<N0, N1toN...>  
    {  
      static constexpr auto value = N0 + sum<N1toN...>::value;  
    };  
  
    template <>  
    struct sum<>  
    {  
      static constexpr auto value = 0;  
    };  
  
    static_assert(sum<>::value == 0, "");  
    static_assert(sum<1>::value == 1, "");  
    static_assert(sum<23>::value == 23, "");  
    static_assert(sum<1, 2>::value == 3, "");  
    static_assert(sum<5, 5, 11>::value == 21, "");  
    static_assert(sum<2, 3, 5, 7, 11, 13>::value == 41, "");  
  
  }  
  
  // http://stackoverflow.com/questions/13728184/template-aliases-and-sfinae  
  // Clang 3.1 fails with headers of libstd++ 4.8.3 when using std::function  
  // because of this.  
  namespace test_template_alias_sfinae  
  {  
  
    struct foo {};  
  
    template<typename T>  
    using member = typename T::member_type;  
  
    template<typename T>  
    void func(...) {}  
  
    template<typename T>  
    void func(member<T>*) {}  
  
    void test();  
  
    void test() { func<foo>(0); }  
  
  }  
  
}  // namespace cxx11  
  
#endif  // __cplusplus >= 201103L  
  
  
  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  eval $cachevar=yes  
else  
  eval $cachevar=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
         CXX="$ac_save_CXX"  
fi  
eval ac_res=\$$cachevar  
           { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_res" >&5  
$as_echo "$ac_res" >&6; }  
      if eval test x\$$cachevar = xyes; then  
        CXX="$CXX $switch"  
        if test -n "$CXXCPP" ; then  
          CXXCPP="$CXXCPP $switch"  
        fi  
        ac_success=yes  
        break  
      fi  
    done  
  fi  
  
    if test x$ac_success = xno; then  
                for alternative in ${ax_cxx_compile_alternatives}; do  
      for switch in -std=c++${alternative} +std=c++${alternative} "-h std=c++${alternative}"; do  
        cachevar=`$as_echo "ax_cv_cxx_compile_cxx11_$switch" | $as_tr_sh`  
        { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX supports C++11 features with $switch" >&5  
$as_echo_n "checking whether $CXX supports C++11 features with $switch... " >&6; }  
if eval \${$cachevar+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_save_CXX="$CXX"  
           CXX="$CXX $switch"  
           cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
  
// If the compiler admits that it is not ready for C++11, why torture it?  
// Hopefully, this will speed up the test.  
  
#ifndef __cplusplus  
  
#error "This is not a C++ compiler"  
  
#elif __cplusplus < 201103L  
  
#error "This is not a C++11 compiler"  
  
#else  
  
namespace cxx11  
{  
  
  namespace test_static_assert  
  {  
  
    template <typename T>  
    struct check  
    {  
      static_assert(sizeof(int) <= sizeof(T), "not big enough");  
    };  
  
  }  
  
  namespace test_final_override  
  {  
  
    struct Base  
    {  
      virtual ~Base() {}  
      virtual void f() {}  
    };  
  
    struct Derived : public Base  
    {  
      virtual ~Derived() override {}  
      virtual void f() override {}  
    };  
  
  }  
  
  namespace test_double_right_angle_brackets  
  {  
  
    template < typename T >  
    struct check {};  
  
    typedef check<void> single_type;  
    typedef check<check<void>> double_type;  
    typedef check<check<check<void>>> triple_type;  
    typedef check<check<check<check<void>>>> quadruple_type;  
  
  }  
  
  namespace test_decltype  
  {  
  
    int  
    f()  
    {  
      int a = 1;  
      decltype(a) b = 2;  
      return a + b;  
    }  
  
  }  
  
  namespace test_type_deduction  
  {  
  
    template < typename T1, typename T2 >  
    struct is_same  
    {  
      static const bool value = false;  
    };  
  
    template < typename T >  
    struct is_same<T, T>  
    {  
      static const bool value = true;  
    };  
  
    template < typename T1, typename T2 >  
    auto  
    add(T1 a1, T2 a2) -> decltype(a1 + a2)  
    {  
      return a1 + a2;  
    }  
  
    int  
    test(const int c, volatile int v)  
    {  
      static_assert(is_same<int, decltype(0)>::value == true, "");  
      static_assert(is_same<int, decltype(c)>::value == false, "");  
      static_assert(is_same<int, decltype(v)>::value == false, "");  
      auto ac = c;  
      auto av = v;  
      auto sumi = ac + av + 'x';  
      auto sumf = ac + av + 1.0;  
      static_assert(is_same<int, decltype(ac)>::value == true, "");  
      static_assert(is_same<int, decltype(av)>::value == true, "");  
      static_assert(is_same<int, decltype(sumi)>::value == true, "");  
      static_assert(is_same<int, decltype(sumf)>::value == false, "");  
      static_assert(is_same<int, decltype(add(c, v))>::value == true, "");  
      return (sumf > 0.0) ? sumi : add(c, v);  
    }  
  
  }  
  
  namespace test_noexcept  
  {  
  
    int f() { return 0; }  
    int g() noexcept { return 0; }  
  
    static_assert(noexcept(f()) == false, "");  
    static_assert(noexcept(g()) == true, "");  
  
  }  
  
  namespace test_constexpr  
  {  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c_r(const CharT *const s, const unsigned long acc) noexcept  
    {  
      return *s ? strlen_c_r(s + 1, acc + 1) : acc;  
    }  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c(const CharT *const s) noexcept  
    {  
      return strlen_c_r(s, 0UL);  
    }  
  
    static_assert(strlen_c("") == 0UL, "");  
    static_assert(strlen_c("1") == 1UL, "");  
    static_assert(strlen_c("example") == 7UL, "");  
    static_assert(strlen_c("another\0example") == 7UL, "");  
  
  }  
  
  namespace test_rvalue_references  
  {  
  
    template < int N >  
    struct answer  
    {  
      static constexpr int value = N;  
    };  
  
    answer<1> f(int&)       { return answer<1>(); }  
    answer<2> f(const int&) { return answer<2>(); }  
    answer<3> f(int&&)      { return answer<3>(); }  
  
    void  
    test()  
    {  
      int i = 0;  
      const int c = 0;  
      static_assert(decltype(f(i))::value == 1, "");  
      static_assert(decltype(f(c))::value == 2, "");  
      static_assert(decltype(f(0))::value == 3, "");  
    }  
  
  }  
  
  namespace test_uniform_initialization  
  {  
  
    struct test  
    {  
      static const int zero {};  
      static const int one {1};  
    };  
  
    static_assert(test::zero == 0, "");  
    static_assert(test::one == 1, "");  
  
  }  
  
  namespace test_lambdas  
  {  
  
    void  
    test1()  
    {  
      auto lambda1 = [](){};  
      auto lambda2 = lambda1;  
      lambda1();  
      lambda2();  
    }  
  
    int  
    test2()  
    {  
      auto a = [](int i, int j){ return i + j; }(1, 2);  
      auto b = []() -> int { return '0'; }();  
      auto c = [=](){ return a + b; }();  
      auto d = [&](){ return c; }();  
      auto e = [a, &b](int x) mutable {  
        const auto identity = [](int y){ return y; };  
        for (auto i = 0; i < a; ++i)  
          a += b--;  
        return x + identity(a + b);  
      }(0);  
      return a + b + c + d + e;  
    }  
  
    int  
    test3()  
    {  
      const auto nullary = [](){ return 0; };  
      const auto unary = [](int x){ return x; };  
      using nullary_t = decltype(nullary);  
      using unary_t = decltype(unary);  
      const auto higher1st = [](nullary_t f){ return f(); };  
      const auto higher2nd = [unary](nullary_t f1){  
        return [unary, f1](unary_t f2){ return f2(unary(f1())); };  
      };  
      return higher1st(nullary) + higher2nd(nullary)(unary);  
    }  
  
  }  
  
  namespace test_variadic_templates  
  {  
  
    template <int...>  
    struct sum;  
  
    template <int N0, int... N1toN>  
    struct sum<N0, N1toN...>  
    {  
      static constexpr auto value = N0 + sum<N1toN...>::value;  
    };  
  
    template <>  
    struct sum<>  
    {  
      static constexpr auto value = 0;  
    };  
  
    static_assert(sum<>::value == 0, "");  
    static_assert(sum<1>::value == 1, "");  
    static_assert(sum<23>::value == 23, "");  
    static_assert(sum<1, 2>::value == 3, "");  
    static_assert(sum<5, 5, 11>::value == 21, "");  
    static_assert(sum<2, 3, 5, 7, 11, 13>::value == 41, "");  
  
  }  
  
  // http://stackoverflow.com/questions/13728184/template-aliases-and-sfinae  
  // Clang 3.1 fails with headers of libstd++ 4.8.3 when using std::function  
  // because of this.  
  namespace test_template_alias_sfinae  
  {  
  
    struct foo {};  
  
    template<typename T>  
    using member = typename T::member_type;  
  
    template<typename T>  
    void func(...) {}  
  
    template<typename T>  
    void func(member<T>*) {}  
  
    void test();  
  
    void test() { func<foo>(0); }  
  
  }  
  
}  // namespace cxx11  
  
#endif  // __cplusplus >= 201103L  
  
  
  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  eval $cachevar=yes  
else  
  eval $cachevar=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
           CXX="$ac_save_CXX"  
fi  
eval ac_res=\$$cachevar  
           { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_res" >&5  
$as_echo "$ac_res" >&6; }  
        if eval test x\$$cachevar = xyes; then  
          CXX="$CXX $switch"  
          if test -n "$CXXCPP" ; then  
            CXXCPP="$CXXCPP $switch"  
          fi  
          ac_success=yes  
          break  
        fi  
      done  
      if test x$ac_success = xyes; then  
        break  
      fi  
    done  
  fi  
  
  ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
  
  if test x$ax_cxx_compile_cxx11_required = xtrue; then  
    if test x$ac_success = xno; then  
      as_fn_error $? "*** A compiler with support for C++11 language features is required." "$LINENO" 5  
    fi  
  fi  
  if test x$ac_success = xno; then  
    HAVE_CXX11=0  
    { $as_echo "$as_me:${as_lineno-$LINENO}: No compiler with C++11 support was found" >&5  
$as_echo "$as_me: No compiler with C++11 support was found" >&6;}  
  else  
    HAVE_CXX11=1  
  
$as_echo "#define HAVE_CXX11 1" >>confdefs.h  
  
  fi  
  
  
  
  if test "${build}" != "${host}"; then  
      ax_cxx_compile_alternatives="11 0x"    ax_cxx_compile_cxx11_required=true  
  ac_ext=cpp  
ac_cpp='$CXXCPP $CPPFLAGS'  
ac_compile='$CXX -c $CXXFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CXX -o conftest$ac_exeext $CXXFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_cxx_compiler_gnu  
  ac_success=no  
      ax_cv_cxx_compile_cxx11_orig_cxx="$CXX"  
    ax_cv_cxx_compile_cxx11_orig_cxxflags="$CXXFLAGS"  
    ax_cv_cxx_compile_cxx11_orig_cppflags="$CPPFLAGS"  
    CXX="$CXX_FOR_BUILD"  
    CXXFLAGS="$CXXFLAGS_FOR_BUILD"  
    CPPFLAGS="$CPPFLAGS_FOR_BUILD"  
      { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX supports C++11 features by default" >&5  
$as_echo_n "checking whether $CXX supports C++11 features by default... " >&6; }  
if ${ax_cv_cxx_compile_cxx11_FOR_BUILD+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
  
// If the compiler admits that it is not ready for C++11, why torture it?  
// Hopefully, this will speed up the test.  
  
#ifndef __cplusplus  
  
#error "This is not a C++ compiler"  
  
#elif __cplusplus < 201103L  
  
#error "This is not a C++11 compiler"  
  
#else  
  
namespace cxx11  
{  
  
  namespace test_static_assert  
  {  
  
    template <typename T>  
    struct check  
    {  
      static_assert(sizeof(int) <= sizeof(T), "not big enough");  
    };  
  
  }  
  
  namespace test_final_override  
  {  
  
    struct Base  
    {  
      virtual ~Base() {}  
      virtual void f() {}  
    };  
  
    struct Derived : public Base  
    {  
      virtual ~Derived() override {}  
      virtual void f() override {}  
    };  
  
  }  
  
  namespace test_double_right_angle_brackets  
  {  
  
    template < typename T >  
    struct check {};  
  
    typedef check<void> single_type;  
    typedef check<check<void>> double_type;  
    typedef check<check<check<void>>> triple_type;  
    typedef check<check<check<check<void>>>> quadruple_type;  
  
  }  
  
  namespace test_decltype  
  {  
  
    int  
    f()  
    {  
      int a = 1;  
      decltype(a) b = 2;  
      return a + b;  
    }  
  
  }  
  
  namespace test_type_deduction  
  {  
  
    template < typename T1, typename T2 >  
    struct is_same  
    {  
      static const bool value = false;  
    };  
  
    template < typename T >  
    struct is_same<T, T>  
    {  
      static const bool value = true;  
    };  
  
    template < typename T1, typename T2 >  
    auto  
    add(T1 a1, T2 a2) -> decltype(a1 + a2)  
    {  
      return a1 + a2;  
    }  
  
    int  
    test(const int c, volatile int v)  
    {  
      static_assert(is_same<int, decltype(0)>::value == true, "");  
      static_assert(is_same<int, decltype(c)>::value == false, "");  
      static_assert(is_same<int, decltype(v)>::value == false, "");  
      auto ac = c;  
      auto av = v;  
      auto sumi = ac + av + 'x';  
      auto sumf = ac + av + 1.0;  
      static_assert(is_same<int, decltype(ac)>::value == true, "");  
      static_assert(is_same<int, decltype(av)>::value == true, "");  
      static_assert(is_same<int, decltype(sumi)>::value == true, "");  
      static_assert(is_same<int, decltype(sumf)>::value == false, "");  
      static_assert(is_same<int, decltype(add(c, v))>::value == true, "");  
      return (sumf > 0.0) ? sumi : add(c, v);  
    }  
  
  }  
  
  namespace test_noexcept  
  {  
  
    int f() { return 0; }  
    int g() noexcept { return 0; }  
  
    static_assert(noexcept(f()) == false, "");  
    static_assert(noexcept(g()) == true, "");  
  
  }  
  
  namespace test_constexpr  
  {  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c_r(const CharT *const s, const unsigned long acc) noexcept  
    {  
      return *s ? strlen_c_r(s + 1, acc + 1) : acc;  
    }  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c(const CharT *const s) noexcept  
    {  
      return strlen_c_r(s, 0UL);  
    }  
  
    static_assert(strlen_c("") == 0UL, "");  
    static_assert(strlen_c("1") == 1UL, "");  
    static_assert(strlen_c("example") == 7UL, "");  
    static_assert(strlen_c("another\0example") == 7UL, "");  
  
  }  
  
  namespace test_rvalue_references  
  {  
  
    template < int N >  
    struct answer  
    {  
      static constexpr int value = N;  
    };  
  
    answer<1> f(int&)       { return answer<1>(); }  
    answer<2> f(const int&) { return answer<2>(); }  
    answer<3> f(int&&)      { return answer<3>(); }  
  
    void  
    test()  
    {  
      int i = 0;  
      const int c = 0;  
      static_assert(decltype(f(i))::value == 1, "");  
      static_assert(decltype(f(c))::value == 2, "");  
      static_assert(decltype(f(0))::value == 3, "");  
    }  
  
  }  
  
  namespace test_uniform_initialization  
  {  
  
    struct test  
    {  
      static const int zero {};  
      static const int one {1};  
    };  
  
    static_assert(test::zero == 0, "");  
    static_assert(test::one == 1, "");  
  
  }  
  
  namespace test_lambdas  
  {  
  
    void  
    test1()  
    {  
      auto lambda1 = [](){};  
      auto lambda2 = lambda1;  
      lambda1();  
      lambda2();  
    }  
  
    int  
    test2()  
    {  
      auto a = [](int i, int j){ return i + j; }(1, 2);  
      auto b = []() -> int { return '0'; }();  
      auto c = [=](){ return a + b; }();  
      auto d = [&](){ return c; }();  
      auto e = [a, &b](int x) mutable {  
        const auto identity = [](int y){ return y; };  
        for (auto i = 0; i < a; ++i)  
          a += b--;  
        return x + identity(a + b);  
      }(0);  
      return a + b + c + d + e;  
    }  
  
    int  
    test3()  
    {  
      const auto nullary = [](){ return 0; };  
      const auto unary = [](int x){ return x; };  
      using nullary_t = decltype(nullary);  
      using unary_t = decltype(unary);  
      const auto higher1st = [](nullary_t f){ return f(); };  
      const auto higher2nd = [unary](nullary_t f1){  
        return [unary, f1](unary_t f2){ return f2(unary(f1())); };  
      };  
      return higher1st(nullary) + higher2nd(nullary)(unary);  
    }  
  
  }  
  
  namespace test_variadic_templates  
  {  
  
    template <int...>  
    struct sum;  
  
    template <int N0, int... N1toN>  
    struct sum<N0, N1toN...>  
    {  
      static constexpr auto value = N0 + sum<N1toN...>::value;  
    };  
  
    template <>  
    struct sum<>  
    {  
      static constexpr auto value = 0;  
    };  
  
    static_assert(sum<>::value == 0, "");  
    static_assert(sum<1>::value == 1, "");  
    static_assert(sum<23>::value == 23, "");  
    static_assert(sum<1, 2>::value == 3, "");  
    static_assert(sum<5, 5, 11>::value == 21, "");  
    static_assert(sum<2, 3, 5, 7, 11, 13>::value == 41, "");  
  
  }  
  
  // http://stackoverflow.com/questions/13728184/template-aliases-and-sfinae  
  // Clang 3.1 fails with headers of libstd++ 4.8.3 when using std::function  
  // because of this.  
  namespace test_template_alias_sfinae  
  {  
  
    struct foo {};  
  
    template<typename T>  
    using member = typename T::member_type;  
  
    template<typename T>  
    void func(...) {}  
  
    template<typename T>  
    void func(member<T>*) {}  
  
    void test();  
  
    void test() { func<foo>(0); }  
  
  }  
  
}  // namespace cxx11  
  
#endif  // __cplusplus >= 201103L  
  
  
  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  ax_cv_cxx_compile_cxx11_FOR_BUILD=yes  
else  
  ax_cv_cxx_compile_cxx11_FOR_BUILD=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $ax_cv_cxx_compile_cxx11_FOR_BUILD" >&5  
$as_echo "$ax_cv_cxx_compile_cxx11_FOR_BUILD" >&6; }  
    if test x$ax_cv_cxx_compile_cxx11_FOR_BUILD = xyes; then  
      ac_success=yes  
    fi  
  
    if test x$ac_success = xno; then  
    for alternative in ${ax_cxx_compile_alternatives}; do  
      switch="-std=gnu++${alternative}"  
      cachevar=`$as_echo "ax_cv_cxx_compile_cxx11_FOR_BUILD_$switch" | $as_tr_sh`  
      { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX supports C++11 features with $switch" >&5  
$as_echo_n "checking whether $CXX supports C++11 features with $switch... " >&6; }  
if eval \${$cachevar+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_save_CXX="$CXX"  
         CXX="$CXX $switch"  
         cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
  
// If the compiler admits that it is not ready for C++11, why torture it?  
// Hopefully, this will speed up the test.  
  
#ifndef __cplusplus  
  
#error "This is not a C++ compiler"  
  
#elif __cplusplus < 201103L  
  
#error "This is not a C++11 compiler"  
  
#else  
  
namespace cxx11  
{  
  
  namespace test_static_assert  
  {  
  
    template <typename T>  
    struct check  
    {  
      static_assert(sizeof(int) <= sizeof(T), "not big enough");  
    };  
  
  }  
  
  namespace test_final_override  
  {  
  
    struct Base  
    {  
      virtual ~Base() {}  
      virtual void f() {}  
    };  
  
    struct Derived : public Base  
    {  
      virtual ~Derived() override {}  
      virtual void f() override {}  
    };  
  
  }  
  
  namespace test_double_right_angle_brackets  
  {  
  
    template < typename T >  
    struct check {};  
  
    typedef check<void> single_type;  
    typedef check<check<void>> double_type;  
    typedef check<check<check<void>>> triple_type;  
    typedef check<check<check<check<void>>>> quadruple_type;  
  
  }  
  
  namespace test_decltype  
  {  
  
    int  
    f()  
    {  
      int a = 1;  
      decltype(a) b = 2;  
      return a + b;  
    }  
  
  }  
  
  namespace test_type_deduction  
  {  
  
    template < typename T1, typename T2 >  
    struct is_same  
    {  
      static const bool value = false;  
    };  
  
    template < typename T >  
    struct is_same<T, T>  
    {  
      static const bool value = true;  
    };  
  
    template < typename T1, typename T2 >  
    auto  
    add(T1 a1, T2 a2) -> decltype(a1 + a2)  
    {  
      return a1 + a2;  
    }  
  
    int  
    test(const int c, volatile int v)  
    {  
      static_assert(is_same<int, decltype(0)>::value == true, "");  
      static_assert(is_same<int, decltype(c)>::value == false, "");  
      static_assert(is_same<int, decltype(v)>::value == false, "");  
      auto ac = c;  
      auto av = v;  
      auto sumi = ac + av + 'x';  
      auto sumf = ac + av + 1.0;  
      static_assert(is_same<int, decltype(ac)>::value == true, "");  
      static_assert(is_same<int, decltype(av)>::value == true, "");  
      static_assert(is_same<int, decltype(sumi)>::value == true, "");  
      static_assert(is_same<int, decltype(sumf)>::value == false, "");  
      static_assert(is_same<int, decltype(add(c, v))>::value == true, "");  
      return (sumf > 0.0) ? sumi : add(c, v);  
    }  
  
  }  
  
  namespace test_noexcept  
  {  
  
    int f() { return 0; }  
    int g() noexcept { return 0; }  
  
    static_assert(noexcept(f()) == false, "");  
    static_assert(noexcept(g()) == true, "");  
  
  }  
  
  namespace test_constexpr  
  {  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c_r(const CharT *const s, const unsigned long acc) noexcept  
    {  
      return *s ? strlen_c_r(s + 1, acc + 1) : acc;  
    }  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c(const CharT *const s) noexcept  
    {  
      return strlen_c_r(s, 0UL);  
    }  
  
    static_assert(strlen_c("") == 0UL, "");  
    static_assert(strlen_c("1") == 1UL, "");  
    static_assert(strlen_c("example") == 7UL, "");  
    static_assert(strlen_c("another\0example") == 7UL, "");  
  
  }  
  
  namespace test_rvalue_references  
  {  
  
    template < int N >  
    struct answer  
    {  
      static constexpr int value = N;  
    };  
  
    answer<1> f(int&)       { return answer<1>(); }  
    answer<2> f(const int&) { return answer<2>(); }  
    answer<3> f(int&&)      { return answer<3>(); }  
  
    void  
    test()  
    {  
      int i = 0;  
      const int c = 0;  
      static_assert(decltype(f(i))::value == 1, "");  
      static_assert(decltype(f(c))::value == 2, "");  
      static_assert(decltype(f(0))::value == 3, "");  
    }  
  
  }  
  
  namespace test_uniform_initialization  
  {  
  
    struct test  
    {  
      static const int zero {};  
      static const int one {1};  
    };  
  
    static_assert(test::zero == 0, "");  
    static_assert(test::one == 1, "");  
  
  }  
  
  namespace test_lambdas  
  {  
  
    void  
    test1()  
    {  
      auto lambda1 = [](){};  
      auto lambda2 = lambda1;  
      lambda1();  
      lambda2();  
    }  
  
    int  
    test2()  
    {  
      auto a = [](int i, int j){ return i + j; }(1, 2);  
      auto b = []() -> int { return '0'; }();  
      auto c = [=](){ return a + b; }();  
      auto d = [&](){ return c; }();  
      auto e = [a, &b](int x) mutable {  
        const auto identity = [](int y){ return y; };  
        for (auto i = 0; i < a; ++i)  
          a += b--;  
        return x + identity(a + b);  
      }(0);  
      return a + b + c + d + e;  
    }  
  
    int  
    test3()  
    {  
      const auto nullary = [](){ return 0; };  
      const auto unary = [](int x){ return x; };  
      using nullary_t = decltype(nullary);  
      using unary_t = decltype(unary);  
      const auto higher1st = [](nullary_t f){ return f(); };  
      const auto higher2nd = [unary](nullary_t f1){  
        return [unary, f1](unary_t f2){ return f2(unary(f1())); };  
      };  
      return higher1st(nullary) + higher2nd(nullary)(unary);  
    }  
  
  }  
  
  namespace test_variadic_templates  
  {  
  
    template <int...>  
    struct sum;  
  
    template <int N0, int... N1toN>  
    struct sum<N0, N1toN...>  
    {  
      static constexpr auto value = N0 + sum<N1toN...>::value;  
    };  
  
    template <>  
    struct sum<>  
    {  
      static constexpr auto value = 0;  
    };  
  
    static_assert(sum<>::value == 0, "");  
    static_assert(sum<1>::value == 1, "");  
    static_assert(sum<23>::value == 23, "");  
    static_assert(sum<1, 2>::value == 3, "");  
    static_assert(sum<5, 5, 11>::value == 21, "");  
    static_assert(sum<2, 3, 5, 7, 11, 13>::value == 41, "");  
  
  }  
  
  // http://stackoverflow.com/questions/13728184/template-aliases-and-sfinae  
  // Clang 3.1 fails with headers of libstd++ 4.8.3 when using std::function  
  // because of this.  
  namespace test_template_alias_sfinae  
  {  
  
    struct foo {};  
  
    template<typename T>  
    using member = typename T::member_type;  
  
    template<typename T>  
    void func(...) {}  
  
    template<typename T>  
    void func(member<T>*) {}  
  
    void test();  
  
    void test() { func<foo>(0); }  
  
  }  
  
}  // namespace cxx11  
  
#endif  // __cplusplus >= 201103L  
  
  
  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  eval $cachevar=yes  
else  
  eval $cachevar=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
         CXX="$ac_save_CXX"  
fi  
eval ac_res=\$$cachevar  
           { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_res" >&5  
$as_echo "$ac_res" >&6; }  
      if eval test x\$$cachevar = xyes; then  
        CXX="$CXX $switch"  
        if test -n "$CXXCPP" ; then  
          CXXCPP="$CXXCPP $switch"  
        fi  
        ac_success=yes  
        break  
      fi  
    done  
  fi  
  
    if test x$ac_success = xno; then  
                for alternative in ${ax_cxx_compile_alternatives}; do  
      for switch in -std=c++${alternative} +std=c++${alternative} "-h std=c++${alternative}"; do  
        cachevar=`$as_echo "ax_cv_cxx_compile_cxx11_FOR_BUILD_$switch" | $as_tr_sh`  
        { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether $CXX supports C++11 features with $switch" >&5  
$as_echo_n "checking whether $CXX supports C++11 features with $switch... " >&6; }  
if eval \${$cachevar+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  ac_save_CXX="$CXX"  
           CXX="$CXX $switch"  
           cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
  
  
// If the compiler admits that it is not ready for C++11, why torture it?  
// Hopefully, this will speed up the test.  
  
#ifndef __cplusplus  
  
#error "This is not a C++ compiler"  
  
#elif __cplusplus < 201103L  
  
#error "This is not a C++11 compiler"  
  
#else  
  
namespace cxx11  
{  
  
  namespace test_static_assert  
  {  
  
    template <typename T>  
    struct check  
    {  
      static_assert(sizeof(int) <= sizeof(T), "not big enough");  
    };  
  
  }  
  
  namespace test_final_override  
  {  
  
    struct Base  
    {  
      virtual ~Base() {}  
      virtual void f() {}  
    };  
  
    struct Derived : public Base  
    {  
      virtual ~Derived() override {}  
      virtual void f() override {}  
    };  
  
  }  
  
  namespace test_double_right_angle_brackets  
  {  
  
    template < typename T >  
    struct check {};  
  
    typedef check<void> single_type;  
    typedef check<check<void>> double_type;  
    typedef check<check<check<void>>> triple_type;  
    typedef check<check<check<check<void>>>> quadruple_type;  
  
  }  
  
  namespace test_decltype  
  {  
  
    int  
    f()  
    {  
      int a = 1;  
      decltype(a) b = 2;  
      return a + b;  
    }  
  
  }  
  
  namespace test_type_deduction  
  {  
  
    template < typename T1, typename T2 >  
    struct is_same  
    {  
      static const bool value = false;  
    };  
  
    template < typename T >  
    struct is_same<T, T>  
    {  
      static const bool value = true;  
    };  
  
    template < typename T1, typename T2 >  
    auto  
    add(T1 a1, T2 a2) -> decltype(a1 + a2)  
    {  
      return a1 + a2;  
    }  
  
    int  
    test(const int c, volatile int v)  
    {  
      static_assert(is_same<int, decltype(0)>::value == true, "");  
      static_assert(is_same<int, decltype(c)>::value == false, "");  
      static_assert(is_same<int, decltype(v)>::value == false, "");  
      auto ac = c;  
      auto av = v;  
      auto sumi = ac + av + 'x';  
      auto sumf = ac + av + 1.0;  
      static_assert(is_same<int, decltype(ac)>::value == true, "");  
      static_assert(is_same<int, decltype(av)>::value == true, "");  
      static_assert(is_same<int, decltype(sumi)>::value == true, "");  
      static_assert(is_same<int, decltype(sumf)>::value == false, "");  
      static_assert(is_same<int, decltype(add(c, v))>::value == true, "");  
      return (sumf > 0.0) ? sumi : add(c, v);  
    }  
  
  }  
  
  namespace test_noexcept  
  {  
  
    int f() { return 0; }  
    int g() noexcept { return 0; }  
  
    static_assert(noexcept(f()) == false, "");  
    static_assert(noexcept(g()) == true, "");  
  
  }  
  
  namespace test_constexpr  
  {  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c_r(const CharT *const s, const unsigned long acc) noexcept  
    {  
      return *s ? strlen_c_r(s + 1, acc + 1) : acc;  
    }  
  
    template < typename CharT >  
    unsigned long constexpr  
    strlen_c(const CharT *const s) noexcept  
    {  
      return strlen_c_r(s, 0UL);  
    }  
  
    static_assert(strlen_c("") == 0UL, "");  
    static_assert(strlen_c("1") == 1UL, "");  
    static_assert(strlen_c("example") == 7UL, "");  
    static_assert(strlen_c("another\0example") == 7UL, "");  
  
  }  
  
  namespace test_rvalue_references  
  {  
  
    template < int N >  
    struct answer  
    {  
      static constexpr int value = N;  
    };  
  
    answer<1> f(int&)       { return answer<1>(); }  
    answer<2> f(const int&) { return answer<2>(); }  
    answer<3> f(int&&)      { return answer<3>(); }  
  
    void  
    test()  
    {  
      int i = 0;  
      const int c = 0;  
      static_assert(decltype(f(i))::value == 1, "");  
      static_assert(decltype(f(c))::value == 2, "");  
      static_assert(decltype(f(0))::value == 3, "");  
    }  
  
  }  
  
  namespace test_uniform_initialization  
  {  
  
    struct test  
    {  
      static const int zero {};  
      static const int one {1};  
    };  
  
    static_assert(test::zero == 0, "");  
    static_assert(test::one == 1, "");  
  
  }  
  
  namespace test_lambdas  
  {  
  
    void  
    test1()  
    {  
      auto lambda1 = [](){};  
      auto lambda2 = lambda1;  
      lambda1();  
      lambda2();  
    }  
  
    int  
    test2()  
    {  
      auto a = [](int i, int j){ return i + j; }(1, 2);  
      auto b = []() -> int { return '0'; }();  
      auto c = [=](){ return a + b; }();  
      auto d = [&](){ return c; }();  
      auto e = [a, &b](int x) mutable {  
        const auto identity = [](int y){ return y; };  
        for (auto i = 0; i < a; ++i)  
          a += b--;  
        return x + identity(a + b);  
      }(0);  
      return a + b + c + d + e;  
    }  
  
    int  
    test3()  
    {  
      const auto nullary = [](){ return 0; };  
      const auto unary = [](int x){ return x; };  
      using nullary_t = decltype(nullary);  
      using unary_t = decltype(unary);  
      const auto higher1st = [](nullary_t f){ return f(); };  
      const auto higher2nd = [unary](nullary_t f1){  
        return [unary, f1](unary_t f2){ return f2(unary(f1())); };  
      };  
      return higher1st(nullary) + higher2nd(nullary)(unary);  
    }  
  
  }  
  
  namespace test_variadic_templates  
  {  
  
    template <int...>  
    struct sum;  
  
    template <int N0, int... N1toN>  
    struct sum<N0, N1toN...>  
    {  
      static constexpr auto value = N0 + sum<N1toN...>::value;  
    };  
  
    template <>  
    struct sum<>  
    {  
      static constexpr auto value = 0;  
    };  
  
    static_assert(sum<>::value == 0, "");  
    static_assert(sum<1>::value == 1, "");  
    static_assert(sum<23>::value == 23, "");  
    static_assert(sum<1, 2>::value == 3, "");  
    static_assert(sum<5, 5, 11>::value == 21, "");  
    static_assert(sum<2, 3, 5, 7, 11, 13>::value == 41, "");  
  
  }  
  
  // http://stackoverflow.com/questions/13728184/template-aliases-and-sfinae  
  // Clang 3.1 fails with headers of libstd++ 4.8.3 when using std::function  
  // because of this.  
  namespace test_template_alias_sfinae  
  {  
  
    struct foo {};  
  
    template<typename T>  
    using member = typename T::member_type;  
  
    template<typename T>  
    void func(...) {}  
  
    template<typename T>  
    void func(member<T>*) {}  
  
    void test();  
  
    void test() { func<foo>(0); }  
  
  }  
  
}  // namespace cxx11  
  
#endif  // __cplusplus >= 201103L  
  
  
  
_ACEOF  
if ac_fn_cxx_try_compile "$LINENO"; then :  
  eval $cachevar=yes  
else  
  eval $cachevar=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
           CXX="$ac_save_CXX"  
fi  
eval ac_res=\$$cachevar  
           { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_res" >&5  
$as_echo "$ac_res" >&6; }  
        if eval test x\$$cachevar = xyes; then  
          CXX="$CXX $switch"  
          if test -n "$CXXCPP" ; then  
            CXXCPP="$CXXCPP $switch"  
          fi  
          ac_success=yes  
          break  
        fi  
      done  
      if test x$ac_success = xyes; then  
        break  
      fi  
    done  
  fi  
      CXX_FOR_BUILD="$CXX"  
    CXXFLAGS_FOR_BUILD="$CXXFLAGS"  
    CPPFLAGS_FOR_BUILD="$CPPFLAGS"  
    CXX="$ax_cv_cxx_compile_cxx11_orig_cxx"  
    CXXFLAGS="$ax_cv_cxx_compile_cxx11_orig_cxxflags"  
    CPPFLAGS="$ax_cv_cxx_compile_cxx11_orig_cppflags"  
  ac_ext=c  
ac_cpp='$CPP $CPPFLAGS'  
ac_compile='$CC -c $CFLAGS $CPPFLAGS conftest.$ac_ext >&5'  
ac_link='$CC -o conftest$ac_exeext $CFLAGS $CPPFLAGS $LDFLAGS conftest.$ac_ext $LIBS >&5'  
ac_compiler_gnu=$ac_cv_c_compiler_gnu  
  
  if test x$ax_cxx_compile_cxx11_required = xtrue; then  
    if test x$ac_success = xno; then  
      as_fn_error $? "*** A compiler with support for C++11 language features is required." "$LINENO" 5  
    fi  
  fi  
  if test x$ac_success = xno; then  
    HAVE_CXX11_FOR_BUILD=0  
    { $as_echo "$as_me:${as_lineno-$LINENO}: No compiler with C++11 support was found" >&5  
$as_echo "$as_me: No compiler with C++11 support was found" >&6;}  
  else  
    HAVE_CXX11_FOR_BUILD=1  
  
$as_echo "#define HAVE_CXX11_FOR_BUILD 1" >>confdefs.h  
  
  fi  
  
  
  fi  
fi  
  
# Check whether --enable-pgo-build was given.  
if test "${enable_pgo_build+set}" = set; then :  
  enableval=$enable_pgo_build; enable_pgo_build=$enableval  
else  
  enable_pgo_build=no  
fi  
  
  
# Issue errors and warnings for invalid/strange PGO build combinations.  
case "$have_compiler:$host:$target:$enable_pgo_build" in  
  *:*:*:no) ;;  
  
  # Allow the PGO build only if we aren't building a compiler and  
  # we are in a native configuration.  
  no:$build:$build:yes | no:$build:$build:lto) ;;  
  
  # Disallow the PGO bootstrap if we are building a compiler.  
  yes:*:*:yes | yes:*:*:lto)  
    as_fn_error $? "cannot perform the PGO bootstrap when building a compiler" "$LINENO" 5 ;;  
  
  *)  
    as_fn_error $? "invalid option for --enable-pgo-build" "$LINENO" 5  
    ;;  
esac  
  
if test "$enable_pgo_build" != "no"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether the compiler supports -fprofile-generate" >&5  
$as_echo_n "checking whether the compiler supports -fprofile-generate... " >&6; }  
  old_CFLAGS="$CFLAGS"  
  PGO_BUILD_GEN_CFLAGS="-fprofile-generate"  
  CFLAGS="$CFLAGS $PGO_BUILD_CFLAGS"  
  
cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
int foo;  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  
else  
  PGO_BUILD_GEN_CFLAGS=  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
  CFLAGS="$old_CFLAGS"  
  if test -n "$PGO_BUILD_GEN_CFLAGS"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    PGO_BUILD_USE_CFLAGS="-fprofile-use"  
  else  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    as_fn_error $? "cannot perform the PGO build without -fprofile-generate" "$LINENO" 5  
  fi  
  
  if test "$enable_pgo_build" = "lto"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking whether the compiler supports -flto=jobserver -ffat-lto-objects" >&5  
$as_echo_n "checking whether the compiler supports -flto=jobserver -ffat-lto-objects... " >&6; }  
    old_CFLAGS="$CFLAGS"  
    PGO_BUILD_LTO_CFLAGS="-flto=jobserver -ffat-lto-objects"  
    CFLAGS="$CFLAGS $PGO_BUILD_LTO_CFLAGS"  
    cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
int foo;  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  
else  
  PGO_BUILD_LTO_CFLAGS=  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
    CFLAGS="$old_CFLAGS"  
    if test -n "$PGO_BUILD_LTO_CFLAGS"; then  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
      { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: LTO is disabled for the PGO build" >&5  
$as_echo "$as_me: WARNING: LTO is disabled for the PGO build" >&2;}  
    fi  
  fi  
fi  
  
  
  
  
# Used for setting $lt_cv_objdir  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for objdir" >&5  
$as_echo_n "checking for objdir... " >&6; }  
if ${lt_cv_objdir+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  rm -f .libs 2>/dev/null  
mkdir .libs 2>/dev/null  
if test -d .libs; then  
  lt_cv_objdir=.libs  
else  
  # MS-DOS does not allow filenames that begin with a dot.  
  lt_cv_objdir=_libs  
fi  
rmdir .libs 2>/dev/null  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $lt_cv_objdir" >&5  
$as_echo "$lt_cv_objdir" >&6; }  
objdir=$lt_cv_objdir  
  
  
  
  
  
cat >>confdefs.h \<<_ACEOF  
#define LT_OBJDIR "$lt_cv_objdir/"  
_ACEOF  
  
  
  
# Check for GMP, MPFR and MPC  
require_gmp=no  
require_mpc=no  
if test -d ${srcdir}/gcc ; then  
  require_gmp=yes  
  require_mpc=yes  
fi  
if test -d ${srcdir}/gdb ; then  
  if test "x$enable_gdb" != xno; then  
   require_gmp=yes  
  fi  
fi  
  
gmplibs="-lmpfr -lgmp"  
if test x"$require_mpc" = "xyes" ; then  
  gmplibs="-lmpc $gmplibs"  
fi  
gmpinc=  
have_gmp=no  
  
# Specify a location for mpc  
# check for this first so it ends up on the link line before mpfr.  
  
# Check whether --with-mpc was given.  
if test "${with_mpc+set}" = set; then :  
  withval=$with_mpc;  
fi  
  
  
# Check whether --with-mpc-include was given.  
if test "${with_mpc_include+set}" = set; then :  
  withval=$with_mpc_include;  
fi  
  
  
# Check whether --with-mpc-lib was given.  
if test "${with_mpc_lib+set}" = set; then :  
  withval=$with_mpc_lib;  
fi  
  
  
if test "x$with_mpc" != x; then  
  gmplibs="-L$with_mpc/lib $gmplibs"  
  gmpinc="-I$with_mpc/include $gmpinc"  
fi  
if test "x$with_mpc_include" != x; then  
  gmpinc="-I$with_mpc_include $gmpinc"  
fi  
if test "x$with_mpc_lib" != x; then  
  gmplibs="-L$with_mpc_lib $gmplibs"  
fi  
if test "x$with_mpc$with_mpc_include$with_mpc_lib" = x && test -d ${srcdir}/mpc; then  
  gmplibs='-L$$r/$(HOST_SUBDIR)/mpc/src/'"$lt_cv_objdir $gmplibs"  
  gmpinc='-I$$s/mpc/src '"$gmpinc"  
  # Do not test the mpc version.  Assume that it is sufficient, since  
  # it is in the source tree, and the library has not been built yet  
  # but it would be included on the link line in the version check below  
  # hence making the test fail.  
  have_gmp=yes  
fi  
  
# Specify a location for mpfr  
# check for this first so it ends up on the link line before gmp.  
  
# Check whether --with-mpfr was given.  
if test "${with_mpfr+set}" = set; then :  
  withval=$with_mpfr;  
fi  
  
  
# Check whether --with-mpfr-include was given.  
if test "${with_mpfr_include+set}" = set; then :  
  withval=$with_mpfr_include;  
fi  
  
  
# Check whether --with-mpfr-lib was given.  
if test "${with_mpfr_lib+set}" = set; then :  
  withval=$with_mpfr_lib;  
fi  
  
  
if test "x$with_mpfr" != x; then  
  gmplibs="-L$with_mpfr/lib $gmplibs"  
  gmpinc="-I$with_mpfr/include $gmpinc"  
fi  
if test "x$with_mpfr_include" != x; then  
  gmpinc="-I$with_mpfr_include $gmpinc"  
fi  
if test "x$with_mpfr_lib" != x; then  
  gmplibs="-L$with_mpfr_lib $gmplibs"  
fi  
if test "x$with_mpfr$with_mpfr_include$with_mpfr_lib" = x && test -d ${srcdir}/mpfr; then  
  # MPFR v3.1.0 moved the sources into a src sub-directory.  
  if ! test -d ${srcdir}/mpfr/src; then  
    as_fn_error $? "Building GCC with MPFR in the source tree is only handled for MPFR 3.1.0+." "$LINENO" 5  
  fi  
  gmplibs='-L$$r/$(HOST_SUBDIR)/mpfr/src/'"$lt_cv_objdir $gmplibs"  
  gmpinc='-I$$r/$(HOST_SUBDIR)/mpfr/src -I$$s/mpfr/src '"$gmpinc"  
  extra_mpc_mpfr_configure_flags='--with-mpfr-include=$$s/mpfr/src --with-mpfr-lib=$$r/$(HOST_SUBDIR)/mpfr/src/'"$lt_cv_objdir"  
  # Do not test the mpfr version.  Assume that it is sufficient, since  
  # it is in the source tree, and the library has not been built yet  
  # but it would be included on the link line in the version check below  
  # hence making the test fail.  
  have_gmp=yes  
fi  
  
# Specify a location for gmp  
  
# Check whether --with-gmp was given.  
if test "${with_gmp+set}" = set; then :  
  withval=$with_gmp;  
fi  
  
  
# Check whether --with-gmp-include was given.  
if test "${with_gmp_include+set}" = set; then :  
  withval=$with_gmp_include;  
fi  
  
  
# Check whether --with-gmp-lib was given.  
if test "${with_gmp_lib+set}" = set; then :  
  withval=$with_gmp_lib;  
fi  
  
  
  
if test "x$with_gmp" != x; then  
  gmplibs="-L$with_gmp/lib $gmplibs"  
  gmpinc="-I$with_gmp/include $gmpinc"  
fi  
if test "x$with_gmp_include" != x; then  
  gmpinc="-I$with_gmp_include $gmpinc"  
fi  
if test "x$with_gmp_lib" != x; then  
  gmplibs="-L$with_gmp_lib $gmplibs"  
fi  
if test "x$with_gmp$with_gmp_include$with_gmp_lib" = x && test -d ${srcdir}/gmp; then  
  gmplibs='-L$$r/$(HOST_SUBDIR)/gmp/'"$lt_cv_objdir $gmplibs"  
  gmpinc='-I$$r/$(HOST_SUBDIR)/gmp -I$$s/gmp '"$gmpinc"  
  extra_mpfr_configure_flags='--with-gmp-include=$$r/$(HOST_SUBDIR)/gmp --with-gmp-lib=$$r/$(HOST_SUBDIR)/gmp/'"$lt_cv_objdir"  
  extra_mpc_gmp_configure_flags='--with-gmp-include=$$r/$(HOST_SUBDIR)/gmp --with-gmp-lib=$$r/$(HOST_SUBDIR)/gmp/'"$lt_cv_objdir"  
  extra_isl_gmp_configure_flags='--with-gmp-builddir=$$r/$(HOST_SUBDIR)/gmp'  
  # Do not test the gmp version.  Assume that it is sufficient, since  
  # it is in the source tree, and the library has not been built yet  
  # but it would be included on the link line in the version check below  
  # hence making the test fail.  
  have_gmp=yes  
fi  
  
if test "x$require_gmp" = xyes && test "x$have_gmp" = xno; then  
  have_gmp=yes  
  saved_CFLAGS="$CFLAGS"  
  CFLAGS="$CFLAGS $gmpinc"  
  # Check for the recommended and required versions of GMP.  
  { $as_echo "$as_me:${as_lineno-$LINENO}: checking for the correct version of gmp.h" >&5  
$as_echo_n "checking for the correct version of gmp.h... " >&6; }  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include "gmp.h"  
int  
main ()  
{  
  
  #define GCC_GMP_VERSION_NUM(a,b,c) (((a) \<< 16L) | ((b) \<< 8) | (c))  
  #define GCC_GMP_VERSION GCC_GMP_VERSION_NUM(__GNU_MP_VERSION,__GNU_MP_VERSION_MINOR,__GNU_MP_VERSION_PATCHLEVEL)  
  #if GCC_GMP_VERSION < GCC_GMP_VERSION_NUM(4,2,3)  
  choke me  
  #endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <gmp.h>  
int  
main ()  
{  
  
  #define GCC_GMP_VERSION_NUM(a,b,c) (((a) \<< 16L) | ((b) \<< 8) | (c))  
  #define GCC_GMP_VERSION GCC_GMP_VERSION_NUM(__GNU_MP_VERSION,__GNU_MP_VERSION_MINOR,__GNU_MP_VERSION_PATCHLEVEL)  
  #if GCC_GMP_VERSION < GCC_GMP_VERSION_NUM(4,3,2)  
  choke me  
  #endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: buggy but acceptable" >&5  
$as_echo "buggy but acceptable" >&6; }  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }; have_gmp=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
  
  # If we have GMP, check the MPFR version.  
  if test x"$have_gmp" = xyes; then  
    # Check for the recommended and required versions of MPFR.  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for the correct version of mpfr.h" >&5  
$as_echo_n "checking for the correct version of mpfr.h... " >&6; }  
    cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <gmp.h>  
    #include <mpfr.h>  
int  
main ()  
{  
  
    #if MPFR_VERSION < MPFR_VERSION_NUM(3,1,0)  
    choke me  
    #endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <gmp.h>  
    #include <mpfr.h>  
int  
main ()  
{  
  
    #if MPFR_VERSION < MPFR_VERSION_NUM(3,1,6)  
    choke me  
    #endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: buggy but acceptable" >&5  
$as_echo "buggy but acceptable" >&6; }  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }; have_gmp=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
  fi  
  
  # Check for the MPC header version.  
  if test "x$require_mpc" = xyes && test x"$have_gmp" = xyes ; then  
    # Check for the recommended and required versions of MPC.  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for the correct version of mpc.h" >&5  
$as_echo_n "checking for the correct version of mpc.h... " >&6; }  
    cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <mpc.h>  
int  
main ()  
{  
  
    #if MPC_VERSION < MPC_VERSION_NUM(0,8,0)  
    choke me  
    #endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <mpc.h>  
int  
main ()  
{  
  
    #if MPC_VERSION < MPC_VERSION_NUM(0,8,1)  
    choke me  
    #endif  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_compile "$LINENO"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: buggy but acceptable" >&5  
$as_echo "buggy but acceptable" >&6; }  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }; have_gmp=no  
fi  
rm -f core conftest.err conftest.$ac_objext conftest.$ac_ext  
  fi  
  
  # Now check the MPFR library.  
  if test x"$have_gmp" = xyes; then  
    saved_LIBS="$LIBS"  
    LIBS="$LIBS $gmplibs"  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for the correct version of the gmp/mpfr libraries" >&5  
$as_echo_n "checking for the correct version of the gmp/mpfr libraries... " >&6; }  
    cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <mpfr.h>  
int  
main ()  
{  
  
    mpfr_t n;  
    mpfr_t x;  
    int t;  
    mpfr_init (n);  
    mpfr_init (x);  
    mpfr_atan2 (n, n, x, MPFR_RNDN);  
    mpfr_erfc (n, x, MPFR_RNDN);  
    mpfr_subnormalize (x, t, MPFR_RNDN);  
    mpfr_clear(n);  
    mpfr_clear(x);  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_link "$LINENO"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }; have_gmp=no  
fi  
rm -f core conftest.err conftest.$ac_objext \  
    conftest$ac_exeext conftest.$ac_ext  
    LIBS="$saved_LIBS"  
  fi  
  
  # Now check the MPC library  
  if test "x$require_mpc" = xyes && test x"$have_gmp" = xyes; then  
    saved_LIBS="$LIBS"  
    LIBS="$LIBS $gmplibs"  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for the correct version of the mpc libraries" >&5  
$as_echo_n "checking for the correct version of the mpc libraries... " >&6; }  
    cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <mpc.h>  
int  
main ()  
{  
  
    mpc_t c;  
    mpc_init2 (c, 53);  
    mpc_set_ui_ui (c, 1, 1, MPC_RNDNN);  
    mpc_cosh (c, c, MPC_RNDNN);  
    mpc_pow (c, c, c, MPC_RNDNN);  
    mpc_acosh (c, c, MPC_RNDNN);  
    mpc_clear (c);  
  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_link "$LINENO"; then :  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }; have_gmp=no  
fi  
rm -f core conftest.err conftest.$ac_objext \  
    conftest$ac_exeext conftest.$ac_ext  
    LIBS="$saved_LIBS"  
  fi  
  
  CFLAGS="$saved_CFLAGS"  
  
# The library versions listed in the error message below should match  
# the HARD-minimums enforced above.  
  if test x$have_gmp != xyes; then  
    if test -d ${srcdir}/gcc ; then  
      as_fn_error $? "Building GCC requires GMP 4.2+, MPFR 3.1.0+ and MPC 0.8.0+.  
Try the --with-gmp, --with-mpfr and/or --with-mpc options to specify  
their locations.  Source code for these libraries can be found at  
their respective hosting sites as well as at  
https://gcc.gnu.org/pub/gcc/infrastructure/.  See also  
http://gcc.gnu.org/install/prerequisites.html for additional info.  If  
you obtained GMP, MPFR and/or MPC from a vendor distribution package,  
make sure that you have installed both the libraries and the header  
files.  They may be located in separate packages." "$LINENO" 5  
    else  
      as_fn_error $? "Building GDB requires GMP 4.2+, and MPFR 3.1.0+.  
Try the --with-gmp and/or --with-mpfr options to specify  
their locations.  If you obtained GMP and/or MPFR from a vendor  
distribution package, make sure that you have installed both the libraries  
and the header files.  They may be located in separate packages." "$LINENO" 5  
    fi  
  fi  
fi  
  
# Flags needed for both GMP, MPFR and/or MPC.  
  
  
  
  
  
  
  
# Libraries to use for stage1 or when not bootstrapping.  
  
# Check whether --with-stage1-libs was given.  
if test "${with_stage1_libs+set}" = set; then :  
  withval=$with_stage1_libs; if test "$withval" = "no" -o "$withval" = "yes"; then  
   stage1_libs=  
 else  
   stage1_libs=$withval  
 fi  
else  
  stage1_libs=  
fi  
  
  
  
# Whether or not to use -static-libstdc++ and -static-libgcc.  The  
# default is yes if gcc is being built; no otherwise.  The reason for  
# this default is that gdb is sometimes linked against GNU Source  
# Highlight, which is a shared library that uses C++ exceptions.  In  
# this case, -static-libstdc++ will cause crashes.  
  
# Check whether --with-static-standard-libraries was given.  
if test "${with_static_standard_libraries+set}" = set; then :  
  withval=$with_static_standard_libraries;  
else  
  with_static_standard_libraries=auto  
fi  
  
if test "$with_static_standard_libraries" = auto; then  
  with_static_standard_libraries=$have_compiler  
fi  
  
# Linker flags to use for stage1 or when not bootstrapping.  
  
# Check whether --with-stage1-ldflags was given.  
if test "${with_stage1_ldflags+set}" = set; then :  
  withval=$with_stage1_ldflags; if test "$withval" = "no" -o "$withval" = "yes"; then  
   stage1_ldflags=  
 else  
   stage1_ldflags=$withval  
 fi  
else  
  stage1_ldflags=  
 # In stage 1, default to linking libstdc++ and libgcc statically with GCC  
 # if supported.  But if the user explicitly specified the libraries to use,  
 # trust that they are doing what they want.  
 if test "$with_static_standard_libraries" = yes -a "$stage1_libs" = "" \  
     -a "$have_static_libs" = yes; then  
   stage1_ldflags="-static-libstdc++ -static-libgcc"  
 fi  
fi  
  
  
  
# Libraries to use for stage2 and later builds.  
  
# Check whether --with-boot-libs was given.  
if test "${with_boot_libs+set}" = set; then :  
  withval=$with_boot_libs; if test "$withval" = "no" -o "$withval" = "yes"; then  
   poststage1_libs=  
 else  
   poststage1_libs=$withval  
 fi  
else  
  poststage1_libs=  
fi  
  
  
  
# Linker flags to use for stage2 and later builds.  
  
# Check whether --with-boot-ldflags was given.  
if test "${with_boot_ldflags+set}" = set; then :  
  withval=$with_boot_ldflags; if test "$withval" = "no" -o "$withval" = "yes"; then  
   poststage1_ldflags=  
 else  
   poststage1_ldflags=$withval  
 fi  
else  
  poststage1_ldflags=  
 # In stages 2 and 3, default to linking libstdc++ and libgcc  
 # statically.  But if the user explicitly specified the libraries to  
 # use, trust that they are doing what they want.  
 if test "$poststage1_libs" = ""; then  
   poststage1_ldflags="-static-libstdc++ -static-libgcc"  
 fi  
fi  
  
case $target in  
  *-darwin2* | *-darwin1[56789]*)  
    # For these versions, we default to using embedded rpaths.  
    if test "x$enable_darwin_at_rpath" != "xno"; then  
      poststage1_ldflags="$poststage1_ldflags -nodefaultrpaths"  
    fi  
  ;;  
  *-darwin*)  
    # For these versions, we only use embedded rpaths on demand.  
    if test "x$enable_darwin_at_rpath" = "xyes"; then  
      poststage1_ldflags="$poststage1_ldflags -nodefaultrpaths"  
    fi  
  ;;  
esac  
  
  
# GCC GRAPHITE dependency isl.  
# Basic setup is inlined here, actual checks are in config/isl.m4  
  
  
# Check whether --with-isl was given.  
if test "${with_isl+set}" = set; then :  
  withval=$with_isl;  
fi  
  
  
# Treat --without-isl as a request to disable  
# GRAPHITE support and skip all following checks.  
if test "x$with_isl" != "xno"; then  
  # Check for isl  
  
  
# Check whether --with-isl-include was given.  
if test "${with_isl_include+set}" = set; then :  
  withval=$with_isl_include;  
fi  
  
  
# Check whether --with-isl-lib was given.  
if test "${with_isl_lib+set}" = set; then :  
  withval=$with_isl_lib;  
fi  
  
  
  # Check whether --enable-isl-version-check was given.  
if test "${enable_isl_version_check+set}" = set; then :  
  enableval=$enable_isl_version_check; ENABLE_ISL_CHECK=$enableval  
else  
  ENABLE_ISL_CHECK=yes  
fi  
  
  
  # Initialize isllibs and islinc.  
  case $with_isl in  
    no)  
      isllibs=  
      islinc=  
      ;;  
    "" | yes)  
      ;;  
    *)  
      isllibs="-L$with_isl/lib"  
      islinc="-I$with_isl/include"  
      ;;  
  esac  
  if test "x${with_isl_include}" != x ; then  
    islinc="-I$with_isl_include"  
  fi  
  if test "x${with_isl_lib}" != x; then  
    isllibs="-L$with_isl_lib"  
  fi  
        if test "x${islinc}" = x && test "x${isllibs}" = x \  
     && test -d ${srcdir}/isl; then  
    isllibs='-L$$r/$(HOST_SUBDIR)/isl/'"$lt_cv_objdir"' '  
    islinc='-I$$r/$(HOST_SUBDIR)/isl/include -I$$s/isl/include'  
    ENABLE_ISL_CHECK=no  
    { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using in-tree isl, disabling version check" >&5  
$as_echo "$as_me: WARNING: using in-tree isl, disabling version check" >&2;}  
  fi  
  
  isllibs="${isllibs} -lisl"  
  
  
  
  if test "${ENABLE_ISL_CHECK}" = yes ; then  
    _isl_saved_CFLAGS=$CFLAGS  
    _isl_saved_LDFLAGS=$LDFLAGS  
    _isl_saved_LIBS=$LIBS  
  
    CFLAGS="${_isl_saved_CFLAGS} ${islinc} ${gmpinc}"  
    LDFLAGS="${_isl_saved_LDFLAGS} ${isllibs} ${gmplibs}"  
    LIBS="${_isl_saved_LIBS} -lisl -lgmp"  
  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for isl 0.15 or later" >&5  
$as_echo_n "checking for isl 0.15 or later... " >&6; }  
    cat confdefs.h - \<<_ACEOF >conftest.$ac_ext  
/* end confdefs.h.  */  
#include <isl/schedule.h>  
int  
main ()  
{  
isl_options_set_schedule_serialize_sccs (NULL, 0);  
  ;  
  return 0;  
}  
_ACEOF  
if ac_fn_c_try_link "$LINENO"; then :  
  gcc_cv_isl=yes  
else  
  gcc_cv_isl=no  
fi  
rm -f core conftest.err conftest.$ac_objext \  
    conftest$ac_exeext conftest.$ac_ext  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: $gcc_cv_isl" >&5  
$as_echo "$gcc_cv_isl" >&6; }  
  
    if test "${gcc_cv_isl}" = no ; then  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: required isl version is 0.15 or later" >&5  
$as_echo "required isl version is 0.15 or later" >&6; }  
    fi  
  
    CFLAGS=$_isl_saved_CFLAGS  
    LDFLAGS=$_isl_saved_LDFLAGS  
    LIBS=$_isl_saved_LIBS  
  fi  
  
  
  
  
  
  
  if test "x${with_isl}" = xno; then  
    graphite_requested=no  
  elif test "x${with_isl}" != x \  
    || test "x${with_isl_include}" != x \  
    || test "x${with_isl_lib}" != x ; then  
    graphite_requested=yes  
  else  
    graphite_requested=no  
  fi  
  
  
  
  if test "${gcc_cv_isl}" = no ; then  
    isllibs=  
    islinc=  
  fi  
  
  if test "${graphite_requested}" = yes \  
    && test "x${isllibs}" = x \  
    && test "x${islinc}" = x ; then  
  
    as_fn_error $? "Unable to find a usable isl.  See config.log for details." "$LINENO" 5  
  fi  
  
  
fi  
  
# If the isl check failed, disable builds of in-tree variant of isl  
if test "x$with_isl" = xno ||  
   test "x$gcc_cv_isl" = xno; then  
  noconfigdirs="$noconfigdirs isl"  
  islinc=  
fi  
  
  
  
  
# Check for LTO support.  
# Check whether --enable-lto was given.  
if test "${enable_lto+set}" = set; then :  
  enableval=$enable_lto; enable_lto=$enableval  
else  
  enable_lto=yes; default_enable_lto=yes  
fi  
  
  
  
  
  
target_elf=no  
case $target in  
  *-darwin* | *-aix* | *-cygwin* | *-mingw* | *-aout* | *-*coff* | \  
  *-msdosdjgpp* | *-vms* | *-wince* | *-*-pe* | \  
  alpha*-dec-osf* | *-interix* | hppa[12]*-*-hpux* | \  
  nvptx-*-none)  
    target_elf=no  
    ;;  
  *)  
    target_elf=yes  
    ;;  
esac  
  
if test $target_elf = yes; then :  
  # ELF platforms build the lto-plugin always.  
  build_lto_plugin=yes  
  
else  
  if test x"$default_enable_lto" = x"yes" ; then  
    case $target in  
      *-apple-darwin[912]* | *-cygwin* | *-mingw* | *djgpp*) ;;  
      # On other non-ELF platforms, LTO has yet to be validated.  
      *) enable_lto=no ;;  
    esac  
  else  
  # Apart from ELF platforms, only Windows and Darwin support LTO so far.  
  # It would also be nice to check the binutils support, but we don't  
  # have gcc_GAS_CHECK_FEATURE available here.  For now, we'll just  
  # warn during gcc/ subconfigure; unless you're bootstrapping with  
  # -flto it won't be needed until after installation anyway.  
    case $target in  
      *-cygwin* | *-mingw* | *-apple-darwin* | *djgpp*) ;;  
      *) if test x"$enable_lto" = x"yes"; then  
    as_fn_error $? "LTO support is not enabled for this target." "$LINENO" 5  
        fi  
      ;;  
    esac  
  fi  
  # Among non-ELF, only Windows platforms support the lto-plugin so far.  
  # Build it unless LTO was explicitly disabled.  
  case $target in  
    *-cygwin* | *-mingw*) build_lto_plugin=$enable_lto ;;  
    *) ;;  
  esac  
  
fi  
  
  
# Check whether --enable-linker-plugin-configure-flags was given.  
if test "${enable_linker_plugin_configure_flags+set}" = set; then :  
  enableval=$enable_linker_plugin_configure_flags; extra_linker_plugin_configure_flags=$enableval  
else  
  extra_linker_plugin_configure_flags=  
fi  
  
  
# Check whether --enable-linker-plugin-flags was given.  
if test "${enable_linker_plugin_flags+set}" = set; then :  
  enableval=$enable_linker_plugin_flags; extra_linker_plugin_flags=$enableval  
else  
  extra_linker_plugin_flags=  
fi  
  
  
  
# Handle --enable-host-pie  
# If host PIE executables are the default (or must be forced on) for some host,  
# we must pass that configuration to the gcc directory.  
gcc_host_pie=  
# Check whether --enable-host-pie was given.  
if test "${enable_host_pie+set}" = set; then :  
  enableval=$enable_host_pie; host_pie=$enableval  
 case $host in  
   *-*-darwin2*)  
     if test x$host_pie != xyes ; then  
       # for Darwin20+ this is required.  
       { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: PIE executables are required for the configured host, host-pie setting ignored." >&5  
$as_echo "$as_me: WARNING: PIE executables are required for the configured host, host-pie setting ignored." >&2;}  
       host_pie=yes  
       gcc_host_pie=--enable-host-pie  
     fi ;;  
  *) ;;  
 esac  
else  
  case $host in  
  *-*-darwin2*)  
    # Default to PIE (mandatory for aarch64).  
    host_pie=yes  
    gcc_host_pie=--enable-host-pie  
    ;;  
  *) host_pie=no ;;  
 esac  
fi  
  
  
  
  
  
# Enable --enable-host-shared.  
# Checked early to determine whether jit is an 'all' language  
# Check whether --enable-host-shared was given.  
if test "${enable_host_shared+set}" = set; then :  
  enableval=$enable_host_shared; host_shared=$enableval  
 case $host in  
   x86_64-*-darwin* | aarch64-*-darwin*)  
     if test x$host_shared != xyes ; then  
       # PIC is the default, and actually cannot be switched off.  
       { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: PIC code is required for the configured host; host-shared setting ignored." >&5  
$as_echo "$as_me: WARNING: PIC code is required for the configured host; host-shared setting ignored." >&2;}  
       host_shared=yes  
     fi ;;  
   *-*-darwin*)  
     if test x$host_pie = xyes -a x$host_shared != xyes ; then  
       { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: PIC code is required for PIE host executables host-shared setting ignored." >&5  
$as_echo "$as_me: WARNING: PIC code is required for PIE host executables host-shared setting ignored." >&2;}  
       host_shared=yes  
     fi ;;  
  *) ;;  
 esac  
else  
  case $host in  
  # 64B x86_64 and Aarch64 Darwin default to PIC.  
  x86_64-*-darwin* | aarch64-*-darwin*) host_shared=yes ;;  
  # 32B and powerpc64 Darwin must use PIC to link PIE exes.  
  *-*-darwin*) host_shared=$host_pie ;;  
  *) host_shared=no;;  
 esac  
fi  
  
  
  
  
if test x$host_shared = xyes; then  
  case $host in  
    *-*-darwin*)  
      # Since host shared is the default for 64b Darwin, and also enabled for  
      # host_pie, ensure that we present the PIE flag when host_pie is active.  
      if test x$host_pie = xyes; then  
        PICFLAG=-fPIE  
      fi  
      ;;  
    *)  
      PICFLAG=-fPIC  
      ;;  
  esac  
elif test x$host_pie = xyes; then  
  PICFLAG=-fPIE  
else  
  PICFLAG=  
fi  
  
  
  
# If we are building PIC/PIE host executables, and we are building dependent  
# libs (e.g. GMP) in-tree those libs need to be configured to generate PIC  
# code.  
host_libs_picflag=  
if test "$host_shared" = "yes" -o "$host_pie" = "yes"; then  
host_libs_picflag='--with-pic'  
fi  
  
  
# By default, C and C++ are the only stage 1 languages.  
stage1_languages=,c,  
  
# Target libraries that we bootstrap.  
bootstrap_target_libs=,target-libgcc,  
  
# Figure out what language subdirectories are present.  
# Look if the user specified --enable-languages="..."; if not, use  
# the environment variable $LANGUAGES if defined. $LANGUAGES might  
# go away some day.  
# NB:  embedded tabs in this IF block -- do not untabify  
if test -d ${srcdir}/gcc; then  
  if test x"${enable_languages+set}" != xset; then  
    if test x"${LANGUAGES+set}" = xset; then  
      enable_languages="${LANGUAGES}"  
        echo configure.ac: warning: setting LANGUAGES is deprecated, use --enable-languages instead 1>&2  
    else  
      enable_languages=default  
    fi  
  else  
    if test x"${enable_languages}" = x ||  
       test x"${enable_languages}" = xyes;  
       then  
      echo configure.ac: --enable-languages needs at least one language argument 1>&2  
      exit 1  
    fi  
  fi  
  enable_languages=`echo "${enable_languages}" | sed -e 's/[     ,][     ,]*/,/g' -e 's/,$//'`  
  
  # 'f95' is the old name for the 'fortran' language. We issue a warning  
  # and make the substitution.  
  case ,${enable_languages}, in  
    *,f95,*)  
      echo configure.ac: warning: 'f95' as language name is deprecated, use 'fortran' instead 1>&2  
      enable_languages=`echo "${enable_languages}" | sed -e 's/f95/fortran/g'`  
      ;;  
  esac  
  
  # If bootstrapping, C++ must be enabled.  
  case ",$enable_languages,:$enable_bootstrap" in  
    *,c++,*:*) ;;  
    *:yes)  
      if test -f ${srcdir}/gcc/cp/config-lang.in; then  
        enable_languages="${enable_languages},c++"  
      else  
        as_fn_error $? "bootstrapping requires c++ sources" "$LINENO" 5  
      fi  
      ;;  
  esac  
  
  # First scan to see if an enabled language requires some other language.  
  # We assume that a given config-lang.in will list all the language  
  # front ends it requires, even if some are required indirectly.  
  for lang_frag in ${srcdir}/gcc/*/config-lang.in .. ; do  
    case ${lang_frag} in  
      ..) ;;  
      # The odd quoting in the next line works around  
      # an apparent bug in bash 1.12 on linux.  
      ${srcdir}/gcc/[*]/config-lang.in) ;;  
      *)  
        # From the config-lang.in, get $language, $lang_requires, and  
        # $lang_requires_boot_languages.  
        language=  
        lang_requires=  
        lang_requires_boot_languages=  
        # set srcdir during sourcing lang_frag to the gcc dir.  
        # Sadly overriding srcdir on the . line doesn't work in plain sh as it  
        # polutes this shell  
        saved_srcdir=${srcdir}  
        srcdir=${srcdir}/gcc . ${lang_frag}  
        srcdir=${saved_srcdir}  
        for other in ${lang_requires} ${lang_requires_boot_languages}; do  
          case ,${enable_languages}, in  
        *,$other,*) ;;  
        *,default,*) ;;  
        *,all,*) ;;  
        *,$language,*)  
          echo " \`$other' language required by \`$language'; enabling" 1>&2  
          enable_languages="${enable_languages},${other}"  
          ;;  
      esac  
        done  
    for other in ${lang_requires_boot_languages} ; do  
      if test "$other" != "c"; then  
        case ,${enable_stage1_languages}, in  
          *,$other,*) ;;  
          *,default,*) ;;  
          *,all,*) ;;  
          *)  
        case ,${enable_languages}, in  
          *,$language,*)  
            echo " '$other' language required by '$language' in stage 1; enabling" 1>&2  
            enable_stage1_languages="$enable_stage1_languages,${other}"  
            ;;  
        esac  
        ;;  
        esac  
          fi  
        done  
        ;;  
    esac  
  done  
  
  new_enable_languages=,c,  
  
  # If LTO is enabled, add the LTO front end.  
  if test "$enable_lto" = "yes" ; then  
    case ,${enable_languages}, in  
      *,lto,*) ;;  
      *) enable_languages="${enable_languages},lto" ;;  
    esac  
    if test "${build_lto_plugin}" = "yes" ; then  
      configdirs="$configdirs lto-plugin"  
    fi  
  fi  
  
  # If we're building an offloading compiler, add the LTO front end.  
  if test x"$enable_as_accelerator_for" != x ; then  
    case ,${enable_languages}, in  
      *,lto,*) ;;  
      *) enable_languages="${enable_languages},lto" ;;  
    esac  
  fi  
  
  missing_languages=`echo ",$enable_languages," | sed -e s/,default,/,/ -e s/,all,/,/ -e s/,c,/,/ `  
  potential_languages=,c,  
  
  enabled_target_libs=  
  disabled_target_libs=  
  
  for lang_frag in ${srcdir}/gcc/*/config-lang.in .. ; do  
    case ${lang_frag} in  
      ..) ;;  
      # The odd quoting in the next line works around  
      # an apparent bug in bash 1.12 on linux.  
      ${srcdir}/gcc/[*]/config-lang.in) ;;  
      *)  
        # From the config-lang.in, get $language, $target_libs,  
        # $lang_dirs, $boot_language, and $build_by_default  
        language=  
        target_libs=  
        lang_dirs=  
        subdir_requires=  
        boot_language=no  
        build_by_default=yes  
        # set srcdir during sourcing.  See above about save & restore  
        saved_srcdir=${srcdir}  
        srcdir=${srcdir}/gcc . ${lang_frag}  
        srcdir=${saved_srcdir}  
        if test x${language} = x; then  
          echo "${lang_frag} doesn't set \$language." 1>&2  
          exit 1  
        fi  
  
    if test "$language" = "c++"; then  
      boot_language=yes  
    fi  
  
        add_this_lang=no  
        # C is always enabled, so no need to add it again  
        if test "$language" != "c"; then  
          case ,${enable_languages}, in  
            *,${language},*)  
              # Language was explicitly selected; include it  
          add_this_lang=yes  
              ;;  
        *,all,*)  
          # All languages are enabled  
          add_this_lang=all  
              ;;  
            *,default,*)  
              # 'default' was selected, select it if it is a default language  
          add_this_lang=${build_by_default}  
              ;;  
          esac  
        fi  
  
        # Disable languages that need other directories if these aren't available.  
    for i in $subdir_requires; do  
      test -f "$srcdir/gcc/$i/config-lang.in" && continue  
      case ${add_this_lang} in  
        yes)  
              # Specifically requested language; tell them.  
              as_fn_error $? "The gcc/$i directory contains parts of $language but is missing" "$LINENO" 5  
              ;;  
            all)  
              { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: The gcc/$i directory contains parts of $language but is missing" >&5  
$as_echo "$as_me: WARNING: The gcc/$i directory contains parts of $language but is missing" >&2;}  
              add_this_lang=unsupported  
              ;;  
            *)  
              # Silently disable.  
              add_this_lang=unsupported  
              ;;  
          esac  
    done  
  
        # Disable Ada if no preexisting GNAT is available.  
        case ${add_this_lang}:${language}:${have_gnat} in  
          yes:ada:no)  
            # Specifically requested language; tell them.  
            as_fn_error $? "GNAT is required to build $language" "$LINENO" 5  
            ;;  
          all:ada:no)  
            { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: GNAT is required to build $language" >&5  
$as_echo "$as_me: WARNING: GNAT is required to build $language" >&2;}  
            add_this_lang=unsupported  
            ;;  
          *:ada:no)  
            # Silently disable.  
            add_this_lang=unsupported  
            ;;  
        esac  
  
        # Disable D if no preexisting GDC is available.  
        case ${add_this_lang}:${language}:${have_gdc} in  
          yes:d:no)  
            # Specifically requested language; tell them.  
            as_fn_error $? "GDC is required to build $language" "$LINENO" 5  
           ;;  
          all:d:no)  
            { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: GDC is required to build $language" >&5  
$as_echo "$as_me: WARNING: GDC is required to build $language" >&2;}  
            add_this_lang=unsupported  
            ;;  
          *:d:no)  
            # Silently disable.  
            add_this_lang=unsupported  
            ;;  
        esac  
  
        # Disable jit if -enable-host-shared not specified  
        # but not if building for Mingw. All code in Windows  
        # is position independent code (PIC).  
        case $target in  
          *mingw*) ;;  
          *)  
          case ${add_this_lang}:${language}:${host_shared} in  
            yes:jit:no)  
               # PR jit/64780: explicitly specify --enable-host-shared  
        as_fn_error $? "  
Enabling language \"jit\" requires --enable-host-shared.  
  
--enable-host-shared typically slows the rest of the compiler down by  
a few %, so you must explicitly enable it.  
  
If you want to build both the jit and the regular compiler, it is often  
best to do this via two separate configure/builds, in separate  
directories, to avoid imposing the performance cost of  
--enable-host-shared on the regular compiler." "$LINENO" 5  
                ;;  
            all:jit:no)  
          { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: --enable-host-shared required to build $language" >&5  
$as_echo "$as_me: WARNING: --enable-host-shared required to build $language" >&2;}  
              add_this_lang=unsupported  
              ;;  
            *:jit:no)  
              # Silently disable.  
              add_this_lang=unsupported  
              ;;  
            esac  
          ;;  
        esac  
  
        # Disable a language that is unsupported by the target.  
    case "${add_this_lang}: $unsupported_languages " in  
      no:*) ;;  
      unsupported:*) ;;  
      *:*" $language "*)  
        { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: ${language} not supported for this target" >&5  
$as_echo "$as_me: WARNING: ${language} not supported for this target" >&2;}  
        add_this_lang=unsupported  
        ;;  
    esac  
  
    case $add_this_lang in  
      unsupported)  
            # Remove language-dependent dirs.  
        disabled_target_libs="$disabled_target_libs $target_libs"  
        noconfigdirs="$noconfigdirs $lang_dirs"  
        ;;  
      no)  
            # Remove language-dependent dirs; still show language as supported.  
        disabled_target_libs="$disabled_target_libs $target_libs"  
        noconfigdirs="$noconfigdirs $lang_dirs"  
            potential_languages="${potential_languages}${language},"  
        ;;  
          all|yes)  
        new_enable_languages="${new_enable_languages}${language},"  
            potential_languages="${potential_languages}${language},"  
        missing_languages=`echo "$missing_languages" | sed "s/,$language,/,/"`  
        enabled_target_libs="$enabled_target_libs $target_libs"  
        case "${boot_language}:,$enable_stage1_languages," in  
          yes:* | *:*,$language,* | *:*,yes, | *:*,all,)  
        # Add to (comma-separated) list of stage 1 languages.  
        case ",$stage1_languages," in  
          *,$language,* | ,yes, | ,all,) ;;  
          *) stage1_languages="${stage1_languages}${language}," ;;  
        esac  
        # We need to bootstrap any supporting libraries.  
        bootstrap_target_libs=`echo "${bootstrap_target_libs}${target_libs}," | sed "s/ /,/g"`  
        ;;  
        esac  
        ;;  
        esac  
        ;;  
    esac  
  done  
  
  # Add target libraries which are only needed for disabled languages  
  # to noconfigdirs.  
  if test -n "$disabled_target_libs"; then  
    for dir in $disabled_target_libs; do  
      case " $enabled_target_libs " in  
      *" ${dir} "*) ;;  
      *) noconfigdirs="$noconfigdirs $dir" ;;  
      esac  
    done  
  fi  
  
  # Check whether --enable-stage1-languages was given.  
if test "${enable_stage1_languages+set}" = set; then :  
  enableval=$enable_stage1_languages; case ,${enable_stage1_languages}, in  
    ,no,|,,)  
      # Set it to something that will have no effect in the loop below  
      enable_stage1_languages=c ;;  
    ,yes,)  
      enable_stage1_languages=`echo $new_enable_languages | \  
    sed -e "s/^,//" -e "s/,$//" ` ;;  
    *,all,*)  
      enable_stage1_languages=`echo ,$enable_stage1_languages, | \  
    sed -e "s/,all,/$new_enable_languages/" -e "s/^,//" -e "s/,$//" ` ;;  
  esac  
  
  # Add "good" languages from enable_stage1_languages to stage1_languages,  
  # while "bad" languages go in missing_languages.  Leave no duplicates.  
  for i in `echo $enable_stage1_languages | sed 's/,/ /g' `; do  
    case $potential_languages in  
      *,$i,*)  
        case $stage1_languages in  
          *,$i,*) ;;  
          *) stage1_languages="$stage1_languages$i," ;;  
        esac ;;  
      *)  
        case $missing_languages in  
          *,$i,*) ;;  
          *) missing_languages="$missing_languages$i," ;;  
        esac ;;  
     esac  
  done  
fi  
  
  
  # Remove leading/trailing commas that were added for simplicity  
  potential_languages=`echo "$potential_languages" | sed -e "s/^,//" -e "s/,$//"`  
  missing_languages=`echo "$missing_languages" | sed -e "s/^,//" -e "s/,$//"`  
  stage1_languages=`echo "$stage1_languages" | sed -e "s/^,//" -e "s/,$//"`  
  new_enable_languages=`echo "$new_enable_languages" | sed -e "s/^,//" -e "s/,$//"`  
  
  if test "x$missing_languages" != x; then  
    as_fn_error $? "  
The following requested languages could not be built: ${missing_languages}  
Supported languages are: ${potential_languages}" "$LINENO" 5  
  fi  
  if test "x$new_enable_languages" != "x$enable_languages"; then  
    echo The following languages will be built: ${new_enable_languages}  
    enable_languages="$new_enable_languages"  
  fi  
  
  
  ac_configure_args=`echo " $ac_configure_args" | sed -e "s/ '--enable-languages=[^ ]*'//g" -e "s/$/ '--enable-languages="$enable_languages"'/" `  
fi  
  
# Handle --disable-<component> generically.  
for dir in $configdirs $build_configdirs $target_configdirs ; do  
  dirname=`echo $dir | sed -e s/target-//g -e s/build-//g -e s/-/_/g`  
  varname=`echo $dirname | sed -e s/+/_/g`  
  if eval test x\${enable_${varname}} "=" xno ; then  
    noconfigdirs="$noconfigdirs $dir"  
  fi  
done  
  
# Check for Boehm's garbage collector  
# Check whether --enable-objc-gc was given.  
if test "${enable_objc_gc+set}" = set; then :  
  enableval=$enable_objc_gc;  
fi  
  
  
# Check whether --with-target-bdw-gc was given.  
if test "${with_target_bdw_gc+set}" = set; then :  
  withval=$with_target_bdw_gc;  
fi  
  
  
# Check whether --with-target-bdw-gc-include was given.  
if test "${with_target_bdw_gc_include+set}" = set; then :  
  withval=$with_target_bdw_gc_include;  
fi  
  
  
# Check whether --with-target-bdw-gc-lib was given.  
if test "${with_target_bdw_gc_lib+set}" = set; then :  
  withval=$with_target_bdw_gc_lib;  
fi  
  
  
case ,${enable_languages},:${enable_objc_gc} in *,objc,*:yes|*,objc,*:auto)  
  { $as_echo "$as_me:${as_lineno-$LINENO}: checking for bdw garbage collector" >&5  
$as_echo_n "checking for bdw garbage collector... " >&6; }  
  if test "x$with_target_bdw_gc$with_target_bdw_gc_include$with_target_bdw_gc_lib" = x; then  
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: using bdw-gc in default locations" >&5  
$as_echo "using bdw-gc in default locations" >&6; }  
  else  
        if test "x$with_target_bdw_gc_include" = x && test "x$with_target_bdw_gc_lib" != x; then  
      as_fn_error $? "found --with-target-bdw-gc-lib but --with-target-bdw-gc-include missing" "$LINENO" 5  
    elif test "x$with_target_bdw_gc_include" != x && test "x$with_target_bdw_gc_lib" = x; then  
      as_fn_error $? "found --with-target-bdw-gc-include but --with-target-bdw-gc-lib missing" "$LINENO" 5  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: using paths configured with --with-target-bdw-gc options" >&5  
$as_echo "using paths configured with --with-target-bdw-gc options" >&6; }  
    fi  
  fi  
esac  
  
# Disable libitm, libsanitizer, libvtv if we're not building C++  
case ,${enable_languages}, in  
  *,c++,*)  
    # Disable libitm, libsanitizer if we're not building libstdc++  
    case "${noconfigdirs}" in  
      *target-libstdc++-v3*)  
        noconfigdirs="$noconfigdirs target-libitm target-libsanitizer"  
        ;;  
      *) ;;  
    esac  
    ;;  
  *)  
    noconfigdirs="$noconfigdirs target-libitm target-libsanitizer target-libvtv"  
    ;;  
esac  
  
case ,${enable_languages}, in  
  *,rust,*)  
    case " ${noconfigdirs} " in  
      *\ target-libstdc++-v3\ *)  
        # Disable target libgrust if we're not building target libstdc++.  
        noconfigdirs="$noconfigdirs target-libgrust"  
        ;;  
    esac  
    ;;  
esac  
  
# If gcc/ is not in the source tree then we'll not be building a  
# target compiler, assume in that case we don't want to build any  
# target libraries or tools.  
#  
# This was added primarily for the benefit for binutils-gdb who reuse  
# this configure script, but don't always have target tools available.  
if test ! -d ${srcdir}/gcc; then  
   skipdirs="${skipdirs} ${target_configdirs}"  
fi  
  
# Remove the entries in $skipdirs and $noconfigdirs from $configdirs,  
# $build_configdirs and $target_configdirs.  
# If we have the source for $noconfigdirs entries, add them to $notsupp.  
  
notsupp=""  
for dir in . $skipdirs $noconfigdirs ; do  
  dirname=`echo $dir | sed -e s/target-//g -e s/build-//g`  
  if test $dir != .  && echo " ${configdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
    configdirs=`echo " ${configdirs} " | sed -e "s/ ${dir} / /"`  
    if test -r $srcdir/$dirname/configure ; then  
      if echo " ${skipdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
    true  
      else  
    notsupp="$notsupp $dir"  
      fi  
    fi  
  fi  
  if test $dir != .  && echo " ${build_configdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
    build_configdirs=`echo " ${build_configdirs} " | sed -e "s/ ${dir} / /"`  
    if test -r $srcdir/$dirname/configure ; then  
      if echo " ${skipdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
    true  
      else  
    notsupp="$notsupp $dir"  
      fi  
    fi  
  fi  
  if test $dir != . && echo " ${target_configdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
    target_configdirs=`echo " ${target_configdirs} " | sed -e "s/ ${dir} / /"`  
    if test -r $srcdir/$dirname/configure ; then  
      if echo " ${skipdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
    true  
      else  
    notsupp="$notsupp $dir"  
      fi  
    fi  
  fi  
done  
  
# Quietly strip out all directories which aren't configurable in this tree.  
# This relies on all configurable subdirectories being autoconfiscated, which  
# is now the case.  
build_configdirs_all="$build_configdirs"  
build_configdirs=  
for i in ${build_configdirs_all} ; do  
  j=`echo $i | sed -e s/build-//g`  
  if test -f ${srcdir}/$j/configure ; then  
    build_configdirs="${build_configdirs} $i"  
  fi  
done  
  
configdirs_all="$configdirs"  
configdirs=  
for i in ${configdirs_all} ; do  
  if test -f ${srcdir}/$i/configure ; then  
    configdirs="${configdirs} $i"  
  fi  
done  
  
target_configdirs_all="$target_configdirs"  
target_configdirs=  
for i in ${target_configdirs_all} ; do  
  j=`echo $i | sed -e s/target-//g`  
  if test -f ${srcdir}/$j/configure ; then  
    target_configdirs="${target_configdirs} $i"  
  fi  
done  
  
# libiberty-linker-plugin is special: it doesn't have its own source directory,  
# so we have to add it after the preceding checks.  
if test x"$extra_linker_plugin_flags$extra_linker_plugin_configure_flags" != x  
then  
  case " $configdirs " in  
    *" libiberty "*)  
      # If we can build libiberty, we can also build libiberty-linker-plugin.  
      configdirs="$configdirs libiberty-linker-plugin"  
      extra_linker_plugin_configure_flags="$extra_linker_plugin_configure_flags \  
        --with-libiberty=../libiberty-linker-plugin";;  
    *)  
      as_fn_error $? "libiberty missing" "$LINENO" 5;;  
  esac  
fi  
  
# Sometimes we have special requirements for the host libiberty.  
extra_host_libiberty_configure_flags=  
case " $configdirs " in  
  *" lto-plugin "* | *" libcc1 "* | *" gdbserver "*)  
    # When these are to be built as shared libraries, the same applies to  
    # libiberty.  
    extra_host_libiberty_configure_flags=--enable-shared  
    ;;  
esac  
  
  
# Sometimes we have special requirements for the host zlib.  
extra_host_zlib_configure_flags=  
case " $configdirs " in  
  *" bfd "*)  
    # When bfd is to be built as a shared library, the same applies to  
    # zlib.  
    if test "$enable_shared" = "yes"; then  
      extra_host_zlib_configure_flags=--enable-host-shared  
    fi  
    ;;  
esac  
  
  
# Produce a warning message for the subdirs we can't configure.  
# This isn't especially interesting in the Cygnus tree, but in the individual  
# FSF releases, it's important to let people know when their machine isn't  
# supported by the one or two programs in a package.  
  
if test -n "${notsupp}" && test -z "${norecursion}" ; then  
  # If $appdirs is non-empty, at least one of those directories must still  
  # be configured, or we error out.  (E.g., if the gas release supports a  
  # specified target in some subdirs but not the gas subdir, we shouldn't  
  # pretend that all is well.)  
  if test -n "$appdirs" ; then  
    for dir in $appdirs ; do  
      if test -r $dir/Makefile.in ; then  
    if echo " ${configdirs} " | grep " ${dir} " >/dev/null 2>&1; then  
      appdirs=""  
      break  
    fi  
    if echo " ${target_configdirs} " | grep " target-${dir} " >/dev/null 2>&1; then  
      appdirs=""  
      break  
    fi  
      fi  
    done  
    if test -n "$appdirs" ; then  
      echo "*** This configuration is not supported by this package." 1>&2  
      exit 1  
    fi  
  fi  
  # Okay, some application will build, or we don't care to check.  Still  
  # notify of subdirs not getting built.  
  echo "*** This configuration is not supported in the following subdirectories:" 1>&2  
  echo "    ${notsupp}" 1>&2  
  echo "    (Any other directories should still work fine.)" 1>&2  
fi  
  
case "$host" in  
  *msdosdjgpp*)  
    enable_gdbtk=no ;;  
esac  
  
# To find our prefix, in gcc_cv_tool_prefix.  
  
# The user is always right.  
if test "${PATH_SEPARATOR+set}" != set; then  
  echo "#! /bin/sh" >conf$$.sh  
  echo  "exit 0"   >>conf$$.sh  
  chmod +x conf$$.sh  
  if (PATH="/nonexistent;."; conf$$.sh) >/dev/null 2>&1; then  
    PATH_SEPARATOR=';'  
  else  
    PATH_SEPARATOR=:  
  fi  
  rm -f conf$$.sh  
fi  
  
  
  get_gcc_base_ver="cat"  
  
# Check whether --with-gcc-major-version-only was given.  
if test "${with_gcc_major_version_only+set}" = set; then :  
  withval=$with_gcc_major_version_only; if test x$with_gcc_major_version_only = xyes ; then  
        get_gcc_base_ver="sed -e 's/^\([0-9]*\).*/\1/'"  
      fi  
  
fi  
  
  
  
  
  
  
if test "x$exec_prefix" = xNONE; then  
        if test "x$prefix" = xNONE; then  
                gcc_cv_tool_prefix=$ac_default_prefix  
        else  
                gcc_cv_tool_prefix=$prefix  
        fi  
else  
        gcc_cv_tool_prefix=$exec_prefix  
fi  
  
# If there is no compiler in the tree, use the PATH only.  In any  
# case, if there is no compiler in the tree nobody should use  
# AS_FOR_TARGET and LD_FOR_TARGET.  
if test x$host = x$build && test -f $srcdir/gcc/BASE-VER; then  
    if test x$with_gcc_major_version_only = xyes ; then  
                gcc_version=`sed -e 's/^\([0-9]*\).*$/\1/' $srcdir/gcc/BASE-VER`  
            else  
        gcc_version=`cat $srcdir/gcc/BASE-VER`  
    fi  
    gcc_cv_tool_dirs="$gcc_cv_tool_prefix/libexec/gcc/$target_noncanonical/$gcc_version$PATH_SEPARATOR"  
    gcc_cv_tool_dirs="$gcc_cv_tool_dirs$gcc_cv_tool_prefix/libexec/gcc/$target_noncanonical$PATH_SEPARATOR"  
    gcc_cv_tool_dirs="$gcc_cv_tool_dirs/usr/lib/gcc/$target_noncanonical/$gcc_version$PATH_SEPARATOR"  
    gcc_cv_tool_dirs="$gcc_cv_tool_dirs/usr/lib/gcc/$target_noncanonical$PATH_SEPARATOR"  
    gcc_cv_tool_dirs="$gcc_cv_tool_dirs$gcc_cv_tool_prefix/$target_noncanonical/bin/$target_noncanonical/$gcc_version$PATH_SEPARATOR"  
    gcc_cv_tool_dirs="$gcc_cv_tool_dirs$gcc_cv_tool_prefix/$target_noncanonical/bin$PATH_SEPARATOR"  
else  
    gcc_cv_tool_dirs=  
fi  
  
if test x$build = x$target && test -n "$md_exec_prefix"; then  
        gcc_cv_tool_dirs="$gcc_cv_tool_dirs$md_exec_prefix$PATH_SEPARATOR"  
fi  
  
  
  
copy_dirs=  
  
  
# Check whether --with-build-sysroot was given.  
if test "${with_build_sysroot+set}" = set; then :  
  withval=$with_build_sysroot; if test x"$withval" != x ; then  
     SYSROOT_CFLAGS_FOR_TARGET="--sysroot=$withval"  
   fi  
else  
  SYSROOT_CFLAGS_FOR_TARGET=  
fi  
  
  
  
  
# Check whether --with-debug-prefix-map was given.  
if test "${with_debug_prefix_map+set}" = set; then :  
  withval=$with_debug_prefix_map; if test x"$withval" != x; then  
     DEBUG_PREFIX_CFLAGS_FOR_TARGET=  
     for debug_map in $withval; do  
       DEBUG_PREFIX_CFLAGS_FOR_TARGET="$DEBUG_PREFIX_CFLAGS_FOR_TARGET -fdebug-prefix-map=$debug_map"  
     done  
   fi  
else  
  DEBUG_PREFIX_CFLAGS_FOR_TARGET=  
fi  
  
  
  
# During gcc bootstrap, if we use some random cc for stage1 then CFLAGS  
# might be empty or "-g".  We don't require a C++ compiler, so CXXFLAGS  
# might also be empty (or "-g", if a non-GCC C++ compiler is in the path).  
# We want to ensure that TARGET libraries (which we know are built with  
# gcc) are built with "-O2 -g", so include those options when setting  
# CFLAGS_FOR_TARGET and CXXFLAGS_FOR_TARGET.  
if test "x$CFLAGS_FOR_TARGET" = x; then  
  if test "x${is_cross_compiler}" = xyes; then  
    CFLAGS_FOR_TARGET="-g -O2"  
  else  
    CFLAGS_FOR_TARGET=$CFLAGS  
    case " $CFLAGS " in  
      *" -O2 "*) ;;  
      *) CFLAGS_FOR_TARGET="-O2 $CFLAGS_FOR_TARGET" ;;  
    esac  
    case " $CFLAGS " in  
      *" -g "* | *" -g3 "*) ;;  
      *) CFLAGS_FOR_TARGET="-g $CFLAGS_FOR_TARGET" ;;  
    esac  
  fi  
fi  
  
  
if test "x$CXXFLAGS_FOR_TARGET" = x; then  
  if test "x${is_cross_compiler}" = xyes; then  
    CXXFLAGS_FOR_TARGET="-g -O2"  
  else  
    CXXFLAGS_FOR_TARGET=$CXXFLAGS  
    case " $CXXFLAGS " in  
      *" -O2 "*) ;;  
      *) CXXFLAGS_FOR_TARGET="-O2 $CXXFLAGS_FOR_TARGET" ;;  
    esac  
    case " $CXXFLAGS " in  
      *" -g "* | *" -g3 "*) ;;  
      *) CXXFLAGS_FOR_TARGET="-g $CXXFLAGS_FOR_TARGET" ;;  
    esac  
  fi  
fi  
  
  
  
  
# Handle --with-headers=XXX.  If the value is not "yes", the contents of  
# the named directory are copied to $(tooldir)/sys-include.  
if test x"${with_headers}" != x && test x"${with_headers}" != xno ; then  
  if test x${is_cross_compiler} = xno ; then  
    echo 1>&2 '***' --with-headers is only supported when cross compiling  
    exit 1  
  fi  
  if test x"${with_headers}" != xyes ; then  
    x=${gcc_cv_tool_prefix}  
    copy_dirs="${copy_dirs} ${with_headers} $x/${target_noncanonical}/sys-include"  
  fi  
fi  
  
# Handle --with-libs=XXX.  If the value is not "yes", the contents of  
# the name directories are copied to $(tooldir)/lib.  Multiple directories  
# are permitted.  
if test x"${with_libs}" != x && test x"${with_libs}" != xno ; then  
  if test x${is_cross_compiler} = xno ; then  
    echo 1>&2 '***' --with-libs is only supported when cross compiling  
    exit 1  
  fi  
  if test x"${with_libs}" != xyes ; then  
    # Copy the libraries in reverse order, so that files in the first named  
    # library override files in subsequent libraries.  
    x=${gcc_cv_tool_prefix}  
    for l in ${with_libs}; do  
      copy_dirs="$l $x/${target_noncanonical}/lib ${copy_dirs}"  
    done  
  fi  
fi  
  
# Set with_gnu_as, with_gnu_ld, and with_system_zlib as appropriate.  
#  
# This is done by determining whether or not the appropriate directory  
# is available, and by checking whether or not specific configurations  
# have requested that this magic not happen.  
#  
# The command line options always override the explicit settings in  
# configure.ac, and the settings in configure.ac override this magic.  
#  
# If the default for a toolchain is to use GNU as and ld, and you don't  
# want to do that, then you should use the --without-gnu-as and  
# --without-gnu-ld options for the configure script.  Similarly, if  
# the default is to use the included zlib and you don't want to do that,  
# you should use the --with-system-zlib option for the configure script.  
  
if test x${use_gnu_as} = x &&  
   echo " ${configdirs} " | grep " gas " > /dev/null 2>&1 ; then  
  with_gnu_as=yes  
  extra_host_args="$extra_host_args --with-gnu-as"  
fi  
  
if test x${use_gnu_ld} = x &&  
   echo " ${configdirs} " | egrep " (go)?ld " > /dev/null 2>&1 ; then  
  with_gnu_ld=yes  
  extra_host_args="$extra_host_args --with-gnu-ld"  
fi  
  
if test x${use_included_zlib} = x &&  
   echo " ${configdirs} " | grep " zlib " > /dev/null 2>&1 ; then  
  :  
else  
  with_system_zlib=yes  
  extra_host_args="$extra_host_args --with-system-zlib"  
fi  
  
# If using newlib, add --with-newlib to the extra_host_args so that gcc/configure  
# can detect this case.  
  
if test x${with_newlib} != xno && echo " ${target_configdirs} " | grep " target-newlib " > /dev/null 2>&1 ; then  
  with_newlib=yes  
  extra_host_args="$extra_host_args --with-newlib"  
fi  
  
# Handle ${copy_dirs}  
set fnord ${copy_dirs}  
shift  
while test $# != 0 ; do  
  if test -f $2/COPIED && test x"`cat $2/COPIED`" = x"$1" ; then  
    :  
  else  
    echo Copying $1 to $2  
  
    # Use the install script to create the directory and all required  
    # parent directories.  
    if test -d $2 ; then  
      :  
    else  
      echo >config.temp  
      ${srcdir}/install-sh -c -m 644 config.temp $2/COPIED  
    fi  
  
    # Copy the directory, assuming we have tar.  
    # FIXME: Should we use B in the second tar?  Not all systems support it.  
    (cd $1; tar -cf - .) | (cd $2; tar -xpf -)  
  
    # It is the responsibility of the user to correctly adjust all  
    # symlinks.  If somebody can figure out how to handle them correctly  
    # here, feel free to add the code.  
  
    echo $1 > $2/COPIED  
  fi  
  shift; shift  
done  
  
# Determine a target-dependent exec_prefix that the installed  
# gcc will search in.  Keep this list sorted by triplet, with  
# the *-*-osname triplets last.  
md_exec_prefix=  
case "${target}" in  
  i[34567]86-pc-msdosdjgpp*)  
    md_exec_prefix=/dev/env/DJDIR/bin  
    ;;  
  *-*-hpux* | \  
  *-*-nto-qnx* | \  
  *-*-solaris2*)  
    md_exec_prefix=/usr/ccs/bin  
    ;;  
esac  
  
extra_arflags_for_target=  
extra_nmflags_for_target=  
extra_ranlibflags_for_target=  
target_makefile_frag=/dev/null  
case "${target}" in  
  spu-*-*)  
    target_makefile_frag="config/mt-spu"  
    ;;  
  loongarch*-*linux* | loongarch*-*gnu*)  
    target_makefile_frag="config/mt-loongarch-gnu"  
    ;;  
  loongarch*-*elf*)  
    target_makefile_frag="config/mt-loongarch-elf"  
    ;;  
  mips*-sde-elf* | mips*-mti-elf* | mips*-img-elf*)  
    target_makefile_frag="config/mt-sde"  
    ;;  
  mipsisa*-*-elfoabi*)  
    target_makefile_frag="config/mt-mips-elfoabi"  
    ;;  
  mips*-*-*linux* | mips*-*-gnu*)  
    target_makefile_frag="config/mt-mips-gnu"  
    ;;  
  nios2-*-elf*)  
    target_makefile_frag="config/mt-nios2-elf"  
    ;;  
  *-*-linux-android*)  
    target_makefile_frag="config/mt-android"  
    ;;  
  *-*-linux* | *-*-gnu* | *-*-k*bsd*-gnu | *-*-kopensolaris*-gnu)  
    target_makefile_frag="config/mt-gnu"  
    ;;  
  *-*-aix4.[3456789]* | *-*-aix[56789].*)  
    # nm and ar from AIX 4.3 and above require -X32_64 flag to all ar and nm  
    # commands to handle both 32-bit and 64-bit objects.  These flags are  
    # harmless if we're using GNU nm or ar.  
    extra_arflags_for_target=" -X32_64"  
    extra_nmflags_for_target=" -B -X32_64"  
    ;;  
esac  
  
alphaieee_frag=/dev/null  
case $target in  
  alpha*-*-*)  
    # This just makes sure to use the -mieee option to build target libs.  
    # This should probably be set individually by each library.  
    alphaieee_frag="config/mt-alphaieee"  
    ;;  
esac  
  
# If --enable-target-optspace always use -Os instead of -O2 to build  
# the target libraries, similarly if it is not specified, use -Os  
# on selected platforms.  
ospace_frag=/dev/null  
case "${enable_target_optspace}:${target}" in  
  yes:*)  
    ospace_frag="config/mt-ospace"  
    ;;  
  :d30v-*)  
    ospace_frag="config/mt-d30v"  
    ;;  
  :m32r-* | :d10v-* | :fr30-* | :i?86*-*-elfiamcu)  
    ospace_frag="config/mt-ospace"  
    ;;  
  no:* | :*)  
    ;;  
  *)  
    echo "*** bad value \"${enable_target_optspace}\" for --enable-target-optspace flag; ignored" 1>&2  
    ;;  
esac  
  
# Some systems (e.g., one of the i386-aix systems the gas testers are  
# using) don't handle "\$" correctly, so don't use it here.  
tooldir='${exec_prefix}'/${target_noncanonical}  
build_tooldir=${tooldir}  
  
# Create a .gdbinit file which runs the one in srcdir  
# and tells GDB to look there for source files.  
  
if test -r ${srcdir}/.gdbinit ; then  
  case ${srcdir} in  
    .) ;;  
    *) cat > ./.gdbinit \<<EOF  
# ${NO_EDIT}  
dir ${srcdir}  
dir .  
source ${srcdir}/.gdbinit  
EOF  
    ;;  
  esac  
fi  
  
# Make sure that the compiler is able to generate an executable.  If it  
# can't, we are probably in trouble.  We don't care whether we can run the  
# executable--we might be using a cross compiler--we only care whether it  
# can be created.  At this point the main configure script has set CC.  
we_are_ok=no  
echo "int main () { return 0; }" > conftest.c  
${CC} -o conftest ${CFLAGS} ${CPPFLAGS} ${LDFLAGS} conftest.c  
if test $? = 0 ; then  
  if test -s conftest || test -s conftest.exe ; then  
    we_are_ok=yes  
  fi  
fi  
case $we_are_ok in  
  no)  
    echo 1>&2 "*** The command '${CC} -o conftest ${CFLAGS} ${CPPFLAGS} ${LDFLAGS} conftest.c' failed."  
    echo 1>&2 "*** You must set the environment variable CC to a working compiler."  
    rm -f conftest*  
    exit 1  
    ;;  
esac  
rm -f conftest*  
  
# Decide which environment variable is used to find dynamic libraries.  
case "${host}" in  
  *-*-hpux*) RPATH_ENVVAR=SHLIB_PATH ;;  
  *-*-darwin*) RPATH_ENVVAR=DYLD_LIBRARY_PATH ;;  
  *-*-mingw* | *-*-cygwin ) RPATH_ENVVAR=PATH ;;  
  *) RPATH_ENVVAR=LD_LIBRARY_PATH ;;  
esac  
  
# On systems where the dynamic library environment variable is PATH,  
# gcc/ will put dynamic libraries into a subdirectory to avoid adding  
# built executables to PATH.  
if test "$RPATH_ENVVAR" = PATH; then  
  GCC_SHLIB_SUBDIR=/shlib  
else  
  GCC_SHLIB_SUBDIR=  
fi  
  
# Adjust the toplevel makefile according to whether bootstrap was selected.  
case $enable_bootstrap in  
  yes)  
    bootstrap_suffix=bootstrap  
    BUILD_CONFIG=bootstrap-debug  
    ;;  
  no)  
    bootstrap_suffix=no-bootstrap  
    BUILD_CONFIG=  
    ;;  
esac  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for default BUILD_CONFIG" >&5  
$as_echo_n "checking for default BUILD_CONFIG... " >&6; }  
  
  
# Check whether --with-build-config was given.  
if test "${with_build_config+set}" = set; then :  
  withval=$with_build_config; case $with_build_config in  
   yes) with_build_config= ;;  
   no) with_build_config= BUILD_CONFIG= ;;  
   esac  
fi  
  
  
if test "x${with_build_config}" != x; then  
  BUILD_CONFIG=$with_build_config  
else  
  case $BUILD_CONFIG in  
  bootstrap-debug)  
    if echo "int f (void) { return 0; }" > conftest.c &&  
       ${CC} -c conftest.c &&  
       mv conftest.o conftest.o.g0 &&  
       ${CC} -c -g conftest.c &&  
       mv conftest.o conftest.o.g &&  
       ${srcdir}/contrib/compare-debug conftest.o.g0 conftest.o.g > /dev/null 2>&1; then  
      :  
    else  
      BUILD_CONFIG=  
    fi  
    rm -f conftest.c conftest.o conftest.o.g0 conftest.o.g  
    ;;  
  esac  
fi  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $BUILD_CONFIG" >&5  
$as_echo "$BUILD_CONFIG" >&6; }  
  
  
# Use same top-level configure hooks in libgcc/libstdc++/libvtv.  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for --enable-vtable-verify" >&5  
$as_echo_n "checking for --enable-vtable-verify... " >&6; }  
# Check whether --enable-vtable-verify was given.  
if test "${enable_vtable_verify+set}" = set; then :  
  enableval=$enable_vtable_verify; case "$enableval" in  
 yes) enable_vtable_verify=yes ;;  
 no)  enable_vtable_verify=no ;;  
 *)   enable_vtable_verify=no;;  
 esac  
else  
  enable_vtable_verify=no  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $enable_vtable_verify" >&5  
$as_echo "$enable_vtable_verify" >&6; }  
  
# Record target_configdirs and the configure arguments for target and  
# build configuration in Makefile.  
target_configdirs=`echo "${target_configdirs}" | sed -e 's/target-//g'`  
build_configdirs=`echo "${build_configdirs}" | sed -e 's/build-//g'`  
bootstrap_fixincludes=no  
  
# If we are building libgomp, bootstrap it.  
if echo " ${target_configdirs} " | grep " libgomp " > /dev/null 2>&1 ; then  
  bootstrap_target_libs=${bootstrap_target_libs}target-libgomp,  
fi  
  
# If we are building libsanitizer and $BUILD_CONFIG contains bootstrap-asan  
# or bootstrap-ubsan, bootstrap it.  
if echo " ${target_configdirs} " | grep " libsanitizer " > /dev/null 2>&1; then  
  case "$BUILD_CONFIG" in  
    *bootstrap-hwasan* | *bootstrap-asan* | *bootstrap-ubsan* )  
      bootstrap_target_libs=${bootstrap_target_libs}target-libsanitizer,  
      bootstrap_fixincludes=yes  
      ;;  
  esac  
fi  
  
# If we are building libvtv and --enable-vtable-verify, bootstrap it.  
if echo " ${target_configdirs} " | grep " libvtv " > /dev/null 2>&1 &&  
   test "$enable_vtable_verify" != no; then  
  bootstrap_target_libs=${bootstrap_target_libs}target-libvtv,  
fi  
  
# If we are building libatomic and the list of enabled languages includes the  
# D frontend, bootstrap it.  
if echo " ${target_configdirs} " | grep " libatomic " > /dev/null 2>&1; then  
  case ,${enable_languages}, in  
    *,d,*)  
      bootstrap_target_libs=${bootstrap_target_libs}target-libatomic,  
      ;;  
  esac  
fi  
  
# Determine whether gdb needs tk/tcl or not.  
# Use 'maybe' since enable_gdbtk might be true even if tk isn't available  
# and in that case we want gdb to be built without tk.  Ugh!  
# In fact I believe gdb is the *only* package directly dependent on tk,  
# so we should be able to put the 'maybe's in unconditionally and  
# leave out the maybe dependencies when enable_gdbtk is false.  I'm not  
# 100% sure that that's safe though.  
  
gdb_tk="maybe-all-tcl maybe-all-tk maybe-all-itcl maybe-all-libgui"  
case "$enable_gdbtk" in  
  no)  
    GDB_TK="" ;;  
  yes)  
    GDB_TK="${gdb_tk}" ;;  
  *)  
    # Only add the dependency on gdbtk when GDBtk is part of the gdb  
    # distro.  Eventually someone will fix this and move Insight, nee  
    # gdbtk to a separate directory.  
    if test -d ${srcdir}/gdb/gdbtk ; then  
      GDB_TK="${gdb_tk}"  
    else  
      GDB_TK=""  
    fi  
    ;;  
esac  
CONFIGURE_GDB_TK=`echo ${GDB_TK} | sed s/-all-/-configure-/g`  
INSTALL_GDB_TK=`echo ${GDB_TK} | sed s/-all-/-install-/g`  
  
# gdb and gdbserver depend on gnulib and gdbsupport, but as nothing  
# else does, only include them if one of these is built.  The Makefile  
# provides the ordering, so it's enough here to add to the list.  
case " ${configdirs} " in  
  *\ gdb\ *)  
    configdirs="${configdirs} gnulib gdbsupport"  
    ;;  
  *\ gdbserver\ *)  
    configdirs="${configdirs} gnulib gdbsupport"  
    ;;  
  *\ sim\ *)  
    configdirs="${configdirs} gnulib"  
    ;;  
esac  
  
# Strip out unwanted targets.  
  
# While at that, we remove Makefiles if we were started for recursive  
# configuration, so that the top-level Makefile reconfigures them,  
# like we used to do when configure itself was recursive.  
  
# Loop over modules.  We used to use the "$extrasub" feature from Autoconf  
# but now we're fixing up the Makefile ourselves with the additional  
# commands passed to AC_CONFIG_FILES.  Use separate variables  
# extrasub-{build,host,target} not because there is any reason to split  
# the substitutions up that way, but only to remain below the limit of  
# 99 commands in a script, for HP-UX sed.  
  
# Do not nest @if/@endif or @unless/@endunless pairs, because  
# configure will not warn you at all.  
  
case "$enable_bootstrap:$ENABLE_GOLD: $configdirs :,$stage1_languages," in  
  yes:yes:*\ gold\ *:*,c++,*) ;;  
  yes:yes:*\ gold\ *:*)  
    as_fn_error $? "in a combined tree, bootstrapping with --enable-gold requires c++ in stage1_languages" "$LINENO" 5  
    ;;  
esac  
  
extrasub_build=  
for module in ${build_configdirs} ; do  
  if test -z "${no_recursion}" \  
     && test -f ${build_subdir}/${module}/Makefile; then  
    echo 1>&2 "*** removing ${build_subdir}/${module}/Makefile to force reconfigure"  
    rm -f ${build_subdir}/${module}/Makefile  
  fi  
  extrasub_build="$extrasub_build  
/^@if build-$module\$/d  
/^@endif build-$module\$/d  
/^@unless build-$module\$/,/^@endunless build-$module\$/d  
/^@if build-$module-$bootstrap_suffix\$/d  
/^@endif build-$module-$bootstrap_suffix\$/d  
/^@unless build-$module-$bootstrap_suffix\$/,/^@endunless build-$module-$bootstrap_suffix\$/d"  
done  
extrasub_host=  
for module in ${configdirs} ; do  
  if test -z "${no_recursion}"; then  
    for file in stage*-${module}/Makefile prev-${module}/Makefile ${module}/Makefile; do  
      if test -f ${file}; then  
    echo 1>&2 "*** removing ${file} to force reconfigure"  
    rm -f ${file}  
      fi  
    done  
  fi  
  case ${module},${bootstrap_fixincludes} in  
    fixincludes,no) host_bootstrap_suffix=no-bootstrap ;;  
    *) host_bootstrap_suffix=$bootstrap_suffix ;;  
  esac  
  extrasub_host="$extrasub_host  
/^@if $module\$/d  
/^@endif $module\$/d  
/^@unless $module\$/,/^@endunless $module\$/d  
/^@if $module-$host_bootstrap_suffix\$/d  
/^@endif $module-$host_bootstrap_suffix\$/d  
/^@unless $module-$host_bootstrap_suffix\$/,/^@endunless $module-$host_bootstrap_suffix\$/d"  
done  
extrasub_target=  
for module in ${target_configdirs} ; do  
  if test -z "${no_recursion}" \  
     && test -f ${target_subdir}/${module}/Makefile; then  
    echo 1>&2 "*** removing ${target_subdir}/${module}/Makefile to force reconfigure"  
    rm -f ${target_subdir}/${module}/Makefile  
  fi  
  
  # We only bootstrap target libraries listed in bootstrap_target_libs.  
  case $bootstrap_target_libs in  
    *,target-$module,*) target_bootstrap_suffix=$bootstrap_suffix ;;  
    *) target_bootstrap_suffix=no-bootstrap ;;  
  esac  
  
  extrasub_target="$extrasub_target  
/^@if target-$module\$/d  
/^@endif target-$module\$/d  
/^@unless target-$module\$/,/^@endunless target-$module\$/d  
/^@if target-$module-$target_bootstrap_suffix\$/d  
/^@endif target-$module-$target_bootstrap_suffix\$/d  
/^@unless target-$module-$target_bootstrap_suffix\$/,/^@endunless target-$module-$target_bootstrap_suffix\$/d"  
done  
  
# Do the final fixup along with target modules.  
extrasub_target="$extrasub_target  
/^@if /,/^@endif /d  
/^@unless /d  
/^@endunless /d"  
  
if test "$enable_pgo_build" != "no"; then  
  extrasub_build="$extrasub_build  
/^@if pgo-build\$/d  
/^@endif pgo-build\$/d"  
fi  
  
# Create the serialization dependencies.  This uses a temporary file.  
  
# Check whether --enable-serial-configure was given.  
if test "${enable_serial_configure+set}" = set; then :  
  enableval=$enable_serial_configure;  
fi  
  
  
case ${enable_serial_configure} in  
  yes)  
    enable_serial_build_configure=yes  
    enable_serial_host_configure=yes  
    enable_serial_target_configure=yes  
    ;;  
esac  
  
# These force 'configure's to be done one at a time, to avoid problems  
# with contention over a shared config.cache.  
rm -f serdep.tmp  
if test "x${enable_serial_build_configure}" = xyes || test "x${enable_serial_host_configure}" = xyes || test "x${enable_serial_target_configure}" = xyes; then  
echo '# serdep.tmp' > serdep.tmp  
fi  
olditem=  
test "x${enable_serial_build_configure}" = xyes &&  
for item in ${build_configdirs} ; do  
  case ${olditem} in  
    "") ;;  
    *) echo "configure-build-${item}: configure-build-${olditem}" >> serdep.tmp ;;  
  esac  
  olditem=${item}  
done  
olditem=  
test "x${enable_serial_host_configure}" = xyes &&  
for item in ${configdirs} ; do  
  case ${olditem} in  
    "") ;;  
    *) echo "configure-${item}: configure-${olditem}" >> serdep.tmp ;;  
  esac  
  olditem=${item}  
done  
olditem=  
test "x${enable_serial_target_configure}" = xyes &&  
for item in ${target_configdirs} ; do  
  case ${olditem} in  
    "") ;;  
    *) echo "configure-target-${item}: configure-target-${olditem}" >> serdep.tmp ;;  
  esac  
  olditem=${item}  
done  
serialization_dependencies=serdep.tmp  
  
  
# Base args.  Strip norecursion, cache-file, srcdir, host, build,  
# target, nonopt, and variable assignments.  These are the ones we  
# might not want to pass down to subconfigures.  The exception being  
# --cache-file=/dev/null, which is used to turn off the use of cache  
# files altogether, and which should be passed on to subconfigures.  
# Also strip program-prefix, program-suffix, and program-transform-name,  
# so that we can pass down a consistent program-transform-name.  
hbaseargs=  
bbaseargs=  
tbaseargs=  
keep_next=no  
skip_next=no  
eval "set -- $ac_configure_args"  
for ac_arg  
do  
  if test X"$skip_next" = X"yes"; then  
    skip_next=no  
    continue  
  fi  
  if test X"$keep_next" = X"yes"; then  
    case $ac_arg in  
      *\'*)  
    ac_arg=`echo "$ac_arg" | sed "s/'/'\\\\\\\\''/g"` ;;  
    esac  
    hbaseargs="$hbaseargs '$ac_arg'"  
    bbaseargs="$bbaseargs '$ac_arg'"  
    tbaseargs="$tbaseargs '$ac_arg'"  
    keep_next=no  
    continue  
  fi  
  
  # Handle separated arguments.  Based on the logic generated by  
  # autoconf 2.59.  
  case $ac_arg in  
    *=* | --config-cache | -C | -disable-* | --disable-* \  
      | -enable-* | --enable-* | -gas | --g* | -nfp | --nf* \  
      | -q | -quiet | --q* | -silent | --sil* | -v | -verb* \  
      | -with-* | --with-* | -without-* | --without-* | --x)  
      separate_arg=no  
      ;;  
    -*)  
      separate_arg=yes  
      ;;  
    *)  
      separate_arg=no  
      ;;  
  esac  
  
  skip_targ=no  
  skip_barg=no  
  case $ac_arg in  
  
  --with-* | --without-*)  
    libopt=`echo "$ac_arg" | sed -e 's,^--[^-_]*[-_],,' -e 's,=.*$,,'`  
  
    case $libopt in  
    *[-_]include)  
      lib=`echo "$libopt" | sed 's,[-_]include$,,'`  
      ;;  
    *[-_]lib)  
      lib=`echo "$libopt" | sed 's,[-_]lib$,,'`  
      ;;  
    *[-_]prefix)  
      lib=`echo "$libopt" | sed 's,[-_]prefix$,,'`  
      ;;  
    *[-_]type)  
      lib=`echo "$libopt" | sed 's,[-_]type$,,'`  
      ;;  
    *)  
      lib=$libopt  
      ;;  
    esac  
  
  
    case $lib in  
    mpc | mpfr | gmp | isl)  
      # If we're processing --with-$lib, --with-$lib-include or  
      # --with-$lib-lib, for one of the libs above, and target is  
      # different from host, don't pass the current argument to any  
      # target library's configure.  
      if test x$is_cross_compiler = xyes; then  
        skip_targ=yes  
      fi  
      ;;  
    libintl | libiconv)  
      # We don't want libintl (and co.) in anything but the host arguments.  
      skip_targ=yes  
      skip_barg=yes  
      ;;  
    esac  
    ;;  
  esac  
  
  case "$ac_arg" in  
    --cache-file=/dev/null | \  
    -cache-file=/dev/null )  
      # Handled here to avoid the test to skip args below.  
      hbaseargs="$hbaseargs '$ac_arg'"  
      bbaseargs="$bbaseargs '$ac_arg'"  
      tbaseargs="$tbaseargs '$ac_arg'"  
      # Assert: $separate_arg should always be no.  
      keep_next=$separate_arg  
      ;;  
    --no*)  
      continue  
      ;;  
    --c* | \  
    --sr* | \  
    --ho* | \  
    --bu* | \  
    --t* | \  
    --program-* | \  
    -cache_file* | \  
    -srcdir* | \  
    -host* | \  
    -build* | \  
    -target* | \  
    -program-prefix* | \  
    -program-suffix* | \  
    -program-transform-name* )  
      skip_next=$separate_arg  
      continue  
      ;;  
    -*)  
      # An option.  Add it.  
      case $ac_arg in  
    *\'*)  
      ac_arg=`echo "$ac_arg" | sed "s/'/'\\\\\\\\''/g"` ;;  
      esac  
      hbaseargs="$hbaseargs '$ac_arg'"  
      if test X"$skip_barg" = Xno; then  
    bbaseargs="$bbaseargs '$ac_arg'"  
      fi  
      if test X"$skip_targ" = Xno; then  
        tbaseargs="$tbaseargs '$ac_arg'"  
      fi  
      keep_next=$separate_arg  
      ;;  
    *)  
      # Either a variable assignment, or a nonopt (triplet).  Don't  
      # pass it down; let the Makefile handle this.  
      continue  
      ;;  
  esac  
done  
# Remove the initial space we just introduced and, as these will be  
# expanded by make, quote '$'.  
hbaseargs=`echo "x$hbaseargs" | sed -e 's/^x *//' -e 's,\\$,$$,g'`  
bbaseargs=`echo "x$bbaseargs" | sed -e 's/^x *//' -e 's,\\$,$$,g'`  
  
# Add in --program-transform-name, after --program-prefix and  
# --program-suffix have been applied to it.  Autoconf has already  
# doubled dollar signs and backslashes in program_transform_name; we want  
# the backslashes un-doubled, and then the entire thing wrapped in single  
# quotes, because this will be expanded first by make and then by the shell.  
# Also, because we want to override the logic in subdir configure scripts to  
# choose program_transform_name, replace any s,x,x, with s,y,y,.  
sed -e "s,\\\\\\\\,\\\\,g; s,','\\\\'',g; s/s,x,x,/s,y,y,/" \<<EOF_SED > conftestsed.out  
${program_transform_name}  
EOF_SED  
gcc_transform_name=`cat conftestsed.out`  
rm -f conftestsed.out  
hbaseargs="$hbaseargs --program-transform-name='${gcc_transform_name}'"  
bbaseargs="$bbaseargs --program-transform-name='${gcc_transform_name}'"  
tbaseargs="$tbaseargs --program-transform-name='${gcc_transform_name}'"  
if test "$silent" = yes; then  
  bbaseargs="$bbaseargs --silent"  
  hbaseargs="$hbaseargs --silent"  
  tbaseargs="$tbaseargs --silent"  
fi  
  
bbaseargs="$bbaseargs --disable-option-checking"  
hbaseargs="$hbaseargs --disable-option-checking"  
tbaseargs="$tbaseargs --disable-option-checking"  
  
if test "$enable_year2038" = no; then  
  baseargs="$baseargs --disable-year2038"  
  tbaseargs="$tbaseargs --disable-year2038"  
fi  
  
# Record and document user additions to sub configure arguments.  
  
  
  
  
# For the build-side libraries, we just need to pretend we're native,  
# and not use the same cache file.  Multilibs are neither needed nor  
# desired.  We can't even use the same cache file for all build-side  
# libraries, as they're compiled differently; some with C, some with  
# C++ or with different feature-enabling options.  
build_configargs="$build_configargs --cache-file=./config.cache ${bbaseargs}"  
  
# For host modules, accept cache file option, or specification as blank.  
case "${cache_file}" in  
"") # empty  
  cache_file_option="" ;;  
/* | [A-Za-z]:[\\/]* ) # absolute path  
  cache_file_option="--cache-file=${cache_file}" ;;  
*) # relative path  
  cache_file_option="--cache-file=../${cache_file}" ;;  
esac  
  
# Host dirs don't like to share a cache file either, horribly enough.  
# This seems to be due to autoconf 2.5x stupidity.  
host_configargs="$host_configargs --cache-file=./config.cache ${extra_host_args} ${hbaseargs}"  
  
target_configargs="$target_configargs ${tbaseargs}"  
  
# Passing a --with-cross-host argument lets the target libraries know  
# whether they are being built with a cross-compiler or being built  
# native.  However, it would be better to use other mechanisms to make the  
# sorts of decisions they want to make on this basis.  Please consider  
# this option to be deprecated.  FIXME.  
if test x${is_cross_compiler} = xyes ; then  
  target_configargs="--with-cross-host=${host_noncanonical} ${target_configargs}"  
fi  
  
# Special user-friendly check for native x86_64-linux build, if  
# multilib is not explicitly enabled.  
case "$target:$have_compiler:$host:$target:$enable_multilib" in  
  x86_64-*linux*:yes:$build:$build:)  
    # Make sure we have a development environment that handles 32-bit  
    dev64=no  
    echo "int main () { return 0; }" > conftest.c  
    ${CC} -m32 -o conftest ${CFLAGS} ${CPPFLAGS} ${LDFLAGS} conftest.c  
    if test $? = 0 ; then  
      if test -s conftest || test -s conftest.exe ; then  
    dev64=yes  
      fi  
    fi  
    rm -f conftest*  
    if test x${dev64} != xyes ; then  
      as_fn_error $? "I suspect your system does not have 32-bit development libraries (libc and headers). If you have them, rerun configure with --enable-multilib. If you do not have them, and want to build a 64-bit-only compiler, rerun configure with --disable-multilib." "$LINENO" 5  
    fi  
    ;;  
esac  
  
# Default to --enable-multilib.  
if test x${enable_multilib} = x ; then  
  target_configargs="--enable-multilib ${target_configargs}"  
fi  
  
# Pass --with-newlib if appropriate.  Note that target_configdirs has  
# changed from the earlier setting of with_newlib.  
if test x${with_newlib} != xno && echo " ${target_configdirs} " | grep " newlib " > /dev/null 2>&1 && test -d ${srcdir}/newlib ; then  
  target_configargs="--with-newlib ${target_configargs}"  
fi  
  
# Different target subdirs use different values of certain variables  
# (notably CXX).  Worse, multilibs use *lots* of different values.  
# Worse yet, autoconf 2.5x makes some of these 'precious', meaning that  
# it doesn't automatically accept command-line overrides of them.  
# This means it's not safe for target subdirs to share a cache file,  
# which is disgusting, but there you have it.  Hopefully this can be  
# fixed in future.  It's still worthwhile to use a cache file for each  
# directory.  I think.  
  
# Pass the appropriate --build, --host, --target and --cache-file arguments.  
# We need to pass --target, as newer autoconf's requires consistency  
# for target_alias and gcc doesn't manage it consistently.  
target_configargs="--cache-file=./config.cache ${target_configargs}"  
  
FLAGS_FOR_TARGET=  
case " $target_configdirs " in  
 *" newlib "*)  
  case " $target_configargs " in  
  *" --with-newlib "*)  
   case "$target" in  
    *-cygwin*)  
      FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -L$$r/$(TARGET_SUBDIR)/winsup/cygwin -isystem $$s/winsup/cygwin/include'  
      ;;  
   esac  
  
   # If we're not building GCC, don't discard standard headers.  
   if test -d ${srcdir}/gcc; then  
     FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -nostdinc'  
  
     if test "${build}" != "${host}"; then  
       # On Canadian crosses, CC_FOR_TARGET will have already been set  
       # by `configure', so we won't have an opportunity to add -Bgcc/  
       # to it.  This is right: we don't want to search that directory  
       # for binaries, but we want the header files in there, so add  
       # them explicitly.  
       FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -isystem $$r/$(HOST_SUBDIR)/gcc/include -isystem $$r/$(HOST_SUBDIR)/gcc/include-fixed'  
  
       # Someone might think of using the pre-installed headers on  
       # Canadian crosses, in case the installed compiler is not fully  
       # compatible with the compiler being built.  In this case, it  
       # would be better to flag an error than risking having  
       # incompatible object files being constructed.  We can't  
       # guarantee that an error will be flagged, but let's hope the  
       # compiler will do it, when presented with incompatible header  
       # files.  
     fi  
   fi  
  
   case "${target}-${is_cross_compiler}" in  
   i[3456789]86-*-linux*-no)  
      # Here host == target, so we don't need to build gcc,  
      # so we don't want to discard standard headers.  
      FLAGS_FOR_TARGET=`echo " $FLAGS_FOR_TARGET " | sed -e 's/ -nostdinc / /'`  
      ;;  
   *)  
      # If we're building newlib, use its generic headers last, but search  
      # for any libc-related directories first (so make it the last -B  
      # switch).  
      FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -B$$r/$(TARGET_SUBDIR)/newlib/ -isystem $$r/$(TARGET_SUBDIR)/newlib/targ-include -isystem $$s/newlib/libc/include'  
  
      # If we're building libgloss, find the startup file, simulator library  
      # and linker script.  
      case " $target_configdirs " in  
    *" libgloss "*)  
    # Look for startup file, simulator library and maybe linker script.  
    FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -B$$r/$(TARGET_SUBDIR)/libgloss/'"$libgloss_dir"  
    # Look for libnosys.a in case the target needs it.  
    FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -L$$r/$(TARGET_SUBDIR)/libgloss/libnosys'  
    # Most targets have the linker script in the source directory.  
    FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -L$$s/libgloss/'"$libgloss_dir"  
    ;;  
      esac  
      ;;  
   esac  
   ;;  
  esac  
  ;;  
esac  
  
case "$target" in  
  x86_64-*mingw* | *-w64-mingw*)  
  # MinGW-w64 does not use newlib, nor does it use winsup. It may,  
  # however, use a symlink named 'mingw' in ${prefix} .  
    FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -L${prefix}/${target}/lib -L${prefix}/mingw/lib -isystem ${prefix}/${target}/include -isystem ${prefix}/mingw/include'  
    ;;  
  *-mingw*)  
  # MinGW can't be handled as Cygwin above since it does not use newlib.  
    FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -L$$r/$(TARGET_SUBDIR)/winsup/mingw -L$$r/$(TARGET_SUBDIR)/winsup/w32api/lib -isystem $$s/winsup/mingw/include -isystem $$s/winsup/w32api/include'  
    ;;  
esac  
  
# Allow the user to override the flags for  
# our build compiler if desired.  
if test x"${build}" = x"${host}" ; then  
  CFLAGS_FOR_BUILD=${CFLAGS_FOR_BUILD-${CFLAGS}}  
  CPPFLAGS_FOR_BUILD=${CPPFLAGS_FOR_BUILD-${CPPFLAGS}}  
  CXXFLAGS_FOR_BUILD=${CXXFLAGS_FOR_BUILD-${CXXFLAGS}}  
  LDFLAGS_FOR_BUILD=${LDFLAGS_FOR_BUILD-${LDFLAGS}}  
fi  
  
# On Canadian crosses, we'll be searching the right directories for  
# the previously-installed cross compiler, so don't bother to add  
# flags for directories within the install tree of the compiler  
# being built; programs in there won't even run.  
if test "${build}" = "${host}" && test -d ${srcdir}/gcc; then  
  # Search for pre-installed headers if nothing else fits.  
  FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -B$(build_tooldir)/bin/ -B$(build_tooldir)/lib/ -isystem $(build_tooldir)/include -isystem $(build_tooldir)/sys-include'  
fi  
  
if test "x${use_gnu_ld}" = x &&  
   echo " ${configdirs} " | grep " ld " > /dev/null ; then  
  # Arrange for us to find uninstalled linker scripts.  
  FLAGS_FOR_TARGET=$FLAGS_FOR_TARGET' -L$$r/$(HOST_SUBDIR)/ld'  
fi  
  
# Search for other target-specific linker scripts and such.  
case "${target}" in  
  mep*)  
    FLAGS_FOR_TARGET="$FLAGS_FOR_TARGET -mlibrary"  
    ;;  
  # The VxWorks support for shared libraries is getting in  
  # incrementally.  Make sure it doesn't get activated implicitly:  
  *vxworks*)  
    if test "${enable_shared-unset}" = unset ; then  
      enable_shared=no  
      # So the build of libraries knows ...  
      target_configargs="${target_configargs} --disable-shared"  
      # So gcc knows ...  
      host_configargs="${host_configargs} --disable-shared"  
    fi  
    ;;  
esac  
  
# Makefile fragments.  
for frag in host_makefile_frag target_makefile_frag alphaieee_frag ospace_frag;  
do  
  eval fragval=\$$frag  
  if test $fragval != /dev/null; then  
    eval $frag=${srcdir}/$fragval  
  fi  
done  
  
  
  
  
  
# Miscellanea: directories, flags, etc.  
  
  
  
  
  
  
  
  
# Build module lists & subconfigure args.  
  
  
  
# Host module lists & subconfigure args.  
  
  
  
  
# Target module lists & subconfigure args.  
  
  
  
# Build tools.  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
# Generate default definitions for YACC, M4, LEX and other programs that run  
# on the build machine.  These are used if the Makefile can't locate these  
# programs in objdir.  
MISSING=`cd $ac_aux_dir && ${PWDCMD-pwd}`/missing  
  
for ac_prog in 'bison -y' byacc yacc  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_YACC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$YACC"; then  
  ac_cv_prog_YACC="$YACC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_YACC="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
YACC=$ac_cv_prog_YACC  
if test -n "$YACC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $YACC" >&5  
$as_echo "$YACC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$YACC" && break  
done  
test -n "$YACC" || YACC="$MISSING bison -y"  
  
case " $build_configdirs " in  
  *" bison "*) YACC='$$r/$(BUILD_SUBDIR)/bison/tests/bison -y' ;;  
esac  
  
for ac_prog in bison  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_BISON+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$BISON"; then  
  ac_cv_prog_BISON="$BISON" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_BISON="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
BISON=$ac_cv_prog_BISON  
if test -n "$BISON"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $BISON" >&5  
$as_echo "$BISON" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$BISON" && break  
done  
test -n "$BISON" || BISON="$MISSING bison"  
  
case " $build_configdirs " in  
  *" bison "*) BISON='$$r/$(BUILD_SUBDIR)/bison/tests/bison' ;;  
esac  
  
for ac_prog in gm4 gnum4 m4  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_M4+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$M4"; then  
  ac_cv_prog_M4="$M4" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_M4="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
M4=$ac_cv_prog_M4  
if test -n "$M4"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $M4" >&5  
$as_echo "$M4" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$M4" && break  
done  
test -n "$M4" || M4="$MISSING m4"  
  
case " $build_configdirs " in  
  *" m4 "*) M4='$$r/$(BUILD_SUBDIR)/m4/m4' ;;  
esac  
  
for ac_prog in flex lex  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LEX+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LEX"; then  
  ac_cv_prog_LEX="$LEX" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LEX="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LEX=$ac_cv_prog_LEX  
if test -n "$LEX"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LEX" >&5  
$as_echo "$LEX" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$LEX" && break  
done  
test -n "$LEX" || LEX="$MISSING flex"  
  
case " $build_configdirs " in  
  *" flex "*) LEX='$$r/$(BUILD_SUBDIR)/flex/flex' ;;  
  *" lex "*) LEX='$$r/$(BUILD_SUBDIR)/lex/lex' ;;  
esac  
  
for ac_prog in flex  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_FLEX+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$FLEX"; then  
  ac_cv_prog_FLEX="$FLEX" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_FLEX="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
FLEX=$ac_cv_prog_FLEX  
if test -n "$FLEX"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $FLEX" >&5  
$as_echo "$FLEX" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$FLEX" && break  
done  
test -n "$FLEX" || FLEX="$MISSING flex"  
  
case " $build_configdirs " in  
  *" flex "*) FLEX='$$r/$(BUILD_SUBDIR)/flex/flex' ;;  
esac  
  
for ac_prog in makeinfo  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_MAKEINFO+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$MAKEINFO"; then  
  ac_cv_prog_MAKEINFO="$MAKEINFO" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_MAKEINFO="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
MAKEINFO=$ac_cv_prog_MAKEINFO  
if test -n "$MAKEINFO"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $MAKEINFO" >&5  
$as_echo "$MAKEINFO" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$MAKEINFO" && break  
done  
test -n "$MAKEINFO" || MAKEINFO="$MISSING makeinfo"  
  
case " $build_configdirs " in  
  *" texinfo "*) MAKEINFO='$$r/$(BUILD_SUBDIR)/texinfo/makeinfo/makeinfo' ;;  
  *)  
  
    # For an installed makeinfo, we require it to be from texinfo 4.7 or  
    # higher, else we use the "missing" dummy.  
    if ${MAKEINFO} --version \  
       | egrep 'texinfo[^0-9]*(4\.([7-9]|[1-9][0-9])|[5-9]|[1-9][0-9])' >/dev/null 2>&1; then  
      :  
    else  
      MAKEINFO="$MISSING makeinfo"  
    fi  
    ;;  
  
esac  
  
# FIXME: expect and dejagnu may become build tools?  
  
for ac_prog in expect  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_EXPECT+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$EXPECT"; then  
  ac_cv_prog_EXPECT="$EXPECT" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_EXPECT="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
EXPECT=$ac_cv_prog_EXPECT  
if test -n "$EXPECT"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $EXPECT" >&5  
$as_echo "$EXPECT" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$EXPECT" && break  
done  
test -n "$EXPECT" || EXPECT="expect"  
  
case " $configdirs " in  
  *" expect "*)  
    test $host = $build && EXPECT='$$r/$(HOST_SUBDIR)/expect/expect'  
    ;;  
esac  
  
for ac_prog in runtest  
do  
  # Extract the first word of "$ac_prog", so it can be a program name with args.  
set dummy $ac_prog; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RUNTEST+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RUNTEST"; then  
  ac_cv_prog_RUNTEST="$RUNTEST" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RUNTEST="$ac_prog"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RUNTEST=$ac_cv_prog_RUNTEST  
if test -n "$RUNTEST"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RUNTEST" >&5  
$as_echo "$RUNTEST" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  test -n "$RUNTEST" && break  
done  
test -n "$RUNTEST" || RUNTEST="runtest"  
  
case " $configdirs " in  
  *" dejagnu "*)  
    test $host = $build && RUNTEST='$$s/$(HOST_SUBDIR)/dejagnu/runtest'  
    ;;  
esac  
  
  
# Host tools.  
ncn_tool_prefix=  
test -n "$host_alias" && ncn_tool_prefix=$host_alias-  
ncn_target_tool_prefix=  
test -n "$target_alias" && ncn_target_tool_prefix=$target_alias-  
  
  
  
if test -n "$AR"; then  
  ac_cv_prog_AR=$AR  
elif test -n "$ac_cv_prog_AR"; then  
  AR=$ac_cv_prog_AR  
fi  
  
if test -n "$ac_cv_prog_AR"; then  
  for ncn_progname in ar; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR"; then  
  ac_cv_prog_AR="$AR" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR=$ac_cv_prog_AR  
if test -n "$AR"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR" >&5  
$as_echo "$AR" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in ar; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR"; then  
  ac_cv_prog_AR="$AR" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR=$ac_cv_prog_AR  
if test -n "$AR"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR" >&5  
$as_echo "$AR" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_AR" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR"; then  
  ac_cv_prog_AR="$AR" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR=$ac_cv_prog_AR  
if test -n "$AR"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR" >&5  
$as_echo "$AR" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_AR" && break  
done  
  
if test -z "$ac_cv_prog_AR" ; then  
  set dummy ar  
  if test $build = $host ; then  
    AR="$2"  
  else  
    AR="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$AS"; then  
  ac_cv_prog_AS=$AS  
elif test -n "$ac_cv_prog_AS"; then  
  AS=$ac_cv_prog_AS  
fi  
  
if test -n "$ac_cv_prog_AS"; then  
  for ncn_progname in as; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AS+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AS"; then  
  ac_cv_prog_AS="$AS" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AS="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AS=$ac_cv_prog_AS  
if test -n "$AS"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS" >&5  
$as_echo "$AS" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in as; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AS+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AS"; then  
  ac_cv_prog_AS="$AS" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AS="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AS=$ac_cv_prog_AS  
if test -n "$AS"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS" >&5  
$as_echo "$AS" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_AS" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AS+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AS"; then  
  ac_cv_prog_AS="$AS" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AS="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AS=$ac_cv_prog_AS  
if test -n "$AS"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS" >&5  
$as_echo "$AS" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_AS" && break  
done  
  
if test -z "$ac_cv_prog_AS" ; then  
  set dummy as  
  if test $build = $host ; then  
    AS="$2"  
  else  
    AS="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$DLLTOOL"; then  
  ac_cv_prog_DLLTOOL=$DLLTOOL  
elif test -n "$ac_cv_prog_DLLTOOL"; then  
  DLLTOOL=$ac_cv_prog_DLLTOOL  
fi  
  
if test -n "$ac_cv_prog_DLLTOOL"; then  
  for ncn_progname in dlltool; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DLLTOOL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DLLTOOL"; then  
  ac_cv_prog_DLLTOOL="$DLLTOOL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DLLTOOL="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DLLTOOL=$ac_cv_prog_DLLTOOL  
if test -n "$DLLTOOL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL" >&5  
$as_echo "$DLLTOOL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in dlltool; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DLLTOOL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DLLTOOL"; then  
  ac_cv_prog_DLLTOOL="$DLLTOOL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DLLTOOL="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DLLTOOL=$ac_cv_prog_DLLTOOL  
if test -n "$DLLTOOL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL" >&5  
$as_echo "$DLLTOOL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_DLLTOOL" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DLLTOOL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DLLTOOL"; then  
  ac_cv_prog_DLLTOOL="$DLLTOOL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DLLTOOL="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DLLTOOL=$ac_cv_prog_DLLTOOL  
if test -n "$DLLTOOL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL" >&5  
$as_echo "$DLLTOOL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_DLLTOOL" && break  
done  
  
if test -z "$ac_cv_prog_DLLTOOL" ; then  
  set dummy dlltool  
  if test $build = $host ; then  
    DLLTOOL="$2"  
  else  
    DLLTOOL="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$DSYMUTIL"; then  
  ac_cv_prog_DSYMUTIL=$DSYMUTIL  
elif test -n "$ac_cv_prog_DSYMUTIL"; then  
  DSYMUTIL=$ac_cv_prog_DSYMUTIL  
fi  
  
if test -n "$ac_cv_prog_DSYMUTIL"; then  
  for ncn_progname in dsymutil; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DSYMUTIL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DSYMUTIL"; then  
  ac_cv_prog_DSYMUTIL="$DSYMUTIL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DSYMUTIL="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DSYMUTIL=$ac_cv_prog_DSYMUTIL  
if test -n "$DSYMUTIL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL" >&5  
$as_echo "$DSYMUTIL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in dsymutil; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DSYMUTIL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DSYMUTIL"; then  
  ac_cv_prog_DSYMUTIL="$DSYMUTIL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DSYMUTIL="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DSYMUTIL=$ac_cv_prog_DSYMUTIL  
if test -n "$DSYMUTIL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL" >&5  
$as_echo "$DSYMUTIL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_DSYMUTIL" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DSYMUTIL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DSYMUTIL"; then  
  ac_cv_prog_DSYMUTIL="$DSYMUTIL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DSYMUTIL="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DSYMUTIL=$ac_cv_prog_DSYMUTIL  
if test -n "$DSYMUTIL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL" >&5  
$as_echo "$DSYMUTIL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_DSYMUTIL" && break  
done  
  
if test -z "$ac_cv_prog_DSYMUTIL" ; then  
  set dummy dsymutil  
  if test $build = $host ; then  
    DSYMUTIL="$2"  
  else  
    DSYMUTIL="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$LD"; then  
  ac_cv_prog_LD=$LD  
elif test -n "$ac_cv_prog_LD"; then  
  LD=$ac_cv_prog_LD  
fi  
  
if test -n "$ac_cv_prog_LD"; then  
  for ncn_progname in ld; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LD+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LD"; then  
  ac_cv_prog_LD="$LD" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LD="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LD=$ac_cv_prog_LD  
if test -n "$LD"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD" >&5  
$as_echo "$LD" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in ld; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LD+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LD"; then  
  ac_cv_prog_LD="$LD" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LD="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LD=$ac_cv_prog_LD  
if test -n "$LD"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD" >&5  
$as_echo "$LD" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_LD" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LD+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LD"; then  
  ac_cv_prog_LD="$LD" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LD="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LD=$ac_cv_prog_LD  
if test -n "$LD"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD" >&5  
$as_echo "$LD" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_LD" && break  
done  
  
if test -z "$ac_cv_prog_LD" ; then  
  set dummy ld  
  if test $build = $host ; then  
    LD="$2"  
  else  
    LD="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$LIPO"; then  
  ac_cv_prog_LIPO=$LIPO  
elif test -n "$ac_cv_prog_LIPO"; then  
  LIPO=$ac_cv_prog_LIPO  
fi  
  
if test -n "$ac_cv_prog_LIPO"; then  
  for ncn_progname in lipo; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LIPO+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LIPO"; then  
  ac_cv_prog_LIPO="$LIPO" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LIPO="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LIPO=$ac_cv_prog_LIPO  
if test -n "$LIPO"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO" >&5  
$as_echo "$LIPO" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in lipo; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LIPO+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LIPO"; then  
  ac_cv_prog_LIPO="$LIPO" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LIPO="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LIPO=$ac_cv_prog_LIPO  
if test -n "$LIPO"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO" >&5  
$as_echo "$LIPO" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_LIPO" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LIPO+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LIPO"; then  
  ac_cv_prog_LIPO="$LIPO" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LIPO="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LIPO=$ac_cv_prog_LIPO  
if test -n "$LIPO"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO" >&5  
$as_echo "$LIPO" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_LIPO" && break  
done  
  
if test -z "$ac_cv_prog_LIPO" ; then  
  set dummy lipo  
  if test $build = $host ; then  
    LIPO="$2"  
  else  
    LIPO="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$NM"; then  
  ac_cv_prog_NM=$NM  
elif test -n "$ac_cv_prog_NM"; then  
  NM=$ac_cv_prog_NM  
fi  
  
if test -n "$ac_cv_prog_NM"; then  
  for ncn_progname in nm; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_NM+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$NM"; then  
  ac_cv_prog_NM="$NM" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_NM="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
NM=$ac_cv_prog_NM  
if test -n "$NM"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM" >&5  
$as_echo "$NM" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in nm; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_NM+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$NM"; then  
  ac_cv_prog_NM="$NM" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_NM="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
NM=$ac_cv_prog_NM  
if test -n "$NM"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM" >&5  
$as_echo "$NM" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_NM" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_NM+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$NM"; then  
  ac_cv_prog_NM="$NM" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_NM="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
NM=$ac_cv_prog_NM  
if test -n "$NM"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM" >&5  
$as_echo "$NM" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_NM" && break  
done  
  
if test -z "$ac_cv_prog_NM" ; then  
  set dummy nm  
  if test $build = $host ; then  
    NM="$2"  
  else  
    NM="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$RANLIB"; then  
  ac_cv_prog_RANLIB=$RANLIB  
elif test -n "$ac_cv_prog_RANLIB"; then  
  RANLIB=$ac_cv_prog_RANLIB  
fi  
  
if test -n "$ac_cv_prog_RANLIB"; then  
  for ncn_progname in ranlib; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RANLIB+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RANLIB"; then  
  ac_cv_prog_RANLIB="$RANLIB" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RANLIB="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RANLIB=$ac_cv_prog_RANLIB  
if test -n "$RANLIB"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB" >&5  
$as_echo "$RANLIB" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in ranlib; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RANLIB+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RANLIB"; then  
  ac_cv_prog_RANLIB="$RANLIB" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RANLIB="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RANLIB=$ac_cv_prog_RANLIB  
if test -n "$RANLIB"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB" >&5  
$as_echo "$RANLIB" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_RANLIB" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RANLIB+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RANLIB"; then  
  ac_cv_prog_RANLIB="$RANLIB" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RANLIB="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RANLIB=$ac_cv_prog_RANLIB  
if test -n "$RANLIB"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB" >&5  
$as_echo "$RANLIB" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_RANLIB" && break  
done  
  
if test -z "$ac_cv_prog_RANLIB" ; then  
  RANLIB="true"  
fi  
  
  
  
if test -n "$STRIP"; then  
  ac_cv_prog_STRIP=$STRIP  
elif test -n "$ac_cv_prog_STRIP"; then  
  STRIP=$ac_cv_prog_STRIP  
fi  
  
if test -n "$ac_cv_prog_STRIP"; then  
  for ncn_progname in strip; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_STRIP+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$STRIP"; then  
  ac_cv_prog_STRIP="$STRIP" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_STRIP="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
STRIP=$ac_cv_prog_STRIP  
if test -n "$STRIP"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP" >&5  
$as_echo "$STRIP" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in strip; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_STRIP+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$STRIP"; then  
  ac_cv_prog_STRIP="$STRIP" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_STRIP="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
STRIP=$ac_cv_prog_STRIP  
if test -n "$STRIP"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP" >&5  
$as_echo "$STRIP" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_STRIP" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_STRIP+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$STRIP"; then  
  ac_cv_prog_STRIP="$STRIP" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_STRIP="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
STRIP=$ac_cv_prog_STRIP  
if test -n "$STRIP"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP" >&5  
$as_echo "$STRIP" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_STRIP" && break  
done  
  
if test -z "$ac_cv_prog_STRIP" ; then  
  STRIP="true"  
fi  
  
  
  
if test -n "$WINDRES"; then  
  ac_cv_prog_WINDRES=$WINDRES  
elif test -n "$ac_cv_prog_WINDRES"; then  
  WINDRES=$ac_cv_prog_WINDRES  
fi  
  
if test -n "$ac_cv_prog_WINDRES"; then  
  for ncn_progname in windres; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDRES+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDRES"; then  
  ac_cv_prog_WINDRES="$WINDRES" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDRES="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDRES=$ac_cv_prog_WINDRES  
if test -n "$WINDRES"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES" >&5  
$as_echo "$WINDRES" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in windres; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDRES+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDRES"; then  
  ac_cv_prog_WINDRES="$WINDRES" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDRES="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDRES=$ac_cv_prog_WINDRES  
if test -n "$WINDRES"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES" >&5  
$as_echo "$WINDRES" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_WINDRES" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDRES+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDRES"; then  
  ac_cv_prog_WINDRES="$WINDRES" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDRES="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDRES=$ac_cv_prog_WINDRES  
if test -n "$WINDRES"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES" >&5  
$as_echo "$WINDRES" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_WINDRES" && break  
done  
  
if test -z "$ac_cv_prog_WINDRES" ; then  
  set dummy windres  
  if test $build = $host ; then  
    WINDRES="$2"  
  else  
    WINDRES="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$WINDMC"; then  
  ac_cv_prog_WINDMC=$WINDMC  
elif test -n "$ac_cv_prog_WINDMC"; then  
  WINDMC=$ac_cv_prog_WINDMC  
fi  
  
if test -n "$ac_cv_prog_WINDMC"; then  
  for ncn_progname in windmc; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDMC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDMC"; then  
  ac_cv_prog_WINDMC="$WINDMC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDMC="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDMC=$ac_cv_prog_WINDMC  
if test -n "$WINDMC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC" >&5  
$as_echo "$WINDMC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in windmc; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDMC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDMC"; then  
  ac_cv_prog_WINDMC="$WINDMC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDMC="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDMC=$ac_cv_prog_WINDMC  
if test -n "$WINDMC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC" >&5  
$as_echo "$WINDMC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_WINDMC" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDMC+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDMC"; then  
  ac_cv_prog_WINDMC="$WINDMC" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDMC="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDMC=$ac_cv_prog_WINDMC  
if test -n "$WINDMC"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC" >&5  
$as_echo "$WINDMC" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_WINDMC" && break  
done  
  
if test -z "$ac_cv_prog_WINDMC" ; then  
  set dummy windmc  
  if test $build = $host ; then  
    WINDMC="$2"  
  else  
    WINDMC="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$OBJCOPY"; then  
  ac_cv_prog_OBJCOPY=$OBJCOPY  
elif test -n "$ac_cv_prog_OBJCOPY"; then  
  OBJCOPY=$ac_cv_prog_OBJCOPY  
fi  
  
if test -n "$ac_cv_prog_OBJCOPY"; then  
  for ncn_progname in objcopy; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJCOPY+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJCOPY"; then  
  ac_cv_prog_OBJCOPY="$OBJCOPY" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJCOPY="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJCOPY=$ac_cv_prog_OBJCOPY  
if test -n "$OBJCOPY"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY" >&5  
$as_echo "$OBJCOPY" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in objcopy; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJCOPY+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJCOPY"; then  
  ac_cv_prog_OBJCOPY="$OBJCOPY" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJCOPY="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJCOPY=$ac_cv_prog_OBJCOPY  
if test -n "$OBJCOPY"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY" >&5  
$as_echo "$OBJCOPY" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_OBJCOPY" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJCOPY+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJCOPY"; then  
  ac_cv_prog_OBJCOPY="$OBJCOPY" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJCOPY="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJCOPY=$ac_cv_prog_OBJCOPY  
if test -n "$OBJCOPY"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY" >&5  
$as_echo "$OBJCOPY" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_OBJCOPY" && break  
done  
  
if test -z "$ac_cv_prog_OBJCOPY" ; then  
  set dummy objcopy  
  if test $build = $host ; then  
    OBJCOPY="$2"  
  else  
    OBJCOPY="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$OBJDUMP"; then  
  ac_cv_prog_OBJDUMP=$OBJDUMP  
elif test -n "$ac_cv_prog_OBJDUMP"; then  
  OBJDUMP=$ac_cv_prog_OBJDUMP  
fi  
  
if test -n "$ac_cv_prog_OBJDUMP"; then  
  for ncn_progname in objdump; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJDUMP+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJDUMP"; then  
  ac_cv_prog_OBJDUMP="$OBJDUMP" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJDUMP="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJDUMP=$ac_cv_prog_OBJDUMP  
if test -n "$OBJDUMP"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP" >&5  
$as_echo "$OBJDUMP" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in objdump; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJDUMP+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJDUMP"; then  
  ac_cv_prog_OBJDUMP="$OBJDUMP" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJDUMP="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJDUMP=$ac_cv_prog_OBJDUMP  
if test -n "$OBJDUMP"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP" >&5  
$as_echo "$OBJDUMP" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_OBJDUMP" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJDUMP+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJDUMP"; then  
  ac_cv_prog_OBJDUMP="$OBJDUMP" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJDUMP="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJDUMP=$ac_cv_prog_OBJDUMP  
if test -n "$OBJDUMP"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP" >&5  
$as_echo "$OBJDUMP" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_OBJDUMP" && break  
done  
  
if test -z "$ac_cv_prog_OBJDUMP" ; then  
  set dummy objdump  
  if test $build = $host ; then  
    OBJDUMP="$2"  
  else  
    OBJDUMP="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$OTOOL"; then  
  ac_cv_prog_OTOOL=$OTOOL  
elif test -n "$ac_cv_prog_OTOOL"; then  
  OTOOL=$ac_cv_prog_OTOOL  
fi  
  
if test -n "$ac_cv_prog_OTOOL"; then  
  for ncn_progname in otool; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OTOOL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OTOOL"; then  
  ac_cv_prog_OTOOL="$OTOOL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OTOOL="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OTOOL=$ac_cv_prog_OTOOL  
if test -n "$OTOOL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL" >&5  
$as_echo "$OTOOL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in otool; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OTOOL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OTOOL"; then  
  ac_cv_prog_OTOOL="$OTOOL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OTOOL="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OTOOL=$ac_cv_prog_OTOOL  
if test -n "$OTOOL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL" >&5  
$as_echo "$OTOOL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_OTOOL" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OTOOL+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OTOOL"; then  
  ac_cv_prog_OTOOL="$OTOOL" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OTOOL="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OTOOL=$ac_cv_prog_OTOOL  
if test -n "$OTOOL"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL" >&5  
$as_echo "$OTOOL" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_OTOOL" && break  
done  
  
if test -z "$ac_cv_prog_OTOOL" ; then  
  set dummy otool  
  if test $build = $host ; then  
    OTOOL="$2"  
  else  
    OTOOL="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
if test -n "$READELF"; then  
  ac_cv_prog_READELF=$READELF  
elif test -n "$ac_cv_prog_READELF"; then  
  READELF=$ac_cv_prog_READELF  
fi  
  
if test -n "$ac_cv_prog_READELF"; then  
  for ncn_progname in readelf; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_READELF+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$READELF"; then  
  ac_cv_prog_READELF="$READELF" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_READELF="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
READELF=$ac_cv_prog_READELF  
if test -n "$READELF"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF" >&5  
$as_echo "$READELF" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
for ncn_progname in readelf; do  
  if test -n "$ncn_tool_prefix"; then  
    # Extract the first word of "${ncn_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_READELF+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$READELF"; then  
  ac_cv_prog_READELF="$READELF" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_READELF="${ncn_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
READELF=$ac_cv_prog_READELF  
if test -n "$READELF"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF" >&5  
$as_echo "$READELF" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  if test -z "$ac_cv_prog_READELF" && test $build = $host ; then  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_READELF+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$READELF"; then  
  ac_cv_prog_READELF="$READELF" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_READELF="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
READELF=$ac_cv_prog_READELF  
if test -n "$READELF"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF" >&5  
$as_echo "$READELF" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  fi  
  test -n "$ac_cv_prog_READELF" && break  
done  
  
if test -z "$ac_cv_prog_READELF" ; then  
  set dummy readelf  
  if test $build = $host ; then  
    READELF="$2"  
  else  
    READELF="${ncn_tool_prefix}$2"  
  fi  
fi  
  
  
  
  
  
  
  
GDCFLAGS=${GDCFLAGS-${CFLAGS}}  
  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for -plugin option" >&5  
$as_echo_n "checking for -plugin option... " >&6; }  
  
plugin_names="liblto_plugin.so liblto_plugin-0.dll cyglto_plugin-0.dll"  
plugin_option=  
for plugin in $plugin_names; do  
  plugin_so=`${CC} ${CFLAGS} --print-prog-name $plugin`  
  if test x$plugin_so = x$plugin; then  
    plugin_so=`${CC} ${CFLAGS} --print-file-name $plugin`  
  fi  
  if test x$plugin_so != x$plugin; then  
    plugin_option="--plugin $plugin_so"  
    break  
  fi  
done  
if test -n "$ac_tool_prefix"; then  
  # Extract the first word of "${ac_tool_prefix}ar", so it can be a program name with args.  
set dummy ${ac_tool_prefix}ar; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR"; then  
  ac_cv_prog_AR="$AR" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR="${ac_tool_prefix}ar"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR=$ac_cv_prog_AR  
if test -n "$AR"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR" >&5  
$as_echo "$AR" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_prog_AR"; then  
  ac_ct_AR=$AR  
  # Extract the first word of "ar", so it can be a program name with args.  
set dummy ar; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_ac_ct_AR+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$ac_ct_AR"; then  
  ac_cv_prog_ac_ct_AR="$ac_ct_AR" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_ac_ct_AR="ar"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
ac_ct_AR=$ac_cv_prog_ac_ct_AR  
if test -n "$ac_ct_AR"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_ct_AR" >&5  
$as_echo "$ac_ct_AR" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  if test "x$ac_ct_AR" = x; then  
    AR=""  
  else  
    case $cross_compiling:$ac_tool_warned in  
yes:)  
{ $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: using cross tools not prefixed with host triplet" >&5  
$as_echo "$as_me: WARNING: using cross tools not prefixed with host triplet" >&2;}  
ac_tool_warned=yes ;;  
esac  
    AR=$ac_ct_AR  
  fi  
else  
  AR="$ac_cv_prog_AR"  
fi  
  
if test "${AR}" = "" ; then  
  as_fn_error $? "Required archive tool 'ar' not found on PATH." "$LINENO" 5  
fi  
touch conftest.c  
${AR} $plugin_option rc conftest.a conftest.c  
if test "$?" != 0; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: Failed: $AR $plugin_option rc" >&5  
$as_echo "$as_me: WARNING: Failed: $AR $plugin_option rc" >&2;}  
  plugin_option=  
fi  
rm -f conftest.*  
if test -n "$plugin_option"; then  
  PLUGIN_OPTION="$plugin_option"  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $plugin_option" >&5  
$as_echo "$plugin_option" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
AR_PLUGIN_OPTION=  
RANLIB_PLUGIN_OPTION=  
if test -n "$PLUGIN_OPTION"; then  
  if $AR --help 2>&1 | grep -q "\--plugin"; then  
    AR_PLUGIN_OPTION="$PLUGIN_OPTION"  
  fi  
  if $RANLIB --help 2>&1 | grep -q "\--plugin"; then  
    RANLIB_PLUGIN_OPTION="$PLUGIN_OPTION"  
  fi  
fi  
  
  
  
# Target tools.  
  
# Check whether --with-build-time-tools was given.  
if test "${with_build_time_tools+set}" = set; then :  
  withval=$with_build_time_tools; case x"$withval" in  
     x/*) ;;  
     *)  
       with_build_time_tools=  
       { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: argument to --with-build-time-tools must be an absolute path" >&5  
$as_echo "$as_me: WARNING: argument to --with-build-time-tools must be an absolute path" >&2;}  
       ;;  
   esac  
else  
  with_build_time_tools=  
fi  
  
  
  
  
if test -n "$CC_FOR_TARGET"; then  
  ac_cv_prog_CC_FOR_TARGET=$CC_FOR_TARGET  
elif test -n "$ac_cv_prog_CC_FOR_TARGET"; then  
  CC_FOR_TARGET=$ac_cv_prog_CC_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_CC_FOR_TARGET"; then  
  for ncn_progname in cc gcc; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC_FOR_TARGET"; then  
  ac_cv_prog_CC_FOR_TARGET="$CC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CC_FOR_TARGET=$ac_cv_prog_CC_FOR_TARGET  
if test -n "$CC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC_FOR_TARGET" >&5  
$as_echo "$CC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_CC_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in cc gcc; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_CC_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_CC_FOR_TARGET"; then  
  for ncn_progname in cc gcc; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC_FOR_TARGET"; then  
  ac_cv_prog_CC_FOR_TARGET="$CC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CC_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CC_FOR_TARGET=$ac_cv_prog_CC_FOR_TARGET  
if test -n "$CC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC_FOR_TARGET" >&5  
$as_echo "$CC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_CC_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CC_FOR_TARGET"; then  
  ac_cv_prog_CC_FOR_TARGET="$CC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CC_FOR_TARGET=$ac_cv_prog_CC_FOR_TARGET  
if test -n "$CC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CC_FOR_TARGET" >&5  
$as_echo "$CC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_CC_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_CC_FOR_TARGET" ; then  
  set dummy cc gcc  
  if test $build = $target ; then  
    CC_FOR_TARGET="$2"  
  else  
    CC_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  CC_FOR_TARGET="$ac_cv_prog_CC_FOR_TARGET"  
fi  
  
  
  
if test -n "$CXX_FOR_TARGET"; then  
  ac_cv_prog_CXX_FOR_TARGET=$CXX_FOR_TARGET  
elif test -n "$ac_cv_prog_CXX_FOR_TARGET"; then  
  CXX_FOR_TARGET=$ac_cv_prog_CXX_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_CXX_FOR_TARGET"; then  
  for ncn_progname in c++ g++ cxx gxx; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CXX_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CXX_FOR_TARGET"; then  
  ac_cv_prog_CXX_FOR_TARGET="$CXX_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CXX_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CXX_FOR_TARGET=$ac_cv_prog_CXX_FOR_TARGET  
if test -n "$CXX_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CXX_FOR_TARGET" >&5  
$as_echo "$CXX_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_CXX_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in c++ g++ cxx gxx; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_CXX_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_CXX_FOR_TARGET"; then  
  for ncn_progname in c++ g++ cxx gxx; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CXX_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CXX_FOR_TARGET"; then  
  ac_cv_prog_CXX_FOR_TARGET="$CXX_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CXX_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CXX_FOR_TARGET=$ac_cv_prog_CXX_FOR_TARGET  
if test -n "$CXX_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CXX_FOR_TARGET" >&5  
$as_echo "$CXX_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_CXX_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_CXX_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$CXX_FOR_TARGET"; then  
  ac_cv_prog_CXX_FOR_TARGET="$CXX_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_CXX_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
CXX_FOR_TARGET=$ac_cv_prog_CXX_FOR_TARGET  
if test -n "$CXX_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $CXX_FOR_TARGET" >&5  
$as_echo "$CXX_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_CXX_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_CXX_FOR_TARGET" ; then  
  set dummy c++ g++ cxx gxx  
  if test $build = $target ; then  
    CXX_FOR_TARGET="$2"  
  else  
    CXX_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  CXX_FOR_TARGET="$ac_cv_prog_CXX_FOR_TARGET"  
fi  
  
  
  
if test -n "$GCC_FOR_TARGET"; then  
  ac_cv_prog_GCC_FOR_TARGET=$GCC_FOR_TARGET  
elif test -n "$ac_cv_prog_GCC_FOR_TARGET"; then  
  GCC_FOR_TARGET=$ac_cv_prog_GCC_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_GCC_FOR_TARGET"; then  
  for ncn_progname in gcc; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GCC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GCC_FOR_TARGET"; then  
  ac_cv_prog_GCC_FOR_TARGET="$GCC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GCC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GCC_FOR_TARGET=$ac_cv_prog_GCC_FOR_TARGET  
if test -n "$GCC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GCC_FOR_TARGET" >&5  
$as_echo "$GCC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_GCC_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in gcc; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_GCC_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_GCC_FOR_TARGET"; then  
  for ncn_progname in gcc; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GCC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GCC_FOR_TARGET"; then  
  ac_cv_prog_GCC_FOR_TARGET="$GCC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GCC_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GCC_FOR_TARGET=$ac_cv_prog_GCC_FOR_TARGET  
if test -n "$GCC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GCC_FOR_TARGET" >&5  
$as_echo "$GCC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_GCC_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GCC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GCC_FOR_TARGET"; then  
  ac_cv_prog_GCC_FOR_TARGET="$GCC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GCC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GCC_FOR_TARGET=$ac_cv_prog_GCC_FOR_TARGET  
if test -n "$GCC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GCC_FOR_TARGET" >&5  
$as_echo "$GCC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_GCC_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_GCC_FOR_TARGET" ; then  
  GCC_FOR_TARGET="${CC_FOR_TARGET}"  
else  
  GCC_FOR_TARGET="$ac_cv_prog_GCC_FOR_TARGET"  
fi  
  
  
  
if test -n "$GFORTRAN_FOR_TARGET"; then  
  ac_cv_prog_GFORTRAN_FOR_TARGET=$GFORTRAN_FOR_TARGET  
elif test -n "$ac_cv_prog_GFORTRAN_FOR_TARGET"; then  
  GFORTRAN_FOR_TARGET=$ac_cv_prog_GFORTRAN_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_GFORTRAN_FOR_TARGET"; then  
  for ncn_progname in gfortran; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GFORTRAN_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GFORTRAN_FOR_TARGET"; then  
  ac_cv_prog_GFORTRAN_FOR_TARGET="$GFORTRAN_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GFORTRAN_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GFORTRAN_FOR_TARGET=$ac_cv_prog_GFORTRAN_FOR_TARGET  
if test -n "$GFORTRAN_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GFORTRAN_FOR_TARGET" >&5  
$as_echo "$GFORTRAN_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_GFORTRAN_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in gfortran; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_GFORTRAN_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_GFORTRAN_FOR_TARGET"; then  
  for ncn_progname in gfortran; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GFORTRAN_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GFORTRAN_FOR_TARGET"; then  
  ac_cv_prog_GFORTRAN_FOR_TARGET="$GFORTRAN_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GFORTRAN_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GFORTRAN_FOR_TARGET=$ac_cv_prog_GFORTRAN_FOR_TARGET  
if test -n "$GFORTRAN_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GFORTRAN_FOR_TARGET" >&5  
$as_echo "$GFORTRAN_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_GFORTRAN_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GFORTRAN_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GFORTRAN_FOR_TARGET"; then  
  ac_cv_prog_GFORTRAN_FOR_TARGET="$GFORTRAN_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GFORTRAN_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GFORTRAN_FOR_TARGET=$ac_cv_prog_GFORTRAN_FOR_TARGET  
if test -n "$GFORTRAN_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GFORTRAN_FOR_TARGET" >&5  
$as_echo "$GFORTRAN_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_GFORTRAN_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_GFORTRAN_FOR_TARGET" ; then  
  set dummy gfortran  
  if test $build = $target ; then  
    GFORTRAN_FOR_TARGET="$2"  
  else  
    GFORTRAN_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  GFORTRAN_FOR_TARGET="$ac_cv_prog_GFORTRAN_FOR_TARGET"  
fi  
  
  
  
if test -n "$GOC_FOR_TARGET"; then  
  ac_cv_prog_GOC_FOR_TARGET=$GOC_FOR_TARGET  
elif test -n "$ac_cv_prog_GOC_FOR_TARGET"; then  
  GOC_FOR_TARGET=$ac_cv_prog_GOC_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_GOC_FOR_TARGET"; then  
  for ncn_progname in gccgo; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GOC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GOC_FOR_TARGET"; then  
  ac_cv_prog_GOC_FOR_TARGET="$GOC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GOC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GOC_FOR_TARGET=$ac_cv_prog_GOC_FOR_TARGET  
if test -n "$GOC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GOC_FOR_TARGET" >&5  
$as_echo "$GOC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_GOC_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in gccgo; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_GOC_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_GOC_FOR_TARGET"; then  
  for ncn_progname in gccgo; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GOC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GOC_FOR_TARGET"; then  
  ac_cv_prog_GOC_FOR_TARGET="$GOC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GOC_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GOC_FOR_TARGET=$ac_cv_prog_GOC_FOR_TARGET  
if test -n "$GOC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GOC_FOR_TARGET" >&5  
$as_echo "$GOC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_GOC_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GOC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GOC_FOR_TARGET"; then  
  ac_cv_prog_GOC_FOR_TARGET="$GOC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GOC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GOC_FOR_TARGET=$ac_cv_prog_GOC_FOR_TARGET  
if test -n "$GOC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GOC_FOR_TARGET" >&5  
$as_echo "$GOC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_GOC_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_GOC_FOR_TARGET" ; then  
  set dummy gccgo  
  if test $build = $target ; then  
    GOC_FOR_TARGET="$2"  
  else  
    GOC_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  GOC_FOR_TARGET="$ac_cv_prog_GOC_FOR_TARGET"  
fi  
  
  
  
if test -n "$GDC_FOR_TARGET"; then  
  ac_cv_prog_GDC_FOR_TARGET=$GDC_FOR_TARGET  
elif test -n "$ac_cv_prog_GDC_FOR_TARGET"; then  
  GDC_FOR_TARGET=$ac_cv_prog_GDC_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_GDC_FOR_TARGET"; then  
  for ncn_progname in gdc; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GDC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GDC_FOR_TARGET"; then  
  ac_cv_prog_GDC_FOR_TARGET="$GDC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GDC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GDC_FOR_TARGET=$ac_cv_prog_GDC_FOR_TARGET  
if test -n "$GDC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GDC_FOR_TARGET" >&5  
$as_echo "$GDC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_GDC_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in gdc; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_GDC_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_GDC_FOR_TARGET"; then  
  for ncn_progname in gdc; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GDC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GDC_FOR_TARGET"; then  
  ac_cv_prog_GDC_FOR_TARGET="$GDC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GDC_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GDC_FOR_TARGET=$ac_cv_prog_GDC_FOR_TARGET  
if test -n "$GDC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GDC_FOR_TARGET" >&5  
$as_echo "$GDC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_GDC_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GDC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GDC_FOR_TARGET"; then  
  ac_cv_prog_GDC_FOR_TARGET="$GDC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GDC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GDC_FOR_TARGET=$ac_cv_prog_GDC_FOR_TARGET  
if test -n "$GDC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GDC_FOR_TARGET" >&5  
$as_echo "$GDC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_GDC_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_GDC_FOR_TARGET" ; then  
  set dummy gdc  
  if test $build = $target ; then  
    GDC_FOR_TARGET="$2"  
  else  
    GDC_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  GDC_FOR_TARGET="$ac_cv_prog_GDC_FOR_TARGET"  
fi  
  
  
  
if test -n "$GM2_FOR_TARGET"; then  
  ac_cv_prog_GM2_FOR_TARGET=$GM2_FOR_TARGET  
elif test -n "$ac_cv_prog_GM2_FOR_TARGET"; then  
  GM2_FOR_TARGET=$ac_cv_prog_GM2_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_GM2_FOR_TARGET"; then  
  for ncn_progname in gm2; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GM2_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GM2_FOR_TARGET"; then  
  ac_cv_prog_GM2_FOR_TARGET="$GM2_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GM2_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GM2_FOR_TARGET=$ac_cv_prog_GM2_FOR_TARGET  
if test -n "$GM2_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GM2_FOR_TARGET" >&5  
$as_echo "$GM2_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_GM2_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in gm2; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_GM2_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_GM2_FOR_TARGET"; then  
  for ncn_progname in gm2; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GM2_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GM2_FOR_TARGET"; then  
  ac_cv_prog_GM2_FOR_TARGET="$GM2_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GM2_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GM2_FOR_TARGET=$ac_cv_prog_GM2_FOR_TARGET  
if test -n "$GM2_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GM2_FOR_TARGET" >&5  
$as_echo "$GM2_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_GM2_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_GM2_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$GM2_FOR_TARGET"; then  
  ac_cv_prog_GM2_FOR_TARGET="$GM2_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_GM2_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
GM2_FOR_TARGET=$ac_cv_prog_GM2_FOR_TARGET  
if test -n "$GM2_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $GM2_FOR_TARGET" >&5  
$as_echo "$GM2_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_GM2_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_GM2_FOR_TARGET" ; then  
  set dummy gm2  
  if test $build = $target ; then  
    GM2_FOR_TARGET="$2"  
  else  
    GM2_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  GM2_FOR_TARGET="$ac_cv_prog_GM2_FOR_TARGET"  
fi  
  
  
  
cat > conftest.c \<< \EOF  
#ifdef __GNUC__  
  gcc_yay;  
#endif  
EOF  
if ($GCC_FOR_TARGET -E conftest.c | grep gcc_yay) > /dev/null 2>&1; then  
  have_gcc_for_target=yes  
else  
  GCC_FOR_TARGET=${ncn_target_tool_prefix}gcc  
  have_gcc_for_target=no  
fi  
rm conftest.c  
  
  
  
  
if test -z "$ac_cv_path_AR_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ar in $with_build_time_tools" >&5  
$as_echo_n "checking for ar in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/ar; then  
      AR_FOR_TARGET=`cd $with_build_time_tools && pwd`/ar  
      ac_cv_path_AR_FOR_TARGET=$AR_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_AR_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_AR_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    AR_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=ar`  
    test $AR_FOR_TARGET = ar && AR_FOR_TARGET=  
    test -n "$AR_FOR_TARGET" && ac_cv_path_AR_FOR_TARGET=$AR_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_AR_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "ar", so it can be a program name with args.  
set dummy ar; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_AR_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $AR_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_AR_FOR_TARGET="$AR_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_AR_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
AR_FOR_TARGET=$ac_cv_path_AR_FOR_TARGET  
if test -n "$AR_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR_FOR_TARGET" >&5  
$as_echo "$AR_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_AR_FOR_TARGET" ; then  
  
  
if test -n "$AR_FOR_TARGET"; then  
  ac_cv_prog_AR_FOR_TARGET=$AR_FOR_TARGET  
elif test -n "$ac_cv_prog_AR_FOR_TARGET"; then  
  AR_FOR_TARGET=$ac_cv_prog_AR_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_AR_FOR_TARGET"; then  
  for ncn_progname in ar; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR_FOR_TARGET"; then  
  ac_cv_prog_AR_FOR_TARGET="$AR_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR_FOR_TARGET=$ac_cv_prog_AR_FOR_TARGET  
if test -n "$AR_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR_FOR_TARGET" >&5  
$as_echo "$AR_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_AR_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in ar; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_AR_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_AR_FOR_TARGET"; then  
  for ncn_progname in ar; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR_FOR_TARGET"; then  
  ac_cv_prog_AR_FOR_TARGET="$AR_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR_FOR_TARGET=$ac_cv_prog_AR_FOR_TARGET  
if test -n "$AR_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR_FOR_TARGET" >&5  
$as_echo "$AR_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_AR_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AR_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AR_FOR_TARGET"; then  
  ac_cv_prog_AR_FOR_TARGET="$AR_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AR_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AR_FOR_TARGET=$ac_cv_prog_AR_FOR_TARGET  
if test -n "$AR_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AR_FOR_TARGET" >&5  
$as_echo "$AR_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_AR_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_AR_FOR_TARGET" ; then  
  set dummy ar  
  if test $build = $target ; then  
    AR_FOR_TARGET="$2"  
  else  
    AR_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  AR_FOR_TARGET="$ac_cv_prog_AR_FOR_TARGET"  
fi  
  
else  
  AR_FOR_TARGET=$ac_cv_path_AR_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_AS_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for as in $with_build_time_tools" >&5  
$as_echo_n "checking for as in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/as; then  
      AS_FOR_TARGET=`cd $with_build_time_tools && pwd`/as  
      ac_cv_path_AS_FOR_TARGET=$AS_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_AS_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_AS_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    AS_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=as`  
    test $AS_FOR_TARGET = as && AS_FOR_TARGET=  
    test -n "$AS_FOR_TARGET" && ac_cv_path_AS_FOR_TARGET=$AS_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_AS_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "as", so it can be a program name with args.  
set dummy as; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_AS_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $AS_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_AS_FOR_TARGET="$AS_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_AS_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
AS_FOR_TARGET=$ac_cv_path_AS_FOR_TARGET  
if test -n "$AS_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS_FOR_TARGET" >&5  
$as_echo "$AS_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_AS_FOR_TARGET" ; then  
  
  
if test -n "$AS_FOR_TARGET"; then  
  ac_cv_prog_AS_FOR_TARGET=$AS_FOR_TARGET  
elif test -n "$ac_cv_prog_AS_FOR_TARGET"; then  
  AS_FOR_TARGET=$ac_cv_prog_AS_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_AS_FOR_TARGET"; then  
  for ncn_progname in as; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AS_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AS_FOR_TARGET"; then  
  ac_cv_prog_AS_FOR_TARGET="$AS_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AS_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AS_FOR_TARGET=$ac_cv_prog_AS_FOR_TARGET  
if test -n "$AS_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS_FOR_TARGET" >&5  
$as_echo "$AS_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_AS_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in as; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_AS_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_AS_FOR_TARGET"; then  
  for ncn_progname in as; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AS_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AS_FOR_TARGET"; then  
  ac_cv_prog_AS_FOR_TARGET="$AS_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AS_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AS_FOR_TARGET=$ac_cv_prog_AS_FOR_TARGET  
if test -n "$AS_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS_FOR_TARGET" >&5  
$as_echo "$AS_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_AS_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_AS_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$AS_FOR_TARGET"; then  
  ac_cv_prog_AS_FOR_TARGET="$AS_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_AS_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
AS_FOR_TARGET=$ac_cv_prog_AS_FOR_TARGET  
if test -n "$AS_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $AS_FOR_TARGET" >&5  
$as_echo "$AS_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_AS_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_AS_FOR_TARGET" ; then  
  set dummy as  
  if test $build = $target ; then  
    AS_FOR_TARGET="$2"  
  else  
    AS_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  AS_FOR_TARGET="$ac_cv_prog_AS_FOR_TARGET"  
fi  
  
else  
  AS_FOR_TARGET=$ac_cv_path_AS_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_DLLTOOL_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for dlltool in $with_build_time_tools" >&5  
$as_echo_n "checking for dlltool in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/dlltool; then  
      DLLTOOL_FOR_TARGET=`cd $with_build_time_tools && pwd`/dlltool  
      ac_cv_path_DLLTOOL_FOR_TARGET=$DLLTOOL_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_DLLTOOL_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_DLLTOOL_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    DLLTOOL_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=dlltool`  
    test $DLLTOOL_FOR_TARGET = dlltool && DLLTOOL_FOR_TARGET=  
    test -n "$DLLTOOL_FOR_TARGET" && ac_cv_path_DLLTOOL_FOR_TARGET=$DLLTOOL_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_DLLTOOL_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "dlltool", so it can be a program name with args.  
set dummy dlltool; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_DLLTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $DLLTOOL_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_DLLTOOL_FOR_TARGET="$DLLTOOL_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_DLLTOOL_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
DLLTOOL_FOR_TARGET=$ac_cv_path_DLLTOOL_FOR_TARGET  
if test -n "$DLLTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL_FOR_TARGET" >&5  
$as_echo "$DLLTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_DLLTOOL_FOR_TARGET" ; then  
  
  
if test -n "$DLLTOOL_FOR_TARGET"; then  
  ac_cv_prog_DLLTOOL_FOR_TARGET=$DLLTOOL_FOR_TARGET  
elif test -n "$ac_cv_prog_DLLTOOL_FOR_TARGET"; then  
  DLLTOOL_FOR_TARGET=$ac_cv_prog_DLLTOOL_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_DLLTOOL_FOR_TARGET"; then  
  for ncn_progname in dlltool; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DLLTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DLLTOOL_FOR_TARGET"; then  
  ac_cv_prog_DLLTOOL_FOR_TARGET="$DLLTOOL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DLLTOOL_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DLLTOOL_FOR_TARGET=$ac_cv_prog_DLLTOOL_FOR_TARGET  
if test -n "$DLLTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL_FOR_TARGET" >&5  
$as_echo "$DLLTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_DLLTOOL_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in dlltool; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_DLLTOOL_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_DLLTOOL_FOR_TARGET"; then  
  for ncn_progname in dlltool; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DLLTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DLLTOOL_FOR_TARGET"; then  
  ac_cv_prog_DLLTOOL_FOR_TARGET="$DLLTOOL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DLLTOOL_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DLLTOOL_FOR_TARGET=$ac_cv_prog_DLLTOOL_FOR_TARGET  
if test -n "$DLLTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL_FOR_TARGET" >&5  
$as_echo "$DLLTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_DLLTOOL_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DLLTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DLLTOOL_FOR_TARGET"; then  
  ac_cv_prog_DLLTOOL_FOR_TARGET="$DLLTOOL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DLLTOOL_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DLLTOOL_FOR_TARGET=$ac_cv_prog_DLLTOOL_FOR_TARGET  
if test -n "$DLLTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DLLTOOL_FOR_TARGET" >&5  
$as_echo "$DLLTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_DLLTOOL_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_DLLTOOL_FOR_TARGET" ; then  
  set dummy dlltool  
  if test $build = $target ; then  
    DLLTOOL_FOR_TARGET="$2"  
  else  
    DLLTOOL_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  DLLTOOL_FOR_TARGET="$ac_cv_prog_DLLTOOL_FOR_TARGET"  
fi  
  
else  
  DLLTOOL_FOR_TARGET=$ac_cv_path_DLLTOOL_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_DSYMUTIL_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for dsymutil in $with_build_time_tools" >&5  
$as_echo_n "checking for dsymutil in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/dsymutil; then  
      DSYMUTIL_FOR_TARGET=`cd $with_build_time_tools && pwd`/dsymutil  
      ac_cv_path_DSYMUTIL_FOR_TARGET=$DSYMUTIL_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_DSYMUTIL_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_DSYMUTIL_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    DSYMUTIL_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=dsymutil`  
    test $DSYMUTIL_FOR_TARGET = dsymutil && DSYMUTIL_FOR_TARGET=  
    test -n "$DSYMUTIL_FOR_TARGET" && ac_cv_path_DSYMUTIL_FOR_TARGET=$DSYMUTIL_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_DSYMUTIL_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "dsymutil", so it can be a program name with args.  
set dummy dsymutil; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_DSYMUTIL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $DSYMUTIL_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_DSYMUTIL_FOR_TARGET="$DSYMUTIL_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_DSYMUTIL_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
DSYMUTIL_FOR_TARGET=$ac_cv_path_DSYMUTIL_FOR_TARGET  
if test -n "$DSYMUTIL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL_FOR_TARGET" >&5  
$as_echo "$DSYMUTIL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_DSYMUTIL_FOR_TARGET" ; then  
  
  
if test -n "$DSYMUTIL_FOR_TARGET"; then  
  ac_cv_prog_DSYMUTIL_FOR_TARGET=$DSYMUTIL_FOR_TARGET  
elif test -n "$ac_cv_prog_DSYMUTIL_FOR_TARGET"; then  
  DSYMUTIL_FOR_TARGET=$ac_cv_prog_DSYMUTIL_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_DSYMUTIL_FOR_TARGET"; then  
  for ncn_progname in dsymutil; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DSYMUTIL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DSYMUTIL_FOR_TARGET"; then  
  ac_cv_prog_DSYMUTIL_FOR_TARGET="$DSYMUTIL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DSYMUTIL_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DSYMUTIL_FOR_TARGET=$ac_cv_prog_DSYMUTIL_FOR_TARGET  
if test -n "$DSYMUTIL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL_FOR_TARGET" >&5  
$as_echo "$DSYMUTIL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_DSYMUTIL_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in dsymutil; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_DSYMUTIL_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_DSYMUTIL_FOR_TARGET"; then  
  for ncn_progname in dsymutil; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DSYMUTIL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DSYMUTIL_FOR_TARGET"; then  
  ac_cv_prog_DSYMUTIL_FOR_TARGET="$DSYMUTIL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DSYMUTIL_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DSYMUTIL_FOR_TARGET=$ac_cv_prog_DSYMUTIL_FOR_TARGET  
if test -n "$DSYMUTIL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL_FOR_TARGET" >&5  
$as_echo "$DSYMUTIL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_DSYMUTIL_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_DSYMUTIL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$DSYMUTIL_FOR_TARGET"; then  
  ac_cv_prog_DSYMUTIL_FOR_TARGET="$DSYMUTIL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_DSYMUTIL_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
DSYMUTIL_FOR_TARGET=$ac_cv_prog_DSYMUTIL_FOR_TARGET  
if test -n "$DSYMUTIL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $DSYMUTIL_FOR_TARGET" >&5  
$as_echo "$DSYMUTIL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_DSYMUTIL_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_DSYMUTIL_FOR_TARGET" ; then  
  set dummy dsymutil  
  if test $build = $target ; then  
    DSYMUTIL_FOR_TARGET="$2"  
  else  
    DSYMUTIL_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  DSYMUTIL_FOR_TARGET="$ac_cv_prog_DSYMUTIL_FOR_TARGET"  
fi  
  
else  
  DSYMUTIL_FOR_TARGET=$ac_cv_path_DSYMUTIL_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_LD_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ld in $with_build_time_tools" >&5  
$as_echo_n "checking for ld in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/ld; then  
      LD_FOR_TARGET=`cd $with_build_time_tools && pwd`/ld  
      ac_cv_path_LD_FOR_TARGET=$LD_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_LD_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_LD_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    LD_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=ld`  
    test $LD_FOR_TARGET = ld && LD_FOR_TARGET=  
    test -n "$LD_FOR_TARGET" && ac_cv_path_LD_FOR_TARGET=$LD_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_LD_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "ld", so it can be a program name with args.  
set dummy ld; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_LD_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $LD_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_LD_FOR_TARGET="$LD_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_LD_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
LD_FOR_TARGET=$ac_cv_path_LD_FOR_TARGET  
if test -n "$LD_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD_FOR_TARGET" >&5  
$as_echo "$LD_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_LD_FOR_TARGET" ; then  
  
  
if test -n "$LD_FOR_TARGET"; then  
  ac_cv_prog_LD_FOR_TARGET=$LD_FOR_TARGET  
elif test -n "$ac_cv_prog_LD_FOR_TARGET"; then  
  LD_FOR_TARGET=$ac_cv_prog_LD_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_LD_FOR_TARGET"; then  
  for ncn_progname in ld; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LD_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LD_FOR_TARGET"; then  
  ac_cv_prog_LD_FOR_TARGET="$LD_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LD_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LD_FOR_TARGET=$ac_cv_prog_LD_FOR_TARGET  
if test -n "$LD_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD_FOR_TARGET" >&5  
$as_echo "$LD_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_LD_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in ld; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_LD_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_LD_FOR_TARGET"; then  
  for ncn_progname in ld; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LD_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LD_FOR_TARGET"; then  
  ac_cv_prog_LD_FOR_TARGET="$LD_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LD_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LD_FOR_TARGET=$ac_cv_prog_LD_FOR_TARGET  
if test -n "$LD_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD_FOR_TARGET" >&5  
$as_echo "$LD_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_LD_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LD_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LD_FOR_TARGET"; then  
  ac_cv_prog_LD_FOR_TARGET="$LD_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LD_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LD_FOR_TARGET=$ac_cv_prog_LD_FOR_TARGET  
if test -n "$LD_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LD_FOR_TARGET" >&5  
$as_echo "$LD_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_LD_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_LD_FOR_TARGET" ; then  
  set dummy ld  
  if test $build = $target ; then  
    LD_FOR_TARGET="$2"  
  else  
    LD_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  LD_FOR_TARGET="$ac_cv_prog_LD_FOR_TARGET"  
fi  
  
else  
  LD_FOR_TARGET=$ac_cv_path_LD_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_LIPO_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for lipo in $with_build_time_tools" >&5  
$as_echo_n "checking for lipo in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/lipo; then  
      LIPO_FOR_TARGET=`cd $with_build_time_tools && pwd`/lipo  
      ac_cv_path_LIPO_FOR_TARGET=$LIPO_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_LIPO_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_LIPO_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    LIPO_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=lipo`  
    test $LIPO_FOR_TARGET = lipo && LIPO_FOR_TARGET=  
    test -n "$LIPO_FOR_TARGET" && ac_cv_path_LIPO_FOR_TARGET=$LIPO_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_LIPO_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "lipo", so it can be a program name with args.  
set dummy lipo; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_LIPO_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $LIPO_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_LIPO_FOR_TARGET="$LIPO_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_LIPO_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
LIPO_FOR_TARGET=$ac_cv_path_LIPO_FOR_TARGET  
if test -n "$LIPO_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO_FOR_TARGET" >&5  
$as_echo "$LIPO_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_LIPO_FOR_TARGET" ; then  
  
  
if test -n "$LIPO_FOR_TARGET"; then  
  ac_cv_prog_LIPO_FOR_TARGET=$LIPO_FOR_TARGET  
elif test -n "$ac_cv_prog_LIPO_FOR_TARGET"; then  
  LIPO_FOR_TARGET=$ac_cv_prog_LIPO_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_LIPO_FOR_TARGET"; then  
  for ncn_progname in lipo; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LIPO_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LIPO_FOR_TARGET"; then  
  ac_cv_prog_LIPO_FOR_TARGET="$LIPO_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LIPO_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LIPO_FOR_TARGET=$ac_cv_prog_LIPO_FOR_TARGET  
if test -n "$LIPO_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO_FOR_TARGET" >&5  
$as_echo "$LIPO_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_LIPO_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in lipo; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_LIPO_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_LIPO_FOR_TARGET"; then  
  for ncn_progname in lipo; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LIPO_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LIPO_FOR_TARGET"; then  
  ac_cv_prog_LIPO_FOR_TARGET="$LIPO_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LIPO_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LIPO_FOR_TARGET=$ac_cv_prog_LIPO_FOR_TARGET  
if test -n "$LIPO_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO_FOR_TARGET" >&5  
$as_echo "$LIPO_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_LIPO_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_LIPO_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$LIPO_FOR_TARGET"; then  
  ac_cv_prog_LIPO_FOR_TARGET="$LIPO_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_LIPO_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
LIPO_FOR_TARGET=$ac_cv_prog_LIPO_FOR_TARGET  
if test -n "$LIPO_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $LIPO_FOR_TARGET" >&5  
$as_echo "$LIPO_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_LIPO_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_LIPO_FOR_TARGET" ; then  
  set dummy lipo  
  if test $build = $target ; then  
    LIPO_FOR_TARGET="$2"  
  else  
    LIPO_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  LIPO_FOR_TARGET="$ac_cv_prog_LIPO_FOR_TARGET"  
fi  
  
else  
  LIPO_FOR_TARGET=$ac_cv_path_LIPO_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_NM_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for nm in $with_build_time_tools" >&5  
$as_echo_n "checking for nm in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/nm; then  
      NM_FOR_TARGET=`cd $with_build_time_tools && pwd`/nm  
      ac_cv_path_NM_FOR_TARGET=$NM_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_NM_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_NM_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    NM_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=nm`  
    test $NM_FOR_TARGET = nm && NM_FOR_TARGET=  
    test -n "$NM_FOR_TARGET" && ac_cv_path_NM_FOR_TARGET=$NM_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_NM_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "nm", so it can be a program name with args.  
set dummy nm; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_NM_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $NM_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_NM_FOR_TARGET="$NM_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_NM_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
NM_FOR_TARGET=$ac_cv_path_NM_FOR_TARGET  
if test -n "$NM_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM_FOR_TARGET" >&5  
$as_echo "$NM_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_NM_FOR_TARGET" ; then  
  
  
if test -n "$NM_FOR_TARGET"; then  
  ac_cv_prog_NM_FOR_TARGET=$NM_FOR_TARGET  
elif test -n "$ac_cv_prog_NM_FOR_TARGET"; then  
  NM_FOR_TARGET=$ac_cv_prog_NM_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_NM_FOR_TARGET"; then  
  for ncn_progname in nm; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_NM_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$NM_FOR_TARGET"; then  
  ac_cv_prog_NM_FOR_TARGET="$NM_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_NM_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
NM_FOR_TARGET=$ac_cv_prog_NM_FOR_TARGET  
if test -n "$NM_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM_FOR_TARGET" >&5  
$as_echo "$NM_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_NM_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in nm; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_NM_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_NM_FOR_TARGET"; then  
  for ncn_progname in nm; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_NM_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$NM_FOR_TARGET"; then  
  ac_cv_prog_NM_FOR_TARGET="$NM_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_NM_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
NM_FOR_TARGET=$ac_cv_prog_NM_FOR_TARGET  
if test -n "$NM_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM_FOR_TARGET" >&5  
$as_echo "$NM_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_NM_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_NM_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$NM_FOR_TARGET"; then  
  ac_cv_prog_NM_FOR_TARGET="$NM_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_NM_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
NM_FOR_TARGET=$ac_cv_prog_NM_FOR_TARGET  
if test -n "$NM_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $NM_FOR_TARGET" >&5  
$as_echo "$NM_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_NM_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_NM_FOR_TARGET" ; then  
  set dummy nm  
  if test $build = $target ; then  
    NM_FOR_TARGET="$2"  
  else  
    NM_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  NM_FOR_TARGET="$ac_cv_prog_NM_FOR_TARGET"  
fi  
  
else  
  NM_FOR_TARGET=$ac_cv_path_NM_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_OBJCOPY_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for objcopy in $with_build_time_tools" >&5  
$as_echo_n "checking for objcopy in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/objcopy; then  
      OBJCOPY_FOR_TARGET=`cd $with_build_time_tools && pwd`/objcopy  
      ac_cv_path_OBJCOPY_FOR_TARGET=$OBJCOPY_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_OBJCOPY_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_OBJCOPY_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    OBJCOPY_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=objcopy`  
    test $OBJCOPY_FOR_TARGET = objcopy && OBJCOPY_FOR_TARGET=  
    test -n "$OBJCOPY_FOR_TARGET" && ac_cv_path_OBJCOPY_FOR_TARGET=$OBJCOPY_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_OBJCOPY_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "objcopy", so it can be a program name with args.  
set dummy objcopy; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_OBJCOPY_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $OBJCOPY_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_OBJCOPY_FOR_TARGET="$OBJCOPY_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_OBJCOPY_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
OBJCOPY_FOR_TARGET=$ac_cv_path_OBJCOPY_FOR_TARGET  
if test -n "$OBJCOPY_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY_FOR_TARGET" >&5  
$as_echo "$OBJCOPY_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_OBJCOPY_FOR_TARGET" ; then  
  
  
if test -n "$OBJCOPY_FOR_TARGET"; then  
  ac_cv_prog_OBJCOPY_FOR_TARGET=$OBJCOPY_FOR_TARGET  
elif test -n "$ac_cv_prog_OBJCOPY_FOR_TARGET"; then  
  OBJCOPY_FOR_TARGET=$ac_cv_prog_OBJCOPY_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_OBJCOPY_FOR_TARGET"; then  
  for ncn_progname in objcopy; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJCOPY_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJCOPY_FOR_TARGET"; then  
  ac_cv_prog_OBJCOPY_FOR_TARGET="$OBJCOPY_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJCOPY_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJCOPY_FOR_TARGET=$ac_cv_prog_OBJCOPY_FOR_TARGET  
if test -n "$OBJCOPY_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY_FOR_TARGET" >&5  
$as_echo "$OBJCOPY_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_OBJCOPY_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in objcopy; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_OBJCOPY_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_OBJCOPY_FOR_TARGET"; then  
  for ncn_progname in objcopy; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJCOPY_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJCOPY_FOR_TARGET"; then  
  ac_cv_prog_OBJCOPY_FOR_TARGET="$OBJCOPY_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJCOPY_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJCOPY_FOR_TARGET=$ac_cv_prog_OBJCOPY_FOR_TARGET  
if test -n "$OBJCOPY_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY_FOR_TARGET" >&5  
$as_echo "$OBJCOPY_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_OBJCOPY_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJCOPY_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJCOPY_FOR_TARGET"; then  
  ac_cv_prog_OBJCOPY_FOR_TARGET="$OBJCOPY_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJCOPY_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJCOPY_FOR_TARGET=$ac_cv_prog_OBJCOPY_FOR_TARGET  
if test -n "$OBJCOPY_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJCOPY_FOR_TARGET" >&5  
$as_echo "$OBJCOPY_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_OBJCOPY_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_OBJCOPY_FOR_TARGET" ; then  
  set dummy objcopy  
  if test $build = $target ; then  
    OBJCOPY_FOR_TARGET="$2"  
  else  
    OBJCOPY_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  OBJCOPY_FOR_TARGET="$ac_cv_prog_OBJCOPY_FOR_TARGET"  
fi  
  
else  
  OBJCOPY_FOR_TARGET=$ac_cv_path_OBJCOPY_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_OBJDUMP_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for objdump in $with_build_time_tools" >&5  
$as_echo_n "checking for objdump in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/objdump; then  
      OBJDUMP_FOR_TARGET=`cd $with_build_time_tools && pwd`/objdump  
      ac_cv_path_OBJDUMP_FOR_TARGET=$OBJDUMP_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_OBJDUMP_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_OBJDUMP_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    OBJDUMP_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=objdump`  
    test $OBJDUMP_FOR_TARGET = objdump && OBJDUMP_FOR_TARGET=  
    test -n "$OBJDUMP_FOR_TARGET" && ac_cv_path_OBJDUMP_FOR_TARGET=$OBJDUMP_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_OBJDUMP_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "objdump", so it can be a program name with args.  
set dummy objdump; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_OBJDUMP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $OBJDUMP_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_OBJDUMP_FOR_TARGET="$OBJDUMP_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_OBJDUMP_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
OBJDUMP_FOR_TARGET=$ac_cv_path_OBJDUMP_FOR_TARGET  
if test -n "$OBJDUMP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP_FOR_TARGET" >&5  
$as_echo "$OBJDUMP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_OBJDUMP_FOR_TARGET" ; then  
  
  
if test -n "$OBJDUMP_FOR_TARGET"; then  
  ac_cv_prog_OBJDUMP_FOR_TARGET=$OBJDUMP_FOR_TARGET  
elif test -n "$ac_cv_prog_OBJDUMP_FOR_TARGET"; then  
  OBJDUMP_FOR_TARGET=$ac_cv_prog_OBJDUMP_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_OBJDUMP_FOR_TARGET"; then  
  for ncn_progname in objdump; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJDUMP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJDUMP_FOR_TARGET"; then  
  ac_cv_prog_OBJDUMP_FOR_TARGET="$OBJDUMP_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJDUMP_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJDUMP_FOR_TARGET=$ac_cv_prog_OBJDUMP_FOR_TARGET  
if test -n "$OBJDUMP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP_FOR_TARGET" >&5  
$as_echo "$OBJDUMP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_OBJDUMP_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in objdump; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_OBJDUMP_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_OBJDUMP_FOR_TARGET"; then  
  for ncn_progname in objdump; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJDUMP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJDUMP_FOR_TARGET"; then  
  ac_cv_prog_OBJDUMP_FOR_TARGET="$OBJDUMP_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJDUMP_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJDUMP_FOR_TARGET=$ac_cv_prog_OBJDUMP_FOR_TARGET  
if test -n "$OBJDUMP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP_FOR_TARGET" >&5  
$as_echo "$OBJDUMP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_OBJDUMP_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OBJDUMP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OBJDUMP_FOR_TARGET"; then  
  ac_cv_prog_OBJDUMP_FOR_TARGET="$OBJDUMP_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OBJDUMP_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OBJDUMP_FOR_TARGET=$ac_cv_prog_OBJDUMP_FOR_TARGET  
if test -n "$OBJDUMP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OBJDUMP_FOR_TARGET" >&5  
$as_echo "$OBJDUMP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_OBJDUMP_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_OBJDUMP_FOR_TARGET" ; then  
  set dummy objdump  
  if test $build = $target ; then  
    OBJDUMP_FOR_TARGET="$2"  
  else  
    OBJDUMP_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  OBJDUMP_FOR_TARGET="$ac_cv_prog_OBJDUMP_FOR_TARGET"  
fi  
  
else  
  OBJDUMP_FOR_TARGET=$ac_cv_path_OBJDUMP_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_OTOOL_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for otool in $with_build_time_tools" >&5  
$as_echo_n "checking for otool in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/otool; then  
      OTOOL_FOR_TARGET=`cd $with_build_time_tools && pwd`/otool  
      ac_cv_path_OTOOL_FOR_TARGET=$OTOOL_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_OTOOL_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_OTOOL_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    OTOOL_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=otool`  
    test $OTOOL_FOR_TARGET = otool && OTOOL_FOR_TARGET=  
    test -n "$OTOOL_FOR_TARGET" && ac_cv_path_OTOOL_FOR_TARGET=$OTOOL_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_OTOOL_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "otool", so it can be a program name with args.  
set dummy otool; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_OTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $OTOOL_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_OTOOL_FOR_TARGET="$OTOOL_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_OTOOL_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
OTOOL_FOR_TARGET=$ac_cv_path_OTOOL_FOR_TARGET  
if test -n "$OTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL_FOR_TARGET" >&5  
$as_echo "$OTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_OTOOL_FOR_TARGET" ; then  
  
  
if test -n "$OTOOL_FOR_TARGET"; then  
  ac_cv_prog_OTOOL_FOR_TARGET=$OTOOL_FOR_TARGET  
elif test -n "$ac_cv_prog_OTOOL_FOR_TARGET"; then  
  OTOOL_FOR_TARGET=$ac_cv_prog_OTOOL_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_OTOOL_FOR_TARGET"; then  
  for ncn_progname in otool; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OTOOL_FOR_TARGET"; then  
  ac_cv_prog_OTOOL_FOR_TARGET="$OTOOL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OTOOL_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OTOOL_FOR_TARGET=$ac_cv_prog_OTOOL_FOR_TARGET  
if test -n "$OTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL_FOR_TARGET" >&5  
$as_echo "$OTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_OTOOL_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in otool; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_OTOOL_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_OTOOL_FOR_TARGET"; then  
  for ncn_progname in otool; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OTOOL_FOR_TARGET"; then  
  ac_cv_prog_OTOOL_FOR_TARGET="$OTOOL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OTOOL_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OTOOL_FOR_TARGET=$ac_cv_prog_OTOOL_FOR_TARGET  
if test -n "$OTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL_FOR_TARGET" >&5  
$as_echo "$OTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_OTOOL_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_OTOOL_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$OTOOL_FOR_TARGET"; then  
  ac_cv_prog_OTOOL_FOR_TARGET="$OTOOL_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_OTOOL_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
OTOOL_FOR_TARGET=$ac_cv_prog_OTOOL_FOR_TARGET  
if test -n "$OTOOL_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $OTOOL_FOR_TARGET" >&5  
$as_echo "$OTOOL_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_OTOOL_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_OTOOL_FOR_TARGET" ; then  
  set dummy otool  
  if test $build = $target ; then  
    OTOOL_FOR_TARGET="$2"  
  else  
    OTOOL_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  OTOOL_FOR_TARGET="$ac_cv_prog_OTOOL_FOR_TARGET"  
fi  
  
else  
  OTOOL_FOR_TARGET=$ac_cv_path_OTOOL_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_RANLIB_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ranlib in $with_build_time_tools" >&5  
$as_echo_n "checking for ranlib in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/ranlib; then  
      RANLIB_FOR_TARGET=`cd $with_build_time_tools && pwd`/ranlib  
      ac_cv_path_RANLIB_FOR_TARGET=$RANLIB_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_RANLIB_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_RANLIB_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    RANLIB_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=ranlib`  
    test $RANLIB_FOR_TARGET = ranlib && RANLIB_FOR_TARGET=  
    test -n "$RANLIB_FOR_TARGET" && ac_cv_path_RANLIB_FOR_TARGET=$RANLIB_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_RANLIB_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "ranlib", so it can be a program name with args.  
set dummy ranlib; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_RANLIB_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $RANLIB_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_RANLIB_FOR_TARGET="$RANLIB_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_RANLIB_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
RANLIB_FOR_TARGET=$ac_cv_path_RANLIB_FOR_TARGET  
if test -n "$RANLIB_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB_FOR_TARGET" >&5  
$as_echo "$RANLIB_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_RANLIB_FOR_TARGET" ; then  
  
  
if test -n "$RANLIB_FOR_TARGET"; then  
  ac_cv_prog_RANLIB_FOR_TARGET=$RANLIB_FOR_TARGET  
elif test -n "$ac_cv_prog_RANLIB_FOR_TARGET"; then  
  RANLIB_FOR_TARGET=$ac_cv_prog_RANLIB_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_RANLIB_FOR_TARGET"; then  
  for ncn_progname in ranlib; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RANLIB_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RANLIB_FOR_TARGET"; then  
  ac_cv_prog_RANLIB_FOR_TARGET="$RANLIB_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RANLIB_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RANLIB_FOR_TARGET=$ac_cv_prog_RANLIB_FOR_TARGET  
if test -n "$RANLIB_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB_FOR_TARGET" >&5  
$as_echo "$RANLIB_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_RANLIB_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in ranlib; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_RANLIB_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_RANLIB_FOR_TARGET"; then  
  for ncn_progname in ranlib; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RANLIB_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RANLIB_FOR_TARGET"; then  
  ac_cv_prog_RANLIB_FOR_TARGET="$RANLIB_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RANLIB_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RANLIB_FOR_TARGET=$ac_cv_prog_RANLIB_FOR_TARGET  
if test -n "$RANLIB_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB_FOR_TARGET" >&5  
$as_echo "$RANLIB_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_RANLIB_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_RANLIB_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$RANLIB_FOR_TARGET"; then  
  ac_cv_prog_RANLIB_FOR_TARGET="$RANLIB_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_RANLIB_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
RANLIB_FOR_TARGET=$ac_cv_prog_RANLIB_FOR_TARGET  
if test -n "$RANLIB_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $RANLIB_FOR_TARGET" >&5  
$as_echo "$RANLIB_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_RANLIB_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_RANLIB_FOR_TARGET" ; then  
  set dummy ranlib  
  if test $build = $target ; then  
    RANLIB_FOR_TARGET="$2"  
  else  
    RANLIB_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  RANLIB_FOR_TARGET="$ac_cv_prog_RANLIB_FOR_TARGET"  
fi  
  
else  
  RANLIB_FOR_TARGET=$ac_cv_path_RANLIB_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_READELF_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for readelf in $with_build_time_tools" >&5  
$as_echo_n "checking for readelf in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/readelf; then  
      READELF_FOR_TARGET=`cd $with_build_time_tools && pwd`/readelf  
      ac_cv_path_READELF_FOR_TARGET=$READELF_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_READELF_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_READELF_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    READELF_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=readelf`  
    test $READELF_FOR_TARGET = readelf && READELF_FOR_TARGET=  
    test -n "$READELF_FOR_TARGET" && ac_cv_path_READELF_FOR_TARGET=$READELF_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_READELF_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "readelf", so it can be a program name with args.  
set dummy readelf; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_READELF_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $READELF_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_READELF_FOR_TARGET="$READELF_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_READELF_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
READELF_FOR_TARGET=$ac_cv_path_READELF_FOR_TARGET  
if test -n "$READELF_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF_FOR_TARGET" >&5  
$as_echo "$READELF_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_READELF_FOR_TARGET" ; then  
  
  
if test -n "$READELF_FOR_TARGET"; then  
  ac_cv_prog_READELF_FOR_TARGET=$READELF_FOR_TARGET  
elif test -n "$ac_cv_prog_READELF_FOR_TARGET"; then  
  READELF_FOR_TARGET=$ac_cv_prog_READELF_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_READELF_FOR_TARGET"; then  
  for ncn_progname in readelf; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_READELF_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$READELF_FOR_TARGET"; then  
  ac_cv_prog_READELF_FOR_TARGET="$READELF_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_READELF_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
READELF_FOR_TARGET=$ac_cv_prog_READELF_FOR_TARGET  
if test -n "$READELF_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF_FOR_TARGET" >&5  
$as_echo "$READELF_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_READELF_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in readelf; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_READELF_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_READELF_FOR_TARGET"; then  
  for ncn_progname in readelf; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_READELF_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$READELF_FOR_TARGET"; then  
  ac_cv_prog_READELF_FOR_TARGET="$READELF_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_READELF_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
READELF_FOR_TARGET=$ac_cv_prog_READELF_FOR_TARGET  
if test -n "$READELF_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF_FOR_TARGET" >&5  
$as_echo "$READELF_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_READELF_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_READELF_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$READELF_FOR_TARGET"; then  
  ac_cv_prog_READELF_FOR_TARGET="$READELF_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_READELF_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
READELF_FOR_TARGET=$ac_cv_prog_READELF_FOR_TARGET  
if test -n "$READELF_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $READELF_FOR_TARGET" >&5  
$as_echo "$READELF_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_READELF_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_READELF_FOR_TARGET" ; then  
  set dummy readelf  
  if test $build = $target ; then  
    READELF_FOR_TARGET="$2"  
  else  
    READELF_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  READELF_FOR_TARGET="$ac_cv_prog_READELF_FOR_TARGET"  
fi  
  
else  
  READELF_FOR_TARGET=$ac_cv_path_READELF_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_STRIP_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for strip in $with_build_time_tools" >&5  
$as_echo_n "checking for strip in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/strip; then  
      STRIP_FOR_TARGET=`cd $with_build_time_tools && pwd`/strip  
      ac_cv_path_STRIP_FOR_TARGET=$STRIP_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_STRIP_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_STRIP_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    STRIP_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=strip`  
    test $STRIP_FOR_TARGET = strip && STRIP_FOR_TARGET=  
    test -n "$STRIP_FOR_TARGET" && ac_cv_path_STRIP_FOR_TARGET=$STRIP_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_STRIP_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "strip", so it can be a program name with args.  
set dummy strip; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_STRIP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $STRIP_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_STRIP_FOR_TARGET="$STRIP_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_STRIP_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
STRIP_FOR_TARGET=$ac_cv_path_STRIP_FOR_TARGET  
if test -n "$STRIP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP_FOR_TARGET" >&5  
$as_echo "$STRIP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_STRIP_FOR_TARGET" ; then  
  
  
if test -n "$STRIP_FOR_TARGET"; then  
  ac_cv_prog_STRIP_FOR_TARGET=$STRIP_FOR_TARGET  
elif test -n "$ac_cv_prog_STRIP_FOR_TARGET"; then  
  STRIP_FOR_TARGET=$ac_cv_prog_STRIP_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_STRIP_FOR_TARGET"; then  
  for ncn_progname in strip; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_STRIP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$STRIP_FOR_TARGET"; then  
  ac_cv_prog_STRIP_FOR_TARGET="$STRIP_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_STRIP_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
STRIP_FOR_TARGET=$ac_cv_prog_STRIP_FOR_TARGET  
if test -n "$STRIP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP_FOR_TARGET" >&5  
$as_echo "$STRIP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_STRIP_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in strip; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_STRIP_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_STRIP_FOR_TARGET"; then  
  for ncn_progname in strip; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_STRIP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$STRIP_FOR_TARGET"; then  
  ac_cv_prog_STRIP_FOR_TARGET="$STRIP_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_STRIP_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
STRIP_FOR_TARGET=$ac_cv_prog_STRIP_FOR_TARGET  
if test -n "$STRIP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP_FOR_TARGET" >&5  
$as_echo "$STRIP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_STRIP_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_STRIP_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$STRIP_FOR_TARGET"; then  
  ac_cv_prog_STRIP_FOR_TARGET="$STRIP_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_STRIP_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
STRIP_FOR_TARGET=$ac_cv_prog_STRIP_FOR_TARGET  
if test -n "$STRIP_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $STRIP_FOR_TARGET" >&5  
$as_echo "$STRIP_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_STRIP_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_STRIP_FOR_TARGET" ; then  
  set dummy strip  
  if test $build = $target ; then  
    STRIP_FOR_TARGET="$2"  
  else  
    STRIP_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  STRIP_FOR_TARGET="$ac_cv_prog_STRIP_FOR_TARGET"  
fi  
  
else  
  STRIP_FOR_TARGET=$ac_cv_path_STRIP_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_WINDRES_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for windres in $with_build_time_tools" >&5  
$as_echo_n "checking for windres in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/windres; then  
      WINDRES_FOR_TARGET=`cd $with_build_time_tools && pwd`/windres  
      ac_cv_path_WINDRES_FOR_TARGET=$WINDRES_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_WINDRES_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_WINDRES_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    WINDRES_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=windres`  
    test $WINDRES_FOR_TARGET = windres && WINDRES_FOR_TARGET=  
    test -n "$WINDRES_FOR_TARGET" && ac_cv_path_WINDRES_FOR_TARGET=$WINDRES_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_WINDRES_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "windres", so it can be a program name with args.  
set dummy windres; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_WINDRES_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $WINDRES_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_WINDRES_FOR_TARGET="$WINDRES_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_WINDRES_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
WINDRES_FOR_TARGET=$ac_cv_path_WINDRES_FOR_TARGET  
if test -n "$WINDRES_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES_FOR_TARGET" >&5  
$as_echo "$WINDRES_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_WINDRES_FOR_TARGET" ; then  
  
  
if test -n "$WINDRES_FOR_TARGET"; then  
  ac_cv_prog_WINDRES_FOR_TARGET=$WINDRES_FOR_TARGET  
elif test -n "$ac_cv_prog_WINDRES_FOR_TARGET"; then  
  WINDRES_FOR_TARGET=$ac_cv_prog_WINDRES_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_WINDRES_FOR_TARGET"; then  
  for ncn_progname in windres; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDRES_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDRES_FOR_TARGET"; then  
  ac_cv_prog_WINDRES_FOR_TARGET="$WINDRES_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDRES_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDRES_FOR_TARGET=$ac_cv_prog_WINDRES_FOR_TARGET  
if test -n "$WINDRES_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES_FOR_TARGET" >&5  
$as_echo "$WINDRES_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_WINDRES_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in windres; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_WINDRES_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_WINDRES_FOR_TARGET"; then  
  for ncn_progname in windres; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDRES_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDRES_FOR_TARGET"; then  
  ac_cv_prog_WINDRES_FOR_TARGET="$WINDRES_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDRES_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDRES_FOR_TARGET=$ac_cv_prog_WINDRES_FOR_TARGET  
if test -n "$WINDRES_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES_FOR_TARGET" >&5  
$as_echo "$WINDRES_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_WINDRES_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDRES_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDRES_FOR_TARGET"; then  
  ac_cv_prog_WINDRES_FOR_TARGET="$WINDRES_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDRES_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDRES_FOR_TARGET=$ac_cv_prog_WINDRES_FOR_TARGET  
if test -n "$WINDRES_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDRES_FOR_TARGET" >&5  
$as_echo "$WINDRES_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_WINDRES_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_WINDRES_FOR_TARGET" ; then  
  set dummy windres  
  if test $build = $target ; then  
    WINDRES_FOR_TARGET="$2"  
  else  
    WINDRES_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  WINDRES_FOR_TARGET="$ac_cv_prog_WINDRES_FOR_TARGET"  
fi  
  
else  
  WINDRES_FOR_TARGET=$ac_cv_path_WINDRES_FOR_TARGET  
fi  
  
  
  
  
if test -z "$ac_cv_path_WINDMC_FOR_TARGET" ; then  
  if test -n "$with_build_time_tools"; then  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for windmc in $with_build_time_tools" >&5  
$as_echo_n "checking for windmc in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/windmc; then  
      WINDMC_FOR_TARGET=`cd $with_build_time_tools && pwd`/windmc  
      ac_cv_path_WINDMC_FOR_TARGET=$WINDMC_FOR_TARGET  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: $ac_cv_path_WINDMC_FOR_TARGET" >&5  
$as_echo "$ac_cv_path_WINDMC_FOR_TARGET" >&6; }  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  elif test $build != $host && test $have_gcc_for_target = yes; then  
    WINDMC_FOR_TARGET=`$GCC_FOR_TARGET --print-prog-name=windmc`  
    test $WINDMC_FOR_TARGET = windmc && WINDMC_FOR_TARGET=  
    test -n "$WINDMC_FOR_TARGET" && ac_cv_path_WINDMC_FOR_TARGET=$WINDMC_FOR_TARGET  
  fi  
fi  
if test -z "$ac_cv_path_WINDMC_FOR_TARGET" && test -n "$gcc_cv_tool_dirs"; then  
  # Extract the first word of "windmc", so it can be a program name with args.  
set dummy windmc; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_path_WINDMC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  case $WINDMC_FOR_TARGET in  
  [\\/]* | ?:[\\/]*)  
  ac_cv_path_WINDMC_FOR_TARGET="$WINDMC_FOR_TARGET" # Let the user override the test with a path.  
  ;;  
  *)  
  as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $gcc_cv_tool_dirs  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_path_WINDMC_FOR_TARGET="$as_dir/$ac_word$ac_exec_ext"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
  ;;  
esac  
fi  
WINDMC_FOR_TARGET=$ac_cv_path_WINDMC_FOR_TARGET  
if test -n "$WINDMC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC_FOR_TARGET" >&5  
$as_echo "$WINDMC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
fi  
if test -z "$ac_cv_path_WINDMC_FOR_TARGET" ; then  
  
  
if test -n "$WINDMC_FOR_TARGET"; then  
  ac_cv_prog_WINDMC_FOR_TARGET=$WINDMC_FOR_TARGET  
elif test -n "$ac_cv_prog_WINDMC_FOR_TARGET"; then  
  WINDMC_FOR_TARGET=$ac_cv_prog_WINDMC_FOR_TARGET  
fi  
  
if test -n "$ac_cv_prog_WINDMC_FOR_TARGET"; then  
  for ncn_progname in windmc; do  
    # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDMC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDMC_FOR_TARGET"; then  
  ac_cv_prog_WINDMC_FOR_TARGET="$WINDMC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDMC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDMC_FOR_TARGET=$ac_cv_prog_WINDMC_FOR_TARGET  
if test -n "$WINDMC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC_FOR_TARGET" >&5  
$as_echo "$WINDMC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
  done  
fi  
  
if test -z "$ac_cv_prog_WINDMC_FOR_TARGET" && test -n "$with_build_time_tools"; then  
  for ncn_progname in windmc; do  
    { $as_echo "$as_me:${as_lineno-$LINENO}: checking for ${ncn_progname} in $with_build_time_tools" >&5  
$as_echo_n "checking for ${ncn_progname} in $with_build_time_tools... " >&6; }  
    if test -x $with_build_time_tools/${ncn_progname}; then  
      ac_cv_prog_WINDMC_FOR_TARGET=$with_build_time_tools/${ncn_progname}  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: yes" >&5  
$as_echo "yes" >&6; }  
      break  
    else  
      { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
    fi  
  done  
fi  
  
if test -z "$ac_cv_prog_WINDMC_FOR_TARGET"; then  
  for ncn_progname in windmc; do  
    if test -n "$ncn_target_tool_prefix"; then  
      # Extract the first word of "${ncn_target_tool_prefix}${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_target_tool_prefix}${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDMC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDMC_FOR_TARGET"; then  
  ac_cv_prog_WINDMC_FOR_TARGET="$WINDMC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDMC_FOR_TARGET="${ncn_target_tool_prefix}${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDMC_FOR_TARGET=$ac_cv_prog_WINDMC_FOR_TARGET  
if test -n "$WINDMC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC_FOR_TARGET" >&5  
$as_echo "$WINDMC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    if test -z "$ac_cv_prog_WINDMC_FOR_TARGET" && test $build = $target ; then  
      # Extract the first word of "${ncn_progname}", so it can be a program name with args.  
set dummy ${ncn_progname}; ac_word=$2  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5  
$as_echo_n "checking for $ac_word... " >&6; }  
if ${ac_cv_prog_WINDMC_FOR_TARGET+:} false; then :  
  $as_echo_n "(cached) " >&6  
else  
  if test -n "$WINDMC_FOR_TARGET"; then  
  ac_cv_prog_WINDMC_FOR_TARGET="$WINDMC_FOR_TARGET" # Let the user override the test.  
else  
as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    for ac_exec_ext in '' $ac_executable_extensions; do  
  if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then  
    ac_cv_prog_WINDMC_FOR_TARGET="${ncn_progname}"  
    $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext" >&5  
    break 2  
  fi  
done  
  done  
IFS=$as_save_IFS  
  
fi  
fi  
WINDMC_FOR_TARGET=$ac_cv_prog_WINDMC_FOR_TARGET  
if test -n "$WINDMC_FOR_TARGET"; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: $WINDMC_FOR_TARGET" >&5  
$as_echo "$WINDMC_FOR_TARGET" >&6; }  
else  
  { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5  
$as_echo "no" >&6; }  
fi  
  
  
    fi  
    test -n "$ac_cv_prog_WINDMC_FOR_TARGET" && break  
  done  
fi  
  
if test -z "$ac_cv_prog_WINDMC_FOR_TARGET" ; then  
  set dummy windmc  
  if test $build = $target ; then  
    WINDMC_FOR_TARGET="$2"  
  else  
    WINDMC_FOR_TARGET="${ncn_target_tool_prefix}$2"  
  fi  
else  
  WINDMC_FOR_TARGET="$ac_cv_prog_WINDMC_FOR_TARGET"  
fi  
  
else  
  WINDMC_FOR_TARGET=$ac_cv_path_WINDMC_FOR_TARGET  
fi  
  
  
RAW_CXX_FOR_TARGET="$CXX_FOR_TARGET"  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target ar" >&5  
$as_echo_n "checking where to find the target ar... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$AR_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $AR_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    AR_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/ar'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$AR_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $AR_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    AR_FOR_TARGET='$(AR)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target as" >&5  
$as_echo_n "checking where to find the target as... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$AS_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $AS_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gas "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    AS_FOR_TARGET='$$r/$(HOST_SUBDIR)/gas/as-new'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$AS_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $AS_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    AS_FOR_TARGET='$(AS)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target cc" >&5  
$as_echo_n "checking where to find the target cc... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$CC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $CC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    CC_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/xgcc -B$$r/$(HOST_SUBDIR)/gcc/'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$CC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $CC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    CC_FOR_TARGET='$(CC)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target c++" >&5  
$as_echo_n "checking where to find the target c++... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$CXX_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $CXX_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  case ,${enable_languages}, in  
    *,c++,*) ;;  
    *) ok=no ;;  
  esac  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    CXX_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/xg++ -B$$r/$(HOST_SUBDIR)/gcc/ -nostdinc++ `if test -f $$r/$(TARGET_SUBDIR)/libstdc++-v3/scripts/testsuite_flags; then $(SHELL) $$r/$(TARGET_SUBDIR)/libstdc++-v3/scripts/testsuite_flags --build-includes; else echo -funconfigured-libstdc++-v3 ; fi` -L$$r/$(TARGET_SUBDIR)/libstdc++-v3/src -L$$r/$(TARGET_SUBDIR)/libstdc++-v3/src/.libs -L$$r/$(TARGET_SUBDIR)/libstdc++-v3/libsupc++/.libs'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$CXX_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $CXX_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    CXX_FOR_TARGET='$(CXX)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target c++ for libstdc++" >&5  
$as_echo_n "checking where to find the target c++ for libstdc++... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$RAW_CXX_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $RAW_CXX_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  case ,${enable_languages}, in  
    *,c++,*) ;;  
    *) ok=no ;;  
  esac  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    RAW_CXX_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/xgcc -shared-libgcc -B$$r/$(HOST_SUBDIR)/gcc -nostdinc++ -L$$r/$(TARGET_SUBDIR)/libstdc++-v3/src -L$$r/$(TARGET_SUBDIR)/libstdc++-v3/src/.libs -L$$r/$(TARGET_SUBDIR)/libstdc++-v3/libsupc++/.libs'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$RAW_CXX_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $RAW_CXX_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    RAW_CXX_FOR_TARGET='$(CXX)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target dlltool" >&5  
$as_echo_n "checking where to find the target dlltool... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$DLLTOOL_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $DLLTOOL_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    DLLTOOL_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/dlltool'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$DLLTOOL_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $DLLTOOL_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    DLLTOOL_FOR_TARGET='$(DLLTOOL)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target dsymutil" >&5  
$as_echo_n "checking where to find the target dsymutil... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$DSYMUTIL_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $DSYMUTIL_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  if expr "x$DSYMUTIL_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $DSYMUTIL_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    DSYMUTIL_FOR_TARGET='$(DSYMUTIL)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target gcc" >&5  
$as_echo_n "checking where to find the target gcc... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$GCC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GCC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    GCC_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/xgcc -B$$r/$(HOST_SUBDIR)/gcc/'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$GCC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GCC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    GCC_FOR_TARGET='$()'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target gfortran" >&5  
$as_echo_n "checking where to find the target gfortran... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$GFORTRAN_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GFORTRAN_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  case ,${enable_languages}, in  
    *,fortran,*) ;;  
    *) ok=no ;;  
  esac  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    GFORTRAN_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/gfortran -B$$r/$(HOST_SUBDIR)/gcc/'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$GFORTRAN_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GFORTRAN_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    GFORTRAN_FOR_TARGET='$(GFORTRAN)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target gccgo" >&5  
$as_echo_n "checking where to find the target gccgo... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$GOC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GOC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  case ,${enable_languages}, in  
    *,go,*) ;;  
    *) ok=no ;;  
  esac  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    GOC_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/gccgo -B$$r/$(HOST_SUBDIR)/gcc/'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$GOC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GOC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    GOC_FOR_TARGET='$(GOC)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target gdc" >&5  
$as_echo_n "checking where to find the target gdc... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$GDC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GDC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  case ,${enable_languages}, in  
    *,d,*) ;;  
    *) ok=no ;;  
  esac  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    GDC_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/gdc -B$$r/$(HOST_SUBDIR)/gcc/'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$GDC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GDC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    GDC_FOR_TARGET='$(GDC)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target gm2" >&5  
$as_echo_n "checking where to find the target gm2... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$GM2_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GM2_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" gcc "*) ;;  
    *) ok=no ;;  
  esac  
  case ,${enable_languages}, in  
    *,m2,*) ;;  
    *) ok=no ;;  
  esac  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    GM2_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/gm2 -B$$r/$(HOST_SUBDIR)/gcc/'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$GM2_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $GM2_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    GM2_FOR_TARGET='$(GM2)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target ld" >&5  
$as_echo_n "checking where to find the target ld... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$LD_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $LD_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" ld "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    LD_FOR_TARGET='$$r/$(HOST_SUBDIR)/ld/ld-new'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$LD_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $LD_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    LD_FOR_TARGET='$(LD)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target lipo" >&5  
$as_echo_n "checking where to find the target lipo... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$LIPO_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $LIPO_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  if expr "x$LIPO_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $LIPO_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    LIPO_FOR_TARGET='$(LIPO)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target nm" >&5  
$as_echo_n "checking where to find the target nm... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$NM_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $NM_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    NM_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/nm-new'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$NM_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $NM_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    NM_FOR_TARGET='$(NM)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target objcopy" >&5  
$as_echo_n "checking where to find the target objcopy... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$OBJCOPY_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $OBJCOPY_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    OBJCOPY_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/objcopy'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$OBJCOPY_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $OBJCOPY_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    OBJCOPY_FOR_TARGET='$(OBJCOPY)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target objdump" >&5  
$as_echo_n "checking where to find the target objdump... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$OBJDUMP_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $OBJDUMP_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    OBJDUMP_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/objdump'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$OBJDUMP_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $OBJDUMP_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    OBJDUMP_FOR_TARGET='$(OBJDUMP)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target otool" >&5  
$as_echo_n "checking where to find the target otool... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$OTOOL_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $OTOOL_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  if expr "x$OTOOL_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $OTOOL_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    OTOOL_FOR_TARGET='$(OTOOL)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target ranlib" >&5  
$as_echo_n "checking where to find the target ranlib... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$RANLIB_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $RANLIB_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    RANLIB_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/ranlib'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$RANLIB_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $RANLIB_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    RANLIB_FOR_TARGET='$(RANLIB)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target readelf" >&5  
$as_echo_n "checking where to find the target readelf... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$READELF_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $READELF_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    READELF_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/readelf'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$READELF_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $READELF_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    READELF_FOR_TARGET='$(READELF)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target strip" >&5  
$as_echo_n "checking where to find the target strip... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$STRIP_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $STRIP_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    STRIP_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/strip-new'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$STRIP_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $STRIP_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    STRIP_FOR_TARGET='$(STRIP)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target windres" >&5  
$as_echo_n "checking where to find the target windres... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$WINDRES_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $WINDRES_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    WINDRES_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/windres'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$WINDRES_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $WINDRES_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    WINDRES_FOR_TARGET='$(WINDRES)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking where to find the target windmc" >&5  
$as_echo_n "checking where to find the target windmc... " >&6; }  
if test "x${build}" != "x${host}" ; then  
  if expr "x$WINDMC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $WINDMC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  else  
    # Canadian cross, just use what we found  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
else  
  ok=yes  
  case " ${configdirs} " in  
    *" binutils "*) ;;  
    *) ok=no ;;  
  esac  
  
  if test $ok = yes; then  
    # An in-tree tool is available and we can use it  
    WINDMC_FOR_TARGET='$$r/$(HOST_SUBDIR)/binutils/windmc'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: just compiled" >&5  
$as_echo "just compiled" >&6; }  
  elif expr "x$WINDMC_FOR_TARGET" : "x/" > /dev/null; then  
    # We already found the complete path  
    ac_dir=`dirname $WINDMC_FOR_TARGET`  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed in $ac_dir" >&5  
$as_echo "pre-installed in $ac_dir" >&6; }  
  elif test "x$target" = "x$host"; then  
    # We can use an host tool  
    WINDMC_FOR_TARGET='$(WINDMC)'  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: host tool" >&5  
$as_echo "host tool" >&6; }  
  else  
    # We need a cross tool  
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: pre-installed" >&5  
$as_echo "pre-installed" >&6; }  
  fi  
fi  
  
  
  
  
  
# Certain tools may need extra flags.  
AR_FOR_TARGET=${AR_FOR_TARGET}${extra_arflags_for_target}  
RANLIB_FOR_TARGET=${RANLIB_FOR_TARGET}${extra_ranlibflags_for_target}  
NM_FOR_TARGET=${NM_FOR_TARGET}${extra_nmflags_for_target}  
  
# When building target libraries, except in a Canadian cross, we use  
# the same toolchain as the compiler we just built.  
COMPILER_AS_FOR_TARGET='$(AS_FOR_TARGET)'  
COMPILER_LD_FOR_TARGET='$(LD_FOR_TARGET)'  
COMPILER_NM_FOR_TARGET='$(NM_FOR_TARGET)'  
if test $host = $build; then  
  case " $configdirs " in  
    *" gcc "*)  
      COMPILER_AS_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/as'  
      COMPILER_LD_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/collect-ld'  
      COMPILER_NM_FOR_TARGET='$$r/$(HOST_SUBDIR)/gcc/nm'${extra_nmflags_for_target}  
      ;;  
  esac  
fi  
  
  
  
  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: checking whether to enable maintainer-specific portions of Makefiles" >&5  
$as_echo_n "checking whether to enable maintainer-specific portions of Makefiles... " >&6; }  
# Check whether --enable-maintainer-mode was given.  
if test "${enable_maintainer_mode+set}" = set; then :  
  enableval=$enable_maintainer_mode; USE_MAINTAINER_MODE=$enableval  
else  
  USE_MAINTAINER_MODE=no  
fi  
  
{ $as_echo "$as_me:${as_lineno-$LINENO}: result: $USE_MAINTAINER_MODE" >&5  
$as_echo "$USE_MAINTAINER_MODE" >&6; }  
  
  
if test "$USE_MAINTAINER_MODE" = yes; then  
  MAINTAINER_MODE_TRUE=  
  MAINTAINER_MODE_FALSE='#'  
else  
  MAINTAINER_MODE_TRUE='#'  
  MAINTAINER_MODE_FALSE=  
fi  
MAINT=$MAINTAINER_MODE_TRUE  
  
# ---------------------  
# GCC bootstrap support  
# ---------------------  
  
# Stage specific cflags for build.  
stage1_cflags="-g"  
case $build in  
  vax-*-*)  
    case ${GCC} in  
      yes) stage1_cflags="-g -Wa,-J" ;;  
      *) stage1_cflags="-g -J" ;;  
    esac ;;  
esac  
  
  
  
# Enable --enable-checking in stage1 of the compiler.  
# Check whether --enable-stage1-checking was given.  
if test "${enable_stage1_checking+set}" = set; then :  
  enableval=$enable_stage1_checking; stage1_checking=--enable-checking=${enable_stage1_checking}  
else  
  if test "x$enable_checking" = xno || test "x$enable_checking" = x; then  
  # For --disable-checking or implicit --enable-checking=release, avoid  
  # setting --enable-checking=gc in the default stage1 checking for LTO  
  # bootstraps.  See PR62077.  
  case $BUILD_CONFIG in  
    *lto*)  
      stage1_checking=--enable-checking=release,misc,gimple,rtlflag,tree,types;;  
    *)  
      stage1_checking=--enable-checking=yes,types;;  
  esac  
  if test "x$enable_checking" = x && \  
     test -d ${srcdir}/gcc && \  
     test x"`cat ${srcdir}/gcc/DEV-PHASE`" = xexperimental; then  
    stage1_checking=--enable-checking=yes,types,extra  
  fi  
else  
  stage1_checking=--enable-checking=$enable_checking,types  
fi  
fi  
  
  
  
# Enable -Werror in bootstrap stage2 and later.  
# Check whether --enable-werror was given.  
if test "${enable_werror+set}" = set; then :  
  enableval=$enable_werror;  
case ${enable_werror} in  
  yes) stage2_werror_flag="--enable-werror-always" ;;  
  *) stage2_werror_flag="" ;;  
esac  
  
else  
  
if test -d ${srcdir}/gcc && test x"`cat $srcdir/gcc/DEV-PHASE`" = xexperimental; then  
  case $BUILD_CONFIG in  
  bootstrap-debug)  
      stage2_werror_flag="--enable-werror-always" ;;  
  "")  
      stage2_werror_flag="--enable-werror-always" ;;  
  esac  
fi  
  
fi  
  
  
  
  
# Specify what files to not compare during bootstrap.  
  
compare_exclusions="gcc/cc*-checksum\$(objext) | gcc/ada/*tools/*"  
compare_exclusions="$compare_exclusions | gcc/m2/gm2-compiler-boot/M2Version*"  
compare_exclusions="$compare_exclusions | gcc/m2/gm2-compiler-boot/SYSTEM*"  
compare_exclusions="$compare_exclusions | gcc/m2/gm2version*"  
case "$target" in  
  hppa*64*-*-hpux*) ;;  
  powerpc*-ibm-aix*) compare_exclusions="$compare_exclusions | *libgomp*\$(objext)" ;;  
esac  
  
  
ac_config_files="$ac_config_files Makefile"  
  
cat >confcache \<<\_ACEOF  
# This file is a shell script that caches the results of configure  
# tests run on this system so they can be shared between configure  
# scripts and configure runs, see configure's option --config-cache.  
# It is not useful on other systems.  If it contains results you don't  
# want to keep, you may remove or edit it.  
#  
# config.status only pays attention to the cache file if you give it  
# the --recheck option to rerun configure.  
#  
# `ac_cv_env_foo' variables (set or unset) will be overridden when  
# loading this file, other *unset* `ac_cv_foo' will be assigned the  
# following values.  
  
_ACEOF  
  
# The following way of writing the cache mishandles newlines in values,  
# but we know of no workaround that is simple, portable, and efficient.  
# So, we kill variables containing newlines.  
# Ultrix sh set writes to stderr and can't be redirected directly,  
# and sets the high bit in the cache file unless we assign to the vars.  
(  
  for ac_var in `(set) 2>&1 | sed -n 's/^\([a-zA-Z_][a-zA-Z0-9_]*\)=.*/\1/p'`; do  
    eval ac_val=\$$ac_var  
    case $ac_val in #(  
    *${as_nl}*)  
      case $ac_var in #(  
      *_cv_*) { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: cache variable $ac_var contains a newline" >&5  
$as_echo "$as_me: WARNING: cache variable $ac_var contains a newline" >&2;} ;;  
      esac  
      case $ac_var in #(  
      _ | IFS | as_nl) ;; #(  
      BASH_ARGV | BASH_SOURCE) eval $ac_var= ;; #(  
      *) { eval $ac_var=; unset $ac_var;} ;;  
      esac ;;  
    esac  
  done  
  
  (set) 2>&1 |  
    case $as_nl`(ac_space=' '; set) 2>&1` in #(  
    *${as_nl}ac_space=\ *)  
      # `set' does not quote correctly, so add quotes: double-quote  
      # substitution turns \\\\ into \\, and sed turns \\ into \.  
      sed -n \  
    "s/'/'\\\\''/g;  
      s/^\\([_$as_cr_alnum]*_cv_[_$as_cr_alnum]*\\)=\\(.*\\)/\\1='\\2'/p"  
      ;; #(  
    *)  
      # `set' quotes correctly as required by POSIX, so do not add quotes.  
      sed -n "/^[_$as_cr_alnum]*_cv_[_$as_cr_alnum]*=/p"  
      ;;  
    esac |  
    sort  
) |  
  sed '  
     /^ac_cv_env_/b end  
     t clear  
     :clear  
     s/^\([^=]*\)=\(.*[{}].*\)$/test "${\1+set}" = set || &/  
     t end  
     s/^\([^=]*\)=\(.*\)$/\1=${\1=\2}/  
     :end' >>confcache  
if diff "$cache_file" confcache >/dev/null 2>&1; then :; else  
  if test -w "$cache_file"; then  
    if test "x$cache_file" != "x/dev/null"; then  
      { $as_echo "$as_me:${as_lineno-$LINENO}: updating cache $cache_file" >&5  
$as_echo "$as_me: updating cache $cache_file" >&6;}  
      if test ! -f "$cache_file" || test -h "$cache_file"; then  
    cat confcache >"$cache_file"  
      else  
        case $cache_file in #(  
        */* | ?:*)  
      mv -f confcache "$cache_file"$$ &&  
      mv -f "$cache_file"$$ "$cache_file" ;; #(  
        *)  
      mv -f confcache "$cache_file" ;;  
    esac  
      fi  
    fi  
  else  
    { $as_echo "$as_me:${as_lineno-$LINENO}: not updating unwritable cache $cache_file" >&5  
$as_echo "$as_me: not updating unwritable cache $cache_file" >&6;}  
  fi  
fi  
rm -f confcache  
  
test "x$prefix" = xNONE && prefix=$ac_default_prefix  
# Let make expand exec_prefix.  
test "x$exec_prefix" = xNONE && exec_prefix='${prefix}'  
  
# Transform confdefs.h into DEFS.  
# Protect against shell expansion while executing Makefile rules.  
# Protect against Makefile macro expansion.  
#  
# If the first sed substitution is executed (which looks for macros that  
# take arguments), then branch to the quote section.  Otherwise,  
# look for a macro that doesn't take arguments.  
ac_script='  
:mline  
/\\$/{  
 N  
 s,\\\n,,  
 b mline  
}  
t clear  
:clear  
s/^[     ]*#[     ]*define[     ][     ]*\([^     (][^     (]*([^)]*)\)[     ]*\(.*\)/-D\1=\2/g  
t quote  
s/^[     ]*#[     ]*define[     ][     ]*\([^     ][^     ]*\)[     ]*\(.*\)/-D\1=\2/g  
t quote  
b any  
:quote  
s/[     `~#$^&*(){}\\|;'\''"<>?]/\\&/g  
s/\[/\\&/g  
s/\]/\\&/g  
s/\$/$$/g  
H  
:any  
${  
    g  
    s/^\n//  
    s/\n/ /g  
    p  
}  
'  
DEFS=`sed -n "$ac_script" confdefs.h`  
  
  
ac_libobjs=  
ac_ltlibobjs=  
U=  
for ac_i in : $LIBOBJS; do test "x$ac_i" = x: && continue  
  # 1. Remove the extension, and $U if already installed.  
  ac_script='s/\$U\././;s/\.o$//;s/\.obj$//'  
  ac_i=`$as_echo "$ac_i" | sed "$ac_script"`  
  # 2. Prepend LIBOBJDIR.  When used with automake>=1.10 LIBOBJDIR  
  #    will be set to the directory where LIBOBJS objects are built.  
  as_fn_append ac_libobjs " \${LIBOBJDIR}$ac_i\$U.$ac_objext"  
  as_fn_append ac_ltlibobjs " \${LIBOBJDIR}$ac_i"'$U.lo'  
done  
LIBOBJS=$ac_libobjs  
  
LTLIBOBJS=$ac_ltlibobjs  
  
  
  
: "${CONFIG_STATUS=./config.status}"  
ac_write_fail=0  
ac_clean_files_save=$ac_clean_files  
ac_clean_files="$ac_clean_files $CONFIG_STATUS"  
{ $as_echo "$as_me:${as_lineno-$LINENO}: creating $CONFIG_STATUS" >&5  
$as_echo "$as_me: creating $CONFIG_STATUS" >&6;}  
as_write_fail=0  
cat >$CONFIG_STATUS \<<_ASEOF || as_write_fail=1  
#! $SHELL  
# Generated by $as_me.  
# Run this file to recreate the current configuration.  
# Compiler output produced by configure, useful for debugging  
# configure, is in config.log if it exists.  
  
debug=false  
ac_cs_recheck=false  
ac_cs_silent=false  
  
SHELL=\${CONFIG_SHELL-$SHELL}  
export SHELL  
_ASEOF  
cat >>$CONFIG_STATUS \<<\_ASEOF || as_write_fail=1  
## -------------------- ##  
## M4sh Initialization. ##  
## -------------------- ##  
  
# Be more Bourne compatible  
DUALCASE=1; export DUALCASE # for MKS sh  
if test -n "${ZSH_VERSION+set}" && (emulate sh) >/dev/null 2>&1; then :  
  emulate sh  
  NULLCMD=:  
  # Pre-4.2 versions of Zsh do word splitting on ${1+"$@"}, which  
  # is contrary to our usage.  Disable this feature.  
  alias -g '${1+"$@"}'='"$@"'  
  setopt NO_GLOB_SUBST  
else  
  case `(set -o) 2>/dev/null` in #(  
  *posix*) :  
    set -o posix ;; #(  
  *) :  
     ;;  
esac  
fi  
  
  
as_nl='  
'  
export as_nl  
# Printing a long string crashes Solaris 7 /usr/bin/printf.  
as_echo='\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\'  
as_echo=$as_echo$as_echo$as_echo$as_echo$as_echo  
as_echo=$as_echo$as_echo$as_echo$as_echo$as_echo$as_echo  
# Prefer a ksh shell builtin over an external printf program on Solaris,  
# but without wasting forks for bash or zsh.  
if test -z "$BASH_VERSION$ZSH_VERSION" \  
    && (test "X`print -r -- $as_echo`" = "X$as_echo") 2>/dev/null; then  
  as_echo='print -r --'  
  as_echo_n='print -rn --'  
elif (test "X`printf %s $as_echo`" = "X$as_echo") 2>/dev/null; then  
  as_echo='printf %s\n'  
  as_echo_n='printf %s'  
else  
  if test "X`(/usr/ucb/echo -n -n $as_echo) 2>/dev/null`" = "X-n $as_echo"; then  
    as_echo_body='eval /usr/ucb/echo -n "$1$as_nl"'  
    as_echo_n='/usr/ucb/echo -n'  
  else  
    as_echo_body='eval expr "X$1" : "X\\(.*\\)"'  
    as_echo_n_body='eval  
      arg=$1;  
      case $arg in #(  
      *"$as_nl"*)  
    expr "X$arg" : "X\\(.*\\)$as_nl";  
    arg=`expr "X$arg" : ".*$as_nl\\(.*\\)"`;;  
      esac;  
      expr "X$arg" : "X\\(.*\\)" | tr -d "$as_nl"  
    '  
    export as_echo_n_body  
    as_echo_n='sh -c $as_echo_n_body as_echo'  
  fi  
  export as_echo_body  
  as_echo='sh -c $as_echo_body as_echo'  
fi  
  
# The user is always right.  
if test "${PATH_SEPARATOR+set}" != set; then  
  PATH_SEPARATOR=:  
  (PATH='/bin;/bin'; FPATH=$PATH; sh -c :) >/dev/null 2>&1 && {  
    (PATH='/bin:/bin'; FPATH=$PATH; sh -c :) >/dev/null 2>&1 ||  
      PATH_SEPARATOR=';'  
  }  
fi  
  
  
# IFS  
# We need space, tab and new line, in precisely that order.  Quoting is  
# there to prevent editors from complaining about space-tab.  
# (If _AS_PATH_WALK were called with IFS unset, it would disable word  
# splitting by setting IFS to empty value.)  
IFS=" ""    $as_nl"  
  
# Find who we are.  Look in the path if we contain no directory separator.  
as_myself=  
case $0 in #((  
  *[\\/]* ) as_myself=$0 ;;  
  *) as_save_IFS=$IFS; IFS=$PATH_SEPARATOR  
for as_dir in $PATH  
do  
  IFS=$as_save_IFS  
  test -z "$as_dir" && as_dir=.  
    test -r "$as_dir/$0" && as_myself=$as_dir/$0 && break  
  done  
IFS=$as_save_IFS  
  
     ;;  
esac  
# We did not find ourselves, most probably we were run as `sh COMMAND'  
# in which case we are not to be found in the path.  
if test "x$as_myself" = x; then  
  as_myself=$0  
fi  
if test ! -f "$as_myself"; then  
  $as_echo "$as_myself: error: cannot find myself; rerun with an absolute file name" >&2  
  exit 1  
fi  
  
# Unset variables that we do not need and which cause bugs (e.g. in  
# pre-3.0 UWIN ksh).  But do not cause bugs in bash 2.01; the "|| exit 1"  
# suppresses any "Segmentation fault" message there.  '((' could  
# trigger a bug in pdksh 5.2.14.  
for as_var in BASH_ENV ENV MAIL MAILPATH  
do eval test x\${$as_var+set} = xset \  
  && ( (unset $as_var) || exit 1) >/dev/null 2>&1 && unset $as_var || :  
done  
PS1='$ '  
PS2='> '  
PS4='+ '  
  
# NLS nuisances.  
LC_ALL=C  
export LC_ALL  
LANGUAGE=C  
export LANGUAGE  
  
# CDPATH.  
(unset CDPATH) >/dev/null 2>&1 && unset CDPATH  
  
  
# as_fn_error STATUS ERROR [LINENO LOG_FD]  
# ----------------------------------------  
# Output "`basename $0`: error: ERROR" to stderr. If LINENO and LOG_FD are  
# provided, also output the error to LOG_FD, referencing LINENO. Then exit the  
# script with STATUS, using 1 if that was 0.  
as_fn_error ()  
{  
  as_status=$1; test $as_status -eq 0 && as_status=1  
  if test "$4"; then  
    as_lineno=${as_lineno-"$3"} as_lineno_stack=as_lineno_stack=$as_lineno_stack  
    $as_echo "$as_me:${as_lineno-$LINENO}: error: $2" >&$4  
  fi  
  $as_echo "$as_me: error: $2" >&2  
  as_fn_exit $as_status  
} # as_fn_error  
  
  
# as_fn_set_status STATUS  
# -----------------------  
# Set $? to STATUS, without forking.  
as_fn_set_status ()  
{  
  return $1  
} # as_fn_set_status  
  
# as_fn_exit STATUS  
# -----------------  
# Exit the shell with STATUS, even in a "trap 0" or "set -e" context.  
as_fn_exit ()  
{  
  set +e  
  as_fn_set_status $1  
  exit $1  
} # as_fn_exit  
  
# as_fn_unset VAR  
# ---------------  
# Portably unset VAR.  
as_fn_unset ()  
{  
  { eval $1=; unset $1;}  
}  
as_unset=as_fn_unset  
# as_fn_append VAR VALUE  
# ----------------------  
# Append the text in VALUE to the end of the definition contained in VAR. Take  
# advantage of any shell optimizations that allow amortized linear growth over  
# repeated appends, instead of the typical quadratic growth present in naive  
# implementations.  
if (eval "as_var=1; as_var+=2; test x\$as_var = x12") 2>/dev/null; then :  
  eval 'as_fn_append ()  
  {  
    eval $1+=\$2  
  }'  
else  
  as_fn_append ()  
  {  
    eval $1=\$$1\$2  
  }  
fi # as_fn_append  
  
# as_fn_arith ARG...  
# ------------------  
# Perform arithmetic evaluation on the ARGs, and store the result in the  
# global $as_val. Take advantage of shells that can avoid forks. The arguments  
# must be portable across $(()) and expr.  
if (eval "test \$(( 1 + 1 )) = 2") 2>/dev/null; then :  
  eval 'as_fn_arith ()  
  {  
    as_val=$(( $* ))  
  }'  
else  
  as_fn_arith ()  
  {  
    as_val=`expr "$@" || test $? -eq 1`  
  }  
fi # as_fn_arith  
  
  
if expr a : '\(a\)' >/dev/null 2>&1 &&  
   test "X`expr 00001 : '.*\(...\)'`" = X001; then  
  as_expr=expr  
else  
  as_expr=false  
fi  
  
if (basename -- /) >/dev/null 2>&1 && test "X`basename -- / 2>&1`" = "X/"; then  
  as_basename=basename  
else  
  as_basename=false  
fi  
  
if (as_dir=`dirname -- /` && test "X$as_dir" = X/) >/dev/null 2>&1; then  
  as_dirname=dirname  
else  
  as_dirname=false  
fi  
  
as_me=`$as_basename -- "$0" ||  
$as_expr X/"$0" : '.*/\([^/][^/]*\)/*$' \| \  
     X"$0" : 'X\(//\)$' \| \  
     X"$0" : 'X\(/\)' \| . 2>/dev/null ||  
$as_echo X/"$0" |  
    sed '/^.*\/\([^/][^/]*\)\/*$/{  
        s//\1/  
        q  
      }  
      /^X\/\(\/\/\)$/{  
        s//\1/  
        q  
      }  
      /^X\/\(\/\).*/{  
        s//\1/  
        q  
      }  
      s/.*/./; q'`  
  
# Avoid depending upon Character Ranges.  
as_cr_letters='abcdefghijklmnopqrstuvwxyz'  
as_cr_LETTERS='ABCDEFGHIJKLMNOPQRSTUVWXYZ'  
as_cr_Letters=$as_cr_letters$as_cr_LETTERS  
as_cr_digits='0123456789'  
as_cr_alnum=$as_cr_Letters$as_cr_digits  
  
ECHO_C= ECHO_N= ECHO_T=  
case `echo -n x` in #(((((  
-n*)  
  case `echo 'xy\c'` in  
  *c*) ECHO_T='    ';;    # ECHO_T is single tab character.  
  xy)  ECHO_C='\c';;  
  *)   echo `echo ksh88 bug on AIX 6.1` > /dev/null  
       ECHO_T='    ';;  
  esac;;  
*)  
  ECHO_N='-n';;  
esac  
  
rm -f conf$$ conf$$.exe conf$$.file  
if test -d conf$$.dir; then  
  rm -f conf$$.dir/conf$$.file  
else  
  rm -f conf$$.dir  
  mkdir conf$$.dir 2>/dev/null  
fi  
if (echo >conf$$.file) 2>/dev/null; then  
  if ln -s conf$$.file conf$$ 2>/dev/null; then  
    as_ln_s='ln -s'  
    # ... but there are two gotchas:  
    # 1) On MSYS, both `ln -s file dir' and `ln file dir' fail.  
    # 2) DJGPP < 2.04 has no symlinks; `ln -s' creates a wrapper executable.  
    # In both cases, we have to default to `cp -pR'.  
    ln -s conf$$.file conf$$.dir 2>/dev/null && test ! -f conf$$.exe ||  
      as_ln_s='cp -pR'  
  elif ln conf$$.file conf$$ 2>/dev/null; then  
    as_ln_s=ln  
  else  
    as_ln_s='cp -pR'  
  fi  
else  
  as_ln_s='cp -pR'  
fi  
rm -f conf$$ conf$$.exe conf$$.dir/conf$$.file conf$$.file  
rmdir conf$$.dir 2>/dev/null  
  
  
# as_fn_mkdir_p  
# -------------  
# Create "$as_dir" as a directory, including parents if necessary.  
as_fn_mkdir_p ()  
{  
  
  case $as_dir in #(  
  -*) as_dir=./$as_dir;;  
  esac  
  test -d "$as_dir" || eval $as_mkdir_p || {  
    as_dirs=  
    while :; do  
      case $as_dir in #(  
      *\'*) as_qdir=`$as_echo "$as_dir" | sed "s/'/'\\\\\\\\''/g"`;; #'(  
      *) as_qdir=$as_dir;;  
      esac  
      as_dirs="'$as_qdir' $as_dirs"  
      as_dir=`$as_dirname -- "$as_dir" ||  
$as_expr X"$as_dir" : 'X\(.*[^/]\)//*[^/][^/]*/*$' \| \  
     X"$as_dir" : 'X\(//\)[^/]' \| \  
     X"$as_dir" : 'X\(//\)$' \| \  
     X"$as_dir" : 'X\(/\)' \| . 2>/dev/null ||  
$as_echo X"$as_dir" |  
    sed '/^X\(.*[^/]\)\/\/*[^/][^/]*\/*$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)[^/].*/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\).*/{  
        s//\1/  
        q  
      }  
      s/.*/./; q'`  
      test -d "$as_dir" && break  
    done  
    test -z "$as_dirs" || eval "mkdir $as_dirs"  
  } || test -d "$as_dir" || as_fn_error $? "cannot create directory $as_dir"  
  
  
} # as_fn_mkdir_p  
if mkdir -p . 2>/dev/null; then  
  as_mkdir_p='mkdir -p "$as_dir"'  
else  
  test -d ./-p && rmdir ./-p  
  as_mkdir_p=false  
fi  
  
  
# as_fn_executable_p FILE  
# -----------------------  
# Test if FILE is an executable regular file.  
as_fn_executable_p ()  
{  
  test -f "$1" && test -x "$1"  
} # as_fn_executable_p  
as_test_x='test -x'  
as_executable_p=as_fn_executable_p  
  
# Sed expression to map a string onto a valid CPP name.  
as_tr_cpp="eval sed 'y%*$as_cr_letters%P$as_cr_LETTERS%;s%[^_$as_cr_alnum]%_%g'"  
  
# Sed expression to map a string onto a valid variable name.  
as_tr_sh="eval sed 'y%*+%pp%;s%[^_$as_cr_alnum]%_%g'"  
  
  
exec 6>&1  
## ----------------------------------- ##  
## Main body of $CONFIG_STATUS script. ##  
## ----------------------------------- ##  
_ASEOF  
test $as_write_fail = 0 && chmod +x $CONFIG_STATUS || ac_write_fail=1  
  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
# Save the log message, to keep $0 and so on meaningful, and to  
# report actual input values of CONFIG_FILES etc. instead of their  
# values after options handling.  
ac_log="  
This file was extended by $as_me, which was  
generated by GNU Autoconf 2.69.  Invocation command line was  
  
  CONFIG_FILES    = $CONFIG_FILES  
  CONFIG_HEADERS  = $CONFIG_HEADERS  
  CONFIG_LINKS    = $CONFIG_LINKS  
  CONFIG_COMMANDS = $CONFIG_COMMANDS  
  $ $0 $@  
  
on `(hostname || uname -n) 2>/dev/null | sed 1q`  
"  
  
_ACEOF  
  
case $ac_config_files in *"  
"*) set x $ac_config_files; shift; ac_config_files=$*;;  
esac  
  
  
  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
# Files that config.status was made for.  
config_files="$ac_config_files"  
  
_ACEOF  
  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
ac_cs_usage="\  
\`$as_me' instantiates files and other configuration actions  
from templates according to the current configuration.  Unless the files  
and actions are specified as TAGs, all are instantiated by default.  
  
Usage: $0 [OPTION]... [TAG]...  
  
  -h, --help       print this help, then exit  
  -V, --version    print version number and configuration settings, then exit  
      --config     print configuration, then exit  
  -q, --quiet, --silent  
                   do not print progress messages  
  -d, --debug      don't remove temporary files  
      --recheck    update $as_me by reconfiguring in the same conditions  
      --file=FILE[:TEMPLATE]  
                   instantiate the configuration file FILE  
  
Configuration files:  
$config_files  
  
Report bugs to the package provider."  
  
_ACEOF  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
ac_cs_config="`$as_echo "$ac_configure_args" | sed 's/^ //; s/[\\""\`\$]/\\\\&/g'`"  
ac_cs_version="\\  
config.status  
configured by $0, generated by GNU Autoconf 2.69,  
  with options \\"\$ac_cs_config\\"  
  
Copyright (C) 2012 Free Software Foundation, Inc.  
This config.status script is free software; the Free Software Foundation  
gives unlimited permission to copy, distribute and modify it."  
  
ac_pwd='$ac_pwd'  
srcdir='$srcdir'  
INSTALL='$INSTALL'  
AWK='$AWK'  
test -n "\$AWK" || AWK=awk  
_ACEOF  
  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
# The default lists apply if the user does not specify any file.  
ac_need_defaults=:  
while test $# != 0  
do  
  case $1 in  
  --*=?*)  
    ac_option=`expr "X$1" : 'X\([^=]*\)='`  
    ac_optarg=`expr "X$1" : 'X[^=]*=\(.*\)'`  
    ac_shift=:  
    ;;  
  --*=)  
    ac_option=`expr "X$1" : 'X\([^=]*\)='`  
    ac_optarg=  
    ac_shift=:  
    ;;  
  *)  
    ac_option=$1  
    ac_optarg=$2  
    ac_shift=shift  
    ;;  
  esac  
  
  case $ac_option in  
  # Handling of the options.  
  -recheck | --recheck | --rechec | --reche | --rech | --rec | --re | --r)  
    ac_cs_recheck=: ;;  
  --version | --versio | --versi | --vers | --ver | --ve | --v | -V )  
    $as_echo "$ac_cs_version"; exit ;;  
  --config | --confi | --conf | --con | --co | --c )  
    $as_echo "$ac_cs_config"; exit ;;  
  --debug | --debu | --deb | --de | --d | -d )  
    debug=: ;;  
  --file | --fil | --fi | --f )  
    $ac_shift  
    case $ac_optarg in  
    *\'*) ac_optarg=`$as_echo "$ac_optarg" | sed "s/'/'\\\\\\\\''/g"` ;;  
    '') as_fn_error $? "missing file argument" ;;  
    esac  
    as_fn_append CONFIG_FILES " '$ac_optarg'"  
    ac_need_defaults=false;;  
  --he | --h |  --help | --hel | -h )  
    $as_echo "$ac_cs_usage"; exit ;;  
  -q | -quiet | --quiet | --quie | --qui | --qu | --q \  
  | -silent | --silent | --silen | --sile | --sil | --si | --s)  
    ac_cs_silent=: ;;  
  
  # This is an error.  
  -*) as_fn_error $? "unrecognized option: \`$1'  
Try \`$0 --help' for more information." ;;  
  
  *) as_fn_append ac_config_targets " $1"  
     ac_need_defaults=false ;;  
  
  esac  
  shift  
done  
  
ac_configure_extra_args=  
  
if $ac_cs_silent; then  
  exec 6>/dev/null  
  ac_configure_extra_args="$ac_configure_extra_args --silent"  
fi  
  
_ACEOF  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
if \$ac_cs_recheck; then  
  set X $SHELL '$0' $ac_configure_args \$ac_configure_extra_args --no-create --no-recursion  
  shift  
  \$as_echo "running CONFIG_SHELL=$SHELL \$*" >&6  
  CONFIG_SHELL='$SHELL'  
  export CONFIG_SHELL  
  exec "\$@"  
fi  
  
_ACEOF  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
exec 5>>config.log  
{  
  echo  
  sed 'h;s/./-/g;s/^.../## /;s/...$/ ##/;p;x;p;x' \<<_ASBOX  
## Running $as_me. ##  
_ASBOX  
  $as_echo "$ac_log"  
} >&5  
  
_ACEOF  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
#  
# INIT-COMMANDS  
#  
extrasub_build="$extrasub_build"  
   extrasub_host="$extrasub_host"  
   extrasub_target="$extrasub_target"  
  
_ACEOF  
  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
  
# Handling of arguments.  
for ac_config_target in $ac_config_targets  
do  
  case $ac_config_target in  
    "Makefile") CONFIG_FILES="$CONFIG_FILES Makefile" ;;  
  
  *) as_fn_error $? "invalid argument: \`$ac_config_target'" "$LINENO" 5;;  
  esac  
done  
  
  
# If the user did not use the arguments to specify the items to instantiate,  
# then the envvar interface is used.  Set only those that are not.  
# We use the long form for the default assignment because of an extremely  
# bizarre bug on SunOS 4.1.3.  
if $ac_need_defaults; then  
  test "${CONFIG_FILES+set}" = set || CONFIG_FILES=$config_files  
fi  
  
# Have a temporary directory for convenience.  Make it in the build tree  
# simply because there is no reason against having it here, and in addition,  
# creating and moving files from /tmp can sometimes cause problems.  
# Hook for its removal unless debugging.  
# Note that there is a small window in which the directory will not be cleaned:  
# after its creation but before its name has been assigned to `$tmp'.  
$debug ||  
{  
  tmp= ac_tmp=  
  trap 'exit_status=$?  
  : "${ac_tmp:=$tmp}"  
  { test ! -d "$ac_tmp" || rm -fr "$ac_tmp"; } && exit $exit_status  
' 0  
  trap 'as_fn_exit 1' 1 2 13 15  
}  
# Create a (secure) tmp directory for tmp files.  
  
{  
  tmp=`(umask 077 && mktemp -d "./confXXXXXX") 2>/dev/null` &&  
  test -d "$tmp"  
}  ||  
{  
  tmp=./conf$$-$RANDOM  
  (umask 077 && mkdir "$tmp")  
} || as_fn_error $? "cannot create a temporary directory in ." "$LINENO" 5  
ac_tmp=$tmp  
  
# Set up the scripts for CONFIG_FILES section.  
# No need to generate them if there are no CONFIG_FILES.  
# This happens for instance with `./config.status config.h'.  
if test -n "$CONFIG_FILES"; then  
  
if $AWK 'BEGIN { getline <"/dev/null" }' </dev/null 2>/dev/null; then  
  ac_cs_awk_getline=:  
  ac_cs_awk_pipe_init=  
  ac_cs_awk_read_file='  
      while ((getline aline < (F[key])) > 0)  
    print(aline)  
      close(F[key])'  
  ac_cs_awk_pipe_fini=  
else  
  ac_cs_awk_getline=false  
  ac_cs_awk_pipe_init="print \"cat \<<'|#_!!_#|' &&\""  
  ac_cs_awk_read_file='  
      print "|#_!!_#|"  
      print "cat " F[key] " &&"  
      '$ac_cs_awk_pipe_init  
  # The final `:' finishes the AND list.  
  ac_cs_awk_pipe_fini='END { print "|#_!!_#|"; print ":" }'  
fi  
ac_cr=`echo X | tr X '\015'`  
# On cygwin, bash can eat \r inside `` if the user requested igncr.  
# But we know of no other shell where ac_cr would be empty at this  
# point, so we can use a bashism as a fallback.  
if test "x$ac_cr" = x; then  
  eval ac_cr=\$\'\\r\'  
fi  
ac_cs_awk_cr=`$AWK 'BEGIN { print "a\rb" }' </dev/null 2>/dev/null`  
if test "$ac_cs_awk_cr" = "a${ac_cr}b"; then  
  ac_cs_awk_cr='\\r'  
else  
  ac_cs_awk_cr=$ac_cr  
fi  
  
echo 'BEGIN {' >"$ac_tmp/subs1.awk" &&  
_ACEOF  
  
# Create commands to substitute file output variables.  
{  
  echo "cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1" &&  
  echo 'cat >>"\$ac_tmp/subs1.awk" \<<\\_ACAWK &&' &&  
  echo "$ac_subst_files" | sed 's/.*/F["&"]="$&"/' &&  
  echo "_ACAWK" &&  
  echo "_ACEOF"  
} >conf$$files.sh &&  
. ./conf$$files.sh ||  
  as_fn_error $? "could not make $CONFIG_STATUS" "$LINENO" 5  
rm -f conf$$files.sh  
  
{  
  echo "cat >conf$$subs.awk \<<_ACEOF" &&  
  echo "$ac_subst_vars" | sed 's/.*/&!$&$ac_delim/' &&  
  echo "_ACEOF"  
} >conf$$subs.sh ||  
  as_fn_error $? "could not make $CONFIG_STATUS" "$LINENO" 5  
ac_delim_num=`echo "$ac_subst_vars" | grep -c '^'`  
ac_delim='%!_!# '  
for ac_last_try in false false false false false :; do  
  . ./conf$$subs.sh ||  
    as_fn_error $? "could not make $CONFIG_STATUS" "$LINENO" 5  
  
  ac_delim_n=`sed -n "s/.*$ac_delim\$/X/p" conf$$subs.awk | grep -c X`  
  if test $ac_delim_n = $ac_delim_num; then  
    break  
  elif $ac_last_try; then  
    as_fn_error $? "could not make $CONFIG_STATUS" "$LINENO" 5  
  else  
    ac_delim="$ac_delim!$ac_delim _$ac_delim!! "  
  fi  
done  
rm -f conf$$subs.sh  
  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
cat >>"\$ac_tmp/subs1.awk" \<<\\_ACAWK &&  
_ACEOF  
sed -n '  
h  
s/^/S["/; s/!.*/"]=/  
p  
g  
s/^[^!]*!//  
:repl  
t repl  
s/'"$ac_delim"'$//  
t delim  
:nl  
h  
s/\(.\{148\}\)..*/\1/  
t more1  
s/["\\]/\\&/g; s/^/"/; s/$/\\n"\\/  
p  
n  
b repl  
:more1  
s/["\\]/\\&/g; s/^/"/; s/$/"\\/  
p  
g  
s/.\{148\}//  
t nl  
:delim  
h  
s/\(.\{148\}\)..*/\1/  
t more2  
s/["\\]/\\&/g; s/^/"/; s/$/"/  
p  
b  
:more2  
s/["\\]/\\&/g; s/^/"/; s/$/"\\/  
p  
g  
s/.\{148\}//  
t delim  
' <conf$$subs.awk | sed '  
/^[^""]/{  
  N  
  s/\n//  
}  
' >>$CONFIG_STATUS || ac_write_fail=1  
rm -f conf$$subs.awk  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
_ACAWK  
cat >>"\$ac_tmp/subs1.awk" \<<_ACAWK &&  
  for (key in S) S_is_set[key] = 1  
  FS = ""  
  \$ac_cs_awk_pipe_init  
}  
{  
  line = $ 0  
  nfields = split(line, field, "@")  
  substed = 0  
  len = length(field[1])  
  for (i = 2; i < nfields; i++) {  
    key = field[i]  
    keylen = length(key)  
    if (S_is_set[key]) {  
      value = S[key]  
      line = substr(line, 1, len) "" value "" substr(line, len + keylen + 3)  
      len += length(value) + length(field[++i])  
      substed = 1  
    } else  
      len += 1 + keylen  
  }  
  if (nfields == 3 && !substed) {  
    key = field[2]  
    if (F[key] != "" && line ~ /^[     ]*@.*@[     ]*$/) {  
      \$ac_cs_awk_read_file  
      next  
    }  
  }  
  print line  
}  
\$ac_cs_awk_pipe_fini  
_ACAWK  
_ACEOF  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
if sed "s/$ac_cr//" < /dev/null > /dev/null 2>&1; then  
  sed "s/$ac_cr\$//; s/$ac_cr/$ac_cs_awk_cr/g"  
else  
  cat  
fi < "$ac_tmp/subs1.awk" > "$ac_tmp/subs.awk" \  
  || as_fn_error $? "could not setup config files machinery" "$LINENO" 5  
_ACEOF  
  
# VPATH may cause trouble with some makes, so we remove sole $(srcdir),  
# ${srcdir} and @srcdir@ entries from VPATH if srcdir is ".", strip leading and  
# trailing colons and then remove the whole line if VPATH becomes empty  
# (actually we leave an empty line to preserve line numbers).  
if test "x$srcdir" = x.; then  
  ac_vpsub='/^[     ]*VPATH[     ]*=[     ]*/{  
h  
s///  
s/^/:/  
s/[     ]*$/:/  
s/:\$(srcdir):/:/g  
s/:\${srcdir}:/:/g  
s/:@srcdir@:/:/g  
s/^:*//  
s/:*$//  
x  
s/\(=[     ]*\).*/\1/  
G  
s/\n//  
s/^[^=]*=[     ]*$//  
}'  
fi  
  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
fi # test -n "$CONFIG_FILES"  
  
  
eval set X "  :F $CONFIG_FILES      "  
shift  
for ac_tag  
do  
  case $ac_tag in  
  :[FHLC]) ac_mode=$ac_tag; continue;;  
  esac  
  case $ac_mode$ac_tag in  
  :[FHL]*:*);;  
  :L* | :C*:*) as_fn_error $? "invalid tag \`$ac_tag'" "$LINENO" 5;;  
  :[FH]-) ac_tag=-:-;;  
  :[FH]*) ac_tag=$ac_tag:$ac_tag.in;;  
  esac  
  ac_save_IFS=$IFS  
  IFS=:  
  set x $ac_tag  
  IFS=$ac_save_IFS  
  shift  
  ac_file=$1  
  shift  
  
  case $ac_mode in  
  :L) ac_source=$1;;  
  :[FH])  
    ac_file_inputs=  
    for ac_f  
    do  
      case $ac_f in  
      -) ac_f="$ac_tmp/stdin";;  
      *) # Look for the file first in the build tree, then in the source tree  
     # (if the path is not absolute).  The absolute path cannot be DOS-style,  
     # because $ac_f cannot contain `:'.  
     test -f "$ac_f" ||  
       case $ac_f in  
       [\\/$]*) false;;  
       *) test -f "$srcdir/$ac_f" && ac_f="$srcdir/$ac_f";;  
       esac ||  
       as_fn_error 1 "cannot find input file: \`$ac_f'" "$LINENO" 5;;  
      esac  
      case $ac_f in *\'*) ac_f=`$as_echo "$ac_f" | sed "s/'/'\\\\\\\\''/g"`;; esac  
      as_fn_append ac_file_inputs " '$ac_f'"  
    done  
  
    # Let's still pretend it is `configure' which instantiates (i.e., don't  
    # use $as_me), people would be surprised to read:  
    #    /* config.h.  Generated by config.status.  */  
    configure_input='Generated from '`  
      $as_echo "$*" | sed 's|^[^:]*/||;s|:[^:]*/|, |g'  
    `' by configure.'  
    if test x"$ac_file" != x-; then  
      configure_input="$ac_file.  $configure_input"  
      { $as_echo "$as_me:${as_lineno-$LINENO}: creating $ac_file" >&5  
$as_echo "$as_me: creating $ac_file" >&6;}  
    fi  
    # Neutralize special characters interpreted by sed in replacement strings.  
    case $configure_input in #(  
    *\&* | *\|* | *\\* )  
       ac_sed_conf_input=`$as_echo "$configure_input" |  
       sed 's/[\\\\&|]/\\\\&/g'`;; #(  
    *) ac_sed_conf_input=$configure_input;;  
    esac  
  
    case $ac_tag in  
    *:-:* | *:-) cat >"$ac_tmp/stdin" \  
      || as_fn_error $? "could not create $ac_file" "$LINENO" 5 ;;  
    esac  
    ;;  
  esac  
  
  ac_dir=`$as_dirname -- "$ac_file" ||  
$as_expr X"$ac_file" : 'X\(.*[^/]\)//*[^/][^/]*/*$' \| \  
     X"$ac_file" : 'X\(//\)[^/]' \| \  
     X"$ac_file" : 'X\(//\)$' \| \  
     X"$ac_file" : 'X\(/\)' \| . 2>/dev/null ||  
$as_echo X"$ac_file" |  
    sed '/^X\(.*[^/]\)\/\/*[^/][^/]*\/*$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)[^/].*/{  
        s//\1/  
        q  
      }  
      /^X\(\/\/\)$/{  
        s//\1/  
        q  
      }  
      /^X\(\/\).*/{  
        s//\1/  
        q  
      }  
      s/.*/./; q'`  
  as_dir="$ac_dir"; as_fn_mkdir_p  
  ac_builddir=.  
  
case "$ac_dir" in  
.) ac_dir_suffix= ac_top_builddir_sub=. ac_top_build_prefix= ;;  
*)  
  ac_dir_suffix=/`$as_echo "$ac_dir" | sed 's|^\.[\\/]||'`  
  # A ".." for each directory in $ac_dir_suffix.  
  ac_top_builddir_sub=`$as_echo "$ac_dir_suffix" | sed 's|/[^\\/]*|/..|g;s|/||'`  
  case $ac_top_builddir_sub in  
  "") ac_top_builddir_sub=. ac_top_build_prefix= ;;  
  *)  ac_top_build_prefix=$ac_top_builddir_sub/ ;;  
  esac ;;  
esac  
ac_abs_top_builddir=$ac_pwd  
ac_abs_builddir=$ac_pwd$ac_dir_suffix  
# for backward compatibility:  
ac_top_builddir=$ac_top_build_prefix  
  
case $srcdir in  
  .)  # We are building in place.  
    ac_srcdir=.  
    ac_top_srcdir=$ac_top_builddir_sub  
    ac_abs_top_srcdir=$ac_pwd ;;  
  [\\/]* | ?:[\\/]* )  # Absolute name.  
    ac_srcdir=$srcdir$ac_dir_suffix;  
    ac_top_srcdir=$srcdir  
    ac_abs_top_srcdir=$srcdir ;;  
  *) # Relative name.  
    ac_srcdir=$ac_top_build_prefix$srcdir$ac_dir_suffix  
    ac_top_srcdir=$ac_top_build_prefix$srcdir  
    ac_abs_top_srcdir=$ac_pwd/$srcdir ;;  
esac  
ac_abs_srcdir=$ac_abs_top_srcdir$ac_dir_suffix  
  
  
  case $ac_mode in  
  :F)  
  #  
  # CONFIG_FILE  
  #  
  
  case $INSTALL in  
  [\\/$]* | ?:[\\/]* ) ac_INSTALL=$INSTALL ;;  
  *) ac_INSTALL=$ac_top_build_prefix$INSTALL ;;  
  esac  
_ACEOF  
  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
# If the template does not know about datarootdir, expand it.  
# FIXME: This hack should be removed a few years after 2.60.  
ac_datarootdir_hack=; ac_datarootdir_seen=  
ac_sed_dataroot='  
/datarootdir/ {  
  p  
  q  
}  
/@datadir@/p  
/@docdir@/p  
/@infodir@/p  
/@localedir@/p  
/@mandir@/p'  
case `eval "sed -n \"\$ac_sed_dataroot\" $ac_file_inputs"` in  
*datarootdir*) ac_datarootdir_seen=yes;;  
*@datadir@*|*@docdir@*|*@infodir@*|*@localedir@*|*@mandir@*)  
  { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: $ac_file_inputs seems to ignore the --datarootdir setting" >&5  
$as_echo "$as_me: WARNING: $ac_file_inputs seems to ignore the --datarootdir setting" >&2;}  
_ACEOF  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
  ac_datarootdir_hack='  
  s&@datadir@&$datadir&g  
  s&@docdir@&$docdir&g  
  s&@infodir@&$infodir&g  
  s&@localedir@&$localedir&g  
  s&@mandir@&$mandir&g  
  s&\\\${datarootdir}&$datarootdir&g' ;;  
esac  
_ACEOF  
  
# Neutralize VPATH when `$srcdir' = `.'.  
# Shell code in configure.ac might set extrasub.  
# FIXME: do we really want to maintain this feature?  
cat >>$CONFIG_STATUS \<<_ACEOF || ac_write_fail=1  
ac_sed_extra="$ac_vpsub  
$extrasub  
_ACEOF  
cat >>$CONFIG_STATUS \<<\_ACEOF || ac_write_fail=1  
:t  
/@[a-zA-Z_][a-zA-Z_0-9]*@/!b  
s|@configure_input@|$ac_sed_conf_input|;t t  
s&@top_builddir@&$ac_top_builddir_sub&;t t  
s&@top_build_prefix@&$ac_top_build_prefix&;t t  
s&@srcdir@&$ac_srcdir&;t t  
s&@abs_srcdir@&$ac_abs_srcdir&;t t  
s&@top_srcdir@&$ac_top_srcdir&;t t  
s&@abs_top_srcdir@&$ac_abs_top_srcdir&;t t  
s&@builddir@&$ac_builddir&;t t  
s&@abs_builddir@&$ac_abs_builddir&;t t  
s&@abs_top_builddir@&$ac_abs_top_builddir&;t t  
s&@INSTALL@&$ac_INSTALL&;t t  
$ac_datarootdir_hack  
"  
eval sed \"\$ac_sed_extra\" "$ac_file_inputs" |  
if $ac_cs_awk_getline; then  
  $AWK -f "$ac_tmp/subs.awk"  
else  
  $AWK -f "$ac_tmp/subs.awk" | $SHELL  
fi \  
  >$ac_tmp/out || as_fn_error $? "could not create $ac_file" "$LINENO" 5  
  
test -z "$ac_datarootdir_hack$ac_datarootdir_seen" &&  
  { ac_out=`sed -n '/\${datarootdir}/p' "$ac_tmp/out"`; test -n "$ac_out"; } &&  
  { ac_out=`sed -n '/^[     ]*datarootdir[     ]*:*=/p' \  
      "$ac_tmp/out"`; test -z "$ac_out"; } &&  
  { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: $ac_file contains a reference to the variable \`datarootdir'  
which seems to be undefined.  Please make sure it is defined" >&5  
$as_echo "$as_me: WARNING: $ac_file contains a reference to the variable \`datarootdir'  
which seems to be undefined.  Please make sure it is defined" >&2;}  
  
  rm -f "$ac_tmp/stdin"  
  case $ac_file in  
  -) cat "$ac_tmp/out" && rm -f "$ac_tmp/out";;  
  *) rm -f "$ac_file" && mv "$ac_tmp/out" "$ac_file";;  
  esac \  
  || as_fn_error $? "could not create $ac_file" "$LINENO" 5  
 ;;  
  
  
  
  esac  
  
  
  case $ac_file$ac_mode in  
    "Makefile":F) sed "$extrasub_build" Makefile |  
   sed "$extrasub_host" |  
   sed "$extrasub_target" > mf$$  
   mv -f mf$$ Makefile ;;  
  
  esac  
done # for ac_tag  
  
  
as_fn_exit 0  
_ACEOF  
ac_clean_files=$ac_clean_files_save  
  
test $ac_write_fail = 0 ||  
  as_fn_error $? "write failure creating $CONFIG_STATUS" "$LINENO" 5  
  
  
# configure is writing to config.log, and then calls config.status.  
# config.status does its own redirection, appending to config.log.  
# Unfortunately, on DOS this fails, as config.log is still kept open  
# by configure, so config.status won't be able to write to it; its  
# output is simply discarded.  So we exec the FD to /dev/null,  
# effectively closing config.log, so it can be properly (re)opened and  
# appended to by config.status.  When coming back to configure, we  
# need to make the FD available again.  
if test "$no_create" != yes; then  
  ac_cs_success=:  
  ac_config_status_args=  
  test "$silent" = yes &&  
    ac_config_status_args="$ac_config_status_args --quiet"  
  exec 5>/dev/null  
  $SHELL $CONFIG_STATUS $ac_config_status_args || ac_cs_success=false  
  exec 5>>config.log  
  # Use ||, not &&, to avoid exiting from the if with $? = 1, which  
  # would make configure fail if this is the last instruction.  
  $ac_cs_success || as_fn_exit 1  
fi  
if test -n "$ac_unrecognized_opts" && test "$enable_option_checking" != no; then  
  { $as_echo "$as_me:${as_lineno-$LINENO}: WARNING: unrecognized options: $ac_unrecognized_opts" >&5  
$as_echo "$as_me: WARNING: unrecognized options: $ac_unrecognized_opts" >&2;}  
fi
```