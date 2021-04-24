# ch1 Linux Basics

### **Linux是一个怎样的系统？**

Linux是一个开发在GNU协议下的类Unix操作系统，具有以下特点

* 开源
* 流行
* 支持多平台

### **安装Linux**

### **引导程序是用来干什么的**

boot loader用来装载并启动linux内核。当机器引导它的操作系统时，BIOS 会读取引导介质上最前面的 512 字节（即人们所知的 *主引导记录（master boot record，MBR）*）由于 BIOS 只能访问很少量的数据，所以大部分引导加载程序分两个阶段进行引导。在引导的第一个阶段中，BIOS 引导一部分引导加载程序，即 *初始程序加载程序（initial program loader，IPL*）。IPL 查询分区表，从而能够加载位于不同介质上任意位置的数据。首先通过这步操作来定位第二阶段引导加载程序（其中包含加载程序的其余部分）。

* 可以传递boot参数给linux内核，比如设备信息
* 可以以优化的方式装载root disk

普遍的Boot loader有

* LILO：Linux Loader
* GRUB：Grand Unified Boot Loader

### **命令行和图形界面各有什么好处**

略

### **安装程序的命令**

### **apt-get原理**

`/etc/apt/sources.list`存放的是软件源站点

* apt-get update

	扫描每一个软件源服务器，并为该服务器所具有软件包资源建立索引文件，存放在本地的/var/lib/apt/lists/目录中

* apt-get install XXX

	1. 扫描本地存放的软件包更新列表（由“apt-get update”命令刷新更新列表，也就是/var/lib/apt/lists/），找到最新版本的软件包；
	2. 进行软件包依赖关系检查，找到支持该软件正常运行的所有软件包；
	3. 从软件源所指 的镜像站点中，下载相关软件包，并存放在/var/cache/apt/archive；
	4. 解压软件包，并自动完成应用程序的安装和配置。

### **文件类型**

（-）普通文件：纯文本文档、二进制文件、数据格式文件等 

（c）字符设备文件：与设备进行交互的文件。Linux 中所有的设备都被抽 象为了对应的文件。字符设备是按字符读取。 

（b）块设备文件：同字符设备文件，但是是按块读取。 

（p）数据输送文件（FIFO，pipe）。他主要的目的在解决多个程序同时存 取一个文件所造成的错误问题。

（s）socket 文件。我们可以启动一个程序来监听客户端的要求，而客户端 就可以通过这个 socket 来进行数据的沟通。如启动 Mysql 服务会创建一 个对应的 socket 文件。一般在/var/run 目录中 

（l）符号链接

（d）目录

### **目录结构**

* /bin：系统必要的命令的二进制文件。包含了会被会被系统管理者和用户使用的命令。大部分常用的命令都在这里。
* /boot：Boot Loader相关的静态文件。包含了所有需要在系统引导阶段使用的文件（如内核镜像等）。
* /dev：设备对应的虚拟文件。
* /etc：系统和软件的配置文件。
* /lib：必要的共享库文件（如.so）或内核模块。
* /media：外部设备通用挂载点的父目录。
* /mnt：临时文件系统的挂载点的父目录。
* /opt：额外的应用软件包安装目录
* /sbin：只有管理员可以使用的命令的二进制文件。是与系统相关的基本命令，如shutdown，reboot等
* /srv：系统提供的有关服务的数据
* /tmp：临时文件
* /usr：Unix System Resources，不是user的简写。用于存放共享、只读的数据。子目录包括/bin，/etc，/lib，/sbin，/tmp等，与根目录下对应的目录对比，这些目录是給后来安装的软件的使用的（而不是系统自带的）。还有/include，/src等文件夹，存放系统编程所需的头文件和源码等。
* /home：用户的家目录的父目录
* /root：root用户的家目录

### **文件权限**

* 三个层次：所有者、所有者所在组、其他用户

* 三种权限：rwx

* ls -l 六个字段含义

	![image-20210421115231394](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210421115231394.png)

* 修改权限的方式

	`chomod <who operator what> filename`

	who: u | g | o | a(all)

	operator: + | - | =

	what: rwx

	https://www.runoob.com/linux/linux-comm-chmod.html

* 默认权限

	umask 默认为0022

	文件默认不能建立未执行文件，必须手工赋予执行权限，666-umask (644)

	目录默认权限最大未777，777-umask (755)

### **进程的概念**

进程是一个正在执行的程序实例。由执行程序、它的当前值、状态信息以及通过操作系统管理此进程执行情况的资源组成

### **层次图**

# ch1-2 Linux Basics2

### 重定向

* 标准输入、标准输出、标准错误
	* 文件描述符：0,1,2
	* C库（stdio.h）的流指针：stdin，stdout，stderr
* 命令行操作符
	* \>：将程序的输出重定向到一个文件设备文件，覆盖原来的文件
	* \>!：同上，但是强制覆盖
	* \>>：同上，但是不覆盖而是在末尾追加
	* <：将程序的输入重定向为某个程序

> 重定向是Linux的重要机制，请描述Linux重定向的用途、使用方法、典型案例，并简要描述其实现机制

用途：

* 输入方向就是数据从哪里流向程序，默认是键盘输入，如果改变了，就从其他地方流入
* 输出方向就是数据从程序流向哪里，默认是显示器，如果改变了，就流向其他地方

使用方法：`>`、`>>`、`<`、`>!`

经典案例：`echo "c.biancheng.net" 1 >log.txt`、`cat <log.txt`

实现机制：

* 在进程创建时，内核为进程默认创建了0、1、2三个特殊的FD，这就是STDIN、STDOUT和STDERR

* I/O重定向的过程中，不变的是FD 0/1/2代表STDIN/STDOUT/STDERR，变化的是文件描述符表中FD 0/1/2对应的具体文件，所以只需要使用dup2系统调用进行重定向即可。比如输出重定向：

	```c
	int fd_in = open("in.txt", O_RDONLY);
	dup2(fd_in, 0);
	...
	```

### 管道

|：将一个进程的标准输出作为另一个进程的标准输入

### 正则表达式

看作业

# ch2 Shell

# ch3-0 Programming Prerequisite

### 编译链接

![image-20210421162924536](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210421162924536.png)

* 预处理：gcc -E，处理预处理指令，如#define，#include....
* 编译：gcc -S，编译一个文件，就只看这个文件，而不看import的其他文件
* 汇编：gcc -c
* 链接：gcc/ld
	* 静态库：如果有f代码，会被拷贝到可执行文件，链接完以后库文件实际上就没有作用了（可执行文件大）
	* 动态库：不拷贝代码，只拷贝接口，运行时要求动态库文件还在，否则可能报错；多个可执行文件可以共享动态库
		* 好处：可执行文件小；方便更新

https://blog.csdn.net/daidaihema/article/details/80902012

### gcc命令、参数（找错）

![image-20210421165409632](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210421165409632.png)

https://blog.csdn.net/wohu1104/article/details/110789570

### makefile

https://seisman.github.io/how-to-write-makefile/index.html

> 为什么Linux引入makefile？和其他脚本的区别？使用makefile编译系统有哪些特点？

* **定义整个工程的编译规则**

	makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。

* **自动化编译**

	makefile带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率

make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。make只管文件的依赖性

# ch3-1 System Programming

### 七种文件类型

* 普通文件类型 
	Linux中最多的一种文件类型, 包括 纯文本文件(ASCII)；二进制文件(binary)；数据格式的文件(data);各种压缩文件.第一个属性为 [-]
* 目录文件
	就是目录， 能用 # cd 命令进入的。第一个属性为 [d]，例如 [drwxrwxrwx]
* 块设备文件
	块设备文件 ： 就是存储数据以供系统存取的接口设备，简单而言就是硬盘。例如一号硬盘的代码是 /dev/hda1等文件。第一个属性为 [b]
* 字符设备
	字符设备文件：即串行端口的接口设备，例如键盘、鼠标等等。第一个属性为 [c]
* 套接字文件
	这类文件通常用在网络数据连接。可以启动一个程序来监听客户端的要求，客户端就可以通过套接字来进行数据通信。第一个属性为 [s]，最常在 /var/run目录中看到这种文件类型
* 管道文件
	FIFO也是一种特殊的文件类型，它主要的目的是，解决多个程序同时存取一个文件所造成的错误。FIFO是first-in-first-out(先进先出)的缩写。第一个属性为 [p]
* 链接文件
	类似Windows下面的快捷方式。第一个属性为 [l]，例如 [lrwxrwxrwx]

### VFS主要原理（四种结构类型

VFS（Virtual Filesystem  Switch）称为虚拟文件系统或虚拟文件系统转换，是一个内核软件层，在具体的文件系统之上抽象的一层，用来处理与Posix文件系统相关的所有调用，表现为能够给各种文件系统提供一个通用的接口，使上层的应用程序能够使用通用的接口访问不同文件系统

* super block 

	超级块，一个超级块对应一个文件系统。

* inode block

	索引节点。一个实际存在的文件实体只有一个inode。inode对象全系统共用。

* dentry object

	目录项。一个目录项对应一个dentry，就是ls -a列出来的每一项就是一个dentry。dentry中有指向inode的指针。多个dentry可以对应同一个inode。dentry对象全系统共用。

* file object

	文件对象。一个打开了的文件对应一个file。file中有指向dentry的指针。文件对象是进程私有的（会以copy-on-write的方式与子进程共享）。一个文件对象包括的内容就是编程语言支持设置的各种文件打开的flag、mode，文件名称、当前的偏移等，其中非常重要的一个字段就是f_op，指向了当前文件所支持的操作集合。

### 硬链接和软链接

> 硬链接与软链接的区别？（至少三点）用shell命令和函数如何创建？

* 软连接

	* 存储被链接文件的文件名（而不是inode）实现链接

	* 可以跨越文件系统

	* 对应系统调用symlink

	* ln -s [source] [target]

		```c
		#include <unistd.h>
		int symlink(const char *oldpath, const char *newpath);
		(Return: 0 if success; -1 if failure)
		```

* 硬链接

	* 与被链接的文件共享同一个inode，dentry不同

	* 不能跨越文件系统

	* 对应系统调用link

	* ln [source] [target]

		```c
		#include <unistd.h>
		int link(const char *oldpath, const char *newpath);
		(Return: 0 if success; -1 if failure)
		```

		

### 主要系统调用

编程题，进程相关系统调用不要求掌握（Fork、execl），ioctl不考

#### IO系统调用

IO系统调用围绕文件描述符fd，一个非负整数进行。标准输入、标准输出、标准错误对应的fd分别是STDIN_FILENO(0)，STDOUT_FILENO(1)，STDERR_FILENO(2)

* `int open(const char *pathname, int flags);`

	`int open(const char *pathname, int flags, mode_t mode);`

	`int creat(const char *pathname, mode_t mode)` 

	pathname：文件路径

	flags：文件打开模式。位域。可选值O_RDONLY、O_WRONLY、O_RDWR、O_APPEND、O_TRUNC（清空文件原来的内容）、O_CREAT（如果不存在则创建）、O_EXCL（和O_CREAT一起使用时，如果原来存在则报错）、O_NONBLOCK（非阻塞模式）

	mode：创建文件时的权限，无符号整数，同chmod的值

	返回值：文件描述符；失败时则-1

* `int close(int fd)`

	fd：文件描述符

	返回值：0；失败则-1 

* `ssize_t read(int fd, void *buf, size_t count);`

	buf：缓冲区

	size_t：要读取的字节数

	返回值：已读取的字节数；若此次调用前已达到文件末尾，则0；出错则-1 

	```c
	#include<fcntl.h>
	
	int main()
	{
	    int fd, nread;
	    char buf[1024];
	    /*open file for reading*/
	    fd=open("data",O_RDONLY);
	    /* read in the data */
	    nread = read(fd, buf, 1024);
	    /* close the file */
	    close(fd);
	}
	```

* `ssize_t write(int fd, const void *buf, size_t count);`

	类比read

* `off_t lseek(int fd, off_t offset, int whence)`

	offset：偏移量

	whence：SEEK_SET：相对文件头偏移+offset处(这里offset不可以为负值)

	​		SEEK_CUR：相对当前位置偏移+offset处（可以为负值）

	​		SEEK_END：偏移到文件末尾+offset处（可以为负值）

	返回值：偏移量；失败则-1 

* `int dup(int oldfd);`

	`int dup2(int oldfd, int newfd);`

	dup复制一个文件文件描述符，返回新的

	dup2复制oldfd到newfd，之前newfd对应的文件将被关闭。

	返回：新的文件描述符；出错则-1

* `int fcntl(int fd, int cmd);`

	`int fcntl(int fd, int cmd, long arg);`

	`int fcntl(int fd, int cmd, struct flock *lock);` 

	cmd: 

	* F_DUPFD：复制文件描述符，返回新的文件描述符
	* F_GETFD/F_SETFD：获取/设置文件描述符标识（目前只有close-on-exec，表示子进程在执行exec族命令时释放对应的文件描述符）。
	* F_GETFL/F_SETFL：获得/设置文件状态标识（open/creat中的flags参数），目前只能更改O_APPEND，O_ASYNC，O_DIRECT，O_NOATIME，O_NONBLOCK
	* F_GETOWN/F_SETOWN：管理I/O可用相关的信号。获得或设置当前文件描述符会接受SIGIO和SIGURG信号的进程或进程组编号
	* F_GETLK/F_SETLK/F_SETLKW：获得/设置文件锁，设置为F_GETLK需要传入flock的指针用于存放锁信息。S_SETLK也传入flock指针表示需要改变的锁的内容，如果不能设置则立即返回EAGAIN。S_SETLKW同S_SETLK，但在无法设置时会阻塞当前进程直到成功

#### 进阶系统调用

* `int stat(const char *filename, struct *buf);`

  `int fstat(int fd, struct stat *buf);`

  `int lstat(const char *filename, struct stat *buf);`

  获取文件的属性。最后一个是遇到符号链接时，能取到被链接的文件的属性（其他的只能取到链接文件自己的属性）。

  返回值：0；失败则-1

  ```c
  struct stat {
  	mode_t st_mode;
      ino_t st_ino;
      dev_t st_rdev;
      nlink_t st_nlink;
      uid_t st_uid;
      gid_t st_gid;
      off_t st_size;
      time_t st_atime;
      time_t st_mtime;
      time_t st_ctime;
      long st_blksize;
      long st_blocks;
  }
  ```

  `st_mode`里存放了类型、权限等信息。

  ```c
  // 判断文件类型，使用定义的宏
  S_ISREG(m)      是否为普通文件
  S_ISDIR(m)      是否为目录
  S_ISCHR(m)      是否为字符设备
  S_ISBLK(m)      是否为块设备
  S_ISFIFO(m)     是否为FIFO(命名管道文件，用于进程通信)
  S_ISLNK(m)      是否为符号链接
  S_ISSOCK(m)     是否为套接字
  // 判断权限，与下面进行&运算
  S_I(R|W|X)(USR|GRP|OTH)
  S_IRWX(U|G|O)
  S_ISUID
  S_ISGID
  S_ISVTX    
  ```

  另外注意下，这几个`time_t`是时间戳也就是long，不是C库里那个time_t

* `int access(const char *path, int mode);`

	根据当前的用户ID和实际组ID测试文件的存取权限

	mode：R_OK，W_OK，X_OK，F_OK（文件是否存在）

	返回值：0；失败则-1

* `int chmod(const char *path, mode_t mode);`

	`int fchmod(int fd, mode_t mode);`

	mode与st_mode中的第九位相同。

	返回值：0；失败则-1

* `int chown(const char *path, uid_t owner, gid_t group);`

	`int fchown(int fd, uid_t owner, gid_t group);`

	`int lchown(const char *path, uid_t owner, gid_t group);`

	更改文件的拥有者和组

	返回值：0；失败则-1

* `mode_t umask(mode_t mask);`

	更改存取权限屏蔽字（默认为022）

	返回值：之前的值

* `int link(const char *oldpath, const char *newpath);`

	`int unlink(const char *pathname);`

	创建/删除一个文件的硬链接

	返回值：0；失败则-1

* `int symlink(const char *oldpath, const char *newpath);`

	`int readlink(const char *path, char *buf, size_t bufsize);`

	创建/读取符号链接的值

	返回值：0；失败则-1

* `int mkdir(const char *pathname, mode_t mode);`

	`int rmdir(const char *pathname);`

	创建/删除空目录

	返回值：0；失败则-1

* `int chdir(const char *path);`

	`int fchdir(int fd);`

	更改当前工作目录

	返回值：0；失败则-1

* `char *getcwd(char *buf, size_t size);`

	获取当前工作目录

	返回值：buf；失败则NULL

* `DIR *opendir(const char *name);`

	打开目录

	返回值：DIR指针，类似FILE；失败则NULL

* `int closedir(DIR *dir);`

	`struct dirent *readdir(DIR *dir);`

	`off_t telldir(DIR *dir);`

	`void seekdir(DIR *dir, off_t offset);`

	![image-20210422144531908](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422144531908.png)

	```c
	struct dirent {
	    long d_ino;
	    off_t d_off;
	    unsigned short d_reclen;
	    unsigned char d_type;
	    char d_name [NAME_MAX + 1];
	}
	```

* `int fcntl(int fd, int cmd, struct flock *lock);`

	cmd:

	* F_GETLK：获得文件的封锁信息 
	* F_SETLK：对文件的某个区域封锁或解除封锁 
	* F_SETLKW：功能同F_SETLK, wait方式
	

![image-20210422144717058](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422144717058.png)
	
* `int lockf(int fd, int cmd, off_t len);`

	cmd：指定的操作类型

	* F_LOCK：给文件夹互斥锁。若已被加锁则阻塞直到成功
	* F_TLOCK：同上，但不会阻塞，直接失败
	* F_ULOCK：解锁
	* F_TEST：测试是否上锁。未上锁则0，否则-1

	len：从当前位置开始要锁住多长

	这个函数是对fcntl的一层封装

### C库（简答题，问xx在C库里怎样完成的？）

> Linux中文件描述符和文件指针FILE \*的区别是什么？

* 文件描述符：在linux系统中打开文件就会获得文件描述符，它是个很小的正整数。每个进程在PCB（Process Control Block）中保存着一份文件描述符表，文件描述符就是这个表的索引，每个表项都有一个指向已打开文件的指针。
* 文件指针：C语言中使用文件指针做为I/O的句柄。文件指针指向进程用户区中的一个被称为FILE结构的数据结构。FILE结构包括一个缓冲区和一个文件描述符。而文件描述符是文件描述符表的一个索引，因此从某种意义上说文件指针就是句柄的句柄

#### IO库函数

* 文件流和FILE结构体

	标准库中的I/O围绕FILE对象，也就是流指针进行。预定义三个流指针，即标准输入stdin，标注你输出stdout，标准错误stderr

* 缓冲模式

	* 块缓冲（全缓冲，full buffered，block bufferd）
	* 行缓冲（如cin
	* 无缓冲

* `void setbuf(FILE *stream, char *buf);`

	`int setvbuf(FILE *stream, char *buf, int mode, size_t size);`

	mode：缓冲模式，_IOFBF（全缓冲），\_IOLBF（行缓冲），\_IONBF（无缓冲）

	buf：缓冲区，如果为NULL且mode不是_IONBF，库会调用malloc分配由size指定的大小的空间

	返回值：0；失败则非0

* `FILE *fopen(const char *filename, const char *mode)`

	mode：打开的模式："r"只读；"w"覆盖写；"a"追加；"r+"读写；"w+"读+覆盖写，且在文件不存在时自动创建；"a+"读+追加写，且在文件不存在时自动创建；"t"文本模式；"b"二进制模式。最后两个可以和之前的组合，如"rb"，"at+"等

	返回值：流指针；失败则NULL

* `int fclose(FILE *stream)`

	返回值：0；失败则-1

* `int getc(FILE *fp);`

	`int fgetc(FILE *fp);`

	`int getchar(void);`

	`getchar`从标准输入读取。

	`getc`使用宏来实现的，所以要注意其参数不能有副作用。但效率会略高于`fgetc`

	返回值：转换成unsigned int的char值；读取到末尾或出错则EOF

* `int putc(int c, FILE *fp);`

	`int fputc(int c, FILE *fp);`

	`int putchar(int c)`

	返回值：写入的字符值；出错则-1

* `char *fgets(char *s, int size, FILE *stream);`

	`char *gets(char *s);`

	s：缓冲区

	后者不推荐，很容易溢出

	注意，会读取size - 1个字符，并在末尾添加\0。遇到文件尾或换行符会停止

	返回值：缓冲区头

* `int fputs(const char *s, FILE *stream);`

	`int puts(const char *s);`

	批量写入直到第一个\0（\0本身不写入）

	返回值：非负整数；出错则EOF

* `size_t fread(void *buf, size_t size, size nmemb, FILE *stream);`

	`size_t fwrite(const void *buf, size_t size, size_t nmemb, FILE *stream);`

	二进制读写

	size：每次读/写的字节数

	nmemb：总共读/写几次。也就是说总共写入的字节数是size \* nmemb。

	返回值：成功读/写的次数

	![image-20210421231427129](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210421231427129.png)

* `int scanf(const char *format, ...);`

	`int fscanf(FILE *stream, const char *format, ...)`

	`int sscanf(const char *str, const char *format, ...);`

	Format Input

	分别从标准输入，流，字符串扫描输入。注意后面的`...`是指可变参数。

	返回值：正确读取的变量个数

* `int printf(const char *format, ...)`

	`int fprintf(FILE *stream, const char *format);`

	`int sprintf(char *str, const char *format);`

	Format Onput

	分别格式化输出到标准输出，流，字符串（包括一个\0）。返回值为写入的字符数，包括\0。

* `int fseek(FILE *stream, long int offset, int whence);`

	和`lseek`差不多

* `long ftell(FILE *stream);`

	返回当前位置的偏移量

* `void rewind(FILE *stream);`

	将流指针移到文件开头

* `int fgetpos(FILE *fp, fpos_t *pos);`

	`int fsetpost(FILE *fp, const fpos_t *pos);`

	也用来获取/移动位置，向/从pos参数存放/读取位置信息。新增这两个函数是为了处理大到超出long int范围的文件

	返回值：0；失败则一个非零值

* `int fflush(FIILE *stream);`

	返回值：0；失败则EOF

* `int fileno(FILE *fp)`

	获取流指针对应的文件描述符

* `FILE *fdopen(int fd, const char *mode);`

	用已打开的文件描述符创建一个流

* `char *tmpnam(char *s);`

	返回一个当前未被使用的文件名

* `FILE *tmpfile(void);`

	创建一个临时文件

### 权限（包括粘滞位）

* 三个层次：所有者、所有者所在组、其他用户

* 三种权限：rwx

* 特殊权限：SUID SGID Sticky bit

  * SUID

  	`chmod u+s FILE `

  	`chmod u-s FILE`

  	`chmod 4xxx FILE`

  	文件属主的x权限,用s代替.表示被设置了SUID

  	如果属主位没有x权限,会显示为大写S,表示有故障(权限无效) 

  	启动为进程之后，其进程的属主为原程序文件的属主；

  * SGID

    `chmod g+s DIR/FILE`

    `chmod g-s DIR/FILE`

    `chmod 2xxx FILE`

    文件属组的x权限,用s代替.表示被设置了SGID

    如果属组位没有x权限,会显示为大写S,表示有故障(权限无效) 

    * 作用在二进制程序上时：执行sgid权限的程序时,此用户将继承此程序的所属组权限

    * 作用于目录上时：此文件夹下所有用户新建文件都自动继承此目录的用户组

  * Sticky bit

  	`chmod o+t DIR`

  	`chmod o-t DIR`

  	`chmod +t DIR`

  	`chmod 1xxx FILE`

  	 文件other位的x权限,用t代替.表示被设置了Sticky

  	 如果other位没有x权限,会显示为大写T,表示有故障(权限无效) 

  	* 对于一个多人可写的目录，如果设置了sticky，则每个用户仅能删除和改名自己的文件或目录；
  	* 只能作用在目录上.普通文件设置无意义,且会被linux内核忽略
  	* 用户在设置Sticky权限的目录下新建的目录不会自动继承Sticky权限 

  https://www.cnblogs.com/Q--T/p/7864795.html

### 文件锁（特殊类型不要求掌握），锁标志位不要求，怎样创建、释放锁的系统调用要求掌握

* 记录锁

* 劝告锁

	检查，加锁由应用程序自己控制

* 强制锁

	检查，加锁由内核控制

	影响 open read write

* 共享锁

* 排它锁

# ch4 内核与驱动

### 什么是内核

![image-20210422151155280](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422151155280.png)

### 加载模块，相关命令

![image-20210422151446237](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422151446237.png)

https://www.cnblogs.com/klb561/p/9236420.html

### 内核模块和应用程序的区别

![image-20210422152622341](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422152622341.png)

* 不能使用C库来开发驱劢程序 
* 没有内存保护机制 
* 小内核栈 
* 并发上的考虑

### 代码，读

### 字符型设备驱动

基本概念

* 申请设备号

* 定义文件操作结构体 file_operations

	![image-20210422152900130](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210422152900130.png)

* 创建并初始化定义结构体 cdev

* 将cdev注册到系统，并和对应的设备号绑定

* 在/dev文件系统中用mknod创建设备文件， 并将该文件绑定到设备号上

# 命令集合



# 企业题目

### openEuler出现的背景

2019年底EulerOS被正式推 送开源社区，命名为openEuler

### openEuler是什么

* openEuler 是一个开源、免费的Linux发行平台；
* 支持x86、ARM、RISC-V等多种处理器架构； 
* 所有开发者、企业、商业组织都可以使用openEuler社区版本，也可以基于社区版本发布自己二次开发的操作系统版本

### 线程间通信ITC

* 互斥机制主要使用自旋锁来实现。openEuler提供了**NUMA感知队列自旋锁**实现互斥机制，减小了NUMA体系结构中使用自旋锁的开销。 
* 同步机制主要使用信号量来实现。openEuler中提供**down**原语与**up**原语，能够实现线程的同步运行。

### 进程间通信IPC

* IPC机制主要包括：信号（Signal）、管道（Pipe）、信号量、共享内存（Shared  Memory）、消息队列（Message Queue）、套接字（Socket）
* openEuler**增强**了两种进程间通信机制：**共享内存与消息传递机制**

### openEuler内存页相关说明

* 页表一般存储在一个地址连续的内存中，丏能随机访问，以快速查找页表中相应的记录。在open  Euler中，**各级页表的表项大小为8B**。 
* 页表的查询通常由与用的硬件内存管理单元（Memory Management Unit，MMU）快速完成，然 后交给OS完成(建表、设置基址寄存器、访存管理) 
* openEuler将标准大页封装为一个**伪文件系统（hugetlbfs）提供给用户程序申请并访问**。 
* 操作系统需要依据一定的页置换策略决定将哪些页进行换出，openEuler采用**Least Recently Used （LRU）**最近最久未使用策略实现页选择换出。
*  页在未来被访问的概率只能预测，不能**精准判断**。

### 鲲鹏处理器

* 鲲鹏处理器是基于**ARMv8-64位RISC指令集**开发的通用处理器
* 使用大量寄存器：通用**X0-X30 （31个，64位）+特殊寄存器+ 系统寄存器**

### openEuler的增强

* 为充分发挥鲲鹏处理器的优势，**openEuler对通用Linux操作系统作了增强**。

* openEuler在**多核调用技术**、**软硬件协同**、**轻量级虚拟化**、**指令级优化**和**智能优化引擎**等斱面做了增强。

* 轻量级虚拟化：**提供iSulad 轻量级容器全场景**解决方案

### 鲲鹏加速引擎KAR

openEuler通过提供鲲鹏加速引擎（Kunpeng Accelerator Engine，KAE）插件，使能Kunpeng硬件加速能力，包括： 

* ==**对称/非对称加密**== 
* ==**数字签名**== 
* ==**压缩解压缩等算法，用于加速SSL/TLS应用和数据压缩**==