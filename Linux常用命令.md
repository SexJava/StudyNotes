# Linux常用命令

## 磁盘管理命令

1. ls命令:列出目录内容

   - 参数:
     - -a  查询所有文件和文件夹.包含隐藏的.
     - -l   查询详细列表(ls -l简写为ll)
     - -h  友好展示信息(ll -h)
   - 需求:展示某个目录下的内容
   - 所在位置:/root
   - 想要查看:/etc目录下的内容( ll -h /etc)

2. cd命令:切换目录

   - cd ../				(向上一层目录)
   - cd /                  (切换到根目录)
   - cd /目录名       (切换到指定目录中)
   - cd ~                  (切换当前用户家目录)
   - cd -                   (切换到上次访问的目录)
   - cd ..                  (上级目录)

   Linux绝对 :  cd /etc /x1     (先切换到/目录,然后在/目录中找到子目录etc,在etc中找到x1目录)

   Linux相对 :  cd x1/x2         (在当前目录寻找x1,进入x1中寻找子目录x2,必须确定x1存在)

3. pwd命令:显示当前所在目录(返回绝对路径)

4. mkdir命令:创建目录

   - mkdir 文件夹名称

     - -p 父目录不存在情况下先生成父目录

   - 需要在/root/t1目录下创建一个t2目录

     位置: /root

     命令: 相对:mkdir t1/t2;绝对:mkdir /root/t1/t2

   - 需要在/root/t3目录下创建t4目录

     位置:/root

     条件:t3和t4都不存在

     命令: mkdir -p t3/t4      

## 文件浏览命令

查看日志文件,xml,properties文件等

1. cat
   - cat 文件名  (快捷查看当前文件的内容,适合查看少量信息的文件.)
2. more
   - more 文件名  (分页显示文件内容)
   - 操作: enter(向下n行,默认1),空格键(向下滚动一屏),B(返回上一屏),Q(退出more)
3. less
   - less -mN 文件名 (分页显示文件内容并显示行号,使用大量数据的查看)
   - 操作: enter(向下n行,默认1),空格键(向下滚动一屏),B(返回上一屏),Q(退出less)
4. tail
   - tail -x 文件名   (可以快速查看文件后x行的内容)

## 文件操作命令

1. 文件复制

   - cp:复制文件或复制目录
     - 复制文件:cp 需要复制文件 复制的位置
       - 需求:把/root/test.txt文件复制到当前目录的t2文件
       - 命令:cp test.txt t2(相对) cp test.txt /root/t2(绝对)
       - 需求:把/root/test.txt文件复制到当前目录的t2文件,改名为111.txt
       - 命令:cp test.txt t2/111.txt
     - 复制目录:cp -r 需要复制目录 复制的位置
       - 需求:把/root/t5目录 复制到 /root/t1
       - 命令: cp -r t5 t1(相对)cp -r /root/t5 /root/t1(绝对)

2. 文件移动

   - mv命令:英东或更名现有的文件或目录
     - 文件/目录移动:
       - 语法:mv 需要移动的文件或目录 移动的位置
       - 需求:把/root/t3 移动到 /root/t3
       - mv t5 t3
     - 文件/目录改名
       - 需求:把/root/text.txt 改名为 111.txt
       - 命令:mv text.txt 111.txt

3. 文件删除

   - rm:删除文件或目录

     - 删除文件:rm 文件名(相对或绝对),rm -f 文件名(强制删除)

     - 删除目录:rm -rf 目录名(强制删除该目录及目录下所有内容)

       rm -rf *(删除当前目录下所有内容)

       rm -rf /*(删除Linux系统根目录下所有内容)

4. 查找文件/目录

   - find [目录] [参数]:查找文件或目录

     -name(指定字符串作为寻找文件或目录的范本样式)

     - 语法:find /home/LiuYunDa/测试 -name 't*'     



## 文档编辑命令

1. vim命令(编辑器)
   - vim 文件名      进入一般模式,按a等进入插入模式,按esc切换到一般模式,按:可以切换到底行模式
   - 插入模式:编辑文件内容
   - 底行模式:可以强制退出(q!)和保存退出(wq)

## grep和管道命令

1. grep:表示正则表达式.字符串搜索工作
   - grep 需要搜索的字符串  搜索的文件(grep -i忽略大小写)
2. 管道命令: 可以连接多个Linux命令
   - 需求:查询当前目录中,所有带t关键字的行数据	
   - 命令:ll | grep t

​		 9

## 压缩,解压命令

Linux压缩包:*.tar(打包,大小不会改变)    *.tar.gz(打包并压缩文件的大小)

1. 压缩命令(参数的顺序不能变)

   - tar  -zvcf  压缩包的名字.tar.gz   需要压缩的内容
   - 例:tar -zcvf haha.tar.gz *   (将当前目录下的所有内容进行打包压缩)

2. 解压命令

   - tar -zxvf  需要解压的压缩包的名称

   - tar -zxvf  需要解压的压缩包的名称  -C  指定压缩路径(注意大写C)

     

## 系统命令

1. ps命令:提供对进程的一次性查看,即执行ps命令的那个时刻的进程信息

   - ps -ef

2. Kill 命令

   强制杀死系统进程: kill -9 pid 号

   需求:查看进程和vim相关的进程,并将vim进程杀死

   命令:ps -ef | grep -i vim

   ​		Kill -9 pid 号

3. ifconfig命令:显示网络设备

   命令:ifconfig

4. ping命令:测试与目标主机的连通性

   命令:ping  主机名或ip地址

5. reboot:重启命令

6. halt:关机命令

## 其他命令

1. 网络设置:setup命令

2. 文件权限:chmod命令

   ![](C:\Users\Administrator\Desktop\Linux权限格式.PNG)

   r: 读取权限  w:写入权限   x:执行权限   -:没有权限

   - 权限更改

     语法: chmod 权限设置 需要更改权限的文件名

     ​		:chmod -R 更改当前目录所有文件的权限
     
   
3. 程序安装命令

   - rpm命令:相当于windows的添加/卸载程序,进行程序的安装,更新,卸载,查看
     - 本地程序安装:rpm -ivh 程序名
     - 本地程序查看: rpm -qa
     - 本地程序卸载: rpm -e --nodeps 程序名
   - yum命令:相当于可以联网的rpm命令(先联网下载程序安装包,自动执行rpm命令)

4. Linux配置防火墙

   - 停止并屏蔽firewalld服务

     ```
     systemctl stop firewalld
     systemctl mask firewalld
     ```

   - 安装iptables-services软件包

     ```
     yum install iptables-services
     ```

   - 在引导时启用iptables服务

     ```
     systemctl enable iptables
     ```

   - 启动iptables服务

     ```
     systemctl start iptables
     ```

   - 保存防火墙规则

     ```
     service iptables save
     ```

   - 管理iptables服务

     ```
     systemctl [stop|start|restart] iptables
     ```

   - 开启端口

     ```
     /sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT
     ```

   - 保存配置

     ```
     service iptables save
     ```

     

