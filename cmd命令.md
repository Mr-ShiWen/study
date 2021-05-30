# Java命令

以下的类全名是：类的所属包名+类简单名称。一般形式是 layOut.next.NextTest，在cmd里面用layOut/next/NextTest表示（javac用斜杠（\）或反斜杠（/）都没问题，但java则只能反斜杠（/））

## 一、编译（javac）

javac   -d 目的路径名       类全名.java / *.java            //编译某个类及其相关类，过后的.class文件放到目的目录下

javac                                  类全名.java / *.java            //编译所有类，过后的.class文件放到源文件的同目录下

javac   -encoding utf-8   类全名.java / *.java             //指定编码格式的编译，这样执行时才能正确输出中文



javac注意：javac的编译行为有两个阶段：编译出目标文件+保存目标文件。这两个阶段是顺序但相互独立的

* 编译阶段
* *   1、按类全名编译时：javac会先按类全名的目录结构寻找源文件、再验证源文件里面的全名与给定的类全名是否符合。只有符合才会编译
  *   2、指定的源文件路径里面有 * ：javac会按照路径找到所有文件并编译，这时候javac不会验证类全名
* 保存阶段
* * 1、指定目的位置：取得class文件里的全名、把class文件按照其全名保存在目的位置的相应结构中。也就是说假如class文件里全名中含有包名，那么将会在目标位置按照包名创建相应的目录与子目录，并把class文件保存在里面。当然，如果目标位置本来就存在对应的目录结构，那就不用新建了，直接保存在相应目录里即可
  * 2、不指定目标位置：直接保存在源文件所在目录，不考虑class文件全名

## 二、打包成jar（jar）

jar   -cvmf   manifest.txt   结果包名字   类全名.class / *.class / 整个文件夹 / *(所有文件与文件夹)  



jar 注意：

* manifest.txt用于指定含有main方法的类，内容例如 Main-Class: layOut.next.NextTest1  ，内容冒号后面空一格，结尾回车
* 执行这个命令前，需要把manifest.txt与相应的包文件（没有包时就是类文件）放在同一相对目录下，然后利用jar把它们打包在一 起，以便执行时，按照manifest.txt找到相应的主类执行。可以认为jar包里面就是该包自身的起始位置，manifest.txt就在这样的起始位置上，执行jar文件时时，是从其起始位置开始寻找类来执行的
* 为了便于观察jar运行，可以打开cmd后，在执行java指令运行jar。如果想双击jar运行后cmd不闪退，可以在程序里加上 System.in.read(); 这里需要做异常 Exception 处理
* 源码文件也可以打包成jar包，但是执行不了，只有字节码文件打包成jar包后才能执行

## 三、执行（java）

java                      主类全名              //这里不用加.class

java        -jar        JAR文件名.jar      //注意：当执行jar包时，不能依赖任何外部的类或包，否则找不到

java       -cp 路径1;...;路径n     主类全名     //这里不用加.class

  //-cp表示指定路径，逻辑上相当于把路径1到路径n的内容组成一个“当前目录”，再在此“当前目录”执行任务。如果涉及到当前目录，则需要用.把它加入“当前目录”。注意这里的路径名就是文件夹路径，不能是文件路径，否则出错。但.jar文件可以看做是一个文件夹，把它加入路径没问题的。路径名间不能有空格

## 四、反编译(javap)

所谓反编译，是指翻译编译后的字节码文件

javap   -verbose   Test   //-verbose表示显示的意思



## 小结

对的情况寥寥数种，错的情况千奇百怪。我们主要关注对的情况即可，出错时寻找哪里不符合对的要求即可



# MySQL相关

## 一、cmd操作mysql的指令

* 启动：net start mysql55（这取决于你的服务名称）

* 停止：net stop mysql55（这取决于你的服务名称）

* 本机3306端口进入服务器：mysql -u 用户名 -p 密码 

* 进入服务器（指定端口号）：mysql -h ip地址 -u 用户名 -p密码 -P 端口号 //【mysql ip 用户名 密码 端口号】
            其中，[-h ip地址] 如果不填就默认为localhost，[-P 端口号] 如果不填就默认3306

    **注意：-p密码 一定是连着写的，或者只写-p。否则-p与密码存在空格的话，cmd认为你的密码是数据库名（连接上服务后选取哪个数据库）**
        
* 导出数据库：mysqldump -u 用户名 -p 数据库名 > 导出的文件名
             如：mysqldump -u root -p -P3307 gaokao>D:\mysql_outData\gaokao_out.sql

* 导入数据库：mysql> source 文件名
             如：source D:/mysql_outData/gaokao_out.sql /*注意这是反斜杠，与windows斜杠相反*/


  ​     导出注意：mysqldump的版本需要与导出数据库所在mysql服务的版本一致，否则容易出错

  ​     导入注意：source是一条mysql语句，需要进入mysql服务并选择数据库后再执行，但这是一条cmd专属语句，其他地方不能用

* 查看正在运行的事务：mysql>SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX\G; 

  //这是一条cmd专属语句

* 退出服务器：exit; //这是mysql语句，需要mysql服务执行

* 查看当前服务运行的端口号：show global variables like 'port'; //这是mysql语句，需要mysql服务执行

## 二、mysql服务的安装与卸载

要理解mysql服务的安装与卸载，我们需要了解“MySQL服务运行逻辑”。（本地搜索引号内容即可）

### 2.1 安装mysql服务

​      1、解压mysql安装程序文件
​      2、创建mysql配置文件my.ini放在解压后的根目录下（其他地方也行，安装时指定位置即可）；并在里面配置相关参数
​      3、cmd在bin目录下执行初始化创建data文件夹：mysqld --initialize --console  //这里要记住显示的初始密码
​      4、cmd在bin目录下安装服务：mysqld --install 服务名 --defaults-file=xxx.ini //服务名自定义
​      5、启动进入服务即可(无法启动时https://www.cnblogs.com/Kitty-/p/11632551.html）

### 2.2 卸载mysql服务

​      1、net stop mysql55 （这取决于你的服务名称）//停止服务
​      2、sc delete mysql55 （这取决于你的服务名称） //删除服务
​      3、删除数据库服务残留的data文件
​      4、清除注册表（一般执行第2步就会自动删除了，无需手动）
​         4.1、HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL55 目录删除
​         4.2、HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL55 目录删除
　　4.3、HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL55 目录删除

### 2.3 安装多个mysql服务

**核心：创建服务并指定ini文件 +通过ini文件制定程序与数据(即ini指定的data文件夹，没有时需要创建) **

#### a、总体思路

​        计算机上，一个MySQL服务对应着一个进程及其对应的数据，而不同进程可以共用一份代码。因此，要启动多个MySQL服务不需要复制多份安装文件，我们只需要一份安装文件即可。而一个服务只需要知道它的程序文件和数据的位置即可正确启动，所以不同服务间的程序可共享，但数据不能共享

   	而通常我们的my.ini文件里面有着（程序位置，数据位置）的信息，因此，我们只需要在安装多个mysql服务时指定不同的xxx.ini文件即可。即上面安装步骤的第4步：mysqld --install 服务名 --defaults-file=xxx.ini

​		这里要注意，我们在安装mysql服务时，需要存在xxx.ini所指定的数据文件。但如果手动创建则往往无法成功启动所安装的服务，因此我们可以利用安装步骤的第3步：mysqld --initialize --console  来创建

#### b、创建需要的data文件夹

​		为了可以成功创建出我们需要的data文件夹，我们先了解一下mysqld的初始化逻辑：如果存在my.ini，则按照它指定的路径创建（未创建就创建，已创建就不再创建），否则，将会在安装文件根目录下创建一个名为data的文件（已经存在就不再创建）

   因此，为了创建出新的data文件夹，我们可以：

​     **方法1**：如果不存在my.ini文件，那么我们可以先把我们的xxx.ini改名为my.ini，从而创建出里面指定的data文件夹。完事后再把初始文件名字改回去即可

​     **方法2**：如果存在my.ini文件，那么我们可以先把它移走，然后按照方法1来即可，完事后再把它移回来

​     **方法3**：如果不存在myini文件，且不存在data文件夹（存在就先移走，完事后再移回来），那么直接初始化生成data文件夹，然后把名字改为xxx.ini指定的即可

​     **方法4**：如果存在my.ini文件，那么我们可以先把它移走，然后按照方法3来即可，完事后再把它移回来

​     总结就是：要利用步骤3创建新的data文件，需要满足（有my.ini，但没有它上面指定的data文件夹）或者（没有my.ini，也没有名为data的data文件夹）

注意：安装服务和初始化这两步没有先后规定，服务安装好了，通过初始化也创建出相应的data文件夹了，那么就可以启动和使用该mysql服务了

   带密码初始化：mysqld --initialize --console 
   无密码初始化：mysqld --initialize-insecure --console  



# 端口号与进程

* 查看所有端口号使用情况：netstat -ano   //结果每一行的PID上的数据就是该端口号上的进程id
* 查看某一端口号使用情况：netstat -ano|findstr “80”   //这里是看80端口号的使用情况，需要查看其它端口号就替换掉80即可
* 查看某个PID对应的进程：tasklist|findstr "10152"  //这里是看PID为10152的进程，需要看其它PID进程，替换掉10152即可
* 结束进程（需要管理员权限）：taskkill /f /t /im java.exe  //这里是结束java.exe这个进程，需要结束其他进程，替换掉java.exe即可



参考：http://www.xitongcheng.com/jiaocheng/win10_article_48264.html



# DNS（域名系统）

* 域名解析：nslookup 需要解析的域名
