## shell和bash
- shell:提供用户操作系统的一个界面,就是系统最外层（壳），用户通过这个来调用内核的操作。
- bash：命令行方式的shell，linux默认的shell
- 为什么要学bash？
  - 所有主机的图形界面都不同，但是所有系统的的shell命令行操作都相同，可以方便对系统进行管理。
  - 远程管理主机的时候传输速度快
- 怎么查看我使用的shell类型：通过`/etc/passwd`来知道。
#### Bash的功能
- 历史命令：存放在.bash_history文件中，文件记录的是用户这次登陆之前的命令，当用户注销后，再将用户执行的命令写到该文件中。
- [Tab] : 命令和文件补全。
- 命令设置别名`alias`
- 程序化脚本
- 查看是否为bash内置命令:`type name`
- `ctrl+u/k/a/e`：分别是删除光标前的字符串、删除光标后的字符串、移动光标到最前面、最后面

### shell的变量
- 查看变量:`echo ${变量名}` 其中`${PATH}`和`$PATH`基本是一个意思，但是在表示数组变量的时候用前者，`"$PATH"`表示将PATH变量格式化成字符串类型，很多时候一样用。
- 变量操作
  - 设置变量：直接在shell中输入`myname=sxy`就行了,通常大写字符是系统自带变量，我们可以将变量写成小写的，方便判断。<br>这样设置的变量仅仅在这个shell进程中能够使用，如果需要让这个线程的子线程也能够使用，那么需要`export myname`将这个变量添加到当前账户的环境变量中。
  - 取消变量：`unset myname`
  - 扩展变量内容：`PATH="$PATH":home\bin`
  - 将变量添加到环境变量：`export 变量`,但是该环境变量仅限于本次登陆操作中，如需永久添加，需要进入.bashrc或/etc/profile中添加用户或者系统环境变量。
  - 在命令的操作数中执行命令：`$()`,例如`version=$(uname -r)`
  - 对变量进行判断`filename=${fileuser:-filename}`:如果fileuser不存在，就取`filename=filename`，如果存在就用fileuser的值。
  - 变量的删除、取代、替换，看书330
- 查看环境变量：`env`
  - 语系编码方式：`LANG`、`LC_ALL`，修改这两个的值来修改当前环境下系统的编码格式；其中LC_ALL的优先级>LANG；要注意的是在tty1-tty6中，bash不能识别zh_CN.UTF8格式，所以会出现乱码。
  - 环境变量的加载：不管是哪个账户，都会去默认的文件路径加载它们的环境变量（有公共的环境变量路径`/etc/profile`和用户自己的环境变量`~/.bashrc`），加载完了之后，在用户使用期间定义的环境变量和添加的环境变量都不会写入默认的环境变量（除非用户去文件里面写），只会写入相应的pid中，供本次登陆的本进程和子进程使用。
- 查看所有变量（包括环境变量和当前bash变量和自定义变量）：`set`
  - PS1:命令行开头显示格式，每次敲下回车后需要显示下一条用户输入的提示符的时候就会读取这个变量。
  - $:本shell的pid
  - ？：上个命令的返回值`echo $?`
- 键盘读取变量
  - `read atest`：此时光标等待输入，输入的值就是atest变量的值。
  - `declare -i/a sum`:将sum定义成整数或数组类型
    - `-x`将后面的变量添加到环境变量
    - `-r`将变量做成只读，慎用，如果搞错了只能注销重新登录了。
    - 系统默认变量为字符串类型。
    - bash环境下默认的数值运算最多只能达到整数形态，没有小数形态。
    - 数组类型：设置：`var[1]="yinweinibuhui"`，读取：`echo ${var[1]}`,必须使用`echo ${}`的形式。
- `ulimit`:限制用户资源使用
  - `-a`:查看当前用户的限制设置情况。
  - `-f`:限制创建文件的容量大小。例：`ulimit -f 10240`:限制创建的文件大小最大为10M。
  - 一般用户只能将-f从大设置到小，如果设置小了想变回去的话只能注销重新启动了。
  

上述对变量的操作基本仅限于本次的shell登陆；但是环境变量和基本变量还是有区别的，环境变量可以提供给本次bash的子进程使用，而基本变量不行。

### 命令别名和历史命令
- 命令别名
  - `alias vi='vim'`:以后输入vi就相当于输入了vim
  - `unalias vi`：取消别名
- 历史命令
  - `history`:列出之前敲的命令，列出多少条与HISTSIZE变量有关。
  - `history -w`:将历史记录写入~/.bash_history文件中。
  - `history -c`：将shell中记录的history清除。
  - `!vi`:执行历史记录中上一条带有vi的命令。

## Bash的操作环境
### 路径
每敲下一个命令的时候，系统是怎么找到这个命令的呢，例如ls这种命令，可能在Bash内置环境和系统文件中均存在，那么系统会优先执行哪个呢？
可以通过`type -a 命令`来查看优先级顺序。
### 登陆与欢迎信息
- bash登陆会读取`/etc/issue`文件，和$PS1一样，通过'/'去读取用户的变量。通过root去改变这个文件的形式，便可以实现对欢迎界面显示的修改。
- `/etc/motd`：root在这文件写的东西，当用户登陆的时候，会显示在用户的登录界面上。
### 环境配置文件
用户进入shell的时候shell会读取一系列文件作为系统的初始变量。
- login shell：登陆过后的shell<br>
  - 需要读取的文件：<br>
  1.  `/etc/profile`：系统整体设置,会从这个文件中调用其他的配置文件。
  2.  `~/bash_profile`:个人配置文件。
![](../流程图/加载顺序.png)
  - 读入环境配置文件`source 文件名`:由于上面两个环境配置文件都是需要电脑重新登陆账户才会进行读取，`source`可以让电脑不需要注销账户，手动加载`.bashrc`等文件
- nologin shell：没有登陆的shell（子进程、X WINDOW）
- 终端环境
  - `stty -a`：查看终端环境设置
  - 通配符
    - `*` :0-无穷个字符
    - `？`：一定有一个字符
    - `[]`：[abcd]代表一定有一个，在abcd之中选一个
    - `[-]`：`[0-9]`代表这个里面的数必须是0-9中的一个
    - `[^]`：`[^abc]`代表只要不是abc中的一个就行
  - 特殊符号:见书345

## 数据流重定向
系统本来输出的路径被拦截，把输出的数据弄到另一个路径中去了。
- `> >>`:其中`>`重定向到文件，如果文件不存在则会建立文件写进去；如果存在的话就会覆盖掉原来的文件内容。`>>`则会在原来文件之后添加。
- `1>>`代表正确输出。系统默认就是1；举个例子，执行`ll / > ~/file`命令，会将本来给屏幕返回的信息重定向到file文件中。
- `2>>`代表错误输出,
  - `find /home -name .bashrc >list_right 2>list_error`：会将正确的输出到list_right文件中，不正确的输出到list_error中。
  - 将正确的和错误的写到同一个文件中：`find /home -name .bashrc >list 2>list`，这种写法是错误的，会导致正确的和错误的交叉写入，最终文件虽然能出来，但是全是乱的；`find /home -name .bashrc >list 2>&1`这种写法才正确。
  - 将错误信息丢弃：`find /home -name .bashrc >list 2>/dev/null`,其中/dev/null是一个虚拟的垃圾桶设备。
- `< <<`：`<`代表使用文件的内容代表键盘的输入。`<<`代表输入的结束符：`cat > catfile << "eof"`执行完后键盘敲到eof的时候就会将键盘的内容写到catfile中。
- `&>`:将正确或错误信息都重定向输出
### 命令执行的判断依据
- `;`:两条命令之间不考虑关联的连续执行
- `&&`:需要第一条正确执行（返回0），第二条才会执行
- `||`:需要第一条错误执行，第二条才会执行<br>
`ls /tmp/abc && touch /tmp/abc/hehe`:代表如果存在abc这个目录才执行创建hehe文件。<br>`ls /tmp/abc || mkdir /tmp/abc`:如果没有这个目录就创建。<br>
`ls /tmp/abc || mkdir /tmp/abc && touch /tmp/abc/hehe`:执行的时候从左到右扫描；如果存在abc目录，它不会执行||，但会执行&&。

## 管道
管道命令能够处理标准输出，对于标准错误会进行忽略；管道`|`后的命令必须能够接受来自前一个命令的数据作为标准输入才行。
- 选取命令
  - `cut`:将输入信息进行“切”操作后输出；它是以行为单位切的
  - `grep`:分析一行信息，如果有我们想要的，就将这一行取出来。
- 排序命令
  - `sort`:按照某些标准进行排序
  - `uniq`:将排序重复的仅仅列出一个显示，`-c`进行计数;<br>`last | cut -d ' ' -f 1 | sort | uniq -c `:查出各个用户登陆的次数
  - `wc`:查看文件的行、列、字符数。
- 双向重定向
  - `tee`:有时候在管道中途需要将某一个输出记录下来，用到`tee`指令，`last | tee last.list | cut -d " " -f1`:将last传过去的值先存到last.list中，再继续从管道上走
- 字符转换
  - `tr`:进行文字的替换，可以指定输入中的字符进行替换和删除
  - `col`:tab键转成空格键
  - `join`:将两个文件之间的数据进行整合，将两个文件中有相同数据的那一行进行合并
  - `paste`:直接将两个文件的同一行贴在一起
  - `expand`:将tab键转成空格
- 划分文件命令
  - `split`:将大文件拆成小文件
    - 对管道输入的进行拆分：`ls -al / | split -l 10 -lsroot`,将根目录下的十行为一个文件，且前缀为lsroot
    - 将大文件变成小文件：`split -b 300k /etc/services services`:将`/etc/services`文件拆成以services为前缀的300k的小文件
- 参数代换：`xargs`:很多命令（如`id 参数`这种）需要参数，但是这些命令又不支持管道，可以用xargs接受管道数据来提供给这些命令以标准输入；本质就是结合管道为这种命令提供符合它格式的参数
- `- 减号的用途`：因为有时候在管道中会用到文件名，`-`可以替代管道前面的输出:`tar -cvf - /home | tar -xvf - -C /tmp/homeback`,管道后面那个'-'其实就是前面那个文件输出的数据。



  

  









