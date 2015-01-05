#ubuntu的init与系统服务设置

##init
  Init是位于/sbin/init的一个程序，它是在linux下，在系统启动过程中，初始化所有的设备驱动程序和数据结构等之后，由内核启动的一个用户级程序，并由此init程序进而完成系统的启动过程。  
ubuntu与传统的linux略有不同，使用upstart完成系统的启动，但表面上仍维持init程序的形式。

##运行级别
  传统上，linux有几种不同的运行级别，包括如下几种：
```cpp
0 - 停机
1 - 单用户模式
2 - 多用户，没有 NFS
3 - 完全多用户模式(标准的运行级)
4 – 系统保留的
5 – X11 （x window)
6 - 重新启动
```

  系统启动后处于哪一种级别由init读取/etc/inittab文件中的缺省级别设置来确定，一半图形界面的系统是进入级别3。
	但是ubuntu与传统的不太一样，默认情况下是找不到/etc/inittab文件的，而且运行级别也有差别，具体分这样几个级别：

```java
0 – 关闭系统
1 – 单用户模式
2~5 – 完整的多用户模式
6 – 重新启动
```

也就是说，默认情况下级别2、3、4、5都是一样的，同时系统的默认级别设定也不是在inittab文件中，而是写在/etc/init/rc-sysinit.conf文件中。打开此文件，可以找到下面一句：
```java
	env DEFAULT_RUNLEVEL=2
```
这表明系统当前默认是进入级别2。
 
另外，在此文见中还有一段以if [ -r /etc/inittab ] 开始的代码，这里保留了使用inittab指定系统默认运行级别的功能，也就是说，如果用户手动创建了/etc/inittab，那么init将以 /etc/inittab中指定的默认运行级别进行系统的启动。比如说用户希望系统以级别3为默认运行级别，则只需在inittab文件中加入如下一行：
```java
	id:3:initdefault:
```
在经过'/etc/init/rc-sysinit.conf'确定运行级别后，init将进一步运行/etc/init.d/rc，然后根据级别进入/etc/rc[?].d启动或关闭相应的服务。

##服务的启动与关闭脚本
ubuntu下启动与关闭服务的脚本存放与/etc/rc[?].d目录下。其中[x]表示0~6，分别对应级别0~6，如/etc目录下的 rc0.d  rc1.d  rc2.d  rc3.d  rc4.d  rc5.d  rc6.d。假设rc-sysinit.conf或inittab中指定的默认级别是2，那么init将执行/etc/rc2.d目录下的脚本启动或关 闭相应服务。

如果打开/etc/rc[?].d目录，会发现这些目录下的文件都是形如Snnxxxx或Knnxxxx的符号链接，而且都是指向/etc /init.d。也就是说不同运行级别下服务的启动或关闭脚本均是放在/etc/init.d下，只不过根据不同级别的需要，在对应/etc /rc[?].d目录下放一个链接，不同的级别会需要不同的服务，因此不同/etc/rc[?].d目录下的链接文件也不尽相同以此区分。
 
其中链接文件中以S开头的表示在调用/etc/init.d目录中对应脚本的时候会传递一个start参数，也就是启动对应服务，而以K开头的则是传递一个stop参数，由此关闭此服务，此处的K表示kill。

S和K后面的nn是一个数字，表示本脚本被执行的先后顺序，小号在前大号在后，这样以解决不同服务之间可能存在的先后依赖关系。比如说ftp服务依赖于网络服务的启动，所以ftp服务的编号就要大于网络服务的编号，在网络服务启动后再行启动。
 
最后的xxxx则是服务的名字。

另外，除了/etc/rc[0~6].d文件外，还有一个/etc/rcS.d目录，这个目录下的服务脚本与/etc/rc[0~6].d格式类似，也为指向/etc/init.d中的脚本的链接，但是会在/etc/rc[0~6].d中的脚本执行前首先被执行。
 
 
##服务的安装
根据上面的介绍，如何将一个软件安装为服务也就比较清楚了，那就是在/etc/init.d添加一个服务的启动脚本，然后在需要启动服务的对应级别的/etc/rc[0~6].d中按照文件名格式添加一个指向/etc/init.d中脚本的符号链接。
 
以apache2为例，默认情况下，apache2编译安装在/usr/local/apache2，apache2的服务器启动脚本是/usr /local/apache2/bin/apachectl，那么安装服务就是要把此apachectl拷贝到需要启动apache2服务器的运行级别对 应的/etc/rc[?].d目录下，一半来说ubuntu是运行在级别2下，所以也就是拷到/etc/rc2.d下，命令如下：
```java
	sudo cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
```
在这里，我们把拷贝的脚本文件名改为惯用的httpd，那么在系统服务中apache2的名称也就是httpd。

手动添加服务的话，就是要在/etc/rc2.d里为httpd作一个形如Snnxxxx的链接。假定启动顺序我们定为80，那么就是要建立一个指向/etc/init.d/httpd的链接/etc/rc2.d/S80httpd，命令如下：
```java
	sudo ln -s /etc/init.d/httpd /etc/rc2.d/S80httpd
```
然后用sysv-rc-conf或者chkconfig –list检查一下就能看到已经在运行级别2下建立的名为httpd的服务，重启后，系统会自动启动apache2。
 
现在要手动启动、关闭或重启httpd服务器可以使用service命令+服务名+参数的形式，如下三句分别是启动、关闭和重启apache2服务器的命令：
```java
	service httpd start

	service httpd stop

	service httpd restart
```
要安装服务除了以上手动作链接外，还可以使用一些工具软件来实现，比如常用的有update-rc.d、chkconfig和sysv-rc-conf等。

这里主要以update-rc.d为例
```java
	update-rc.d
```
如用update-rc.d在运行级别2,3,4,5中都添加httpd服务启动命令，并指定启动序号是80，可以如下（注意命令最后的那个点号不能少）：
```java
	sudo update-rc.d httpd start 80 2 3 4 5 .
```
如果要在运行级别2,3,4,5中都添加httpd服务关闭命令，并指定关闭序号是80，则可以如下（注意命令最后的那个点号不能少）：
```java
	sudo update-rc.d httpd stop 80 2 3 4 5 .
```
而如果要删除httpd服务，则用以下命令就删掉/etc/rc[?].d中所有存在的指向/etc/init.d/httpd的链接：   
```java
	sudo update-rc.d httpd remove
```
另外，还可以用带有defaults参数的形式同时向运行级别2,3,4,5中添加启动服务命令，即Start命令，并同时向运行级别0,1,6添加关闭命令，即Kill命令，其中start命令的序号是80，kill命令的序号是90：
```java
	sudo update-rc.d httpd defaults 80 90
```
这条命令也等同于（注意命令stop前面和最后的那两个点号不能少）：
```java
	sudo update-rc.d httpd start 80 2 3 4 5 . stop 90 0 1 6 .
```
以上两条命令的效果就是作了如下几个链接：
```java
	/etc/rc0.d/K90httpd -> ../init.d/httpd

	/etc/rc1.d/K90httpd -> ../init.d/httpd

	/etc/rc6.d/K90httpd -> ../init.d/httpd

	/etc/rc2.d/S80httpd -> ../init.d/httpd

	/etc/rc3.d/S80httpd -> ../init.d/httpd

	/etc/rc4.d/S80httpd -> ../init.d/httpd

	/etc/rc5.d/S80httpd -> ../init.d/httpd
```

##chkconfig
查看所有服务在所有级别下的情况：

chkconfig –list

查看某服务的情况，如下将查看httpd服务在哪些级别下是被启动的：

chkconfig httpd

在此，如果httpd在2,3,4,5下是被启动，将返回信息：

httpd 2345

##sysv-rc-conf
 
这个软件是有图形界面的，比较直观简单，就不多说了，看看就明白了。

rc.local
在/etc/rc[2~5].d目录下都会有一个S99rc.local，这是一个指向/etc/init.d/rc.local的链接，可以看 出，在正常的2~5级别启动的最后都会调用这个rc.local脚本，而/etc/init.d/rc.local中又会检查是否存在/etc /rc.local，并运行之，因此，我们就可以在/etc/rc.local中写入代码，来随系统启动某些程序，实现类似服务的功能。
 
系统的启动过程
综上，可以看到，系统的启动调用过程就如下过程：

>内核 → /etc/init/rc-sysinit.conf → [/etc/inittab] → /etc/init.d/rc → /etc/rc[?].d → /etc/init.d/rc.local → /etc/rc.local

其他linux系统
在其他系统下以上的文件结构和过程略有不同，以Redhat系的CentOS5为例，系统中默认init是使用/etc/inittab文件的，然后读取/etc/rc.sysinit，再根据运行级别进入/etc/rc[?].d。

其中，/etc/rc.sysinit是指向/etc/rc.d/rc.sysinit的链接，/etc/rc[?].d是指向/etc/rc.d /rc[?].d的链接，/etc/rc.local是指向/etc/rc.d/rc.local的链接，所以系统启动的顺序就变成如下：

>内核 → /etc/inittab → /etc/ rc.sysinit（/etc/rc.d/rc.sysinit） → /etc/rc[?].d（/etc/rc.d/rc[?].d） → /etc/rc.local（/etc/rc.d/rc.local）


