# 路径

* 在 JavaSE 中路径分为**相对于工程的路径**和**相对于磁盘的路径**

```
相对于工程的路径:
	不同的情境下，相对路径不同。最普遍的相对路径是相对于classPath。但在idea的模块上执行时，有时候相对路径不一定是classPath，甚至同一份代码通过main调用执行与通过@Test执行时，其相对路径都不同，这点要注意。但一般单独执行class或jar时（不通过IDE），相对路径都是classPath
	classPath获取：        
		URL url = this.getClass().getResource("/");
        String path = url.getPath();

   
    System.getProperty("user.dir")，这个可以获取当前工程空间路径，工程空间路径与类所在文件夹没有联系，是jvm的系统属性，置于这个属性怎么来的，暂时还不清楚


相对于磁盘的路径:
    盘符:/目录/文件名 ，如 ‪D:\IntelliJ IDEA\project\src\book.xml
```



* 在 web 中路径分为相对路径和绝对路径两种

```txt
资源文件（html、jsp等）里面的相对路径:  
          . 表示当前文件所在的目录 或者是 当前目录
          .. 表示当前文件所在的上一级目录 或者是 上一级目录
          文件名 表示当前文件所在目录的文件,相当于 ./文件名 ./ 可以省略 
          
    注意：这里的相对路径，在没有base标签的情况下，将以浏览器地址栏中的地址为参考系，如果存在base标签，将以base标签为参考系。以base标签为参考系能够使得按相对地址的跳转不受浏览器地址栏地址的影响，这样可以避免当资源路径与浏览器地址栏的路径不同时（例如请求转发送得到的资源，其路径与地址栏路径不同）发生的跳转错误
    
    

普通java路径（与JavaSE一样）：

   相对于工程的路径：
      从tomcat的bin目录算起（因为项目的class文件是连同tomcat一起在同一个jvm上面执行的，可以认为这些class文件属于tomcat代码的一部分，而tomcat本身的java代码所在的空间就是其安装目录下的bin目录）
      
   相对于磁盘的路径：
       盘符:/目录/文件名 ，如 ‪D:\IntelliJ IDEA\project\src\book.xml
       
       
       
工程路径：
    当里面涉及到按工程解析路径时，一般把 "/" 解析为 http://ip:port/工程名。工程路径可以出现在java中，也可以出现在其他资源中，关键看是否需要工程路径


绝对路径: 
          正确格式是: http://ip:port/工程名/资源路径 
          错误格式是: 盘符:/目录/文件名
```

一般来说：

​		web阶段：base标签+相对地址

​		框架阶段：绝对地址（http://ip:port/工程名/资源路径 ）

# 前后端数据传递

## 一、传递方式

### 1.1、普通函数调用

前端数据传递给后端主要通过 **$.post** 或者 **$.ajax** 方法，关键的地方在于 url（字符串形式）、参数（对象形式）、与回调函数，具体细节如下：

```javascript
   
    //$.post 与 $.ajax 的实参与返回数据接收形式类似; 

    //后端通过request.getParameter("keyName")取得实参数据

    //后端返回数据直接return即可，但是函数需要用@ResponseBody注解，这样后端数据才会转换成JSON或XML数据，以便返回

	//前端通过data接受的后端返回数据是已经被解码回来的，即 后端数据->(编码json或xml)->(解码回数据或对象)->前端接收，中间编码解码过程有框架实现了，前后端不需要再实现

    //置于前端到后端的过程能否实现编码解码由框架自动完成，还需要实验探究

	//前端调用的后端方法一定要用@ResponseBody注解，这样前端的回调函数才能执行。否则由于没有返回，导致前端不知调用是否结束，就不会往下执行回调函数了。另外，@ResponseBody注解不能让函数类型是void，否则出错

	$.post("${ctx}/operationcenter/taskhead/taskHead/method1",  {name:name, age:age},function(data){
                        if(data){
                            $("#forwarderAddr").val(data[0]);
                            $("#forwarderContact").val(data[1]);
                            $("#forwarderTelephone").val(data[2]);
                        }
   });


    var peoples=[{name:"张三丰", age:88},{name:"刘德华", age:43}];
    $.ajax({
        type:'POST',
        data:{peoples:JSON.stringify(peoples), word:"hello world"},
        /* contentType :'application/json; charsetset=utf-8',*/
        dataType:'json',
        url :"${ctx}/operationcenter/taskhead/taskHead/dataTransmission2",
        success :function(data) {
            alert(data);
        },
        error :function(e) {
            alert("error");
        }
    });
```

### 1.2、页面跳转

前端通过调用后端的相关函数，实现页面跳转

```html

<!-- 参数直接放在跳转链接上，参数与链接通过？号隔开，参数之间通过&号隔开 -->
<!-- 后端通过request.getParameter("keyName")取得实参数据 -->
<!-- 由于是跳转页面，后端将返回页面位置的url字符串，而参数存放在model里面，以便在新的页面通过"${keyName}"取得 -->
<!-- 置于后端返回的url与model怎么处理，到最终打开新页面，这个过程暂时不清楚 -->

<a href="${ctx}/operationcenter/taskhead/taskHead/sendUserFrame?selTdUserID="+selTdUserID+"&selTdUserName="+selTdUserName+"&articalID="+articalID>发货单管理列表</a>

```



## 二、数据形式（entity、json）

### 2.1、entity

后端把数据库信息保存在entity的成员变量上，再传递给前端相应的页面字段；前端在保存的时候把相应的页面字段数据保存在entity的成员变量中再返回给后端。可以认为entity是后端与前端的通信中介。更进一步，数据库与前端的通信中介就是entity，后端是它们的中间层而已。entity本质保存在域中（request域、session域、servletContext域，注意不能是pageContext域，因为它作用范围仅限于本页面），请求转发后在jsp页面取出来即可



注意：

* 前端通过entiy提交给后台的json字符串里面的引号字符 \” 会被额外转换成  & quot;  ，可以通过StringEscapeUtils.unescapeHtml4 (jsonStr) 方法转换回来，以便json工具处理这些json字符串

* 后端通过entity传递给前端的json字符串本来是正常的，但前端通过"${xxx}"无法识别json字符串里面的双引号 \“，可能因为"${xxx}"在页面加载前执行，无法使用html处理。不然正常的html是可以识别出json字符串里面的双引号 \“ 的。**这种情况可以通过把json字符串里面的双引号（即 \ "）替换成单引号(即 ' )即可。但是在前端一定要把单引号替换回双引号，否则无法把json字符串转换回正常数据**。如何替换请看https://www.w3school.com.cn/jsref/jsref_replace.asp

  

* 前端通过**$.post**， **$.ajax** 调用后端方法，后端通过 request.getParameter（“keyName”）得到的json字符串则正常，不会被转换为 & quot

* 后端方法被调用完毕返回的json字符串在前端也能正常识别。说明json字符串格式在前端后端都可以正确处理。当然后端返回数据一般不需要转成json字符串，通过@ResponseBody注解会自动完成数据的编码解码，这样后端只管传送数据，前端只管获取数据即可，中间的编码解码过程由@ResponseBody注解自动完成



### 2.2、json

下面主要围绕传递的数据形式进行讨论，即前后端通过什么样的数据进行交流

前后端数据传递主要使用 json 字符串，它们把需要传递的数据转为json字符串后传递给对方，对方接收后，把json字符串解析为自己的数据即可。其中前端主要是通过 JSON这个类操作，后端则可以通过不同的json解析工具操作，其中org.json和json-lib比较简单

#### json定义

json字符串里面内容可以分为数据、格式字符，格式字符主要有“[ ]”与“{ }”，前者代表数组，后者代表对象，数据又分整型、浮点型、字符串、布尔型等。需要注意，如果是字符串数据，其引号需要用“\”这个符号转义，因为这是json字符串里面的字符串。对象有多个属性名与值，这在json字符串中通过键值对来表示，key是一个字符串数据，表示属性的名字，value则可以是各种类型数据，表示属性的值



json字符串常用来传递对象或者数组，当然也可以用来传递基本类型数据，不过这样意义不大，基本数据直接传递即可，何必多此一举搞成json字符串呢



可以把json字符串里面的内容类比我们的编程语句，只不过这个语句有比较大的限制罢了。具体可以看下面的例子



**json字符串里面可以包含json字符串，即字符串可以层层包含字符串，这在json解析时会处理的，不用我们操心**



**注意：后端通过@ResponseBody注解返回json字符串时：**

​      1、会把原 null 写成 undefined   

​      2、返回String时不转换为json字符串格式。因此在前端将会是error的结果

#### 前端处理json

前端的对象一般不会类型之分，可以任意设置属性与方法。因此json字符串与前端对象可以很方便的转换

* 数据（普通数据或对象）转为json字符串：JSON.stringify(data);
*  json字符串转为数据（普通数据或对象）：JSON.parse(jsonStr);

``` javascript
    //随便计算
    function calculate() {
        //本身
        var a=getArray();
        alert("本身:"+a+",  length:"+a.length);
        //转换为json
        var b=JSON.stringify(a);
        alert("json:"+b); 
        //从json转换回来
        var c=JSON.parse(b);
        alert("json翻译: "+c);
    }
    function getArray() {
        var a=[{name:"张三丰", age:88},{name:"刘德华", age:43}];
        var b=[1,2,3,4];
        var c=[[1,2,3],[4,5,6]];
        var d={name:"张三丰", age:88};
        var e="hello world"
        return a;
    }
```



#### 后端处理json

后端比如java，由于java对象是有类型的区分的，因此json字符串要转化为对象需要指明转换成什么对象。一般网上提供的json工具包，都把json字符串转换为 JSONObject或JSONArray类对象，当然，这两个类也是工具包提供人自己定义的。不同工具包，往往有不同的定义。当然。可以自己写个方法或类来解析或生产json字符串，这可以作为自己的练习，但工作中可以利用别人提供的这些工具包

#### org.json工具包

``` java
    public static void main(String[] args)  {
        String jsonStr="{name:\"张三丰\",age:88}";
        String jsonArrStr1="[{name:\"张三丰\", age:88},{name:\"刘德华\", age:43}]";
        String jsonArrStr2="[\"张三丰\",\"太极拳\",\"张无忌\",\"九阳真经\"]";
        try{
            //把json字符串解析为JSONObject类对象（json对象），或者JSONArray类对象（json数组）
            JSONObject jsonObject=new JSONObject(jsonStr);
            JSONArray jsonArray1=new JSONArray(jsonArrStr1);
            JSONArray jsonArray2=new JSONArray(jsonArrStr2);
            
            //获取json对象里面的值
            String name=jsonObject.getString("name");
            int age=jsonObject.getInt("age");
            
            //获取json数组里面的值（数组元素是json对象）
            for(int i=0;i<jsonArray1.length();i++){
                JSONObject e=jsonArray1.getJSONObject(i);//这里如果getString就会异常，因为数组元素是JsonObject类型
                name=e.getString("name");
                age=e.getInt("age");
            }

            //获取json数组里面的值（数组元素是基本数据）
            for (int i = 0; i < jsonArray2.length(); i++) {
                String elem=jsonArray2.getString(i);
            }

        }catch ( JSONException e){
            e.printStackTrace();
        }
    }
```

#### json-lib工具包

待续



## 三、前端传数据给后端的中文乱码问题

​		有时候，为了让中文字符适应某些特殊要求，可能会通过将中文字符按照字节方式来编码的情况， 如在按照**post**方式提交时，http header头要求其内容必须为iso8859-1编码

​		这样，前端传递的 A（utf-8）字符串将会只改变编码变成B（ISO-8859-1），但其背后的字节信息是不变的，在后端如果按照传输过来的数据及其编码来解读的话，将会得到B（ISO-8859-1），为了得到A（utf-8），只需要把编码方式从ISO-8859-1更换成utf-8即可.

猜测：当B是比较特殊的字符时，将会显示乱码

### 方案一（不推荐）：

得到B（ISO-8859-1），再通过ISO-8859-1编码方式得到B的字节（也就是A的字节，因为字节不变），更换编码方式为utf-8

```java
String name = new String(request.getParameter("name").getBytes("ISO-8859-1"),"utf-8");
```

这里，String 的getBytes（decode）是为了得到字符串的原字节信息，以便重新指定编码

### 方案二（推荐）：

```java
request.setCharacterEncoding("UTF-8");
```

这里将直接设置数据的编码格式（不改变信息字节），不过值得注意的是，**这句一定要在获取数据前设置才有效，request一旦获取过任何数据后，这种设置就无效了**



## 四、后端传数据给前端的中文乱码问题

​	当服务器通过 response 给客户端返回信息时，如果出现中文，在浏览器上面就会出现乱码。因为response默认采用ISO-8859-1编码，但这个编码不支持中文，即中文无法被编码出来，即使设置浏览器同为ISO-8859-1也无法解决问题。要解决这个问题只能用另一种编码代替，例如utf-8。有两种方案：

### 方案一(不推荐)

步骤一：设置返回数据编码为 utf-8  //目的是让服务器按照这个格式编码字符串

步骤二：设置响应头的 contentType 键对应的值为 text/html；charset=utf-8  // 目的是告诉浏览器这是文本内容，且编码格式是utf-8 



代码如下：

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        //设置返回数据编码格式
        response.setCharacterEncoding("utf-8");

        //设置响应头部Content-Type，告诉浏览器这是什么内容，什么编码
        response.setHeader("Content-Type","text/html;charset=utf-8");
        
        //往客户端回传 字符串 数据
        PrintWriter writer = response.getWriter();
        writer.write("国哥很帅!!! ");
    }
```



### 方案二（推荐）

步骤一：设置ContentType即可    //这一步将完成方案一中两步的效果,即既设置了服务端的返回数据编码，也设置了响应头的ContentType键

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //设置ContentType即可
        response.setContentType("text/html;charset=utf-8");

        //往客户端回传 字符串 数据
        PrintWriter writer = response.getWriter();
        writer.write("国哥很帅!!! ");
    }
```

**注意：此方法需要在获取流之前使用才能生效，否则无效**



# 依赖添加

我们的程序运行往往需要依赖别的代码资源，依赖添加可以帮我们简便开发，不用全都自己写

以下有两个概念：**资源项**（指资源文件、文件夹等）、**依赖项**（即运行时依赖的文件、文件夹等）。其实依赖项就是被依赖使用的资源项罢了。如idea的libararies里面是资源项，模块的Dependencies 才是真正的依赖项



**注意：资源项、依赖项都只能是两种形式：**

​		**1、jar**   

​		**2、符合类全名的包**   

**这是为了在执行时在模块空间里能正确访问这些资源，其他形式的资源由于不能正确访问（比如类全名不符合包路径的类），因为无法依赖，这在代码编写的时候ide会提示没有该类**

一般library里面不能再套library，eclipse与idea都不能再套

## IDEA

1、给本地项目添加依赖：

 File ->  Project Structure ->Modules ->Dependencies 下添加依赖的 文件（jar）、文件夹（即包） 、 lib

​    当添加的资源不来自项目的文件、文件夹（即包）时，这些资源会自动添加到 External Libraries目录下，即使是libraries也认为是来自外部的资源项

​    module依赖来源：A、项目内部的 jar 包、文件夹（即包）、lib     B、项目外部的 jar 包、文件夹（即包）



2、idea里面依赖项是按照模块隔离的，不同的模块可以设置其不同的依赖，模块也只能在它自己的依赖上寻找，不能到别的模块的依赖里寻找。可以说一个模块就是一个执行空间，里面有自定义的类、包和依赖的类、包



3、添加第1点说的 Library（资源项） 在：

File ->  Project Structure ->Libraries 目录下，可以按需添加lib， 添加方式：

​      方式A ：Libraries 目录下，点击“+”号，选择文件或文件夹即可

​      方式B：在项目下创建文件夹或模块，把依赖资源黏贴进去，右击该文件夹或模块，点 Add as Library  并选择用于project即可。**如果只是用于module，则不会添加到 Libraries为工程所有模块共享，只会形成专属于该模块的 library**



这些Library只是资源项，只有在第1步的module模块里面设置引用了，才能被模块使用，否则模块不能使用。模块里面的才是它的依赖项



4、给web项目添加依赖项：web项目由于运行在容器里面（例如 Tomcat）

因此还要选择 Artifacts 选项，将类库，添加到打包部署中：

![image-20201213185640710](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213185640710.png)



总结如下图：

![module的依赖](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/module的依赖.jpg)





参考：

​    1、https://blog.csdn.net/hwt1070359898/article/details/90517291

​    2、https://blog.csdn.net/yinyanyao1747/article/details/90751024

​    3、尚硅谷javaWeb第5课（xml与tomcat）的资料



## eclipse

1、给平台即eclipse添加资源项以备使用：

​     window-> preferences-> Java-Build Path-> User Libraries  ,之后点击new按钮创建自己的library，可以自己命名（相比之下，idea的library不能自命名）。创建后往里面添加资源项即可



2、给本地项目添加依赖项：

​    右键该项目->（最下方）properties-> Java Build Path-> Libraries窗口-> 点击添加选择资源（可以是jar、文件或者是平台的library）即可



3、给web项目添加依赖项：web项目由于运行在容器里面（例如 Tomcat），因此依赖需要添加在 WEB-INF/lib 目录下，否则找不到：

  右键该项目->（最下方）properties->deployment assembly-> 点击Add-> 选择 Java Build Path Entries（第2步里面的资源）或其他资源- >选择依赖资源-> finish即可



参考：https://blog.csdn.net/yogima/article/details/81186092



# java测试api

​      我们通常的写的非main方法是不能直接测试的，只能通过main方法调用。为了可以更方便的测试我们写的方法，可以通过在方法上加上注解 @Test 即可，当然这里的Test来自org.junit.Test，需要引入单元测试包 **junit-4.12.jar** 包，由于这个版本的junit包不含有hamcrest，因此还需要引入 **hamcrest-core-1.3.jar **包

​	总的来说，引入 **junit-4.12.jar** 包与 **hamcrest-core-1.3.jar** 包后，通过给方法加上 @Test 注解即可测试该方法

​    注意：@Test的单元测试不能测试含有参数的方法，如果需要测试还有参数的方法，给它套一层无参的调用层即可



# IDEA的debug

## 准备

1、断点，只需要在代码需要停的行的左边上单击，就可以添加和取消 

2、Debug 启动 Tomcat 运行代码： 

![image-20210114210003714](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210003714.png)

## 测试工具栏

![image-20210114210119395](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210119395.png)

1、让代码往下执行一步

![image-20210114210146311](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210146311.png)

2、可以进入当前方法体内（自己写的代码，非框架源码）

![image-20210114210826567](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210826567.png)

3、跳出当前方法体外 

![image-20210114210847506](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210847506.png)

4、强制进入当前方法体内

![image-20210114210909454](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210909454.png)

5、一直执行，最后停在光标所在行（相当于临时断点） 

![image-20210114210931241](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114210931241.png)

## 变量窗口

它可以查看当前方法范围内所有有效的变量

![image-20210114212435493](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114212435493.png)

## **方法调用栈窗口** 

1、方法调用栈可以查看当前线程有哪些方法调用信息 

2、下面的调用上一行的方法

![image-20210114213023050](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114213023050.png)



## 其他相关按钮

![image-20210114214116551](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210114214116551.png)



# 页面三剑客

​		页面三剑客主要是html、css与javascript，其中html是负责页面显示内容，css负责显示的样式，javaScript负责页面的执行代码。css和javaScript是结合进html才能生效的

​		为了简化操作，jquery又在javascript基础上进一步封装，通过符号`$`大大简化了运行代码

​		layui则是为了美化页面而来的，其底层依然是css与javaScript



## HTML常用标签

HTML的标签主要的作用是告诉浏览器怎么显示、显示什么。一个标签是一个显示元素。

HTML的标签分为单标签与双标签，其中单标签格式是<xxx/> 或<xxx>（实际dom元素里的html是<xxx>)，双标签格式是<xxx></xxx>

### 表格

* **table**：表格标签     （border：设置边框    width:设置宽度    height：设置高度   align：设置对齐   cellspacing：单元格间距）

* * **tr**：行标签 
  * * **td**：单元格标签。扩展时会影响其他行的水平分布，通过删除相应单元可以对齐    （align：设置内容对齐   colspan：列方向扩展  rowspan：行方向扩展）
    * **th**：表头单元格标签。与<td align="center"><b>内容</ b></td>效果相同，其中 b标签是加粗标签。即**加粗居中**

* * **colgroup**：列组标签，只要放在 table 里面即可，无论在table里什么位置，都作用于整个table
  * * **col**：列标签，用于设置每列的样式等，可以利用span属性进行列扩展

* * **thead**：表头标签，里面存放 tr、th 标签
* * **tbody**：表身标签，里面存放 tr、th 标签 
  
  ```html
  <table border="1">
    <colgroup>
      <col span="2" style="background-color:red">
      <col style="background-color:yellow">
    </colgroup>
    <tr>
      <th>ISBN</th>
      <th>Title</th>
      <th>Price</th>
    </tr>
    <tr>
      <td>3476896</td>
      <td>My first HTML</td>
      <td>$53</td>
   </tr>
  </table>
  ```
  
  

注意：cellspacing设置单元格间距为0时只是把边框紧挨在一起，但并没有合并。若要合并，需要用到border-collapse： collapse 样式

### 列表

* ul：无序列表标签，unordered list （type：类型）
* * li：列表项标签，list item
* ol：有序列表标签，order list

参考：<a href="https://blog.csdn.net/qingyisuo/article/details/78921185">有序列表与无序列表</a>

### iframe框架

* iframe：框架标签，用于存放页面（src：页面或资源路径  name：框架名称，与a标签配合）。注意iframe里面存放的页面是iframe所在页面的子页面

* 子页面和父页面可以相互访问方法与变量，以及dom对象 https://www.cnblogs.com/xcsn/p/9475716.html

* 补充子页面访问父页面的dom对象：$("#selIds",parent.document)  //通过 Jquery ，需要两个参数（选择器，父页面的document）

  

### 超链接（a标签）

* a：超链接标签（href：链接，即资源路径    target：目标，用于设置新页面打开位置，设为iframe的name时，将在iframe打开） href="#"时表示链接到当前页面

target：_Blank(新页面打开)

  注意：a标签的跳转是触发a标签单击事件而实现的，但是必须是通过a标签的后代标签冒泡上来触发操作才有效，也就是说，直接触发a标签单击事件只能响应用户设定的响应函数但不会跳转，只有点击a标签的后代标才可以跳转，a标签只有监听到从后代冒泡上来的单击触发动作才会触发跳转。

  页面上，点击a标签里面的文本即可激发跳转

  函数里：就要激发a标签的后代标签才能激发跳转

  总之：用户响应函数只要单击就行，而跳转则需要后代冒泡上来的单击才有效

当跳转时，真正的地址是  **href值?name1=val1&name2=val2&...&namek=valk** 的形式



### 表单（form）

* form：表单标签  （action：提交的服务器地址   method：提交方式，有get与post，默认为get）
* * input：输入标签    （type：输入类型    name：名称   value：默认值   checked：选择情况  style='width: 15px;'：设置宽度） 

​           1、 文本输入框：type=“text”

​           2、密码输入框：type=“password”   输入的密码将会被用小圆点隐藏起来

​           3、单选输入框：type=“radio”，配合name属性可以实现分组，同name同组。可用checked属性实现默认选择，checked属性值为true或false，也可用“checked”代表true

​            4、复选框：type=“checkbox” ，可用checked属性实现默认选择，“checked”表示选中，在标签里面设置时，不能用true或false，因为任何值都解读为选中，但是通过dom对象可以用true或false设置会否选中。checked未定义时默认为false

​            5、重置按钮：type=“reset”，将输入框重置为默认值，配合value属性修改按钮上面的文字

​            6、提交按钮：type=“submit”，将信息提交，配合value属性修改按钮上面的文字

​            7、按钮：type=“button” ，纯按钮，配合value属性修改按钮上面的文字

​            8、文件选择按钮：type=“file”，用于文件选择

​            9、隐藏域：type=“hidden”，隐藏，可设置name、value等，但不显示。用于存放不需要用户看见的信息，提交时会发送

* * select：选择标签，形成一个下拉选择框（name：名称），可以通过multiple属性决定是否多选， 值为true或者“multiple”时是多选，否则是单选。**select的value就是选中的option的value，没选时默认第一个option选中。如果select下面没有option，其value=null**
  * * option：下拉列表框中的选项标签。配合selected属性可实现默认选择
* * textarea：多行文本输入框标签 ，默认显示标签里面的内容    （rows：行数   cols：列数，即字符数  name：名称） 这里行数只是显示行数，输入更多行是没有问题的
* * button：按钮标签，与<input type="button">类似

**注意：**

  0、form表单里面，任意按钮（包括< button>、< input  type="submit">、< input  type="button">）的点击都会默认触发form提交，除非在自定义的响应函数里面返回false才能阻止

  1、为了更加整齐的显示表单里面的数据，可以利用table标签，把输入放到一行一行上面

  2、不同标签其属性对应情况不尽相同，例如 <input type="text">的value对应文本框内容，而<input type="checkbox">的value则不对应页面内容，是它自己本身的属性

 3、表单项有些属性是逻辑上固有的，比如单选复选的value、checked，即使未定义也有默认值（on、false），访问时会返回这个默认值。当然通过jQ对象的attr（）方法可以得到undefined值，但官方已经说明这是错误的，不符合这种逻辑固有属性，因此在某些属性上推荐使用prop（）方法替换attr（）方法

 4、只读属性可以通过readonly或者readOnly设置，但获取的时候只能通过readOnly获取，否则返回undefined

**提交给服务器时的三种情况：**

​	0、所有的参数都将以（name，value）的键值对形式提交，因此，只有设置name的表单项会提交，不设置name的表单项是不会提交的，get提交的形式是：**路径 ?name=value&name=value&name=value** 的形式

​    1、没有设置name属性的表单项不提交 / 设置了modelAttribute属性的form表单里面，没有设置path属性的表单项不提交

​    2、form之外的表单项不提交

​    3、单选、复选、下拉的option标签，都需要添加value属性，否则发送其默认值。单选、复选的默认值为on，这可以通过获取它的dom对象的value属性值查看。option的默认值则是标签内容。

​    4、下拉选项select的值就是选中的option的value，所以只需要在select标签上设置name属性即可以提交



**两种提交模式特点**

​    一、get：

​         1、浏览器中地址栏是：action值+？+请求参数。其中请求参数是 name=value&name=value&...&name=value的格式

​         2、不安全。地址栏中参数的value可见。

​         3、非acsii字符或者超过100个字符，则必须使用post（见HTML参考手册）

   二、post：

​         1、浏览器中地址栏是：action值

​         2、安全

​         3、理论上没有字符长度限制

### 内容类标签

* div：默认独占一行，前后的内容不能进入该行。且里面每个对象都会独占一行或多行显示
* span：继续在该行显示
* p：段落标签。默认在上方或下方空出一行，如果已有空行就不再空了。

这些标签主要是用来显示内容的，内容可以是文字、或者标签（如图片标签）

### css样式标签

* style：css样式标签，专用于定义css样式，里面存放css代码 。为了可读性，该标签一般用在head里面 （type：类型）

### 字体标签（font）常用属性

* color：颜色
* face：字体（例如 宋体、黑体等）
* size：字体大小，值为1~7，其中1最小，7最大

### 标题标签

即 h 标签，有1到6级，1级最大，6级最小

```html
    <h1 align="left">标题1</h1>
	<h2 align="center">标题2</h2>
	<h3 align="right">标题3</h3>
	<h4>标题4</h4>
	<h5>标题5</h5>
	<h6>标题6</h6>
```

### base标签

​		当我们点击页面的a标签进行跳转时，如果使用到相对路径，那么浏览器将参考地址栏的地址进行跳转，如果地址栏地址确实就是资源路径，那么没什么问题。但如果地址栏地址不是资源路径（如在请求转发的情况下地址栏的就是直接请求的地址，而非请求转发后资源的地址），那么这种跳转往往跳到错误的地方去

​		为了解决这个问题，我们需要一个固定的参考系，也就是说，无论浏览器地址栏的路径到底是什么，点击a标签跳转时，将固定参考某个路径（可以设置为资源自身的路径），这就是base标签的作用

​		**简而言之，base标签就是a标签跳转的参考系，如果存在base标签，那么a标签通过相对地址的跳转将以base标签的地址为参考系，而不是以浏览器地址栏路径为参考系**

​		以base标签为参考系能够使得按相对地址的跳转不受浏览器地址栏地址的影响，这样可以避免当资源路径与浏览器地址栏的路径不同时（例如请求转发送得到的资源，其路径与地址栏路径不同）发生的跳转错误

​		注意：

* base标签最多具体到文件夹，否则就是出错，因此，base标签href属性值最后必须是 **/** 这个符号 ，否则认为最后是个文件  
* 使用base标签后，按照相对路径或者绝对路径的方式跳转即可，注意相对路径不能以斜杠/ 开头，因为页面相对路径只能以 点（.）或者点点（..）或者路径名（xxx）或者文件名（xxx.yyy）开头，servlet里的路径才能以斜杠开头，表示http://ip:port/工程名/

### 特殊属性

* disabled：表现是否可用，值为true（或“disable”）时不可用，否则可用。不可用标签仅仅意味着该标签在页面上不能产生任何交互，包括输入、选择、单选、复选、点击等，但通过dom对象依然能对它操作、赋值等。最主要的是，设置了disabled属性的表单项是不会被提交的

  



## CSS代码

所谓的css，主要是用于对html标签的样式作**分类应用**。这是通过选择器来声明样式所应用的标签，有标签名选择器、id选择器、class选择器、组合选择器等。可以理解为 *选择器主动选择相应标签来引用样式*

### 语法规则

```css
/* 语法结构如下
选择器{
    键值对1;     
    键值对2;
    ...
    键值对n;
}

这里的选择器是用来声明样式所应用的标签，它说明了中括号内的样式是应用在这种标签上的。
样式是通过一组键值对来确定的，所谓键值对就是（属性：值）的形式，另外复合属性则是（属性：值1 值2 ... 值n）的形式
多个键值对需要用冒号隔开
*/

div{
    border: 1px solid red;
}
span{
    border: 1px solid red;
}
p{
    border: 1px solid red;
}
```

### css与html结合的3种方式

* 方式一：直接把键值对放到标签的style属性里面，可以看做是样式的直接注入

  ```html
   <div style="border: 1px solid red; width: 300px; height: 50px">div 标签 2</div>
  ```

* 方式二：通过style标签，应用css代码

  ```html
      <style type="text/css">
          /*需求 1：分别定义两个 div、span 标签，分别修改每个 div 标签的样式为：边框 1 个像素，实线，红色。*/
          div{
              border: 1px solid red;
              width: 300px;
              height: 50px;
          }
          span{
              border: 1px solid red;
          }
          p{
              border: 1px solid red;
          }
      </style>
  ```

* 方式三：css单独写成一个文件，在html里面通过 link 标签引入即可

   html的link标签

  ```html
      <!--  rel：用来声明所链接文件与本文件的关系   type：指明所链接文件的类型  href：链接文件的路径  -->
      <link rel="stylesheet" type="text/css" href="样式1.css">
  ```

  css的样式代码

  ```css
  div{
      border: 1px solid red;
      width: 300px;
      height: 50px
  }
  span{
      border: 1px solid red;
  }
  p{
      border: 1px solid red;
  }
  ```

总结：

方式1需要在每个标签的stype属性上面写样式，是样式的直接注入

方式2则通过style标签里面的css代码作用于相应类型的标签，简化了单个html文件配置

方式3则是通过link标签链接外部css代码作用于相应类型的标签，简化了多个html文件配置

### css选择器的4种类型

#### 1、基本选择器

* 标签（TagName）选择器：即选择器是标签类型名，按标签类型选择。注意这不是标签的name属性值，而是标签类型名字

  ```css
  /* 语法结构如下
  标签类型名{
      键值对1;     
      键值对2;
      ...
      键值对n;
  }
  */
  div{
      border: 1px solid red;
      width: 300px;
      height: 50px
  }
  span{
      border: 1px solid red;
  }
  p{
      border: 1px solid red;
  }
  ```



* id选择器：即选择器是 **#id属性值** 的形式，按照id属性值选择

  ```css
  /* 语法结构如下
  #id属性值{
      键值对1;     
      键值对2;
      ...
      键值对n;
  }
  */
  #id001{
      color: blue;
      font-size: 30px;
      border: 1px solid yellow;
  }
  #id002{
      color: red;
      font-size: 20px;
      border: 5px dotted blue;
  }
  ```



* class选择器：即选择器是 **.class属性值** 的形式，按照class属性值选择

  ```css
  /* 语法结构如下
  .class属性值{
      键值对1;     
      键值对2;
      ...
      键值对n;
  }
  */
  .class01{
      color: blue;
      font-size: 30px;
      border: 1px solid yellow;
  }
  .class02{
      color: #bfbfbb;
      font-size: 26px;
      border: 1px solid red;
  }
  ```



#### 2、层级选择器

所谓层级选择器，往往由一个符号与基本选择器构成，需要按与逻辑接在前面的结果上（不只是前面一个而是前面所有选择器的最终选择结果），形成一个层次选择


* 有后代选择器、子元素选择器、相邻元素选择器、之后兄弟元素选择器。具体可以参考jQuery的层级选择器

  ```css
  //后代选择器
  .showmore a span {
      padding-left: 15px;
      background: url(img/down.gif) no-repeat 0 0;
  }
  ```



#### 3、复合选择器

所谓复合选择器，就是由多个选择器结合在一起的，这些选择器可以是**基本选择器、层级选择器**


* 所谓复合选择器就是多个选择器排在一起，从左到右按与逻辑或者是或逻辑结合，即选择器可以是类似 **选择器1选择器2，选择器3，...，选择器n** 的形式（选择器2按与逻辑结合，选择器3按或逻辑结合），其中 **选择器i** 的形式可以是 标签选择器、id选择器、class选择器中的任意一种

  ```css
  /* 语法结构如下
  选择器1...选择器n{
      键值对1;     
      键值对2;
      ...
      键值对n;
  }
  */
  //全是或逻辑
  .class01,#id01,div{
      color: blue;
      font-size: 20px;
      border: 1px solid yellow;
  }
  //全是与逻辑
  div.class01#id01{
      color: blue;
      font-size: 20px;
      border: 1px solid yellow;
  }
  //既有与逻辑，也有或逻辑
  div.class01,#id01{
      color: blue;
      font-size: 20px;
      border: 1px solid yellow;
  }
  
  ```

*这与 JS （注意不是 JQ）通过  TagName、id、name 访问不同*

### css样式的即时性与使用（class属性）

* 当标签发生变化时，css选择器的匹配也会发生变化，从而导致标签样式发生改变，这个改变是可以马上显示在页面上的，称之为样式的即时性。

* 通常，由于标签的class属性可以有多个值（不同的值用空白符隔开），因此常常利用class选择器，这样通过dom对象的class属性值就可以应用多种样式，通过修改class属性值就可以修改标签的样式了

### css选择器优先级

id选择器 > class选择器 > 标签选择器

当发生选择器的样式冲突的时候，优先级高的选择器会覆盖掉优先级低的选择器

### 常用样式

* 字体：
* * 颜色：color
  * 大小：font-size
  * 文本对齐：text-align
* 标签：
* * 宽度：width
  * 高度：height
  * 左空白填充：margin-left
  * 右空白填充：margin-right
  * 居中：margin-left：auto    margin-right：auto  //对a标签没用，对dev、p标签有用
  * 背景颜色：background-color
* 边框：
* * 边框：border
  * 边框合并：border-collapse： collapse   //用于table标签将会合并边框
* 超链接
* * 下划线：text-decoration
* 列表修饰符：
* * list-style: none   //用于去除列表前面的修饰符
* 表格边框：
* * 表格边框合并：border-collapse： collapse   //用于table标签将会合并边框



### 属性自适应

  元素的宽高在没有设定的情况下是会随着其子元素的大小而变化的，当然如果设定好了，那无论什么情况下都按照设定值显示

子元素是会显示在父元素的上面一层的，所以如果设定死了，有时会出现一些不符合预期的覆盖现象

### 元素在页面定位

元素在页面的定位是依据其左上角的左边确定的，如下图：

<img src="http://typora-imges.oss-cn-beijing.aliyuncs.com/img/元素的页面位置.jpg" alt="元素的页面位置" style="zoom: 67%;" />

可见，元素的top值就是y坐标值，left值就是x坐标值

## JavaScript

JavaScript是运行在客户端浏览器上面的代码，在html文件加载的时候就会运行

### JS与html结合的2种方式

* 方式一：在script标签里面，写 JavaScript 代码

  ```html
      <script type="text/javascript">
          alert("hello JavaScript");
      </script>
  ```

* 方式二：JavaScript单独写成一个文件，在html里面通过 script 标签引入即可

  html的 script 标签

  ```html
      <script type="text/javascript" src="1.js">
      </script>
  ```

  JavaScript代码

  ```javascript
  alert("hello JavaScript");
  ```

* 方式三：在html的标签属性里面使用 JavaScript 代码，典型的就是标签的事件属性



注意：

​       html里面一个 script 标签只能使用方式一或者方式二来执行JavaScript代码，如果同时使用两种，则只会执行方式二

​       如果需要执行多份JavaScript代码，只需要用多个 script 标签即可



### JS的数据

​    JavaScript是一门弱类型语言，意味着其变量是没有类型的，可以存放不同类型的数据。但是数据是有类型的，且在实现层面数据本身会包含其类型信息。JavaScript语言提供一个 **typeof（）** 函数来获取数据的类型信息



**JS的数据类型：**

* number：数值类型   // Java会细分为 short、int、long、float、double，但JavaScript统一为number
* string：字符串类型
* boolean：布尔类型
* object：对象类型
* function：函数类型
* undefined：默认值undefined的类型



**JS的特殊数据：**

* undefined：未定义。所有JavaSript变量没有赋值时，默认值为undefined。且其类型为undefined

* null：空值。属于 object 类型，指一个空对象

* NAN：全称是 not a number，指不是一个数字。属于 number 类型。但我们不能直接使用，只能是在做不允许的运算时由系统产生



**JS数据类型转换：**

* string <---> number ：
* * 1. 调用Number()来对string进行值类型转换 ，如 Number（"123"）
  * 2. parseInt() ，如 parseInt（"123"）
  * 3. parseFloat() ，如 parseFloat（"123"）
  * 4. 对字符串变量采用++运算，将会自动转换为数字类型计算并把数字结果保存在变量里。注意：++运算不能是字符串本身，必须为字符串变量，否则报错
  * 5. 两个string类型通过 "-" 号运算时，将转换成number数据做减法运算
  * 6. 两个string类型通过 "+" 号运算时，不会转换成number数据运算，而是做字符串连接操作
  * 7. 一个string一个number通过"+"号运算时，number先转换成string，再做字符串连接操作
  * 8. 两个string比较时，将按照字典序比较，如"12">"2"是 false
  * 9. 一个string一个number比较时，string将转换成number再比较，如"12">2是 true



**Js字符串：**

* 子串的替换：https://www.w3school.com.cn/jsref/jsref_replace.asp

  

**JS数据打印**

* 打印非对象数据时，直接打印内容
* 打印对象数据（包括普通对象，dom对象）时，打印的是对象的具体类型信息，当然a标签对象比较特殊，打印的是跳转地址



### JS增强for循环

```javascript
    //这里的增强for循环，并不是直接取数组元素，而是取其下标（从小到大）
    var a="a,b,c,";
    var strs=a.split(",");
    for(var k in strs){
        alert("一个元素："+strs[k]);
    }
```



### JS的关系运算

* **常见有：>、>=、<、<=、==、!=  **      

 // 这些关系运算符号形式与Java是相同的，但意义不同，这里是基于数据字面值比较，例如 ”12“==12 是true。也就是当数据类型不同的数据比较时，浏览器会统一转换为numbe类型数据再作比较。如 ”13“转为13，true转为1

* **特殊：===**       

 //这是一个全等于符号，是更为严格的相同判断，与Java中的==意义相同，只有 数据类型与值 都相同时才会是true

### JS的逻辑运算

* **运算符：&&（与）、||（或）、！（非）**

注意：逻辑运算是基于布尔类型值进行的，而 JavaScript 所有类型的值都能作为布尔类型。

可以当做 false 的值有：

​         number类型：0，NAN
​         string类型：""     //这是一个空串
​         boolean类型：false
​         object类型：null
​         undefined类型：undefined

* **运算规律**
* * && ：结果为true时，返回最后一个  `true数据 `  ；结果为false时，返回第一个  `false数据`
  * || ：结果为true时，返回第一个  `true数据`  ；结果为false时，返回最后一个  `false数据`
  * ！ ：结果为true时，返回true，结果为false时，返回false

注意：&& 与 || 的返回结果是   `true数据`   或  `false数据`  ，而不是true或false。只有！才是true或false。例如 12&&"abc" 返回"abc" ，false||null  返回null， !"abc" 返回 false

### JS的数组

JavaScript的数组类似于Java的数组，变量可以看做是一个引用变量，存放指向数组的引用。在作为参数传递给函数时，如果在函数内部改变数组，那么这种变化对于外部的引用是可见的。数组属于JS里面的对象

**JS数组格式**：

​      var  数组名=[] ; //定义一个空数组

​       var  数组名=[1，“abc”，true] ;   //定义数组，初始化其值

**JS数组可以动态扩增长度，例如：**

```javascript
 //写操作可以扩容数组  
var ar=[];
alert(ar.length);//0，空数组
ar[0]=1;
ar[2]="abc";
alert(ar.length);//3，即数组已经根据赋值的最大下标值扩容了，下标为0、1、2
alert(ar[1]);//undefined，未定义数据，这是由于a[1]我们没有赋值，系统赋予默认值了

//读操作不能扩容数组
var ar=[];
ar[0]=1;
ar[2]="abc";
alert(ar.length); //3，即数组已经根据赋值的最大下标值扩容了，下标为0、1、2
alert(ar[9]);   //undefined,未定义数据
alert(ar.length);  //3，读操作不能扩容
```

**JS数组的遍历：**

```javascript
//方式1：for循环
var ar=[];
ar[0]=1;
ar[3]="abc";
for(var i=0;i<ar.length;i++){
    alert(ar[i]);
}
//方式2：alert整个数组,仅适用alert
alert(ar);
```

**JS数组的赋值：**

```javascript
//通过下标赋值
var ar=[];
ar[0]=12;  
alert(ar.length); //1

//通过字符串赋值（实际上字符串会被按照字符拆分出一个字符串数组），a存放的是["a"，"b"，"c"]
ar="abc";  
alert(ar.length); //3

//在末尾添加 push（）方法
ar.push(1);
ar.push(2);

//非法赋值!!!
ar=10;  
```

### JS的函数

#### 函数定义

* **第一种定义方式： function 函数名 （形参列表）{ 函数体 }**

  ```javascript
  //无参函数
  function fun1() {
      alert("这是一个无参函数");
  }
  
  fun1();//这是一个无参函数
  
  //有参函数，注意参数不用加类型，否则报错
  function fun2(a,b) {
      alert("这是一个有参函数，a: "+a+" ,b: "+b);
  }
  
  fun2("apple",123);  //这是一个有参函数，a: apple ,b: 123
  
  //带返回值函数
  function sum(a,b) {
      var ans=a+b;
      return ans;
  }
  
  alert(sum(3,4));//7，函数返回的计算结果
  ```

* **第二种定义方式：var 函数名=function（形参列表）{ 函数体 }**

  ```javascript
  //无参函数
  var fun1=function () {
      alert("这是一个无参函数");
  }
  
  fun1();//这是一个无参函数
  
  //有参函数，注意参数不用加类型，否则报错
  var fun2=function (a,b) {
      alert("这是一个有参函数，a: "+a+" ,b: "+b);
  }
  
  fun2("apple",123);  //这是一个有参函数，a: apple ,b: 123
  
  //带返回值函数
  var sum=function (a,b) {
      var ans=a+b;
      return ans;
  }
  alert(sum(3,4));//7，函数返回的计算结果
  ```

总结：方式二只是把函数名当成一个变量而已，使用的时候与方式一完全一样

**注意：在JS当中，function  （形参列表）{ 函数体 } 通常作为一个代码块而非函数使用，因为它没有函数名。而代码块与一段代码的效果完全相同**

#### 函数不能重载

JS不允许函数重载，后面的重名函数会覆盖前面的函数

```javascript
//函数不允许重载，后面会覆盖前面
function fun() {
    alert("这是一个无参函数");
}
function fun(a,b) {
    alert("这是一个有参函数，a: "+a+", b: "+b);
}
fun(1,2); //这是一个有参函数，a: 1, b: 2
fun();  //这是一个有参函数，a: undefined, b: undefined
```

#### 函数形参实参匹配问题

* 当传递的实参数量不足时，会自动加上undefined数据不全

  ```javascript
  function fun(a,b) {
      alert("这是一个有参函数，a: "+a+", b: "+b);
  }
  fun(1,2); //这是一个有参函数，a: 1, b: 2
  fun(1);  //这是一个有参函数，a: 1, b: undefined
  ```

* 当传递的实参数量过多时，会都传过去，但形参只会接受前面的实参，多余的不管

  ```javascript
  function fun(a,b) {
      alert("这是一个有参函数，a: "+a+", b: "+b);
  }
  fun(1,2,3); //这是一个有参函数，a: 1, b: 2
  ```

* 隐形形参**arguments**会接受传递过来的所有参数，可以把**arguments**当做一个参数数组处理（除了整体alert操作外）

  ```javascript
  function fun(a,b) {
      alert(arguments.length);
      alert(arguments[0]);
      alert(arguments[1]);
      alert(arguments[2]);
      alert("这是一个有参函数，a: "+a+", b: "+b);
  }
  fun(1,2,3);// 3; 1; 2; 3; 这是一个有参函数，a: 1, b: 2
  ```

* 隐形形参arguments的存在使得函数可以实现不定参数的功能

  ```javascript
  //sum函数可以计算不定数量的参数和，利用了arguments
  function sum() {
      var ans=0;
      for(var i=0;i<arguments.length;i++){
          ans+=arguments[i];
      }
      return ans;
  }
  alert(sum(1,2)); //3
  alert(sum(1,2,3)); //6
  
  //为了可读性，可以加入两个形参,不影响arguments
  function sum(a,b) {
      var ans=0;
      for(var i=0;i<arguments.length;i++){
          ans+=arguments[i];
      }
      return ans;
  }
  alert(sum(1,2)); //3
  alert(sum(1,2,3)); //6
  ```

总结：JS函数调用时的参数遵循以下规则：

   1、实参不够，undefined补全。

   2、实参过多，不做处理

   3、显式形参取**前面的实参**

   4、隐形形参arguments取**所有实参**，可把arguments当做数组处理（除整体alert操作外）

隐形形参arguments的存在使得函数可以实现不定参数的功能。

#### 函数作用域

JS函数的作用域是整个文件的所有 script 标签，如一个script 标签里面的代码可以调用文件中任意一个script 标签里面的函数

#### 常用的api函数

* alert（参数）：在页面上弹出个提示框，内容就是里面的参数
* confirm（参数）：在页面上 弹出个确认框，内容就是里面的参数，点击确认时返回true，点击取消时返回false
* console.log（参数）：在浏览器的控制台上打印参数
* submit（）：提交表单。**表单内的元素使用submit（）时优先使用这个api，而非自己定义的同名函数。表单外的元素则使用自己定义的同名函数**

注意：

​     当参数为对象时，得到的是对象的类型信息（a标签对象比较特殊，得到的是跳转地址）

​     当参数为非对象时，得到的是数据



### JS的对象

​		js的对象大都是通用对象，即不属于某个类，只是包含属性与方法；当然也有专用对象，这些对象属于某个类；所谓的专用对象出生便具有类所描绘的属性与方法。但其属性、方法的访问与添加与通用对象无异

* **对象结构：属性+方法    //与Java是一样的**

* **对象创建方式一：通过 new  Object（）的形式**

  ```javascript
  /*
      对象定义
         var 对象名= new Object();              //定义一个空对象
           对象名.属性名=值;                          //定义一个属性
           对象名.方法名=function(参数列表){ 函数体 }     //定义一个方法
           
         var 对象名= new Object(value); //返回一个value的包装类对象，例如string则返回String
  
       对象的访问
           对象名.属性/方法
  */
  
      var obj=new Object();  //定义空对象
      obj.name="张三丰";     //定义属性
      obj.age=88;            //定义属性
      obj.getInfo=function () {   //定义方法
          alert("name:"+this.name+", age:"+this.age);  //即使在内部访问属性也要
      }
      obj.getInfo();  //name:张三丰, age:88
  ```
  
* **对象创建方式二：通过花括号的形式，这里花括号表示对象而非代码块，因此后面需要加冒号；**

  ```javascript
  /*
        对象定义
            var 对象名= {};              //定义一个空对象
            对象名.属性名=值;                          //定义一个属性
            对象名.方法名=function(参数列表){ 函数体 }     //定义一个方法
  
            var 对象名= {           //定义一个非空对象
                属性名：值，
                属性名：值，
                函数名：function( 参数列表 ){ 函数体 }
            };
  
         对象的访问
             对象名.属性/方法
  */
  
    var obj={
        name: "张三丰",
        age:88,
        getInfo:function () {
            alert("name:"+this.name+", age:"+this.age); //即使内部访问属性，也要用this
        }
    };
    obj.getInfo(); //name:张三丰, age:88
  ```

* **对象创建方式三：通过类**

  ```javascript
    var patt=new RegExp("e");//创建一个正则表达式对象，"e"是传入的正则表达式
  ```

  

总结：

​     1、new object（）：创建空对象

​     2、new object（value）：创建value的包装类对象

​     3、{}：可以创建空对象，也可以创建非空对象

​     4、对象可以动态添加成员（属性或方法）

​     5、当然还可以利用类来创建对象，这与Java一样，例如创建正则表达式对象

### JS的事件

所谓事件，就是电脑设备与页面的交互响应。广义的说就是：发生了事件--->做出响应

要实现**“发生了事件--->做出响应”**，就需要监听器与响应函数。监听器是 JS 提供给我们的，我们只需要写响应函数即可。通常说的事件既可以是发生的事件也可以是事件监听器，应视具体语境而定。

即事件可以在标签中按类似属性赋值的方式注册，但事件不是属性，dom对象操作事件的方式只有一种，就是dom对象.事件名，形式上类似属性

注意，当对象绑定了事件时，发生事件后会先执行用户的响应函数再执行默认动作（如果存在的话），而如果响应函数返回false，则可以阻止dom对象继续执行默认行为。例如表单的提交、a标签的跳转这些默认动作可以通过在onsubmit、onclick事件的响应函数中返回false狙击掉

  **常见的事件：**

​          onload：加载完成事件，即页面加载完成

​          onclick：单击事件

​          onblur：失去焦点事件

​          onchange：内容发生改变事件

​          onsubmit：表单提交事件   //这个事件只会在form标签出现，不会在< input type="submit" >标签出现

这里所指的事件就是发生的事情

注意，在事件的响应函数中，有一个隐含的this参数，它代表响应的dom对象本身，即dom对象调用响应函数时，会把自己的引用以及事件对象传递过去，this参数是隐含的，事件对象则需要设置形参接受或者直接通过另一个隐含参数arguments获取。（所谓隐含参数是指不用显示写明，但编译器会帮我们自动添加上去的参数）

#### 事件注册

所谓事件注册，实质上就是把事件监听器注册给对象并设置响应代码

* **静态注册：**通过 html 标签的事件属性直接赋于事件响应后的  `JS代码或者JS函数`  ，这种方式我们叫静态注册。这里我们可以把这些属性看做是一个个事件监听器， 以onload事件为例

  ```html
  <!--onload属性里直接写js代码-->
  <body onload="alert('静态注册onload事件')">
  
  </body>
  
  <!--onload属性里调用JS函数-->
  <head>
      <meta charset="UTF-8">
      <title>onload</title>
      <script type="text/javascript">
          function fun() {
              alert("这是onload事件的静态注册");
          }
      </script>
  </head>
  <body onload="fun()">
  
  </body>
  ```

* **动态注册：**是指先通过 js 代码得到标签的 dom 对象（下一节讲到），然后再通过 dom 对象.事件 = function(){}  这种形式赋于事件响应后的代码，叫动态注册

  ```html
  <head>
     <script type="text/javascript">
         /*前置知识
          1、window对象是全局的，代表一个窗口，浏览器会为每个html文件创建window对象，也会为页面中每个frame框架或者iframe框架创造额外windows对象。且window对象在每个页面执行前产生。因此可以在JavaScript代码中给它注册事件
          2、而页面中的标签对象需要待页面加载它之后才能存在，因此，如果要给页面标签注册事件。要么把注册代码放到该标签后面，要么把注册代码放在window的onload事件里
          */
  //<!-- ------------------------动态注册1(直接注册响应代码)----------------------- -->    
        //这个是给window对象注册onload事件，并把其他对象的事件注册代码放到里面，这样页面加载完毕后就会执行并注册其他标签对象
          window.onload=function () {
              var btnObj=document.getElementById("btn02");
              btnObj.onclick=function () {
                  alert("动态注册onclick事件");
              }
          }
      </script>
  </head>
  <body>
     <button onclick="onclickFun()">按钮1</button>  <!--静态注册-->
     <button id="btn02">按钮2</button>   <!--动态注册-->
  </body>
  
  <!-- --------------------------动态注册2(直接注册响应代码)--------------------- --> 
  <body>
     <button onclick="onclickFun()">按钮1</button>  <!--静态注册-->
     <button id="btn02">按钮2</button>   <!--动态注册-->
  
     <script>
         var btnObj=document.getElementById("btn02");
         btnObj.onclick=function () {
             alert("动态注册onclick事件");
         }
     </script>
  
  </body>
  
  <!-- -------------------动态注册3(注册响应代码，代码里面调用别的函数)------------- --> 
  <head>
    <script>
         funtion funname(){
             
         }
         var btnObj=document.getElementById("btn02");
         btnObj.onclick=funname;//注意这里不能是funname（），因为这表示执行函数，而非函数本身
     </script>
  </head> 
  <body>
     <button onclick="onclickFun()">按钮1</button>  <!--静态注册-->
     <button id="btn02">按钮2</button>   <!--动态注册-->
  </body>
  
  <!-- --------------------------错误的动态注册（页面还没加载完成）------------------ -->
  <head>
     <script>
         //后面还没执行，不存在标签对象，因此注册会失败
         var btnObj=document.getElementById("btn02");
         btnObj.onclick=function () {
             alert("动态注册onclick事件");
         }
     </script>
  </head>  
  <body>
     <button onclick="onclickFun()">按钮1</button>  <!--静态注册-->
     <button id="btn02">按钮2</button>   <!--动态注册-->
  </body>
  
  
  ```

​      注意：

1、事件的动态注册需要等到页面加载完毕后才能进行，否则因为获取不到标签对象而导致注册不成功，当然window对象除外。因此动态注册代码一般放在window的onload事件里面，即页面加载完毕后再进行动态注册

2、**静态注册我们只需要在字符串里面写响应代码，编译器自动帮我们包装成一个函数作为响应函数注册给事件**

3、动态注册则是需要我们把函数（匿名函数与非匿名函数）作为响应函数来注册：

​     1）通过function：dom对象.事件=function（）{响应代码};

​     2）通过已有的函数：dom对象.事件=funname;   //注意这里不能是funname（），因为这表示执行函数，而非函数本身

**注意：这两种方式都是把函数（匿名函数或者非匿名函数）注册给事件**

4、所以静态注册和动态注册归根结底都是把一个响应函数注册给事件，只不过，静态注册只需要写响应代码，编译器自动包装成函数注册给事件，而动态注册则需要显式地以函数的形式作为响应函数注册给事件

5、**事件的响应函数里面可以用 this，它代表响应事件的dom对象，这不需要显式设置形参。同时可以通过设置一个形参例如event来接受事件对象。实现上可能编译器自动帮我们把this形参加到第一个形参里面。调用响应函数的实参第一个就是响应事件的dom对象，第二个是事件对象**

#### 事件注册时this的意义

```html
//案例1：显示window对象  因为静态注册时字符串里面的是响应代码而非响应函数，里面只是个函数调用语句，而this是在调用函数里面，而非响应函数里面
<scrip>
    function hello(){
    	alert(this);
    }
</scrip>
<body>
    <button onclick="hello()">测试</button>
</body>


//案例2：显示button对象  因为静态注册时字符串里面的是响应代码，this是在包装后响应函数里面，代表的是响应对象本身
<body>
    <button onclick="alert(this);">测试</button>
</body>

//案例3：显示button对象  因为动态注册直接把函数作为响应函数，里面的this就是响应函数里面的this
<scrip>
    window.onload=function(){
    	var btn=document.getElementById("abc");
    	btn.onclick=function(){
    		alert(this);
    	}
    }
</scrip>
<body>
    <button id="abc">测试</button>
</body>


//案例4：显示button对象  因为动态注册直接把函数作为响应函数，里面的this就是响应函数里面的this
<scrip>
    function hello(){
    	alert(this);
    }
    window.onload=function(){
    	var btn=document.getElementById("abc");
    	btn.onclick=hello;//这里不能加括号，否则表示一个表达式（函数调用），而非函数本身
    }
    
</scrip>

<body>
    <button id="abc">测试</button>
</body>


```

总结：this代表的意义取决于它是在普通函数里面（window），还是在响应函数里面（响应对象）。特别注意**案例1**与**案例4**，形式上貌似都是把hello（）函数注册给事件，但结果却不一样，区别在于**案例1**中的 “hello（）”只是响应函数里面的调用语句，而**案例4**里面，则hello（）函数本身就作为一个响应函数了

响应函数可以使用this，是因为编译器帮我们在响应函数的形参上，加上一个this在首位。另外，我们还可以添加第二位形参even来接收事件对象



#### onload事件

具体注册方法见上面的**事件注册**的例子

#### onclick事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>onclick</title>
    <script type="text/javascript">
        function onclickFun() {
            alert("静态注册onclick事件")
        }

        window.onload=function () {
            var btnObj=document.getElementById("btn02");
            btnObj.onclick=function () {
                alert("动态注册onclick事件");
            }
        }
    </script>
</head>
<body>
   <button onclick="onclickFun()">按钮1</button>  <!--静态注册-->
   <button id="btn02">按钮2</button>   <!--动态注册-->
</body>
</html>
```

#### onblur事件（失去焦点）

```html
<head>
    <meta charset="UTF-8">
    <title>onblur</title>
    <script type="text/javascript">
        function onblurFun() {
            console.log("静态注册失去焦点事件");//console对象，将会在浏览器控制台上打印信息。可以                                                 通过F12查看
        }
    </script>
</head>
<body>
    用户名：<input type="text" onblur="onblurFun()"> <br>
    密码：<input type="text">
</body>
```

#### onchange事件

```html
<head>
    <meta charset="UTF-8">
    <title>onchange</title>
    <script type="text/javascript">
        function onchangeFun() {
            alert(" 女神已经改变了 ");
        }
        window.onload=function () {
            var manObj=document.getElementById("idol");
            manObj.onchange=function () {
                alert(" 男神已经改变了 ");
            }
        }
    </script>
</head>
<body>
    请选择你心中的女神：
     <select onchange="onchangeFun()">
         <option>--女神--</option>
         <option>芳芳</option>
         <option>佳佳</option>
         <option>娘娘</option>
     </select>
    请选择你心中的男神：
    <select id="idol">
        <option>--男神--</option>
        <option>国哥</option>
        <option>华仔</option>
        <option>富城</option>
    </select>
</body>
```

#### onsubmit事件

*提交事件只有注册在form标签上才会有效，注册在input标签无效，虽然提交按钮是个input标签，但是最后的提交行为是form表单执行的，即input标签只是触发了form标签的提交事件而已，因此需要把onsubmit注册在form表单上*

```html
<head>
    <meta charset="UTF-8">
    <title>onsubmit</title>
    <script type="text/javascript">
        function submitFun() {
            alert("静态注册submit事件");
        }
    </script>
</head>
<body>
   <form action="https://www.baidu.com/?tn=sitehao123_15" method="get" onsubmit="submitFun()" >
       <input type="submit" >
   </form>
</body>
```

另外 onsubmit事件的响应函数如果返回false，是可以阻止表单提交的

```html
<!-------------------阻止表单提交1------------------>
<body>
   <form action="https://localhost:8080" method="get" onsubmit="return false" >
       <input type="submit" >
   </form>
</body>
<!-------------------阻止表单提交2------------------>
<head>
    <meta charset="UTF-8">
    <title>onsubmit</title>
    <script type="text/javascript">
        function submitFun() {
            alert("静态注册submit事件");
            return false;
        }
    </script>
</head>
<body>
   <form action="https://localhost:8080" method="get" onsubmit="return submitFun()" >
       <input type="submit" >
   </form>
</body>
<!-------------------阻止表单提交错误实例------------------>
<head>
    <meta charset="UTF-8">
    <title>onsubmit</title>
    <script type="text/javascript">
        function submitFun() {
            alert("静态注册submit事件");
            return false;
        }
    </script>
</head>
<body>
    <!--submitFun()函数虽然返回false，但它只是响应函数里面的调用，而相应函数本身却没有返回false。
         改为return submitFun()就可以了-->
   <form action="https://localhost:8080" method="get" onsubmit="submitFun()" >    
       <input type="submit" >
   </form>
</body>
<!-----------------------阻止表单提交3---------------------->
<head>
    <meta charset="UTF-8">
    <title>onsubmit</title>
    <script type="text/javascript">
        window.onload=function () {
            var obj=document.getElementById("form01");
            //动态注册是把function里面的响应代码赋予事件，而非打包成函数再赋予
            //因此里面的return是直接向表单返回数据的
            //而静态注册的函数如果要向表单返回数据，是需要在 retrun fun（)而非仅仅fun（）
            obj.onsubmit=function () {
                alert("动态注册submit事件");
                return false;
            }
        }
    </script>
</head>
<body>
   <form action="https://localhost:8080" method="get" id="form01" >
       <input type="submit" value="动态注册">  
   </form>
</body>
```

### dom模型（标签对象）

全称是 Document Object Model 即文档对象模型

将标签、属性与文本内容当做对象来管理，所有的标签都用一个dom对象表示

![文本对象](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/文本对象.PNG)

 #### document 对象：

​     **第一点：Document 它管理了所有的 HTML文档内容**

​     **第二点：document它是一种树结构的文档。有层级关系**

​     **第三点：它让我们把所有的标签都 对象化**

​     **第四点：我们可以通过document访问所有的标签对象**

* document对象有以下4个方法获取标签对象：

​    1、getElementsByTagName   //通过TagName，返回拥有该TagName的标签对象集合，可以按数组方式操作

​    2、getElementById   //通过标签id，返回第一个拥有该id的标签对象

​    3、getElementsByName   //通过name，返回拥有该name的标签对象集合，可以按数组方式操作

​    4、getElementByClassName //通过class属性值，返回拥有该class属性值的标签对象集合，可以按数组方式操作

*理论上，js代码可以通过遍历按照任意条件筛选dom对象，这与css样式只能通过 TagName、id、class 访问不同*



* document对象还提供1个创建对象的方法：

​    1、createElement("tagName");  //通过标签名创建该类标签的对象。配合innerHTML属性设置后得到一个标签对象



* document对象提供一个body属性，里面就是body标签的对象。这是出于方便考虑

#### dom对象大体结构

![标签对象](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/标签对象.PNG)

#### dom对象常用的属性与方法

**一、方法：**

* getElementsByTagName("xxx")：获取指定标签名的标签对象的集合，搜索范围是当前节点下面的所有结点

  

* appendChild( oChildNode )：添加孩子结点，oChildNode 是要添加的孩子节点。通过document提供的createElement("tagName")方法创建dom对象，设置它的innerHTML属性后，再添加到该节点作为孩子节点



* removeChild（childNode）：删除孩子结点



* remove（）：删除本节点

  

* setAttribute("属性名","属性值")：设置属性



* getAttribute('属性')：获取自定义的属性值。区别于domElem.属性 一般用于获取元素本身内置的属性值

  



注意：普通dom对象集合遍历所有结点来清空时，需要从尾部到头部遍历，否则删除过程中数组长度动态变化变化影响操作最终无法删除完毕

​            但是jQ对象里面的dom对象集合的删除则不需要从尾部到头部，因为删除过程中，数组长度不会发生动态变化（无论是for遍历，还是each遍历都一样）

**二、属性**

* childNodes：属性，获取当前节点的所有子节点。文本（包括空白字符）也是被封装成对象的，故孩子结点不止标签对象，可能还存在文本对象
* firstChild：属性，获取当前节点的第一个子节点
* lastChild：属性，获取当前节点的最后一个子节点
* parentNode：属性，获取当前节点的父节点
* nextSibling：属性，获取当前节点的下一个兄弟节点
* previousSibling：属性，获取当前节点的上一个兄弟节点
* className：**用于获取或设置标签的 class 属性值，注意class属性比较特殊，这里不能通过 dom对象.class访问，只能通过dom对象.className访问**
* innerHTML：属性，表示获取/设置起始标签和结束标签中的内容。包含了所有的文本与标签。如果设置innerHTML属性，那么新的文本内容将会覆盖掉原来两标签间的所有内容（包括文本与标签）。此属性主要用于存放dom对象对应的起源html（对象属性的动态改变不会导致其起源html的改变），当此属性改变时，里面的dom对象也要跟着改变
* innerText：属性，表示获取/设置起始标签和结束标签中的文本（包括换行符与空白字符），注意，innerText包括本身及其后代的全部文本。如果设置innerText属性，那么新的文本内容将会覆盖掉原来两标签间的所有内容（包括文本与标签）
* defaultValue：记录dom对象默认的value值，一般是上一次的value值。常用于修改后恢复原值

*innerHTML属性可读可写（写时必须以字符串形式，即html中标签间的内容被认为是HTML内容，JS中需要把它包装成字符串，赋值时再去掉双引号，按HTML内容解析（可能是标签，可能是文本等）后赋值），value属性（如 input标签的value属性）可读可写，这些可以通过标签对象进行操作*



**注意：程序中动态改变dom对象的属性值是不会反映到对应的html上面的。但改变其html可以通过以下方法：**

​     **1、重新设置其父dom对象的innerHTML**

​     **2、通过JQuery对象的 attr（）设置属性**



#### 标签对象、文本对象

在html页面上，标签，文本（包含空白字符）都以dom对象形式存在，但通常的获取dom对象的api函数都是在标签对象的基础上进行的。要获取文本对象，可以通过对象的孩子属性获取



### 正则表达式对象

正则表达式主要用在两个地方：字符串的方法 与  RegExp 对象，具体可以参考：https://www.runoob.com/js/js-regexp.html

var patt=new RegExp(pattern,modifiers);  或  var patt=/pattern/modifiers; 

- pattern（模式） 描述了表达式的模式 
- modifiers(修饰符) 用于指定全局匹配、区分大小写的匹配和多行匹配 

一般用 var patt=new RegExp(pattern);  或 var patt=/pattern/;  就行

即  /pattern/modifiers 与 new RegExp(pattern,modifiers)等效

RegExp有一个test（“content”）方法，可以检测参数字符串是否含有可以匹配到正则表达式的子串，有则返回true，否则返回false

### 两种常见的验证效果

1、通过span标签的innerHTML属性展示文字：

```javascript
//获取span标签对象
var spanElement=document.getElementById("userNameSpan");
//设置innerHTML内容为文字，innerHTML是可读写的
spanElement.innerHTML="xxx";   
```

2、通过span标签的innerHTML属性展示图片：

```javascript
//获取span标签对象
var spanElement=document.getElementById("userNameSpan");
//设置innerHTML内容为图片标签（图片需要通过标签显示），innerHTML是可读写的
spanElement.innerHTML=" <img src='right.png' width='15' height='15' > ";
```

注意：innerHTML属性可读可写（写时必须以字符串形式，即html中标签间的内容被认为是html内容，JS中需要把它包装成字符串，赋值时再去掉双引号，按html内容解析（可能是标签，可能是文本等）后赋值）

### location对象

javascript语言中提供了一个location地址栏对象，它有一个属性叫 href，可以获取浏览器地址栏中的地址，且href属性可读、可写，当写的时候除了改变href属性值外，还会跳转到该属性值的地址



## JQuery

是 JavaScript 和查询（Query），它就是辅助 JavaScript 开发的 js 类库。

### 使用前提

要使用jQuery类库，需要先引入它的js文件，如

```html
<script type="text/javascript" src="../script/jquery-1.7.2.js"></script>
```

### 事件动态注册

获取jQuery对象（应该是dom对象的一个对应），通过该对象的方法去注册事件。即 jQ对象.方法名（function（）{ }）；  下面代码中用到的 $ 是一个函数，在核心函数节再细说

```javascript
var $btnObj=$("#btnId");  //按照id值获取jQ对象，本质上就是一个dom对象对应的js对象
$btnObj.click(function () {  //注册单击事件
   alert("jQuery 绑定的单击事件");
});
```

与通过dom对象注册事件类似，需要等到页面加载完毕之后再注册，因此上面的代码可以按照以下两种方式放置：

```javascript
//方法一：注册类window的onload事件
    window.onload=function () {
        var $btnObj=$("#btnId");  //按照id值获取jQ对象，本质上就是一个dom对象对应的js对象
        $btnObj.click(function () {   //注册单击事件
            alert("jQuery 绑定的单击事件");
        });
    }
//方法二：放到$(function(){})里面，dom对象加载完毕后，执行function里面的代码
    $(function () {
        var $btnObj=$("#btnId");  //按照id值获取jQ对象，本质上就是一个dom对象对应的js对象
        $btnObj.click(function () {   //注册单击事件
            alert("jQuery 绑定的单击事件");
        });
    });
```



### 核心函数 $

$ 是 jQuery 的核心函数，能完成 jQuery 的很多功能。$()就是调用$这个函数

* 1、传入参数为 [ 函数 ] 时： 

​         表示dom对象准备完成之后，执行函数

​        注意：$(function(){ }) 是 $(docment).ready(function(){ }) 的简写，相当于给document对象注册ready事件。与document.ready=function（）{} 等效。另外，document的ready事件比window的onload事件更快，前者只要dom对象加载完毕之后就会响应，后者还需要等页面加载完后才响应。

​       $(function(){ })出现n次，就会给document对象注册n个响应函数，待dom对象加载完成后就会按照注册顺序执行这些响应函数

​       window.onload=function（）{ } 出现n次，只会给window对象注册最后一个响应函数（前面的被覆盖了），待页面加载完成后只会执行这最后一个



* 2、传入参数为 [ HTML 字符串 ] 时，**创建对象**： 

​         会对我们创建这个 html 标签对象 ，包装成一个jQ对象返回



* 3、传入参数为 [ 选择器字符串 ] 时，**查询对象**： 

​          $(“标签名”)：标签名选择器，根据指定的标签名查询dom对象 ，包装成一个jQ对象返回

​          $(“#id 属性值”)： id 选择器，根据 id 查询dom对象 ，包装成一个jQ对象返回

​          $(“.class 属性值”)：类型选择器，可以根据 class 属性查询dom对象 ，包装成一个jQ对象返回



* 4、传入参数为 [ DOM 对象 ] 时，**转换对象**： 

​           会把这个 dom 对象转换为 jQuery 对象 返回

### jQuery特殊对象$

$除了可以是jQuery的核心函数之外，还可以是一个对象。通过 $.post()，$.get()，$.ajax()等这些方式使用

### jQuery对象与dom对象

#### 定义区别

* **dom对象：**

  1、查询：document.getElementsByTagName()、document.getElementsById()、       document.getElementsByName()

  2、创建：document.createElement("tagName")  //创建一个某类标签的dom对象

* **jQuery对象：**

  1、查询： $(“tagName”)、$(“#id 属性值”)、$(“.class 属性值”)

  2、创建：$("html字符串")

  3、转换：$(dom对象)  //把dom对象转换为 jQuery 对象



#### 三大板块对dom对象的访问方式

  1、css：tagName、id、class

  2、JS：tagName、id、class、name

  3、jQ：tagName、id、class 、*、过滤选择器    

#### jQuery对象的本质

jQuery 对象是 dom 对象的数组 + jQuery 提供的一系列功能函数。主要用来对dom对象进行处理。对其元素的访问与数组一样，即 jQ对象[index] 的方式，另外它里面还有各种属性和方法提供使用，使用方式与对象一样

#### jQuery对象和dom对象使用的区别

* jQuery 对象不能使用 DOM 对象的属性和方法 

* DOM 对象也不能使用 jQuery 对象的属性和方法

#### jQuery对象与dom对象的转换

* dom对象--->jQuery对象：$(dom对象)
* jQuery对象--->dom对象：jQuery对象[下标]   //访问数组元素

#### jQuery对象常用方法

* 注册事件：jq对象.事件方法（function（）{}），如 $btnObj.click( funtion(){ } )     // 给里面的dom对象注册 onclick事件

  

* 获取或设置样式属性： jq对象.css()  // css函数不同参数有不同的意义。

* * jq对象.css("键")，取第一个元素的css样式中指定键对应的值； 
  * jq对象.css( {键1：值1，键2：值2，...，键n：值n} )，设置所有元素样式； 
  *  jq对象.css(“键”，值)，设置所有元素样式 

  

* 设置value属性值：jq对象.val(值)    //更详细请看后面jQuery对象的元素属性操作方法

  

* 删除dom元素：jq对象.remove（）//删除里面所有的dom元素，当然附带的后代元素肯定也删除了

  

* 遍历里面的dom对象：jq对象.each（function（）{}） // 表示遍历每个对象时都执行一次function，其中function可以读写外部的本函数局部变量（可能实现的时候是把外部变量的地址传递过去），也可以通过this使用到本次遍历到的dom对象

  ```javascript
  var str="";
  $obj.each(function () {
     str+=" ";
     str+=this.value;
  });
  alert(str);
  ```

### jQuery选择器

所谓的选择器，常常是指选择条件，不同类型的条件的表达形式不同，处理的代码逻辑不同

#### 一、基本选择器

* 标签名选择器：$(“tagName”)

  

* id选择器：$(“#id 属性值”)

  

* class选择器：$(“.class 属性值”)

  

* 全选选择器：$(“*”)  选择所有的标签对象，不包括文本对象

  

注意：选取结果的顺序是按照它们在页面上的顺序排列的



#### 二、过滤选择器

​		过滤器是由一些限定条件组合而成，其中重要的符号有“：”、“[ ]” 等表示过滤的意思。**过滤选择器往往用作合成选择器的一部分，同于“与”逻辑过滤掉元素。**下面例子大都是 *与选择器*，过滤选择器仅仅是其中一部分

​		所谓过滤器就是在已经选择的元素基础上再过滤掉一些元素，而不是像选择器那样去选取新的元素



##### 3.1 基本过滤选择器

* ：first    //获取第一个元素，如 $（"button:first"）表示获取button元素中的第一个元素

  

* ：last    //获取最后一个元素，如 $（"button:last"）表示获取button元素中的最后一个元素

  

* ：not（选择器） //获取没有匹配的元素，如$（"input:not(#01)"）表示获取input元素中，id不为01的元素。$（"input:not(:checked)"）表示获取在input元素中，没有被选择的元素。注意， 这里的:checked是一个选择器



更多基本过滤选择器可以查看该节chm文档

##### 3.2 内容过滤选择器

* ：contains（text）//获取包含给定文本的元素，如 $("div:contains('John')") 表示获取div元素中含有John文本的元素，是否含有文本是检查该标签内的所有text内容（即包括自己的text以及后代的text）

  

* ：has（selector） //匹配 **含有选择器所匹配的后代元素** 的元素，如 $("div:has(p)")表示获取div元素中含有p元素的元素，这里的含有是指该元素的后代元素含有而非仅仅孩子元素含有



更多内容过滤选择器可以查看该节chm文档



##### 3.3 属性过滤选择器

* [attribute]    //匹配包含给定属性的元素 ，如 $("div[id]")

  

* [attribute=value]  //匹配包含给定属性的元素,且该属性值为value的元素,如 $("div[id='abc']")



更多内容过滤选择器可以查看该节chm文档

 ##### 3.4 表单过滤选择器

* ：text    //匹配所有的单行文本框，如$(":text")



更多表当过滤选择器可以查看该节chm文档

##### 3.5表单对象属性过滤选择器

* ：enable  //匹配所有可用元素，如$("input:enabled")表示匹配input元素中的可用元素

  

* ：disable  //匹配所有不可用元素，如$("input:disabled")表示匹配input元素中的不可用元素

  

* ：checked   //匹配所有选中的被选中元素(复选框、单选框等，不包括select中的option)

  

* ：selected    // 匹配所有选中的option元素，注意，这里不是select元素，而是里面的option元素



#### 三、层级选择器

所谓层级选择器，**本质上是一个符号**，按与逻辑接在前面的结果上（不只是前面一个而是前面所有选择器的最终选择结果），形成一个层次选择

* 后代选择器：ancestor descendant  中的 **空格符**，  表示后代

  ​    //在给定的祖先元素下匹配所有的后代元素 ，注意：祖先元素可能多个，每个祖先元素下可能有多个后代元素匹配。所谓后代指该对象为根的整个树上的所有非根结点。当一个指定类型的后代元素有多个不同层级的指定类型的祖先元素时，该后代元素在结果中依然只出现一次，不能重复 

  

* 子元素选择器：parent > child    中的  **>** ，表示孩子

  //在给定的父元素下匹配所有的子元素，这里的子元素指孩子结点，比后代范围窄

  

* 相邻元素选择器：prev + next  中的  **+** ，表示相邻元素

  //匹配所有紧接在 prev 元素后的 next 元素

  

* 之后的兄弟元素选择器：prev ~ sibings   中的  **~** ，表示之后的兄弟

  //匹配 prev 元素之后的所有 siblings 元素




#### 四、复合选择器

所谓复合选择器，就是由多个选择器结合在一起的，这些选择器可以是**基本选择器、过滤选择器、层级选择器**



* 或结合：$(“选择器1，选择器2，...，选择器N”)  ，相当于"或"逻辑

  

* 与结合：$(“选择器1选择器2...选择器N”)，即同时满足不同的选择器，后面的选择器是在前面的选择器的选择结果上再做选择的，相当于"与"逻辑。这里不同选择器要紧挨不能留空白，否则报错。因此，如果存在tagName选择器，那需要放在首位，否则由于无法分割而认为它属于前一个选择器

  

* 或&与结合：$(“选择器1选择器2，选择器3...，选择器N”)，即选择器1与选择器2做与结合，在于选择器3做或结合，这样按从左到右的顺序不断结合（与结合或或结合）进新的选择器

#### 总结

   单体选择器有三大类：基本选择器、过滤选择器、层级选择器。

   而复合选择器是单体选择器从左到右的排列在一起，并从左到右结合选择器，每个选择器可以按照与逻辑（紧挨）也可以按照或逻辑（逗号隔开）进行选择，与逻辑是基于前面结果的，或逻辑不基于前面结果。

  其中层级选择器只能按照与逻辑选择

### jQuery对象的元素筛选方法

所谓jQuery元素的筛选方法，其实就是jQ对象提供的方法。这些方法能够从本jQ对象中的dom对象集合出发选出符合条件的dom对象后，放置于一个新的jQ对象里面，返回这个新的jQ对象。或者从本jQ对象中的dom对象集合出发做一些判断，返回true/false。

* eq（index）：获取指定索引的元素。 即 jQ对象.eq（index）返回指定位置的元素。当然，不是返回元素本身，而是返回包含这个指定元素的新的jQ对象

  

* first（）：获取第一个元素。返回包含这个元素的新的jQ对象

  

* nextAll（）：查找当前元素之后所有的同辈元素。这里的当前元素是指jQ对象中的每一个元素，对每个元素查找后面的所有同辈元素，并把所有的结果去重排序（按照它们在页面中的顺序排序）后加到新的jQ对象中，返回这新的jQ对象

  
  
* is(选择器)：如果里面有至少一个元素匹配选择器，就返回true，否则返回false

  

更多筛选方法请看该节的chm文档

### jQuery对象的元素属性操作方法

#### 1、html、文本、值

* html（）：无参数表示获取**第一个元素的innerHTML内容**；有参数表示设置所有元素的innerHTML 内容。新内容将会覆盖标签内所有html内容（包括文本和标签）

  

* text（）：无参数表示获取**所有元素的innerText**，每个元素的innerText包括本身及其后代的全部文本；有参数表示设置所有元素的innerText。新内容将会覆盖每个元素标签内所有html内容（包括文本和标签）

  

* val（）：无参数表示获取**第一个元素的value值**，有参数表示设置所有元素的value值。注意这里的value值只有表单项才会有

    **val（）方法还可以设置表单项的选中状态：**

  ​     1、单选：$(":radio").val([值])  //将会把单选元素集合中，value=值的元素置为选中状态

  ​     2、多选：$(":checkbox").val([值1，值2，...，值n])  //将会把复选元素集合中，value=值i（i=1,2，...，n）的元素置为选中状态

  ​     3、下拉多选：$("#multiple").val([值1，值2，...，值n]); //将会把select标签下的option标签集合中，value=值i（i=1,2，...，n）的元素置为选中状态。*如果jQ对象有多个select对象，那就对每个select对象执行上面的操作*

  ​     4、下拉单选：$("#single").val([值])  //将会把select标签下的option标签集合中，value=值的元素置为选中状态。*如果jQ对象有多个select对象，那就对每个select对象执行上面的操作*

  ​     5、一次性操作：$(":radio,:checkbox,#multiple,#single").val([值1，值2，...，值n]);   //表示把选中的所有元素集合中，value=值i（i=1,2，...，n）的元素置为选中状态。其中如果含有select元素，则是对其里面的option元素进行操作

  **val（）设置选中状态的局限：只能对设置选中（true）状态，不能设置未选中（false）状态**

  **val（）设置选中状态的优势：可以通过value值指定选中的元素**

  对于属性的设置还可以通过attr( )与prop（）方法进行

#### 2、属性

* attr()：可以设置和获取属性的值，不推荐操作 checked、readOnly、selected、disabled 等等，因为如果不存在该属性的时候获取属性值则会返回undifined，官方认为这是一个错误，不符合这些属性的逻辑固有特性（即该属性无需定义也是存在的，并存在默认值）

  attr（）方法还可以操作非标准的属性（即自定义的属性），比如自定义的abc等属性

  

* prop（）： 可以设置和获取属性的值,只推荐操作 checked、readOnly、selected、disabled 等等，因为如果不存在该属性的时候获取属性值则会返回false

 注意，attr（）与prop（）获取属性值只会返回**第一个元素的属性**，而设置属性则会设置全部元素的属性

**特别注意：attr（）对元素属性的设置会反映到其html上面，属于强设置，这是 JQ中比较特别的方法，而其他方法都是弱设置**

### jQuery对象里元素的增删改

#### 0、普通插入（作为孩子）

* append（）：参数可以是html字符串或者是dom对象，把dom对象添加到所有元素的孩子列尾部，成为他们最后一个孩子结点

#### 1、内部插入（作为孩子）

* appendTo() ： a.appendTo(b)   // 把a中的所有dom元素插入到b中的每个dom元素的孩子列的尾部。成为其最后一批孩子，完成后将删除a中所有元素，如果a中的元素来自页面，那么页面中原来的a元素将被删除。b可以是一个jQ对象，也可以是一个选择器。

  

* prependTo()：  a.prependTo(b)   // 把a中的所有dom元素插入到b中的每个dom元素的孩子列的首部。成为其第一批孩子，完成后将删除a中所有元素，如果a中的元素来自页面，那么页面中原来的a元素将被删除。b可以是一个jQ对象，也可以是一个选择器。



**注意：** 上述插入是复制后插入。a中的全部元素将会复制 b.length 份分别插入到b中的每个元素的孩子列中，而非把同一个元素的引用插入到所有的b元素中，这可以通过获取前后相应标签数量来验证。

#### 2、外部插入（作为兄弟）

* insertAfter() ：a.insertAfter(b)   // 把a中的所有dom元素插入到b中的每个dom元素的紧邻后面，成为其后面一排邻居，完成后将删除a中所有元素，如果a中的元素来自页面，那么页面中原来的a元素将被删除。b可以是一个jQ对象，也可以是一个选择器

  

* insertBefore() ：a.insertBefore(b)  // 把a中的所有dom元素插入到b中的每个dom元素的紧邻前面，成为其前面一排邻居，完成后将删除a中所有元素，如果a中的元素来自页面，那么页面中原来的a元素将被删除。b可以是一个jQ对象，也可以是一个选择器

  

**注意：** 上述插入是复制后插入。a中的全部元素将会复制 b.length 份分别插入到b中的每个元素的前面或后面，而非把同一个元素的引用插入到所有的b元素的前面或后面，这可以通过获取前后相应标签数量来验证。

#### 3、替换

* replaceWith() ：a.replaceWith(b)   // 把b中的所有dom元素替换掉a中的所有dom元素，即删除a中所有dom元素，在a中最后一个dom元素的位置上插入b的所有dom元素，完成后将删除b中所有元素，如果b中的元素来自页面，那么页面中原来的b元素将被删除。b可以是一个jQ对象，也可以是一个选择器

  

* replaceAll() ：a.replaceAll(b)   // 把b中的所有dom元素替换掉a中的每个dom元素，即删除a中每个dom元素，并把b中全部元素的一份拷贝替换进去，完成后将删除a中所有元素，如果a中的元素来自页面，那么页面中原来的a元素将被删除。b可以是一个jQ对象，也可以是一个选择器



**注意**：a.replaceWith(b) 是用b整体替换a整体，a.replaceAll(b)则是用a整体的 b.length 份拷贝分别替换b中的每一个元素

#### 4、删除

* remove() ：a.remove()    //删除 a 中所有dom元素

  

* empty() ：a.empty();   //清空 a 里所有元素的内容 ，包括文本与后代标签

### jQuery对象里元素的css样式操作

#### 1、直接操作元素的css样式

通过css（）方法：jq对象.css()   // 获取、添加样式

* * jq对象.css("键")，取第一个元素的css样式中指定键对应的值； 
  * jq对象.css( {键1：值1，键2：值2，...，键n：值n} )，给所有元素添加样式； 
  *  jq对象.css(“键”，值)，给所有元素添加样式

**注意是css（）是添加样式，不是设置样式，如果有重复键，新的将会覆盖旧的**

#### 2、操作元素的class属性值，间接改变元素样式

元素的class属性可以有多个值（不同值用空格分开），因此常常利用css的class选择器，这样通过dom对象的class属性值就可以应用多种样式，通过修改class属性值就可以修改标签的样式了

* addClass(“class属性值”) ：给所有元素添加class属性值（在元素的class属性值里面去重）

  

* removeClass() ：每个元素删除所有class属性值

* removeClass(“class属性值”)：每个元素删除指定的class属性值

  

* toggleClass(“class属性值”) ：每个元素，有就删除class属性值，没有就添加class属性值

  

#### 3、操作元素的定位

* offset（）：获取元素的（top，left）定位，返回的是第一个元素的定位（放在一个普通 js对象 的top与left属性里）
* offset（ {top：xx ， left：yy} ）：设置所有元素的定位  //一般只有一个元素才会使用，否则所有元素将会重叠

### jQuery对象里元素的动画操作

动画是通过不断的切换样式实现的，静态样式的快速切换就呈现了动画效果

#### 1、基本动画（隐藏/显示）

* show() ：将隐藏的元素显示 （该方法只对隐藏元素有效）

  


* hide() ：将可见的元素隐藏（该方法只对可见的元素有效）

  

* toggle() ：可见就隐藏，不可见就显示



以上方法都是立即呈现的，可以添加参数改变：

​       1、第一个参数是动画 执行的时长，以毫秒为单位 

​       2、第二个参数是动画的回调函数 (动画完成后自动调用的函数)

**注意**：

  1、通过递归函数执行toggle方法，可以让动画不间断呈现

  2、这些貌似重载的函数应该是通过函数内部对参数进行判断，不同参数执行不同功能。这可能利用到隐形参数arguments

#### 2、淡入淡出动画

* fadeIn() ：淡入（慢慢可见），只对隐藏元素有效

  

* fadeOut()：淡出（慢慢消失），只对可见元素有效

  

* fadeToggle():  淡入淡出切换

  

* fadeTo（speed，opacity）：指定时间内淡出到某个透明度（0透明 1可见），speed是完成时间，单位ms，opacity是0到1的某个透明度值



以上 fadeIn、fadeOut、fadeToggle 都是立即呈现的，可以添加参数改变：

​       1、第一个参数是动画 执行的时长，以毫秒为单位 

​       2、第二个参数是动画的回调函数 (动画完成后自动调用的函数)

而 fadeTo 方法必须存在speed、opacity参数才能执行，如果需要，可以添加一个回调函数

### jQuery对象里元素的事件操作

* click（）：单击事件。存在函数参数时，是绑定事件[ click( function(){xxx} )  或者  click（函数名不带括号） ]；没有参数时是触发事件

  

* mouseover() ：鼠标移入事件 。存在函数参数时，是绑定事件；没有参数时是触发事件

  

* mouseout() ：鼠标移出事件。存在函数参数时，是绑定事件；没有参数时是触发事件

  

* bind( “事件1 事件2 ... 事件n”，function（）{} ) ：可以给元素一次性绑定一个或多个事件。这些事件将用同一个响应函数。注意，**这个方法不能触发事件，只能绑定事件**

  ```javascript
      //给h5绑定单击、鼠标移入、鼠标移除事件，这几个事件共用一个响应函数
      $("h5").bind("click mouseover mouseout",function () {
          console.log("这是bind绑定的事件");
      });
  ```



* one（ “事件1 事件2 ... 事件n”，function（）{} ) ：用法和意义与bind一样，也是只能绑定事件不能触发事件，不同的是，one注册的所有事件都只会响应一次

  

* unbind( “事件1 事件2 ... 事件n”) ：跟 bind 方法相反的操作，解除指定的事件的绑定，也是只能绑定事件不能触发事件

  

* live() ：也是用来绑定事件，用法同 bind 。它可以用来给与选择器匹配的所有元素绑定事件。**哪怕这个元素是后面动态创建出来的也有效**

  ​     也就是说 live 除了绑定本jQ对象里面的元素外，还能一直监视新创建的元素，如果新创建的元素匹配本jQ对象的选择器，那就给它绑定指定的事件

  ​      **注意**：live（）方法工作原理决定了其有效的前提是jQ对象存在选择器，也就是说这个jQ对象是通过 $("选择器") 的方式得到的，如果是通过 $("标签的html字符串") 或者 $(dom对象) 得到的则 live（）方法无效

**注意：上述用到匿名函数（即function（）{ }）的地方，都可以用非匿名函数的函数名来代替，后面不加括号，否则表示一个调用语句而非函数**



### 事件冒泡

事件的冒泡是指，父子元素同时监听同一个事件。当触发子元素的事件的时候，父元素的事件也会被触发。简单来说就是事件触发动作能够从子元素穿透到父元素

阻止触发动作：在子元素的响应函数中 return false 即可

### 事件对象

**定义：**事件对象，是封装有触发的事件信息的一个 javascript 对象。也就是事件触发动作发生后，会产生一个事件对象，该对象包含该事件的很多信息，例如事件的类型。而我们所谓的注册事件就是用一个事件监听器去监听事件，发现事件对象后就执行响应函数

**获取：**每次调用响应函数的时候都会把事件对象传过去，可以在响应函数上用一个形参接收，当然了，隐形参数arguments 也可以接收该事件对象

**用处：**大多时候，响应函数接收的事件对象的类型是确定的，比如，注册onclick事件时，其响应函数得到的就是click类型的事件对象。但是在使用bind注册多个事件时，由于共用一个响应函数，因此该响应函数在不同监听器上接收到的事件对象类型是变化的，而我们可以通过在函数体里面获取其事件对象类型来判断事件进而做出不同的行为，使得一个函数可以精准响应多个事件

```javascript
$("h5").bind("click mouseover mouseout",function (event) {
    if(event.type=="mouseover"){
        console.log("鼠标进来了");
    }else if(event.type=="mouseout"){
        console.log("鼠标出去了");
    }else {
        console.log("单击");
    }
});
```



### 表单序列化 serialize（）

serialize()可以把表单中所有表单项（设置了disabled属性为true或"disabled"的表单项除外）的内容都获取到，并以 name=value&name=value 的形式进行拼接

示例代码：

```javascript
var serialize = $("#form01").serialize();
```





## layui

layui（谐音：类UI) 是一款采用自身模块规范编写的前端 UI 框架，遵循原生 HTML/CSS/JS 的书写与组织形式，门槛极低，拿来即用。它坚持采用经典模块化，也正是能让人避开工具的复杂配置，重新回归到原生态的 HTML/CSS/JavaScript本身

### 使用前提

要使用layui，首先下载layui框架并复制到工程文件里面，然后再在html页面通过标签引入`layui.css`与`layui.js` 。这样就可以使用layui了

### 表格

layui对表格的渲染分为3种方式：方法渲染、自动渲染、静态转换。  其中方法渲染和自动渲染只产生一个表格，而静态渲染则在页面上产生3个表格（原表格、layui产生的只有 thead [内容来自原表格thead] 的表格 、layui产生的只有 tbody [内容来自原表格tbody] 的表格    ）

#### 表格渲染方法

所谓渲染，就是对表格呈现的外表进行处理，layui的表格渲染需要对表头和表整体进行参数设定，**特别是表头的field参数不能缺少**，否则数据显示不出

##### 1、方法渲染

其实这是“自动化渲染”的手动模式，本质类似，只是“方法级渲染”将基础参数的设定放在了JS代码中，且原始的 table 标签只需要一个 *选择器*，举例如下：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>table1</title>
    <!-- 引入 layui.css与layui.js -->
    <link rel="stylesheet" href="../layui/css/layui.css" media="all">
    <script src="../layui/layui.js"></script>
    <!--方法渲染-->
    <script type="text/javascript">
        layui.use('table', function(){
            var table = layui.table;
            //第一个实例
            table.render({
                elem: '#demo'
                ,height: 500
                ,width:950
                ,url: 'user' //数据接口
                ,page: true //开启分页
                ,cols: [[ //表头
                    {field: 'id', title: 'ID', width:80, sort: true, fixed: 'left'}
                    ,{field: 'username', title: '用户名', width:80}
                    ,{field: 'sex', title: '性别', width:80, sort: true}
                    ,{field: 'city', title: '城市', width:80}
                    ,{field: 'sign', title: '签名', width: 177}
                    ,{field: 'experience', title: '积分', width: 80, sort: true}
                    ,{field: 'score', title: '评分', width: 80, sort: true}
                    ,{field: 'classify', title: '职业', width: 80}
                    ,{field: 'wealth', title: '财富', width: 135, sort: true}
                ]]
            });

        });
    </script>

</head>
<body>
    <table id="demo" class="layui-table" lay-filter="test" ></table>
</body>
</html>
```

##### 2、自动渲染

在一段 table 容器中配置好相应的参数，由 table 模块内部自动对其完成渲染，而无需你写初始的渲染方法。其特点在上文也有阐述。你需要关注的是以下三点：

  1）带有 *class="layui-table"* 的 *< table>* 标签

  2）对table标签设置属性 *lay-data=""* 用于配置一些基础参数

  3）在 *< th>* 标签中设置属性*lay-data=""*用于配置表头信息

##### 3、静态转换

假设你的页面已经存在了一段有内容的表格，它由原始的table标签组成，这时你需要把它转换为layui表格

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格2-静态转换</title>
    <!-- 引入 layui.css与layui.js -->
    <link rel="stylesheet" href="../layui/css/layui.css"  media="all">
    <script type="text/javascript" src="../layui/layui.js"></script>
    <!--静态转换-->
    <script type="text/javascript">
        layui.use('table',function () {
            var table = layui.table;
            //转换静态表格
            table.init('demo', {
                height: 250 //设置高度
                ,width:870
                ,limit: 10 //注意：请务必确保 limit 参数（默认：10）是与你服务端限定的数据条数一致
                ,page: true //开启分页
                //支持所有基础参数
            });
        });
        window.onload=function () {
            alert("加载完成了");
        }
    </script>
</head>
<body>
    <table  lay-filter="demo" >
        <thead>
            <tr>
                <th lay-data="{field:'userName', width:150}">昵称</th>
                <th lay-data="{field:'joinTime', width:200}">加入时间</th>
                <th lay-data="{field:'signature',width:500}">签名</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>贤心</td>
                <td>2016-11-29</td>
                <td>人生就像是一场修行</td>
            </tr>
            <tr>
                <td>许闲心</td>
                <td>2016-11-28</td>
                <td>于千万人之中遇见你所遇见的人，于千万年之中，时间的无涯的荒野里…</td>
            </tr>
            <tr>
                <td>陈奕迅</td>
                <td>2016-11-28</td>
                <td>于千万人之中遇见你所遇见的人，于千万年之中，时间的无涯的荒野里…</td>
            </tr>
            <tr>
                <td>周杰伦</td>
                <td>2016-11-28</td>
                <td>于千万人之中遇见你所遇见的人，于千万年之中，时间的无涯的荒野里…</td>
            </tr>
            <tr>
                <td>古天乐</td>
                <td>2016-11-28</td>
                <td>于千万人之中遇见你所遇见的人，于千万年之中，时间的无涯的荒野里…</td>
            </tr>

        </tbody>
    </table>
</body>
</html>
```

**注意：**而静态渲染则在页面上产生3个表格：原table、layui产生的只有 thead [内容来自原表格thead] 的table  、layui产生的只有 tbody  [内容来自原表格tbody]   的table 。而只有后面两个新表格显示，原来的表格被隐藏，即 display=none

新的thead里面，每一个th由原来的**< th>xxx< /th>**  变为 **< th>< div> < span>xxx< /span> < /div>< /th>**

新的tbody里面，每一个td由原来的**< td>xxx< /td>** 变为 **< td>< div> xxx< /div>< /td>**

**新表格：**

   1、表格标签（table、thead、tbody、tr、th、td）：基本不保存原来表格对应标签的设置，只有th 的lay-data属性保留，且**< th>< /th>**变成**< th>< div> < span>< /span> < /div>< /th>**

**< td>< /td>** 变为 **< td>< div> < /div>< /td>**

   2、非表格标签：与原表格里面对应的非表格标签相同，css直接对其渲染；或者在下面额外产生一个关联标签，显示时只显示这个额外标签（例如表格里面的checkbox）



**原表格到新表格的过程：复制原表格副本--->对副本的表格标签的属性重新设定（清洗框架）--->包装内容（th里面的用< div> < span>< /span> < /div>包装，而td里面的用< div> < /div>包装)--->渲染内容（不同内容的渲染方式不同）--->table分裂（一个包含thead、一个包含tbody）**      这里包装用的div标签的class属性值是 class="layui-table-cell  laytable-cell-x-y-z"



**因此在选择表格项里面的值的时候，要特别注意，我们需要的是选择新的表格里面的选项值，因为新表格才会显示在界面上与用户交互，原表格被layui隐藏了**



**这里查询的时候要注意，按照id查询后只会查询到匹配的第一个id，因此，如果原表格里面有非表格标签，如果按id查询将只会查询到原表格里面的选项**

#### 表格刷新（重载）

表格刷新意思是重新渲染一次

* 1、方法渲染时，准备好异步数据后，重新渲染一遍即可
* 2、静态转换时，重新处理表格（一般是对tbody部分增删）后，再重新静态转换一遍即可

#### 改变静态表格里非表格标签的显示状态

* 方法1：直接获取新表格里面对应的非表格标签dom对象（注意layui对其的渲染，有可能直接渲染，也有可能产生一个关联显示层标签再渲染如checkbox），设置其相关属性即可
* 方法2：获取旧表格的dom对象的父对象，删除dom对象，通过新的html创建新的dom对象加入父对象的孩子列中原来位置，重新刷新即可

注意：方法2，不能通过改变旧表dom对象的属性值+刷新表格 的方法改变显示状态，因为虽然改变了旧表dom对象的属性值，但是该dom对象的html并没有发生改变（html不随着dom对象属性改变而该表，是dom对象创建时就确定好的），而刷新表格时，实际上是重新生成一张表格，而这张表格是根据原表格的html而新生成的，而非根据原表格的dom对象生成的，因此只有改变原表格的html才能改变新表格。当然 JQuerry的attr（）方法在改变dom元素的属性时也会反映在其html的改变上，因此，通过attr（）方法改变属性+刷新表格 也是可行的

#### 常见问题

* 表格里面，下拉选择项被挡住了：这是由于超出部分被隐藏的元素，设置一下相关样式即可

  ```html
  方法1(不推荐)，通过 jS 获取dom对象,设置style属性值即可
  <scipt>
      var tablebody=document.getElementByClassName("layui-table-body");
      for(var i=0;i<tablebody.length;i++){
          tablebody[i].style="overflow:visible;";
      }
                                           
      var tablebox=document.getElementByClassName("layui-table-box");
      for(var i=0;i<tablebox.length;i++){
          tablebox[i].style="overflow:visible;";
      }
      
      var tablecell=document.getElementByClassName("layui-table-cell");
      for(var i=0;i<tablecell.length;i++){
          tablecell[i].style="overflow:visible;";
      }
  </scipt>
      
  方法2，通过 jQuery 获取dom对象，设置样式
  <scipt>
      $(".layui-table-body, .layui-table-box, .layui-table-cell").css('overflow', 'visible');
  </scipt>
  
  方法3，通过 css 样式代码即可
  <style>
      .layui-table-body, .layui-table-box, .layui-table-cell{
          overflow:visible
      }
  </style>
  
  ```

* 设置表格的每一行高度

  ```html
  方法1(不推荐)，通过 jS 获取dom对象，再设置style属性即可
  <scipt>
      var tablebody=document.getElementByClassName("layui-table-body");
      for(var i=0;i<tablebody.length;i++){
          tablebody[i].style=" height:100px; line-height:100px ";
      }                                  
  </scipt>
      
  方法2，通过 jQuery 获取dom对象设置
  <scipt>
      $(".layui-table-cell").css('height', '100px');
       $(".layui-table-cell").css('line-height', '100px');//可有可无
  </scipt>
      
  方法3，通过 css 设置样式
  <style>
      .layui-table-cell {
        height: 100px;  
        line-height: 100px;  //可有可无
      }
  </style>
  ```



### 表单

这里，约定一个词汇的意义：**表单架**：指**class="layui-form" 的标签** 或者 **渲染了的表格（新表格）**。前者称之为 **form表单架** 后者称之为 **table表单架**。table表单架由于是渲染后新的表格，因此一般不是form表单架（渲染后，新的表格元素不保留原来的设置），当然，可以动态添加layui-form的class属性给它，这样它既是table表单架又是form表单架了

layui会对放到表单架里面的表单项进行默认渲染（有的需要做相应设置如input需要设置class=“layui-input”，有的则不需要设置如checkbox）。layui对表单元素的渲染的要求是元素放在表单架内部，即使是后面说的重新渲染也需要是表单架内部元素。以下的讨论都是基于内部表单元素的，外部的元素渲染没有官方的说明，我们当然可以利用layui的渲染对外部元素做一些class设置，但渲染结果如何，则需要自己摸索，这是不推荐的方式。

**表单元素的完整渲染大体有两类：**

   1、直接样式属性等直接渲染元素本身  

​    2、创建原元素的关联显示层元素，原元素隐藏，显示层元素显示。操作显示层元素时，原元素也会随之改变

#### 0、渲染前提

除了引入`layui.css`与`layui.js` ，还需要写一句layui.use（“form”,function(){}）或者layui.use（“table”,function(){}）等，可能需要这些模块里面的东西

#### 1、checkbox

##### checkbox的渲染策略

a、layui对checkbox的渲染策略是，在原checkbox标签下面创建一个新的div标签，用这个div标签（及其后代）作为checkbox的显示层，而checkbox则隐藏起来

b、当点击这个显示层时除了会导致显示层本身的选中状态发生变化之外，还会导致checkbox的选中状态发生变化。即checkbox标签与这个显示层标签是一个逻辑整体，单击时，他们的选中状态都会发生改变。

c、**但如果直接改变显示层的选中状态，是不会导致checkbox的选中状态也跟着变化的，必须通过触发显示层的单击事件才能让他们的选中状态都发生改变。内在的逻辑应该是：触发单击事件--->设置显示层已选+设置本体已选**

b、显示层的选中状态是通过class属性的 “layui-form-checked” 值体现的，而checkbox则是通过checked属性的true值或者“checked”值体现的。我们可以直接改变这些值从而改变它们的选中状态

注意：在layui渲染下、checkbox可以通过 title 属性值指定旁边的文字显示，通过 lay-skin 属性值指定文字与复选框的结合形式（primary表示复选框在前，默认title在前，复选框在后）

```html
 <! layui渲染前 -->
  <form class="layui-form">
      <input id="a1"  name="tdId" type="checkbox"  value="政治" title="政治" lay-skin="primary" />
  </form>

<! layui渲染后 -->
 <form class="layui-form">
     <input id="a1"  name="tdId" type="checkbox"  value="政治" title="政治" lay-skin="primary" />
     <div class="layui-unselect layui-form-checkbox" lay-skin="primary">
         <span>政治</span>
         <i class="layui-icon layui-icon-ok">
             ::before
         </i>
      </div>
  </form>
```

##### 对渲染后checkbox的监视

通过layui.use（“需要的模块名”，匿名函数）来实现，在匿名函数里面可以监视渲染后的checkbox，当然需要指定所要监视的复选框的 lay-filter 属性值。注意，只会监视到经过渲染的checkbox，未经渲染的checkbox即使lay-filter符合也不会受监视

```html
  <script>
      //监视复选框
      layui.use('form', function () {
          var form = layui.form;
          //注意：lay-filter="addRowData"
          form.on('checkbox(addRowData)', function (data) {
              //data.elem   得到checkbox原始DOM对象
              //data.elem.checked   是否被选中，true或者false
              //data.value   复选框value值，也可以通过data.elem.value得到
              //data.othis  得到美化后的DOM对象
              alert("点击了一下");
              alert(data.elem.parentNode.innerHTML);
              if (data.elem.checked) {//如果当前复选框改为选中状态
                  alert("选中");
              }
              else {
                  alert("取消");
              }
          });
      });
 </script>      

```

#### 2、radio

渲染策略，监测方法与checkbox相同，都是新生成一个关联的显示层标签，点击选择时，原标签与显示层标签一同变化。但如果通过直接改变显示层的选中状态，是不会导致原标签选中状态跟着改变的。只有通过单击选择，才可以一起变化

#### 3、select

渲染策略，监测方法与checkbox相同，都是新生成一个关联的显示层标签，点击选择时，原标签与显示层标签一同变化。但如果通过直接改变显示层的选中状态，是不会导致原标签选中状态跟着改变的。只有通过单击选择，才可以一起变化

**注意：select渲染的显示层分为两大部分： 头部框（input）+下拉列表（dl）**

   头部框的value取下拉列表的已经选择的label，placehoder=“请选择”

  而下拉列表页面上不会显示label为空的选项，且第一个选项是默认选择的，不管它有没有label。当然可以通过函数选择label为空的选项，但页面只能选择label为非空的选项

#### 4、input

通过设置class="layui-input" 即可渲染，layui对input的渲染是直接设置其css样式实现的，不需要生成新的标签，通常放在div标签内

```html
    <div class="layui-input-inline">
        <input id="ipt3" htmlEscape="false" maxlength="50" class="layui-input "/>
    </div>
```

#### 5、a标签

通过设置class="layui-btn layui-btn-xs"即可渲染，layui对a的渲染是直接设置其css样式实现的，不需要生成新的标签

#### 6、div标签水平化

div标签默认是另起一行的意思，但通过class="layui-input-inline"，使得layui可以更改其样式，从而达到水平化的目的

#### 表单元素的重新渲染

当需要改变表单元素时，有两种情况：

​     1、表单元素是直接渲染，这时候直接改变原表单元素就可以改变其显示了

​     2、表单元素是间接渲染（即通过关联显示层），这样改变原表单元素后是改变不了显示的，需要重新渲染，表单元素的重新渲染方式：**layui.form.render("标签名", filter);**  或者  **layui.form.render("标签名");**     即通过 form 模块渲染元素

​      这里filter不是指要渲染的元素的filter，而是form表单架（class=“layui-form”的元素）的filter，所以

**layui.form.render("标签名", lay-filter)  意思是：**找到 含有指定lay-filter的form表单架，对其内部的指定标签进行渲染。 

**layui.form.render("标签名",)  意思是：**找到所有的表单架，包括form表单架与table表单架，对其内部的指定标签进行渲染。

**注意：表单元素的重新渲染需要元素是表单架内部元素，即class=“layui-form”的标签里面，或者是渲染了的表格（新表格）里面**

表单元素只需要 改变dom对象属性+刷新 即可改变显示， 而表格里面的刷新，则需要 改变html+刷新 才能改变显示。原因就是表格刷新时会按照html复制一份到下面，而dom元素的动态值不反映在html上面，因此其动态值在复制时会丢失。



# xml

**定义**：xml 是可扩展的标记性语言(所谓可扩展，意思是标签名可以自定义，这是与html不同的地方)

**xml 的主要作用有：** 

1、用来保存数据，而且这些数据具有自我描述性 

2、它还可以做为项目或者模块的配置文件 

3、还可以做为网络传输数据的格式（现在 JSON 为主，用xml的不多了）

## xml语法

xml的基本语法包含以下几个模块：

1. 文档声明
2. xml 注释 
3. 元素（标签） 
4. xml 属性 

5. 文本区域（CDATA 区）

### 1.文档声明

所谓文档声明就是声明这是一个xml文件

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!--
    以上内容就是xml文件声明，表示这是个xml文件
    version="1.0"  ：表示xml版本
    encoding="utf-8"   ：表示xml文件本省的编码
-->
```

### 2.xml注释

这与html一样，都是 <!--xx-->

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!--
    以上内容就是xml文件声明，表示这是个xml文件
    version="1.0"  ：表示xml版本
    encoding="utf-8"   ：表示xml文件本省的编码
-->
<books>   <!--books表示多个图书信息-->
    <book sn="SN12341234">  <!--book表示一个图书信息 sn表示该图书序列号-->
        <name>时间简史</name>
        <author>霍金</author>
        <price>75</price>
    </book>
    <book sn="SN12341235">  <!--book表示一个图书信息 sn表示该图书序列号-->
        <name>java从入门到放弃</name>
        <author>沙老师</author>
        <price>9.9</price>
    </book>
</books>
```

### 3.元素

元素是指开始标签到结束标签的内容，本质上就是一个标签。元素里面可以包含文本、子元素或者二者的混合

#### 3.1 元素命名规则

元素的命名，就是标签名的命名

* 名称可以含字母、数字以及其他的字符
* 名称不能以数字或者标点符号开始 
* ~~名称不能以字符 “xml”（或者 XML、Xml）开始~~ （现在是可以的）
* 名称不能包含空格

#### 3.2 xml也分单标签双标签

* 单标签

  ```xml
  <book sn="SN12341236" name="辟邪剑谱" author="林平之" price="9999" />
  ```

* 双标签

  ```xml
      <book sn="SN12341235">  <!--book表示一个图书信息 sn表示该图书序列号-->
          <name>java从入门到放弃</name>
          <author>沙老师</author>
          <price>9.9</price>
      </book>
  ```

### 4. xml 属性

 xml 的标签属性和 html 的标签属性是非常类似的，**属性可以提供元素的额外信息**

在标签上可以书写属性： 

​        一个标签上可以书写多个属性。**每个属性的值必须使用 引号 引起来**。 即xml把一切信息当做字符串来处理，而html则还可以是其他值，例如true/false  数字等

​         其他的规则和标签的书写规则一致

### 5. 语法规则

#### 5.1 所有XML元素必须有关闭标签

![image-20201210193153986](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201210193153986.png)

#### 5.2 XML 标签对大小写敏感

![image-20201210193411525](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201210193411525.png)

#### 5.3 XML必须正确的嵌套

![image-20201210193508124](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201210193508124.png)

#### 5.4 XML必须有根元素

![image-20201210193712211](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201210193712211.png)

#### 5.5 XML属性值必须加引号

  一个标签上可以书写多个属性。**每个属性的值必须使用 引号 引起来**。 即xml把一切信息当做字符串来处理，而html则还可以是其他值，例如true/false  数字等

#### 5.6 XML中的特殊字符

与html的字符实体类似，XML也有它的字符实体，即通过一些符号来表达出这些特殊字符：

* 小于号 < ：&lt ；

其它字符实体参考文档中（三剑客手册）的XML语法规则

#### 5.7 文本区域（CDATA 区）

CDATA 语法可以告诉 xml 解析器，我 CDATA 里的文本内容，只是纯文本，不需要 xml 语法解析 。因此一些XML中的特殊字符可以直接在CDATA区里面书写即可，不需要用到字符实体

​    格式：

​         **<![CDATA[** 这里可以把你输入的字符原样显示，不会解析 xml **]]>**

## xml解析技术

不管是 html 文件还是 xml 文件它们都是标记型文档，都可以使用 w3c 组织制定的 dom 技术来解析。所谓xml解析，就是把xml文本内容转化成对象树，以便处理

![image-20201210212801977](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201210212801977.png)

document 对象表示的是整个文档（可以是 html 文档，也可以是 xml 文档）

dom 解析技术是 W3C 组织制定的，而所有的编程语言都对这个解析技术使用了自己语言的特点进行实现。 Java 对 dom 技术解析标记也做了实现

第三方的解析： 

* jdom 在 dom 基础上进行了封装 

* **dom4j** 又对 jdom 进行了封装

* pull 主要用在 Android 手机开发，是在跟 sax 非常类似都是事件机制解析 xml 文件

这个 Dom4j 它是第三方的解析技术。我们需要使用第三方给我们提供好的类库才可以解析 xml 文件

## dom4j 解析技术

这个 Dom4j 它是第三方的解析技术。我们需要使用第三方给我们提供好的类库才可以解析 xml 文件。因此我们需要用到 `dom4j-1.6.1.jar`这个包，也可以到官网https://dom4j.github.io/下载其他版本的包

依赖包准备好后，可以写代码解析xml了

### 代码示例

```java
    @Test
    //SAXReader、Document与Element都是dom4j包里的类
    //SAXReader读取xml文件，转化为Document对象(代表整个文件)，可以通过Document对象获取根元素的Element对象
    //Element对象就对应xml里面的标签
    //Element对象的使用与js里面的dom对象类似，具体看下面
    public void teat2() throws DocumentException {
        //读取books.xml，获取document对象
        SAXReader saxReader=new SAXReader();
        Document document=saxReader.read("src/books.xml");
        
        //通过document对象获取根元素
        Element rootElement=document.getRootElement();
//        System.out.println(rootElement);
        
        //通过根元素获取book标签对象
        List<Element> list=rootElement.elements("book");
        
        //遍历，处理每个book标签转化为book类
        for(Element e:list){
            Element nameElement=e.element("name");
            //获取name
            String name=nameElement.getText();//取本对象的内部text
            //获取price
            String price=e.elementText("price");//取标签名为price的子对象的内部text
            //获取作者
            String author=e.elementText("author");//取标签名为author的子对象的内部text
            //获取sn属性值
            String sn=e.attributeValue("sn");//取本对象的sn属性的值
            //转换为Book对象
            System.out.println(new Book(sn,name,Double.parseDouble(price),author));
            System.out.println();
        }
    }
```

### 常见对象及其方法

#### 一、SAXReader对象

输入流对象，用于读取xml文件，转换成Document对象

方法：

* read（“url”）： 用于读取xml文件，转换成Document对象并返回

#### 二、Document对象

代表整个xml文件

方法：

* getRootElement()：获取根元素的Element对象并返回

#### 三、Element对象

一个Element对象代表xml里面的一个元素，与html里面的dom对象意义类似

方法：

* element（“name”）：根据标签名获取对应的**第一个子元素**
*  elements（“name”）：根据标签名获取对应的**所有子元素**，形成列表（List）返回
* getText()：获取本对象的内部text，这里不包含子元素或其他后代元素的text，仅仅是自身的text。这是与js中的dom对象的不同之处
* elementText("name")：根据签名取对应的**第一个子对象**的内部text。不存在指定标签名的**子对象**时返回null
* attributeValue（"attributeName"）：获取属性值



注意上述方法中，很多查找范围仅仅在子元素这个范围内，不包含更深的后代元素，这与js的dom对象有很大区别，注意不要混淆了



# Tomcat

## 安装

找到你需要用的 Tomcat 版本对应的 zip 压缩包，解压到需要安装的目录即可

## 目录介绍

* bin ：专门用来存放 Tomcat 服务器的可执行程序 

* conf ：专门用来存放 Tocmat 服务器的配置文件 

* lib ：专门用来存放 Tomcat 服务器的 jar 包 

* logs ：专门用来存放 Tomcat 服务器运行时输出的日记信息 

* temp ：专门用来存放 Tomcdat 运行时产生的临时数据 

* webapps ：专门用来存放部署的 Web 工程。 里面一个子目录代表一个工程

* work ：是 Tomcat 工作时的目录，用来存放 Tomcat 运行时 jsp 翻译为 Servlet 的源码，和 Session 钝化的目录

## 启动 **Tomcat** 服务器

找到 Tomcat 目录下的 bin 目录下的 startup.bat 文件，双击，就可以启动 Tomcat 服务器

打开浏览器，在浏览器地址栏中输入以下地址测试： 

1、http://localhost:8080 

2、http://127.0.0.1:8080 

3、http://真实 ip:8080 

当出现如下界面，说明 Tomcat 服务器启动成功！！！

![image-20201212213135318](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201212213135318.png)

常见的启动失败的情况有，双击 startup.bat 文件，就会出现一个小黑窗口一闪而来。 

这个时候，失败的原因基本上都是因为没有配置好 JAVA_HOME 环境变量 

## 启动多个tomcat实例

### 方法一（一对一）

核心：多个安装文件，多个tomcat实例。即每个安装文件对应一个启动的实例

* 步骤1：直接复制安装文件到别的地方:
* 更改conf目录下的server.xml文件里面的几个端口号，以免启动时端口号冲突。

连接端口、SSL的连接端口、Apache的侦听端口、停止Tomcat的端口等端口(默认8080、8443、8009、8005)

### 方法二（一对多）

核心：单个安装文件，多个tomcat实例。关键在于：程序代码共用一份，工作空间分开即可，类似于mysql的多个服务启动情形

* 步骤1：在别的地方建立tomcat1、tomcat2、...、tomcatN目录

  

* 步骤2：每个tomcat目录，与原始的tomcat安装目录下的子目录结构保持一致

![img](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/201206191420374641.png)



* 步骤3：在新的tomcat目录里的bin目录下，编写新的 startup.bat 和 shutdown.bat 文件

startup.bat

```text
rem 设置数据路径(即工作空间)
set CATALINA_BASE=D:\Tomcat\tomcat1
for %%x in ("%CATALINA_BASE%") do set CATALINA_BASE=%%~sx

rem 设置程序路径(即安装路径)
set CATALINA_HOME=D:\Tomcat\apache-tomcat-8.5.57
for %%x in ("%CATALINA_HOME%") do set CATALINA_HOME=%%~sx

rem 启动，这里调用程序文件里的startup.bat
call %CATALINA_HOME%\bin\startup.bat
```

shutdown.bat

```text
rem 设置数据路径(即工作空间)
set CATALINA_BASE=D:\Program Files\tomcat1
for %%x in ("%CATALINA_BASE%") do set CATALINA_BASE=%%~sx

rem 设置程序路径(即安装路径)
set CATALINA_HOME=D:\Program Files\apache-tomcat-7.0.14
for %%x in ("%CATALINA_HOME%") do set CATALINA_HOME=%%~sx

rem 停止，这里调用程序文件里的shutdown.bat
call %CATALINA_HOME%\bin\shutdown.bat
```

注意：

这里的启动、停止，是设置CATALINA_BASE与CATALINA_HOME（即设置实例的程序路径与数据路径），真正的启动停止程序是在程序安装路径下的startup.bat/shutdown.bat，这里可以体现出来 startup.bat/shutdown.bat 的执行只与CATALINA_BASE与CATALINA_HOME这两个环境变量有关，与它们本身所在位置无关。

启动时，会依据CATALINA_BASE与CATALINA_HOME启动，包括端口设置等，都是查询CATALINA_BASE与CATALINA_HOME下相关文件的

即真正的startup.bat/shutdown.bat在执行时与其本身位置无关，一切的启动/停止操作需要的数据都通过CATALINA_BASE与CATALINA_HOME这两个路径获取



* 步骤4：复制tomcat安装目录的 conf/server.xml 到新的tomcat目录里的 conf 目录下，并更改里面的几个端口号，以免启动时端口号冲突。连接端口、SSL的连接端口、Apache的侦听端口、停止Tomcat的端口等端口(默认8080、8443、8009、8005)

  

* 步骤5：复制tomcat安装目录的 conf/tomcat-users.xml与   conf/web.xml 到新的tomcat目录里的 conf 目录下

  

* 步骤6：双击新的tomcat目录里的 bin 目录下的 startup.bat/shutdown.bat 即可实现一个实例的启动停止



可见，对程序的相关配置并非放在程序目录里面，而是放到数据目录里面的，以便启动不同的实例



参考：https://www.cnblogs.com/erik/archive/2012/06/19/2554715.html



## Tomcat 的停止

1、点击 tomcat 服务器窗口的 x 关闭按钮 

2、把 Tomcat 服务器窗口置为当前窗口，然后按快捷键 Ctrl+C 

3、**找到 Tomcat 的 bin 目录下的 shutdown.bat 双击，就可以停止 Tomcat 服务器**

## Tomcat相关配置

### 一、启动出现中文乱码

字节流解码为字符串时，使用了的字符集和编码所用字符集不一致。windows系统中，其命令行窗口在解码字节数组时，默认使用本地字符集（对于我们就是**GBK**），而tomcat默认输出的启动信息是通过**utf8**进行编码的，这就导致编码与解码所使用字符集的不一致，从而出现了乱码情况！

解决办法就是让tomcat把打印到控制台的信息按照**GBK**编码即可：

​    步骤1：tomcat目录的conf子目录中，找到一个名为 "**logging.properties**" 的文件

​    步骤2：打开这个文本文件，找到如下配置项：java.util.logging.ConsoleHandler.encoding = **UTF-8**

​    步骤3：将 UTF-8 修改为 GBK，修改后的效果为：java.util.logging.ConsoleHandler.encoding = **GBK**

​    步骤4：保存后，重启tomcat！

### 二、更改服务端口号

Tomcat 默认的端口号是：8080，如果想要更改，可以通过以下配置

   步骤1：找到 Tomcat 目录下的 conf 目录，找到 server.xml 配置文件

   步骤2：打开这个文本文件，找到Connector元素标签

![Connector](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Connector.PNG)

​    步骤3：更改该标签的 port属性值即可

​    步骤4：重启tomcat服务器

## 部署web工程到tomcat

### 方法一

只需要把 web 工程的目录拷贝到 Tomcat 的 webapps目录下即可

访问：http://ip:port/工程名/目录下/文件名 

提示：http://ip:port 访问到 Tomcat 的 webapps目录，要访问具体工程需要在后面继续添加对应的工程路径

### 方法二

不同于方法一把项目目录直接拷贝到webapps目录下，方法二是通过相关配置，映射到外部的工程目录，具体操作如下：

  步骤1：找到 Tomcat 下的 conf \Catalina\localhost\ 下,创建如下的配置文件

  ![image-20201213113703430](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213113703430.png)

abc.xml 配置文件内容如下：

```xml
<!-- Context 表示一个工程上下文 
     path 表示工程的逻辑访问路径:/abc （逻辑上指tomcat工作空间里的abc工程，当然这是逻辑上的）
     docBase 表示你的工程目录实际路径 --> 
<Context path="/abc" docBase="E:\book" />
```

访问这个工程的路径如下:http://ip:port/abc/ 

就表示访问 E:\book 目录，即我们只需要输入工程的逻辑访问路径，内部就会转化为实际访问路径并访问

## 手托html到浏览器与在浏览器用http访问html的区别

### 一、手托方式

![image-20201213154527750](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213154527750.png)

### 二、通过http访问

![image-20201213154613105](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213154613105.png)



## ROOT的工程的访问，以及默认index.htm页面的访问

当我们在浏览器地址栏中输入访问地址： http://ip:port/ 

====>>>> 

没有工程名的时候，默认访问的是 ROOT 工程里面的 **index.xxx**



当我们在浏览器地址栏中输入的访问地址： http://ip:port/工程名/ 

====>>>> 

没有资源名，默认访问 **index.xxx** 页面



上面提到的 **index.xxx**，既可以是 **index.jsp** 也可以是 **index.html** 

## idea整合Tomcat

操作的菜单如下：File | Settings | Build, Execution, Deployment | Application Servers

![image-20201213160441219](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213160441219.png)

配置你的 Tomcat 安装目录：

![image-20201213160640617](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213160640617.png)

就可以通过创建一个 Model 查看是不是配置成功！！！ 

![image-20201213160702157](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213160702157.png)

## idea中动态web工程的操作

### 0、工程项目与Tomcat的总体关系

具体如下图所示

![工程项目与Tomcat的总体关系](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/工程项目与Tomcat的总体关系-1609735189354.jpg)

### 一、创建web工程

1、创建一个新模块：

![image-20201213161859688](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213161859688.png)

2、选择你要创建什么类型的模块：

![image-20201213161919917](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213161919917.png)

3、输入你的模块名，点击【Finish】完成创建

![image-20201213161952551](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213161952551.png)

4、创建成功如下图：

![image-20201213162016997](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213162016997.png)

### 二、web工程目录介绍

![image-20201213163053861](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213163053861.png)

注意：创建的工程文件的WEB-INF下本来没有lib的，需要我们手动添加进去。当然添加jar包进lib文件夹还不行，这只是在tomcat运行时会把它放到工程空间调用。但是本地运行时不会放到执行空间，需要右键jar包，点击Add as Library 才能正真把jar包暴露在本地工程空间里，这样编译器才不会报找不到包的错误

当然，这里还涉及到 Artifacts ，具体有什么作用还需要探究https://www.cnblogs.com/cmyBlog/p/10892713.html

**猜测：执行时，tomcat里面真正的工程空间里存放的是src里面的java代码与web里面的资源，不再存在src目录与web目录**。通过 http://ip:port/工程名 默认访问工程空间的index.jsp文件

### 三、如何给动态web工程添加额外jar包

参考<a href="#依赖添加">依赖添加</a> 中idea项的第4点

### 四、在IDEA中部署工程到Tomcat上运行

#### 1、建议修改 web 工程对应的 Tomcat 运行实例名称：

![image-20201213192640413](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213192640413.png)



#### 2、确认你的 Tomcat 实例中有你要部署运行的 web 工程模块：

在创建web工程时会指定一个tomcat实例，但运行该实例时可以自由设定需要运行的模块，不一定就是默认的模块（即创建时选择该tomcat实例的web工程模块）

![image-20201213193131429](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213193131429.png)



#### 3、你还可以修改你的 Tomcat 实例启动后默认的访问地址：

![image-20201213193419629](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213193419629.png)



#### 4、在 IDEA 中如何运行，和停止 Tomcat 实例

启动tomcat时，控制台可以看见本次启动的tomcat实例的工作空间（CATALINA_BASE）。里面有我们的工程，一般在 conf \Catalina\localhost\ 下的xml里有我们的工程的实际地址

  4.1、正常启动 Tomcat 实例：

![image-20201213193608997](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213193608997.png)

  4.2、Debug 方式启动 Tomcat 运行实例：

![image-20201213193637846](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213193637846.png)

  4.3、停止 Tomcat 运行实例：

![image-20201213193703618](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213193703618.png)

  4.4、重启 Tomcat 运行实例：

![image-20201213193722618](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213193722618.png)

![tomcat重启](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/tomcat重启.PNG)



### 五、修改工程访问路径

本质上只是修改在内存里服务器工作空间上的工程名字而已，我们也可以在Tomcat 实例工作目录下的 conf \Catalina\localhost\里的相应的xml文件配置工程名称，效果一样的。详见<a href="#方法二">部署web工程到tomcat的方法二</a>

![image-20201213195343408](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213195343408.png)

### 六、修改运行的端口号

![image-20201213195750867](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213195750867.png)

### 七、修改运行使用的浏览器

![image-20201213200100789](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213200100789.png)

### 八、配置资源热部署

![image-20201213200314618](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201213200314618.png)

注意：对于html或者jsp或者css等资源文件，更改后会自动部署进服务器；但java类文件就不会自动部署进服务器了，需要我们手动点击 redeploy （重启按钮，选择redeploy） 以重新部署工程，当然 restart Server 也行，不过没必要

## Tomcat相关问题

* 1、Tomcat实例运行时是把工程加载到内存还是直接在硬盘上读写？

  答：应该是把工程加载进内存里面，Tomcat 的 webapps目录下的工程或者 Tomcat 下的 conf \Catalina\localhost\ 下的xml文件所映射的工程。所有工程都加载tomcat的工作空间里



* 2、访问工程时，通过 **http://实例ip地址: 端口号/工程名/文件路径**   这种访问方式是如何实现的？

  答：需要查找相关资料，另外，可能与http协议有关，需要了解http协议  。应该是当有请求访问tomcat里面的资源时，tomcat将把请求封装成Request，另生成一个Response对象，带着两个对象去访问资源。资源那里可以通过request得到一些请求参数，通过response返回数据



* 3、通过http协议访问工程的对象（例如Servlet对象）是如何做到的？tomcat本身是java写的，运行在jvm上面，那么tomcat里面的工程的java代码又是怎么运行的？

​    答：这里可能涉及到类加载器，即在java程序里面去加载类，具体参考jvm的类加载器章节。猜测：工程里面的对象是由tomcat创建的（tomcat利用类加载器加载工程里的类并创建对象），即tomcat创建工程里面的对象并存放到一个map里面（这个map存放的是<url,对象>，通过http寻找的时候，先检测是否含有url，如果含有就调用对象的方法，否则再去工程里面按照路径寻找目标文件），这个map与具体工程是绑定的，不同工程都有属于自己的map

​      这里有个关键点就是，由于工程里的java代码是由tomcat调用运行的，也就是说工程代码与tomcat本身都运行在同一个jvm上，这样如果不同工程类名出现重复怎么办？**答案就是根据其类加载器不同而区分，对于不同工程的类文件，用不同的类加载器加载就可以隔离同名类了，这样保证了不同工程都能用上自己的类**。具体参考：https://www.zhihu.com/question/385233613/answer/1393721692

​     获取被某个加载器加载的类：Class.forName(String name, boolean initialize, ClassLoader loader)

![tomcat里的工程](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/tomcat里的工程.jpg)





# Servlet

## 定义

* Servlet 是 JavaEE 规范之一。规范就是接口 
* Servlet 就 JavaWeb 三大组件之一。三大组件分别是：Servlet 程序、Filter 过滤器、Listener 监听器 
* Servlet 是运行在服务器上的一个 java 小程序，它可以接收客户端发送过来的请求，并响应数据给客户端

## 使用前提

​      下面所描述的使用Servlet以及相关的类，都来自javax包，这在tomcat安装目录的lib目录里的 **servlet-api.jar** 里面含有

​      一般来说，把tomcat整合进idea时，都会自动带上 **jsp-api.jar** 与 **servlet-api.jar** 这两个jar包，所以新建的web工程选择tomcat服务器后就会自动把这两个jar包添加到依赖里了。但如果没有添加，我们也可以到tomcat安装目录的lib目录里把相应的jar包添加到依赖里

## 实现 Servlet 程序

### 实现步骤

1、编写一个类去实现 Servlet 接口 

2、实现 service 方法，处理请求，并响应数据 。即访问该Servlet类时，就是访问service方法

```java
public class HelloServlet implements Servlet {
    public static void main(String[] args) {
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {

    }

    @Override
    public ServletConfig getServletConfig() {
        return null;
    }

    /* service方法专门用来处理请求和响应 */
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("HelloServlet 被访问了，kk");
        System.out.println("哈哈哈");
    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }
}
```



3、到 web.xml 中去配置 servlet 对象及其访问地址

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- servlet 标签给 Tomcat 配置 Servlet 程序 -->
    <servlet>

        <!--servlet-name 标签 Servlet 对象的别名（考虑到可读性，一般是类名） -->
        <servlet-name>HelloServlet</servlet-name>

        <!--servlet-class 是 Servlet 对象的全类名-->
        <servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>

    </servlet>

    <!--servlet-mapping 标签给 servlet 程序配置访问地址-->
    <servlet-mapping>

        <!--servlet-name 标签的作用是告诉服务器，我当前配置的地址给哪个 Servlet 对象使用，通过Servlet程序的别名选择-->
        <servlet-name>HelloServlet</servlet-name>

        <!--url-pattern 标签配置访问地址，这里访问地址可以任意设定（但必须以/开头），浏览器按照设定的地址即可访问指定的servlet程序 <br/>
        / 斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径 <br/>
        /hello 表示地址为：http://ip:port/工程路径/hello <br/> -->
        <url-pattern>/hello</url-pattern>

    </servlet-mapping>


</web-app>
```

### xml配置Servlet的实质

xml所谓配置Servlet，实质是配置Servlet对象（<servelet>）及其相应的指针资源(</servlet-mapping>)

* 一个< servlet>标签就代表一个Servlet类对象，而非一个Servlet类。因此，不同的< servlet>标签的类可以一样，但对象不同，也就是说即使两个< servlet>标签含有相同的类，但访问的时候其实访问的是相同类的不同对象而已，这里可以通过在service方法打印对象看出
* < servlet-mapping>与< servlet>之间的通过< servlet-name>连接起来
* < servlet-mapping>配置的url实质上是配置指针资源的url，该指针资源指向相应的 Servlet 对象，访问该指针资源就是指针其指向的Servlet对象的service方法

![servlet的路径配置（xml里面）](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/servlet的路径配置（xml里面）-1609063738996.jpg)



### 常见错误

常见的错误 1：url-pattern 中配置的路径没有以斜杠打头

![image-20201220213750307](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201220213750307.png)

常见错误 2：servlet-name 配置的值不存在：

![image-20201220213820397](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201220213820397.png)

常见错误 3：servlet-class 标签的全类名配置错误：

![image-20201220213847147](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201220213847147.png)

## url 地址到 Servlet程序的访问(指针资源)

![image-20201220215758594](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201220215758594.png)

注意：就是浏览器通过 url 访问到相应的指针资源，执行该指针资源指向的Servlet对象的service方法。**猜测：加载工程后，根据 web.xml 的 < servlet> 与 < servlet-mapping> 在设置好的 url 指定的位置创建指针资源，指向相应的Servlet对象**，详见<a href="#xml配置Servlet的实质">xml配置Servlet的实质</a>





## Servlet 的生命周期

1、执行 Servlet 构造器方法 

2、执行 init 初始化方法 （来自override）

第一、二步，是在第一次访问，的时候创建 Servlet 程序会调用 



3、执行 service 方法 （来自override）

第三步，每次访问都会调用 



4、执行 destroy 销毁方法 （来自override）

第四步，在 web 工程停止的时候调用

## GET 和 POST 请求的分发处理

Servlet的service方法可以分辨请求类型，通过把ServletRequest转换成其子类型HttpServletRequest，调用HttpServletRequest的getMethod（）方法即可，如果返回值是“POST”，那就是post类型请求；如果返回值是“GET"。那就是get类型请求

```java
public class HelloServlet implements Servlet {

    /* service方法专门用来处理请求和响应 */
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("3. service方法==hello servlet");
        //类型转化(把ServletRequest转换成其子类HttpServletRequest)
        HttpServletRequest httpServletRequest= (HttpServletRequest) servletRequest;
        //获取请求方式
        String method=httpServletRequest.getMethod();

        if(method.equals("GET")){
            doGet();
        }else if(method.equals("POST")) {
            doPost();
        }
    }

    /**
     * 处理get请求
     */
    private void doGet(){
        System.out.println("处理get请求");
    }

    /**
     * 处理post请求
     */
    private void doPost(){
        System.out.println("处理post请求");
    }
}
```

## 通过继承 HttpServlet 实现 Servlet 程序

般在实际项目开发中，都是使用继承 HttpServlet 类的方式去实现 Servlet 程序，即HttpServlet 类已经实现Servlet了，我们只需要根据业务需要重写一些方法即可：

   1、编写一个类去继承 HttpServlet 类 

  2、根据业务需要重写 doGet 或 doPost 方法 

  3、到 web.xml 中的配置 Servlet 程序的访问地址

```java
public class HelloServlet2 extends HttpServlet {
    /**
     *
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("hello2处理get请求");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("hello2处理post请求");
    }
}
```



## Servlet 类的继承体系

其中GenericServlet是一个抽象类

![image-20201226160953270](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201226160953270.png)

## HttpServlet结构

![HttpServlet结构](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/HttpServlet结构.jpg)

## 继承 HttpServlet 的 Servlet 的结构

大致结构如下图所示，其中GenericServlet是一个抽象类，只是实现了Servlet的部分方法，还有部分方法是在HttpServlet上实现的，HttpServlet也是个抽象类

![继承 HttpServlet 的 Servlet 的结构](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/继承 HttpServlet 的 Servlet 的结构-1609060841050.jpg)



## 使用 IDEA 创建 Servlet 程序

这种方法的优点就是：

* 创建继承HttpServlet的类
* 自动配置xml文件，当然还需要自己去配置< servlet-mapping>



菜单：new ->Servlet 程序

![image-20201226112533826](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201226112533826.png)

配置 Servlet 的信息：

![image-20201226112553578](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201226112553578.png)



## ServletConfig 类

### 定义

* ServletConfig 类从类名上来看，就知道是 Servlet 程序的配置信息类

* Servlet 程序和 ServletConfig 对象都是由 Tomcat 负责创建，我们负责使用

* Servlet 程序默认是第一次访问的时候创建，ServletConfig 是每个 Servlet 程序创建时，就创建一个对应的 ServletConfig 对象

总的来说，ServletConfig主要用于存在Servlet程序的配置信息，这些配置信息是我们在< servlet>标签里面定义的，具体看后面的web.xml 中的配置

### 作用

​	1、可以获取 Servlet 程序的别名 servlet-name 的值 

​	2、获取初始化参数 init-param 

​	3、获取 ServletContext 对象

### 配置

web.xml 中的配置：

```xml
    <servlet>

        <!--servlet-name 标签 Servlet 程序的别名（考虑到可读性，一般是类名） -->
        <servlet-name>HelloServlet</servlet-name>

        <!--servlet-class 是 Servlet 程序的全类名-->
        <servlet-class>com.atguigu.servlet.HelloServlet</servlet-class>

        <!-- init-param是初始化参数，主要通过键值对的形式配置参数 -->
        <!-- 该Servlet标签里的所有 init-param 里面的键值对，将会存放在一个对应的 ServletConfig 对象里面 -->
        <init-param>
            <!--参数名-->
            <param-name>userName</param-name>
            <!--参数值-->
            <param-value>root</param-value>
        </init-param>
        <init-param>
            <!--参数名-->
            <param-name>url</param-name>
            <!--参数值-->
            <param-value>jdbc:mysql://localhost:3306/test</param-value>
        </init-param>

    </servlet>
```

Servlet 中的代码：

```java
    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        /*System.out.println("2. init方法");*/

//        1、可以获取 Servlet 程序的别名 servlet-name 的值
        System.out.println("HelloServlet程序的别名是："+servletConfig.getServletName());

//        2、获取初始化参数 init-param
        System.out.println("初始化参数userName的值是："+servletConfig.getInitParameter("userName"));
        System.out.println("初始化参数url的值是："+servletConfig.getInitParameter("url"));

//        3、获取 ServletContext 对象
        System.out.println("ServletContext: "+servletConfig.getServletContext());
    }
```

### 注意

* ServletConfig 是每个 Servlet 程序创建时，就创建一个对应的 ServletConfig 对象。并且把此 ServletConfig 对象作为实参去调用Servlet对象的 init方法 ，如果我们想以后使用它，可以在 init 方法里面把 ServletConfig 对象保存起来放到属性里面

*  GenericServlet 类实现了 Servlet 类，且已经实现了 init 方法，就是把 ServletConfig 对象保存在属性里，可通过 getServletConfig() 方法获取

* 如果我们通过继承 HttpServlet 类的方式去实现 Servlet 程序，如果需要重写 init 方法，那么就要特别注意，最好调用 super.init() 方法把 ServletConfig 对象保存起来，以防后面要用到

## ServletContext

### 定义

1、ServletContext 是一个接口，它表示 Servlet 上下文对象 

2、一个 web 工程，只有一个 ServletContext 对象实例

3、ServletContext 对象是一个**域对象**

4、ServletContext 是在 web 工程部署启动的时候创建。在 web 工程停止的时候销毁



**什么是域对象?** 

域对象，是可以像 Map 一样存取数据的对象，叫域对象。 

这里的域指的是存取数据的操作范围，整个 web 工程。可以把它看做是整个web工程的“全局变量”

**域对象的数据操作：**

​    1、添加数据：setAttribute（）

​	2、获取数据：getAttribute（）

​	3、删除数据：removeAttrbute（）

### 作用

​	1、获取 web.xml 中配置的上下文参数 context-param （这些参数以（key，value）的形式保存在ServletContext中）

​	2、获取当前的工程路径，格式: /工程路径 

​	3、获取工程部署后在服务器硬盘上的绝对路径 

​	4、像 Map 一样存取数据 （以（key，value）的形式存取数据）



**web.xml 中的配置：**

```xml
    <!-- context-param是上下文参数(它属于整个web工程),主要通过键值对实现 -->
    <!-- web工程所有的context-param里面的键值对，都会存放在一个 ServletContext 对象里 -->
    <context-param>
        <param-name>userName</param-name>
        <param-value>context</param-value>
    </context-param>
    <context-param>
        <param-name>passWord</param-name>
        <param-value>888</param-value>
    </context-param>
```

**ServletContext 演示代码：**

```java
public class ContextServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println();

        //0、准备好ServletContext对象，有如下两种方法，都是来自GenericServlet
        ServletContext servletContext = getServletConfig().getServletContext();
        ServletContext servletContext1 = getServletContext();
        
        //1、获取 web.xml 中配置的上下文参数 context-param
        System.out.println("context-param userName: "+servletContext.getInitParameter("userName"));
        System.out.println("context-param passWord: "+servletContext.getInitParameter("passWord"));


  	    //2、获取当前的工程路径，格式: /工程路径
        System.out.println("当前工程路径："+servletContext.getContextPath());

	    //3、获取工程部署后在服务器硬盘上的绝对路径，即本tomcat实例上运行的工程的绝对路径（在tomcat工作空间里，一般在 conf \Catalina\localhost\ 下的xml里也可以看到我们的工程的实际地址）。启动时控制台可以看见本次的tomcat实例的工作空间（CATALINA_BASE）
         
        /**
         * 斜杠 / 在服务器解析的时候，表示地址为：http://ip:port/工程路径   所谓工程路径，实际映射到的是idea代码的web目录，也就是说一个工程，实际上对应的是idea代码的web目录
         这里的getRealPath("/")相当于把http://ip:port/工程路径转化为工程在硬盘上的绝对路径
         */
        System.out.println("工程部署的路径："+servletContext.getRealPath("/"));
        System.out.println("工程下css的路径："+servletContext.getRealPath("/css"));
        System.out.println("工程下img的1.jpg路径："+servletContext.getRealPath("/img/1.jpg"));

	    //4、像 Map 一样存取数据
        System.out.println("context1中，保存之前获取域数据key1的值："+servletContext.getAttribute("key1"));

        servletContext.setAttribute("key1","value1");

        System.out.println("context1中，获取域数据key1的值："+servletContext.getAttribute("key1"));
    }
}
```

## Servlet、ServletConfig与ServletContext

注意：Servlet 对象是否保存了 ServletConfig 对象取决于其 init 方法，一般继承自 HttpServlet 时，init是会保存ServletConfig 对象的，否则用户实现 init 方法时可以自行决定

![Servlet、ServletConfig与ServletContext的关系](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Servlet、ServletConfig与ServletContext的关系-1609065304342.jpg)



## HttpServletRequest 类

### 作用

​     每次只要有请求进入 Tomcat 服务器，Tomcat 服务器就会把请求过来的 HTTP 协议信息解析好封装到 Request 对象中。 然后传递到 service 方法（doGet 和 doPost）中给我们使用。我们可以通过 HttpServletRequest 对象，获取到所有请求的信息

### 常用方法

#### i. getRequestURI()    

获取请求的资源路径   如  

```txt
/07_servlet/requestAPI
```



#### ii. getRequestURL() 

获取请求的统一资源定位符（绝对路径） 如

```txt
 http://localhost:8080/07_servlet/requestAPI
```



#### iii. getRemoteHost() 	  	

获取客户端的 ip 地址 

```txt
 在IDEA中，客户端使用localhost访问时，得到的客户端地址是  0:0:0:0:0:0:0:1 <br>
 在IDEA中，客户端使用127.0.0.1访问时，得到的客户端地址是  127.0.0.1 <br>
 在IDEA中，客户端使用 真实ip 访问时，得到的客户端地址是  真实ip <br>
```



#### iv. getHeader("key") 		 	

获取请求头 ,由于请求头是键值对的集合，因此，我们需要指明获取哪一个 键 对应的 值。例如request.getHeader("referer")则表示得到请求端的页面地址，即从哪个地址发起请求的，这个信息一般用于重定向回原来页面



#### v. getParameter("name")

​		获取请求的参数 ,参数是通过（name，value）的键值对传过来的，由于传送过来的键值对存在相同name的情况（如表单里的复选框），tomcat把这些键值对按照（name，valueList）的形式存放在map里面并放入HttpServletRequest里面。

​		而getParameter("name")将获取的对应name的valueList的第一个值

注意：前端通过post方式提交给后端时，容易出现中文乱码问题，具体原因和解决方法参考<a href="#三、前端传数据给后端的中文乱码问题">后端获取数据的中文乱码问题</a>  



#### vi. getParameterValues("name") 	 	

获取请求的参数（多个值的时候使用） ，将获得对应name的整个valueList，并以数组的形式传回

注意：前端通过post方式提交给后端时，容易出现中文乱码问题，具体原因和解决方法参考<a href="#三、前端传数据给后端的中文乱码问题">后端获取数据的中文乱码问题</a>  



#### vii. getMethod() 		 

获取请求的方式 GET 或 POST 



#### viii. setAttribute(key, value); 	 	

设置域数据 （以便请求转发时别的Servlet对象可以访问，常用于数据共享，类似于ServletContext）



#### ix. getAttribute(key); 		 

获取域数据 



#### x. getRequestDispatcher() 		 

获取请求转发对象 

```java
//获取请求转发对象，里面封装了资源路径（不一定是Servlet对象，也可以是其他资源，如html），以便访问
RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");

//通过请求转发对象，访问另一个资源（不一定是Servlet对象，也可以是其他资源，如html）
requestDispatcher.forward(request,response);
```



#### xi. 其他方法

如下：

```java
        request.getScheme(); //获取协议，如http
        request.getServerName(); //获取服务器 ip，如localhost
        request.getServerPort(); //获取服务器端口，如8080
        request.getContextPath(); //获取工程路径,如 /book
        request.getMethod(); //获取请求方法
        request.getRemoteHost(); //获取客户端 ip 地址
        session.getId(); //获取会话的 id 编号
```







### 获取数据的中文乱码问题

​		按照**post**方式提交时，浏览器对于中文参数将采用基于utf-8的url编码，而服务器则是默认基于ISO-8859-1字符集的url解码，因此会中文乱码问题（get方式提交不会出现中文乱码，因为get方式提交时，服务器是默认基于utf-8编码的url解码）

​		这样，前端传递的 A（utf-8）字符串将会只改变编码变成B（ISO-8859-1），但其背后的字节信息是不变的，因为url解码是会把原来的字节信息翻译出来的，就是从字节信息到字符串这一步由于采用的字符集不同导致得到的信息失真。在后端如果按照传输过来的数据及其编码来解读的话，将会得到B（ISO-8859-1），为了得到A（utf-8），只需要把字符集从ISO-8859-1更换成utf-8即可



#### 方案一（不推荐）：

得到B（ISO-8859-1），再通过ISO-8859-1编码方式得到B的字节（也就是A的字节，因为字节不变），更换编码方式为utf-8

```java
String name = new String(request.getParameter("name").getBytes("ISO-8859-1"),"utf-8");
```

这里，String 的getBytes（decode）是为了得到字符串的原字节信息，以便重新指定编码

#### 方案二（推荐）：

```java
request.setCharacterEncoding("UTF-8");
```

这里将直接设置数据的字符集（不改变信息字节），不过值得注意的是，**这句一定要在获取数据前设置才有效，request一旦获取过任何数据后，这种设置就无效了**



猜想：如果客户端发送过来的信息含有编码信息就好了，服务端就能根据编码信息进行解码。因为服务端传回数据给客户端时，就带有编码信息。出现乱码只是因为默认的编码不支持中文而已，详见HttpServletResponse的中文乱码问题



### 请求转发

#### 定义

请求转发是指，服务器收到请求后，从一个资源跳转到另一个资源的操作叫请求转发。请求转发不能理解为函数或对象的调用，这些或许是实现途径，真正应该理解为带着request和response去访问另一个资源，这才是最终目的

![image-20210106155259958](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210106155259958.png)

#### 实现

* 步骤1：先获取带有目标资源路径的 RequestDispatcher 对象，也就是请求转发对象，这可以通过HttpServletRequest对象完成，如下

  ```java
  //获取请求转发对象，里面封装了资源路径（不一定是Servlet对象，也可以是其他资源，如html），以便访问
  RequestDispatcher requestDispatcher = request.getRequestDispatcher("/servlet2");
  ```

* 步骤2：通过 RequestDispatcher 对象，也就是请求转发对象去访问资源

  ```java
  //通过请求转发对象，访问另一个资源（不一定是Servlet对象，也可以是其他资源,如html）
  requestDispatcher.forward(request,response);
  ```

  

#### 特点

* 1、浏览器地址栏没变化

* 2、它们是一次请求

* 3、它们共享Request中的域数据 

* 4、可以转发访问WEB-INF目录下的资源，这些资源不能通过浏览器直接访问，但可以通过请求转发间接访问

* 5、不可以访问工程以外的资源，例如把请求转发对象的路径初始化为 http://www.baidu.com  但实际访问时，真正的路径是   http://ip:port/工程名/http://www.baidu.com  意味着只能访问内部资源，因此这个百度路径是无法打开的

  ​        请求转发的路径是从当前工程名开始，无论用什么路径初始化请求转发对象，最终其完整的路径都是：http://ip:port/工程名/你的初始化路径

### request的 parameter 与 attribute

​		二者都是存放数据的map，不同的是parameter只能存放客户端传递过来的参数，一般是（String，String）类型、或者（String，String[] ）类型，后端只能取数据。而attribute能存放（String，Object）类型，且后端可以取数据，也可以存数据



### base 标签的作用

​		当我们点击页面的a标签进行跳转时，如果使用到相对路径，那么浏览器将参考地址栏的地址进行跳转，如果地址栏地址确实就是资源路径，那么没什么问题。但如果地址栏地址不是资源路径（如在请求转发的情况下地址栏的就是直接请求的地址，而非请求转发后资源的地址），那么这种跳转往往跳到错误的地方去

​		为了解决这个问题，我们需要一个固定的参考系，也就是说，无论浏览器地址栏的路径到底是什么，点击a标签跳转时，将固定参考某个路径（可以设置为资源自身的路径），这就是base标签的作用

​		**简而言之，base标签就是a标签跳转的参考系，如果存在base标签，那么a标签通过相对地址的跳转将以base标签的地址为参考系，而不是以浏览器地址栏路径为参考系**

​		以base标签为参考系能够使得按相对地址的跳转不受浏览器地址栏地址的影响，这样可以避免当资源路径与浏览器地址栏的路径不同时（例如请求转发送得到的资源，其路径与地址栏路径不同）发生的跳转错误

​		注意：

* base标签最多具体到文件夹，否则就是出错，因此，base标签href属性值最后必须是 **/** 这个符号 ，否则认为最后是个文件  
* 使用base标签后，按照相对路径或者绝对路径的方式跳转即可，注意相对路径不能以斜杠/ 开头，因为页面相对路径只能以 点（.）或者点点（..）或者路径名（xxx）或者文件名（xxx.yyy）开头，servlet里的路径才能以斜杠开头，表示http://ip:port/工程名/

## HttpServletResponse 类

### 作用

​		HttpServletResponse 类和 HttpServletRequest 类一样。每次请求进来，Tomcat 服务器都会创建一个Response 对象传递给 Servlet 程序去使用。HttpServletRequest 表示请求过来的信息，HttpServletResponse 表示所有响应的信息

​		我们如果需要设置返回给客户端的信息，都可以通过 HttpServletResponse 对象来进行设置

### 两个输出流

#### 概况

* 字节流    	getOutputStream(); 	常用于下载（传递二进制数据） 
* 字符流 		getWriter(); 		常用于回传字符串（常用）

#### 结构

![HttpServletResponse与它的两个输出流](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/HttpServletResponse与它的两个输出流.jpg)

#### 注意

response的两个流同时只能使用一个。 使用了字节流，就不能再使用字符流，反之亦然，否则就会报错。



如下是使用两个流的代码及其报错信息

```java
 response.getWriter();
        response.getOutputStream();
```

![image-20210106224651740](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210106224651740.png)

#### 字节流与字符流的区别

​		字节流是直接处理信息的字节数据，至于这些字节数据表示什么信息则不关心。读取时返回字节数组，输出时把字节信息直接输出

​		字符流则把信息的字节数据看做是字符（字节+隐含编码），读取时，把这些字符读入，写时，把这些字符写出（字节+隐含编码）

​		这里有个比较容易迷惑的地方就是，java里，字符最终还是由二进制构成的字节组成，因此字符流和字节流是不是没有区别呢？

​		非也，字符底层不单单只有字节构成，还有类型（默认utf-8编码），二者结合才确定这个字节是字符，总之，我们只需要知道java里面有一套确定字符的方法即可，置于它怎么确定的，这里不需要关系。不然这个字节也可以是整数、浮点数等。

​		因此，字符流是专用于处理字符的，即把信息当做字符来输入输出。而信息上的字符和java里面字符的实现无需关心，只要字符本身能正确输入输出即可

​		Java字符 <= 字符流 => 信息字符



### 往客户端回传数据

```java
public class ResponseIOServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //往客户端回传 字符串 数据
        PrintWriter writer = response.getWriter();
        writer.write("response's content!!! ");
    }
}
```



### 响应的中文乱码解决

​		当服务器通过 response 给客户端返回信息时，如果出现中文，在浏览器上面就会出现乱码。因为response默认采用ISO-8859-1编码，但这个编码不支持中文，即中文无法被编码出来，即使设置浏览器同为ISO-8859-1也无法解决问题。要解决这个问题只能用另一种编码代替，例如utf-8。有两种方案：

#### 方案一（不推荐）

步骤一：设置返回数据编码为 utf-8

步骤二：设置响应头的 contentType 键对应的值为 text/html；charset=utf-8。 目的是告诉浏览器这是文本内容，且编码格式是utf-8 



代码如下：

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        //设置内容编码格式
        response.setCharacterEncoding("utf-8");

        //设置响应头部Content-Type，告诉浏览器这是什么内容，什么编码
        response.setHeader("Content-Type","text/html;charset=utf-8");
        
        //往客户端回传 字符串 数据
        PrintWriter writer = response.getWriter();
        writer.write("国哥很帅!!! ");
    }
```

#### 方案二（推荐）

步骤一：设置ContentType即可   //这一步将完成方案一中两步的效果,即既设置了服务端的返回数据编码，也设置了响应头的ContentType键

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //设置ContentType即可
        response.setContentType("text/html;charset=utf-8");

        //往客户端回传 字符串 数据
        PrintWriter writer = response.getWriter();
        writer.write("国哥很帅!!! ");
    }
```

**注意：此方法需要在获取流之前使用才能生效，否则无效**

这两种方案在设置服务端的编码同时，也需要通过响应头告诉客户端自己的编码，否则客户端不知道从而采用不同的编码导致信息失真



### 请求重定向

#### 定义

​		请求重定向，是指客户端给服务器发请求，然后服务器告诉客户端说。我给你一些地址。你去新地址访问。浏览器收到新地址后，就按照新地址访问资源。这就是请求重定向（因为之前的地址可能已经被废弃）

![image-20210107215643912](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210107215643912.png)

**注意：请求重定向的地址也可以是斜杠开头，开头的斜杠表示 http://ip:port/  ，也可以不以斜杠开头直接就是完整路径。而请求转发只能以斜杠开头（不能访问工程外的数据），且斜杠表示http://ip:port/ 工程名/  ，这是二者的不同。一般请求重定向地址：request.getContextPath()+"xx/yy/..."**



#### 实现

##### 方案一（不推荐）

步骤一：设置响应行 302状态码，表示重定向

步骤二：设置响应头的 Location 键值对，用于表示重定向到的新地址

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    System.out.println("曾到此一游，response1");

    //设置302状态码，表示重定向
    response.setStatus(302);

    //设置响应头，说明新的地址
    response.setHeader("Location","http://localhost:8080/07_servlet/response2");
}
```

##### 方案二（推荐）

步骤一：直接sendRedirect即可。  这一步就完成了方案一中两步的内容，包括设置响应行的302响应码，与响应头的Location键值对

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("曾到此一游，response1");

        response.sendRedirect("http://localhost:8080/07_servlet/response2");
    }
```



**注意：请求重定向的地址也可以是斜杠开头，开头的斜杠表示 http://ip:port/  ，也可以不以斜杠开头直接就是完整路径。而请求转发只能以斜杠开头（不能访问工程外的数据），且斜杠表示http://ip:port/ 工程名/  ，这是二者的不同。一般请求重定向地址：request.getContextPath()+"xx/yy/..."**





#### 特点

* 1、浏览器地址会发生变化
* 2、两次请求
* 3、不共享Request域中的数据（因为是两次请求，两次的request是不同的，它们之间是独立的，数据不会共享）
* 4、可以访问工程以外的路径（这点与请求转发不同。请求转发只能访问工程内部资源。请求转发的路径是从当前工程名开始，无论用什么路径初始化请求转发对象，最终其完整的路径都是：http://ip:port/工程名/你的初始化路径）



## web中斜杠的意义

在 web 中 / 斜杠 是一种绝对路径



/ 斜杠 如果被浏览器解析，得到的地址是：http://ip:port/ 

```html
< a href="/"> 斜杠</a> 
```



/ 斜杠 如果被服务器解析，得到的地址是：http://ip:port/工程路径 

1、<**url-pattern**>/servlet1</**url-pattern**> 

2、servletContext.getRealPath(“/”); 

3、request.getRequestDispatcher(“/”); 



特殊情况： response.sendRediect(“/”); 

把斜杠发送给浏览器解析。得到 http://ip:port/ 

## 域对象

Servlet程序涉及的域对象有：HttpRequest对象（用于请求转发时数据共享）、ServletContext（用于不同Servlet对象间数据共享）。域对象主要通过 attribute 这个map进行数据的读写（getAttribute、setAttribute）,当然域对象本身还有 parameter这个map，一般parameter都只能读、不能写

​    另外，jsp对应的servlet程序中，还有PageContextImpl  对象、HttpSession 对象也是域对象

# http协议

所谓 HTTP 协议，就是指，客户端和服务器之间通信时，发送的数据，需要遵守的规则，叫 HTTP 协议。 HTTP 协议中的数据又叫报文

http传递参数时，一般是String类型，即name=value中，name与value都要是String类型。当然value也可以是文件。Servlet接受时，request的getParameter需要String类的name，返回String类的value

## 请求的http协议

客户端给服务器发送数据叫请求。 服务器给客户端回传数据叫响应。 

请求又分为 GET 请求，和 POST 请求两种

### get请求

1、请求行 

​	(1) 请求的方式 		GET 

​	(2) 请求的资源路径 [+?+请求参数] 

​	(3) 请求的协议的版本号 	HTTP/1.1 

2、请求头 

​	key : value  	组成 	不同的键值对，表示不同的含义

具体请看下图，其中User-Agent表示用户代理，也即浏览器相关信息

![image-20201227215449693](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201227215449693.png)



### post请求

1、请求行

​	(1) 请求的方式 		POST 

​	(2) 请求的资源路径

​	(3) 请求的协议的版本号 	HTTP/1.1 

2、请求头

​	1) key : value   不同的请求头，有不同的含义 

   **空行** 

3、请求体 ===>>> 就是发送给服务器的数据

具体请看下图，其中User-Agent表示用户代理，也即浏览器相关信息

![image-20201228152643392](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201228152643392.png)



### get请求与post请求的差异

* get请求传递的数据直接在请求的资源路径后面 如：资源路径+?+请求参数；而post请求的数据则放在请求体



### 常用的请求头

​	Accept: 表示客户端可以接收的数据类型 

​	Accpet-Languege: 表示客户端可以接收的语言类型 

​	User-Agent: 表示客户端浏览器的信息 

​	Host： 表示请求时的服务器 ip 和 端口号

​	referer：表示发起请求的页面地址。本质是指发起请求时浏览器窗口的地址。注意不是请求的目的地址，而是发起请求的源地址



### 常见的get请求与post请求

GET 请求： 

​	1、form 标签 method=get 

​	2、a 标签 

​	3、link 标签引入 css 

​	4、Script 标签引入 js 文件 

​	5、img 标签引入图片 

​	6、iframe 引入 html 页面 

​	7、在浏览器地址栏中输入地址后敲回车

POST 请求：

​	8、form 标签 method=post



### get请求与post请求的中文乱码问题

* 当使用get请求时，对于参数，浏览器默认用基于utf-8字符集的url编码，而服务器也是默认基于utf-8字符集的url解码，因此不会出现中文乱码问题
* 当使用post请求时，对于参数，浏览器默认用基于utf-8字符集的url编码，而服务器也是默认基于ISO-8859-1字符集的url解码，因此会中文乱码问题，解决方法见<a href="#获取数据的中文乱码问题">获取数据的中文乱码问题</a>  



## 响应的http协议

### 响应格式

1、响应行 

​	(1) 响应的协议和版本号 

​	(2) 响应状态码 

​	(3) 响应状态描述符 

2、响应头 

​	(1) key : value 	不同的响应头，有其不同含义 

​	**空行** 

3、响应体 	---->>> 	就是回传给客户端的数据

具体看下图：

![image-20201228160756279](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201228160756279.png)

### 常见响应吗

​	200 	表示请求成功 

​	302 	表示请求重定向（明天讲） 

​	404 	表示请求服务器已经收到了，但是你要的数据不存在（请求地址错误） 

​	500 	表示服务器已经收到请求，但是服务器内部错误（代码错误）

### MIME 类型说明

​	MIME 是 HTTP 协议中数据类型

​	MIME 的英文全称是"Multipurpose Internet Mail Extensions" 多功能 Internet 邮件扩充服务。MIME 类型的格式是“大类型/小 类型”，并与某一种文件的扩展名相对应

常见的 MIME 类型：

![MIME1](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/MIME1.PNG)

![MIME2](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/MIME2.PNG)

## 浏览器查看http信息

### ie浏览器

安装HttpWatch，在菜单栏选择启动或停止 HttpWatch

### Chrome浏览器

F12，选择 Network，具体如下图

其中，Response Header 、 Request Header 都办含了相应的 **请求行/响应行** 与 **请求头/响应头**，而非仅仅字面上的头部

![image-20201228191809818](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201228191809818.png)

### FireFox浏览器

同chrome

其中，Response Header 、 Request Header 都办含了相应的 **请求行/响应行** 与 **请求头/响应头**，而非仅仅字面上的头部

![image-20201228192343495](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20201228192343495.png)

# 数据库JDBC

## JDBC

​		JDBC是一套数据库连接规范，旨在编写数据库应用程序时有统一的接口而无需依赖特定数据库的api。达到“ 一次开发，适用所有数据库”。虽然实际开发中，经常会因为使用了数据库特定的语法、数据类型或函数等而无法达到目标，但JDBC的标准还是大大简化了开发工作。

​		而JDBC驱动则是JDBC规范中驱动接口的具体实现，不同厂商按照自己的DBMS去实现驱动接口，从而为上层提供统一的访问接口。JDBC规定了这个接口为Driver类，且加载一个Driver类时，必须在初始化阶段主动把自己注册到DriverManager类里面。

​       通过mysql的jdbc驱动连接数据库，具体见[JDBC连接逻辑](F:\研究生学习资料\流程图\数据库相关\JDBC连接逻辑.jpg)

![JDBC连接逻辑](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/JDBC连接逻辑-1613710330784.jpg)

**注意：只有Driver类是厂商提供的，其他的相关类/接口都是来自java.sql这个包，属于java标准类库**



## 数据库连接池

​		为了提高性能，避免频繁创建数据库连接，使用到数据库连接池技术

​		数据连接池的本质作用是提供和回收Connection 对象：通过druidDataSource.getConnection() 得到connection，通过connection.close（）回收connection。

​		注意这里Connection只是一个接口，java标准库提供的Connection对象close会关闭连接，但`druid-1.1.9.jar` 包提供的Connection对象close只是回归数据库连接池druidDataSource，并非关闭连接。

### 使用前提

​		需要引入数据库连接池包 `druid-1.1.9.jar` 与 数据库驱动包 `mysql-connector-java-5.1.7-bin.jar`  前者一般是阿里的数据库连接池包，后者是JDBC连接数据库需要的厂商驱动，这里是mysql驱动，如果需要连接其他数据库，那就需要其他的驱动包

### 创建数据库连接池

   具体看下图

![数据库连接池](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/数据库连接池-1610440547109.jpg)

代码如下：

```java
    private static DruidDataSource dataSource;//数据库连接池

    static {
        try {

            //读取jdbc.properties配置文件，得到字节流
            InputStream inputStream=JdbsUtils.class.getClassLoader().getResourceAsStream("jdbc.properties");

            //创建Properties对象， 从流中加载数据
            Properties properties=new Properties();
            properties.load(inputStream);

            //创建数据库连接池
            dataSource= (DruidDataSource) DruidDataSourceFactory.createDataSource(properties);

        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

### 使用数据库连接池

通过 DruidDataSource 对象（数据库连接池对象）获取connection，用完后，关闭connection，具体代码如下：

```java
public class JdbcUtils {

    private static DruidDataSource dataSource;//数据库连接池

    static {
       //创建数据库连接池
    }

    /**
     * 获取数据库连接池中的连接
     * @return 如果返回null，说明获取连接失败<br/> 有值就是获取连接成功
     */
    public static Connection getConnection(){
        Connection connection=null;
        try {
            connection=dataSource.getConnection();
        }catch (Exception e){
            e.printStackTrace();
        }
        return connection;
    }

    /**
     * 关闭连接,返回数据库连接池
     * @param connection
     */
    public static void close(Connection connection){
        if(connection!=null){
            try {
                connection.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

}
```

### 关于Connection的close方法

* 通过 DriverManager获取的 Connection 对象，在调用 close（）方法后，会关闭与数据库的 TCP 连接
* 而数据库连接池中的Connection对象，执行 con.close 并不会关闭与数据库的 TCP 连接，而是将连接还回到池中去，如果不 close 掉的话，这个连接将会一直被占用，直接连接池中的连接耗尽为止

​		**这是由于Connection是个标准类库的接口，可以有不同实现。在数据库连接池中，它的close方法实现为将连接还回到池中去，而非真正关闭TCP连接。它的其他方法与 DriverManager获取的 Connection 对象差不多**



### 注意

* 使用数据库连接池得到Connection，如果调用关闭后是不能再用来执行sql的，否则报错：java.sql.SQLException: connection holder is null。这应该是数据库连接池对连接的管理，要使用是能重新通过数据库连接池获取





## 数据库连接池与DBUtils

数据库连接池主要是获取和回收connection，也就是管理 JDBC 的connections；DBUtils工具包主要是用来执行sql语句的（查询时把结果放到bean里面），需要用到connection



# JavaEE

## JavaEE的三层架构

![image-20210108221628897](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210108221628897.png)

分层的目的是为了解耦。解耦就是为了降低代码的耦合度。方便项目后期的维护和升级

web 层    	com.atguigu.web/servlet/controller 

service 层   	com.atguigu.service 	Service 接口包 	

​						com.atguigu.service.impl 	Service 接口实现类 

dao 持久层 	com.atguigu.dao 	Dao 接口包 

​						com.atguigu.dao.impl 	Dao 接口实现类 

实体 bean 对象 	com.atguigu.pojo/entity/domain/bean 	JavaBean 类 

测试包 		com.atguigu.test/junit 

工具类 		com.atguigu.utils



## JavaWeb项目模块

![JavaWeb项目模块](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/JavaWeb项目模块-1610633823640.jpg)



## Dao类（DBUtils工具）

### 数据库交互工具DBUtils

​		Dao类主要用于与数据库交互，如果只使用JDBC进行开发，会发现冗余代码过多，为了简化JDBC开发，采用apache commons组件一个成员：DBUtils工具

​		DBUtils是java编程中的数据库操作实用工具，小巧简单实用

​		DBUtils封装了对JDBC的操作，简化了JDBC操作，可以少写代码

​		Dbutils三个核心功能：

​				**1、QueryRunner中提供对sql语句操作的API**

​				**2、ResultSetHandler接口，用于定义select操作后，怎样封装结果集**

​				**3、DbUtils类，它就是一个工具类,定义了关闭资源与事务处理的方法**

​		总的来说，DBUtils是建立在JDBC上的一套用于对数据库进行交互的工具，更具体地说，是建立在JDBC的Connection之上的对数据库的Bean交互

​		**本质上是通过QueryRunner执行sql语句，需要我们提供connection**



为了使用DBUtils工具，需要引入 `commons-dbutils-1.3.jar` 这个jar包

QueryRunner 类对象对数据库的增删改查模式如下：

```java
    //使用DbUtils操作数据库
    private static QueryRunner queryRunner=new QueryRunner();

    public static void main(String[] args) {
        try {
            Connection conn= JdbcUtils.getConnection();
            String sql="xxx";

            //增删改
            int affectedRowsNum=queryRunner.update(conn,sql,args);

            //查一行,User是Bean类，可以根据需要更换别的Bean
            User user= queryRunner.query(conn,sql,new BeanHandler<User>(User.class),args);

            //查多行,User是Bean类，可以根据需要更换别的Bean
            //注意，BeanListHandler<X> implements ResultSetHandler<List<X>>，因此调用时，query方法的T实际上是List<X>
            List<User> userList= queryRunner.query(conn,sql,new BeanListHandler<User>(User.class),args);

            //查某个值（实际返回数据类型视数据库相应数据类型而定，如果数据库数据是string，那么返回数据就是String，如果数据库数据是int，那么返回数据就是Integer）
            Object obj=queryRunner.query(conn,sql,new ScalarHandler(),args);

        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

​		注意：args，可变长参数，主要是对sql语句的补充，args参数的元素将会替换sql的占位符“？”从而形成真正的sql语句。注意args的参数只需要按照相应类型写即可，例如sql语句："select id from t_user where username=?"  args为："wzg168" 即可，无需 " 'wzg168' "。**args参数的类型本身将会被解读成sql语句里面的参数的类型**

​		注意：**通过jdbc查取的数据全是String类型，而queryRunner把信息保存到 bean 里面会根据 bean 相应字段进行数据类型转换（主要是通过反射获取bean的所有set方法，按照查到的数据的字段名把数据set进bean里面）**





### BaseDao（基于BDUtils）

​		使用BaseDao的理由：

* 默认基于某个数据库：这样，就不用每次使用DBUtils时指定数据库连接connection了

* bean通用：一般来说，我们如果需要不同的 Bean 去存取数据库，那么就得写不同的 xxDao，而这些不同的xxDao的实现代码除了用于与数据库交互的 Bean 不同之外，其他操作模式都高度相似，为了较少重复代码。我们可以写一个对数据库进行增删改查的 BaseDao,它里面的方法是泛型方法，不同Bean的情况下都可以使用它。这样，这个BaseDao相当于一个Bean通用的Dao，而xxDao这些Bean专用的Dao就可以使用通用的BaseDao实现自己的功能了

* 实现基本增删改查：这样xxDao就可以利用这些基本操作实现数据库交互

  

​		**BaseDao的增删改查模式，方法入口是这样的（bean类型，sql语句，args参数）**

​		可以把BaseDao设置成抽象类，这样专用的xxDao可以继承它，使用它的方法

具体使用如下：

  注意，代码里的 sql 字符串有占位符 '?'，需要用 args里面的参数替换掉这些占位符后才能得到真正的 sql 语句 ，**args参数的类型本身将会被解读成sql语句里面的参数的类型**

```java
package com.atguigu.dao.impl;

import com.atguigu.utils.JdbcUtils;
import com.sun.media.sound.RIFFInvalidDataException;
import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.apache.commons.dbutils.handlers.ScalarHandler;

import java.net.ConnectException;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.List;

public  abstract class BaseDao {

    //使用DbUtils操作数据库
    private QueryRunner queryRunner=new QueryRunner();

    /**
     * update() 方法用来执行insert into、delete from、 update 语句
     * @return 如果返回-1，表示执行失败<br/> 返回其他表示影响行数
     */
    public int update(String sql,Object... args){
        Connection connection= JdbcUtils.getConnection();
        try {
            return queryRunner.update(connection,sql,args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            JdbcUtils.close(connection);
        }
        return -1;
    }

    /**
     * 查询返回一条javaBean的sql语句
     * @param type 返回的对象类型
     * @param sql  执行的sql语句
     * @param args sql对应的参数值
     * @param <T> 返回的类型的泛型
     * @return
     */
    public <T> T queryForOne(Class<T> type, String sql, Object ... args){
        Connection conn=JdbcUtils.getConnection();
        try {
            return queryRunner.query(conn,sql,new BeanHandler<T>(type),args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            JdbcUtils.close(conn);
        }
        return null;
    }

    /**
     * 查询返回多个javaBean的sql语句
     * @param type 返回的对象类型
     * @param sql  执行的sql语句
     * @param args sql对应的参数值
     * @param <T> 返回的类型的泛型
     * @return
     */
    public <T> List<T> queryForList(Class<T> type, String sql, Object ... args){
        Connection conn=JdbcUtils.getConnection();
        try {
            return queryRunner.query(conn,sql,new BeanListHandler<T>(type),args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            JdbcUtils.close(conn);
        }
        return null;
    }

    /**
     * 执行返回一行一列的某个值的sql语句
     * @param sql 执行的sql语句
     * @param args sql对应的参数值
     * @return
     */
    public Object queryForSingleValue(String sql, Object ... args){
        Connection con=JdbcUtils.getConnection();
        try {
            queryRunner.query(con,sql,new ScalarHandler(),args);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return null;
    }

}
```

### xxDao（如UserDao）

专用的Dao，通过继承可以利用通用的BaseDao实现业务需要的数据读写，具体代码如下：

```java
package com.atguigu.dao.impl;

import com.atguigu.dao.UserDao;
import com.atguigu.pojo.User;

public class UserDaoImpl extends BaseDao implements UserDao {


    @Override
    public User queryUserByUsername(String username) {
        String sql="select id, username,password,email from t_user where username=?";
        return queryForOne(User.class,sql,username);
    }

    @Override
    public User queryUserByUsernameAndPassword(String username, String password) {
        String sql="select id, username,password,email from t_user where username=? and password=?";
        return queryForOne(User.class,sql,username,password);
    }

    @Override
    public int saveUser(User user) {
        String sql="insert into t_user (username, password, email) VALUES (?,?,?)";
        return update(sql,user.getUsername(),user.getPassword(),user.getEmail());
    }
}
```



### DBUtils、BaseDao与xxDao

具体关系如下图所示：

![DBUtils、BaseDao与 xxDao](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/DBUtils、BaseDao与 xxDao.jpg)



## 代码优化

### 代码优化一：合并 Servlet 程序

​		在实际的项目开发中，一个模块，一般只使用一个 Servlet 程序。这里我们合并 LoginServlet 和 RegistServlet 程序为 UserServlet 程序

![image-20210207225308620](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210207225308620.png)



### 优化代码二：使用反射优化大量 else if 代码

​		优化一中在业务比较多的时候就会存在大量 if else ，不利于可读性与可维护性。我们可以通过反射来调用方法，从而替换掉大量if else的逻辑判断

```java
    protected void doPost(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
        String action = request.getParameter("action");//得到页面标识（我们设置了对应的方法名就是页面标识）

        try {
            //获取方法
            Class requestClass= javax.servlet.http.HttpServletRequest.class;
            Class responseClass=javax.servlet.http.HttpServletResponse.class;


            Method method=this.getClass().getDeclaredMethod(action,requestClass, responseClass);

            //调用方法
            method.invoke(this,request,response);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

注意：参数类型的Class对象用全类名比用简单类名要稳妥，有时候简单类名可能被认为是另一个类



### 代码优化三：抽取 BaseServlet 程序

​		把公共部分抽取出来，放到BaseUtils里面，别的类直接继承即可，不需要再重写公共逻辑。上述的公共逻辑就是doPost里面的逻辑（获取action、反射获取处理方法、反射调用处理方法）

```java
    protected void doPost(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
        String action = request.getParameter("action");
        System.out.println(action);

        try {
            //获取方法
            Class requestClass= javax.servlet.http.HttpServletRequest.class;
            Class responseClass=javax.servlet.http.HttpServletResponse.class;


            Method method=this.getClass().getDeclaredMethod(action,requestClass, responseClass);

            //调用方法
            method.invoke(this,request,response);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```



## 数据的封装和抽取（BeanUtils的使用）

BeanUtils 工具类，它可以一次性的把所有请求的参数注入到 JavaBean 中

BeanUtils 工具类，经常用于把 Map 中的值注入到 JavaBean 中，或者是对象属性值的拷贝操作



BeanUtils 它不是 Jdk 的类。而是第三方的工具类。所以需要导包：

​		1、导入需要的 jar 包： 

​				commons-beanutils-1.8.0.jar 

​				commons-logging-1.1.1.jar	

​	

​		2、编写 WebUtils 工具类使用：

```java
   public class WebUtils {

    /**
     * 把map的值注入到bean属性中
     * @param map
     * @param bean
     * @throws InvocationTargetException
     * @throws IllegalAccessException
     */
    public static <T> T copyParamToBean(Map<String,String[]> map, T bean) throws InvocationTargetException, IllegalAccessException {

        System.out.println("注入之前："+bean);

        /**
         * 把所有请求参数注入到User对象中(参数Map存放(name,value)键值对，设置User的状态遍历)
         */
        BeanUtils.populate( bean,map );
        System.out.println("注入之后："+bean);
        return bean;
    }

}
```

​		关键就是 BeanUtils.populate( bean,map )  这个就是通过BeanUtils把参数注入bean。一般通过request.getParameterMap()  获取参数map。这里bean作为一个Object，注入时是通过反射使用setter来把数据写入bean的属性的

​		注意：**把数据注入bean时，会根据bean的属性类型做出相应的类型转换后再set进去（主要是通过反射获取bean的所有set方法，按照查到的数据的字段名把数据set进bean里面）**



使用代码：

```java
User user= WebUtils.copyParamToBean(request.getParameterMap(),new User());
```



## MVC 概念

MVC 全称：Model 模型、 View 视图、 Controller 控制器



MVC 最早出现在 JavaEE 三层中的 Web 层，它可以有效的指导 Web 层的代码如何有效分离，单独工作

* View 视图：只负责数据和界面的显示，不接受任何与显示数据无关的代码，便于程序员和美工的分工合作—— JSP/HTML

* Controller 控制器：只负责接收请求，调用业务层的代码处理请求，然后派发页面，是一个“调度者”的角色——Servlet

  ​	转到某个页面。或者是重定向到某个页面。 

* Model 模型：将与业务逻辑相关的数据封装为具体的 JavaBean 类，其中不掺杂任何与数据处理相关的代码——  JavaBean/domain/entity/pojo



**MVC是一种思想** 

MVC 的理念是将软件代码拆分成为组件，单独开发，组合使用（**目的还是为了降低耦合度**）

![image-20210209122942470](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210209122942470.png)

MVC 的作用还是为了降低耦合。让代码合理分层。方便后期升级和维护



## 页面



### 分页

#### 分页信息

​		分页的关键信息有三个：totalSize（总记录数）、pageSize（每页记录数）、pageNo（当前页码）。前二者决定页面分布，当前页码则决定取哪一页

​		页面展示信息则包含：totalSize（总记录数）、pageSize（每页记录数）、pageNo（当前页码）、totalPage（总页数），items（该页信息）

​		其中，totalPage（总页数）虽然是可由关键三项信息推出的冗余信息，但展示在页面有利于读者对分页有着更直观的感受

​		![分页信息](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/分页信息.jpg)



#### 分页逻辑

![分页逻辑](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/分页逻辑.jpg)

### 页面访问逻辑

​		页面访问逻辑（例如分页、更新/添加页面）是javaWeb的基石，页面的直接呈现或间接呈现（例如删除数据后呈现页面）都需要用到页面的访问逻辑

​		当一个jsp页面涉及到多种访问时，需要调整使之能适应从而能满足每种访问要求



### 页面base

​		为了方便操作的统一性，页面跳转所基于的base标签的地址最好设置为：http://ip:port/工程路径/



### 抽取公共页面

​		如果多个jsp页面含有相同部分，那么可以抽取出来作为一个公共页面，这些页面通过使用静态包含把公共页面包含进去



## 免用户名登录与登录后用户信息的保存

* 免用户名登录：登录成功后把用户名信息保存在cookie里，每次登录时由客户端发送给服务器，填充在登录界面上从而实现免用户名登录
* 登陆后用户名信息保存：保存在session的域中，访问页面时可以根据域中是否含有用户对象来判断用户是否已经登录



## 谷歌验证码

### 使用步骤

谷歌验证码 kaptcha 使用步骤如下： 

​		1、导入谷歌验证码的 jar 包 

​				kaptcha-2.3.2.jar 

​		2、在 web.xml 中去配置用于生成验证码的 Servlet 程序

```xml
    <servlet>
        <servlet-name>KaptchaServlet</servlet-name>
        <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>KaptchaServlet</servlet-name>
        <url-pattern>/kaptcha.jpg</url-pattern>
    </servlet-mapping>
```



3、在表单中使用 img 标签去显示验证码图片并使用它

```jsp
    <form action="registServlet" method="get">
        用户名：<input type="text" name="username"> <br>
        验证码：<input type="text" name="code" style="width: 80px">
        <img src="kaptcha.jpg" style="width: 100px; height: 28px"> <br>

        <input type="submit" value="登录">
    </form>
```



4、在服务器获取谷歌生成的验证码和客户端发送过来的验证码比较使用

```java
 import static com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY;

public class RegistServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取session中的验证码
        String token = (String) request.getSession().getAttribute(KAPTCHA_SESSION_KEY);

        //删除session中的验证码
        request.getSession().removeAttribute(KAPTCHA_SESSION_KEY);

        //获取表单验证码
        String code=request.getParameter("code");


        //获取用户名
        String username = request.getParameter("username");

        if(code!=null&&token.equals(code)){
            System.out.println("保存到数据库："+username);
        }else {
            System.out.println("请不要重复提交表单");
        }

        response.sendRedirect(request.getContextPath()+"/ok.jsp");
    }
}
```



**注意**：com.google.code.kaptcha.servlet.KaptchaServlet 在生成验证码图片时，会往session的域中保存对应的验证码，key为com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY对应值就是验证码值



### 验证码切换

对img标签绑定单击事件，每次单击时，对其src属性写一次，为了避免浏览器缓存，src值除了必要的servlet地址外可以加上每次不同的参数，这样浏览器不会因为相同的地址导致取缓存值



# jsp

## 定义

jsp 的全换是 java server pages。Java 的服务器页面

jsp 的主要作用是代替 Servlet 程序回传 html 页面的数据

因为 Servlet 程序回传 html 页面数据是一件非常繁锁的事情。开发成本和维护成本都极高

Servlet 回传 html 页面数据的代码：

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //通过响应的输出流回传html数据
        response.setContentType("text/html;charset=utf-8");

        PrintWriter writer = response.getWriter();

        writer.write("<!DOCTYPE html>\r\n");
        writer.write("<html lang=\"en\">\r\n");
        writer.write("<head>\r\n");
        writer.write("<meta charset=\"UTF-8\">\r\n");
        writer.write("<title>Title</title>\r\n");
        writer.write("</head>\r\n");
        writer.write("<body>\r\n");
        writer.write("这是html页面数据\r\n");
        writer.write("</body>\r\n");
        writer.write("</html>\r\n");
        
    }
```

试想一下，如果html页面很复杂，那么通过servlet程序回传就会特别麻烦

## jsp与html区别

html是静态页面，jsp是动态页面。

html完成后，页面内容不变，除非更改页面代码

而jsp 将会被服务器翻译成一个servlet程序，通过servlet程序回传jsp页面上的html数据。由于jsp本质上是一个servlet程序，而servlet程序里面可以通过java代码动态获取数据，因此不动人不同时间打开jsp看到的内容都不一样相同。例如张三登录成功显示“欢迎张三”，李四登录成功显示“欢迎李四”。这是html做不到的动态效果



## jsp页面的创建与访问

### 创建

![image-20210116220130316](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210116220130316.png)

输入文件名回车即可

![image-20210116220157943](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210116220157943.png)

### 访问

jsp 页面和 html 页面一样，都是存放在 web 目录下。访问也跟访问 html 页面一样。

web 目录

a.html 页面 		访问地址是 =======>>>>>> http://ip:port/工程路径/a.html 

b.jsp 页面 			访问地址是 =======>>>>>> http://ip:port/工程路径/b.jsp

## jsp的本质

### 本质是Servlet程序

jsp 页面本质上是一个 Servlet 程序

当我们第一次访问 jsp 页面的时候。Tomcat 服务器会帮我们把 jsp 页面翻译成为一个 java 源文件。并且对它进行编译成 为.class 字节码程序。我们打开 java 源文件不难发现其里面的内容是： 

![image-20210116221022549](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210116221022549.png)

​		我们跟踪原代码发现，HttpJspBase 类。它直接地继承了 HttpServlet 类。也就是说。jsp 翻译出来的 java 类，它间接了继 承了 HttpServlet 类。也就是说，翻译出来的是一个 Servlet 程序

​		HttpJspBase 类重写了HttpServlet的service方法，里面调用的是 _jspService（），而 _ jspService（）是在a_jsp这个用户的类实现的，注意这里调用无论是get还是post都是执行_jspService（）方法

![image-20210116225007937](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210116225007937.png)

​		总结：通过翻译的 java 源代码我们就可以得到结果：jsp 就是 Servlet 程序

​		也可以去观察翻译出来的 Servlet 程序的源代码，不难发现。其底层实现，也是通过输出流。把 html 页面数据回传给客户端

​		对jsp的理解应当看做它是为了生成servlet而写的，而非servlet是jsp的实现。因为jsp里面可以嵌入java代码，我们在写jsp的时候是从利用jsp构造servlet的角度去写的。不能脱离servlet去理解jsp，故不能看做servlet是jsp的实现

### HttpJspBase结构

![HttpJspBase结构](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/HttpJspBase结构.jpg)



## jsp的三种语法

### 一、头部page指令

jsp 的 page 指令可以修改 jsp 页面中一些重要的属性，或者行为

```jsp
<%@ page import="java.util.Map" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

i. language 属性    	   表示 jsp 翻译后是什么语言文件。暂时只支持 java 



ii. contentType 属性     	 表示 jsp 返回的数据类型是什么。也是源码 _jspService() 方法中 response.setContentType()参数值 



iii. pageEncoding 属性    	  表示当前 jsp 页面文件本身的字符集



iv. import 属性   	   跟 java 源代码中一样。用于导包，导类

========================两个属性是给 out 输出流使用============================= 

v. autoFlush 属性   	设置当 out 输出流缓冲区满了之后，是否自动刷新冲级区。默认值是 true



vi. buffer 属性 		设置 out 缓冲区的大小。默认是 8kb

![image-20210119230137405](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210119230137405.png)

========================两个属性是给 out 输出流使用============================= 

vii. errorPage 属性 		设置当 jsp 页面运行时出错，自动跳转去的错误页面路径。 

​     // *errorPage* *表示错误后自动跳转去的路径*  .*这个路径一般都是以斜杠打头，它表示请求地址为* *http://ip:port/**工程路径**/**映射到代码的* *Web* *目录* 。这是由于jsp文件最终还是被翻译成Servlet程序，因此需要按照Servlet程序的地址解析规则去写地址，例如  errorPage="/error500.jsp"



viii. isErrorPage 属性 	设置当前 jsp 页面是否是错误信息页面。默认是 false。如果是 true ，源代码就会出现异 											常对象，可以 获取异常信息

 

ix. session 属性 		设置访问当前 jsp 页面，是否会创建 HttpSession 对象。默认是 true

 

x. extends 属性 		设置 jsp 翻译出来的 java 类默认继承谁

### 二、jsp中的常用脚本

#### i. 声明脚本(极少使用)

​		声明脚本的格式是：	 **<%!**  声明 java 代码  **%>** 

​		作用：	可以给 jsp 翻译出来的 java 类定义属性和方法甚至是静态代码块、内部类等

```jsp
<%--1、声明类属性--%>
    <%!
        private Integer id;
        private String name;
        private static Map<String, Object> map;
    %>
<%--2、声明 static 静态代码块--%>
    <%!
        static {
            map = new HashMap<>();
            map.put("key1", "value1");
            map.put("key2", "value2");
            map.put("key3", "value3");
        }
    %>
<%--3、声明类方法--%>
    <%!
        public int abc() {
            return 12;
        }
    %>
<%--4、声明内部类--%>
    <%!
        public static class A {
            private Integer id = 12;
            private String abc = "abc";
        }
    %>
```

这些声明代码翻译到java类里面：

![image-20210121004556214](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210121004556214.png)

#### ii. 表达式脚本（常用）

表达式脚本的格式是：**<%=**表达式**%>** 

表达式脚本的作用是：的 jsp 页面上输出数据 （整型，浮点型，字符串，对象（toString方法的信息））

例子：

```jsp
   <%=12%> <br>
    <%=12.12%> <br>
    <%="我是字符串"%> <br>
    <%=cat%> <br>
    <%=request.getParameter("username")%>  <%--可以直接使用request，因为它来自 _jspService()--%>
```

表达式脚本的特点： 

​	1、所有的表达式脚本都会被翻译到 _jspService() 方法中 

​	2、表达式脚本都会被翻译成为 out.print(表达式) 输出到页面上 

​	3、由于表达式脚本翻译的内容都在 _jspService()  方法中,所以 _jspService() 方法中的对象都可以直接使用

​	4、表达式脚本中的表达式不能以分号结束，因为分号会带入待out.print（）当中，参考第2点



#### iii. 代码脚本

代码脚本的格式是： **<%** java 语句 **%>** 

代码脚本的作用是：可以在 jsp 页面中，编写我们自己需要的功能（写的是 java 语句）



代码脚本的特点是： 

​	1、代码脚本翻译之后都在_jspService 方法中 

​	2、代码脚本由于翻译到_jspService()方法中，所以在 _jspService()方法中的现有对象都可以直接使用。 

​	3、还可以由多个代码脚本块组合完成一个完整的 java 语句

​	4、代码脚本还可以和表达式脚本一起组合使用，在 jsp 页面上输出数据

注意：3与4使得代码脚本可以和前面的脚本或者html组合使用，非常灵活



例如：

```jsp
    <table border="1" cellspacing="0">
        <%
            for(int j=0;j<10;j++){
        %>
        <tr>
            <td>第 <%=j%> 行</td>
        </tr>
        <%
            }
        %>
    </table>
```

结果

![捕获](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/捕获-1611243749428.PNG)

这时候，只有从java源码的角度去理解jsp才比较好：

​    html是 源码里面的 out.write（html），即使是html注释也会被write给客户端

​    代码脚本则直接搬到源代码里面

​    表达式脚本就是 out.print（表达式） 

jsp就是这样一行一行转换到 java 源代码里面的



#### 几种脚本的比较

* 声明脚本用于类整体的属性、方法、内部类、静态代码块；而表达式脚本与代码脚本则仅仅是作用于_jspService()方法内



### jsp 中的三种注释

#### i. html 注释

格式：*<!--* *这是* *html* *注释* -->

html 注释会被翻译到 java 源代码中。在_jspService 方法里，以 out.writer 输出到客户端。



#### ii. java 注释

格式：//单行注释    、     /* 这是多行注释 */

java 注释会被翻译到 java 源代码中。 并不会被write给客户端



#### iii. jsp 注释

格式：*<%--* *这是* *jsp* *注释* *--%>*

jsp 注释可以注掉，jsp 页面中所有代码。是jsp页面真正的注释，而html注释和java注释对jsp来说都是有用的内容



## jsp九大内置对象

jsp 中的内置对象，是指 Tomcat 在翻译 jsp 页面成为 Servlet 源代码后，内部提供的九大对象，叫内置对象。

![image-20210123231827790](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210123231827790.png)

注意：exception对象需要 page 指令中 isErrorPage="true" 才能有

## jsp 四大域对象

四个域对象分别是： 

​		request 	(HttpServletRequest 类)、 	一次请求内有效 （请求转发后依然属于一次请求）



​		pageContext 	(PageContext 类) 	本次请求&&当前 jsp 页面 有效

​																	//（pageContext对象需要page页面与本次请求共同约束产生的） 



​		session 	(HttpSession 类)、 	一个会话范围内有效（打开浏览器访问服务器，直到关闭浏览器） 



​		application 	(ServletContext 类) 	整个 web 工程范围内都有效（只要 web 工程不停止，数据都在）



域对象是可以像 Map 一样存取数据的对象。四个域对象功能一样。不同的是它们对数据的存取范围



四个域在使用的时候，优先顺序分别是，他们从小到大的范围的顺序。 

​				pageContext ====>>> request ====>>> session ====>>> application

主要从节省内存角度考虑，不用的时候就释放，因此，作用域小的优先使用



## **jsp** 中的 out 输出和 response.getWriter 输出

response 中表示响应，我们经常用于设置返回给客户端的内容（输出） 

out 也是给用户做输出使用的

![image-20210125230422692](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210125230422692.png)

这种输出顺序是因为 out是JspWriterImpl对象，里面调用了response.getWriter进行输出。当out.write时，调用response.getWriter.write。当然由于设置缓冲区，out.write只有缓冲区满了才会调用response.getWriter.write，才会出现上述的输出顺序



由于 jsp 翻译之后，底层源代码都是使用 out 来进行输出，所以一般情况下。我们在 jsp 页面中统一使用 out 来进行输出。避 免打乱页面输出内容的顺序

​		out.write() 输出字符串没有问题 

​		out.print() 输出任意数据都没有问题（都转换成为字符串后调用的 write 输出）

深入源码，浅出结论：在 jsp 页面中，可以统一使用 out.print()来进行输出



## jsp 的常用标签

### jsp 静态包含

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 施汶
  Date: 2021/1/26
  Time: 22:36
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    
    头部信息 <br>
    主体内容 <br>
    <%--
        <%@include file=""%> 就是静态包含
        file属性指定你要包含的jsp页面路径

        地址中，第一个斜杠表示为 http://ip:port:工程名/ 映射到代码web目录
    --%>
    
    <%@include file="/include/footer.jsp"%>
    
</body>
</html>
```

*静态包含的特点：* 

​	1、静态包含不会翻译被包含的 *jsp* 页面

​	2、静态包含其实是把被包含的 *jsp* 页面的代码拷贝到包含的位置执行输出

​    本质上就是访问 jsp时，只会生成的一个servlet输出jsp页面，遇到静态包含时，就把另一个jsp页面也一起写出去

### jsp 动态包含

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 施汶
  Date: 2021/1/26
  Time: 22:36
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    
    头部信息 <br>
    主体内容 <br>

    <%--
      <jsp:include page="/include/footer.jsp"></jsp:include> 是动态包含
      page属性是指定你要包含的jsp页面路径
      动态包含也可以像静态包含一样。把被包含的内容执行输出到包含位置
    --%>

    <jsp:include page="/include/footer.jsp">
        <jsp:param name="username" value="bbj"/>
        <jsp:param name="password" value="123"/>
    </jsp:include>
    
</body>
</html>
```

动态包含的特点： 

​	1、动态包含会把包含的 jsp 页面也翻译成为 java 代码 

​	2、动态包含底层代码使用如下代码去调用被包含的 jsp 页面执行输出。 JspRuntimeLibrary.include(request, response, "/include/footer.jsp", out, false); 

​	3、动态包含，还可以传递参数（底层是通过request写入参数，被调用方可以通过request获取参数）



底层原理

![image-20210126232342145](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210126232342145.png)



### jsp 标签-转发

```jsp
    <%--
    下面两种方式都可以实现请求转发
    --%>

    <%
        request.getRequestDispatcher("/scope2.jsp").forward(request,response);
    %>

    <jsp:forward page="/scope2.jsp"></jsp:forward>
```



## Listener 监听器

* 1、Listener 监听器它是 JavaWeb 的三大组件之一。JavaWeb 的三大组件分别是：Servlet 程序、Filter 过滤器、Listener 监听器

* 2、Listener 它是 JavaEE 的规范，就是接口
* 3、监听器的作用是，监听某种事物的变化。然后通过回调函数，反馈给客户（程序）去做一些相应的处理。

### ServletContextListener 监听器

​	ServletContextListener 它可以监听 ServletContext 对象的创建和销毁。 

​	ServletContext 对象在 web 工程启动的时候创建，在 web 工程停止的时候销毁。 

​	监听到创建和销毁之后都会分别调用 ServletContextListener 监听器的方法反馈。

```java
public interface ServletContextListener extends EventListener {

   
   /**
     *在 ServletContext 对象创建之后马上调用，做初始化
     */
    public void contextInitialized(ServletContextEvent sce);

    /**
     *在 ServletContext 对象销毁之后调用
     */
    public void contextDestroyed(ServletContextEvent sce);
}
```

如何使用 ServletContextListener 监听器监听 ServletContext 对象。 

使用步骤如下： 

​	1、编写一个类去实现 ServletContextListener 

​	2、实现其两个回调方法 

​	3、到 web.xml 中去配置监听器



监听器实现

```java
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;

public class MyServletContextListenerImpl implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("ServletContext对象被创建了");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("ServletContext对象被销毁了");
    }
}
```

web.xml 中的配置

```xml
    <!--配置监听器-->
    <listener>
        <listener-class>com.atguigu.listener.MyServletContextListenerImpl</listener-class>
    </listener>
```



## 动态base标签

​		base标签动态获取使得工程可以部署在不同的服务器上而不需要修改base标签，因为可以动态获取到本服务器上的工程路径，用这个路径作为base标签的路径

```jsp
<%
    String basePath=request.getScheme()
            + "://"
            + request.getServerName()
            + ":"
            + request.getServerPort()
            + request.getContextPath()
            +"/"
            ;
%>

<base href="<%=basePath%>">
```





# EL表达式

## 定义和作用

​		EL 表达式的全称是：Expression Language。是表达式语言

​		EL 表达式的作用：EL 表达式主要是代替 jsp 页面中的表达式脚本在 jsp 页面中进行数据的输出 ，但是表达式脚本可以输出任意局部变量，但EL表达式只能访问特定的对象（参见EL表达式的11个隐含对象）

因为 EL 表达式在输出数据的时候，要比 jsp 的表达式脚本要简洁很多

```jsp
    <%
        request.setAttribute("key","值");
    %>
    表达式脚本输出key的值是：<%=request.getAttribute("key1")%><br>
    EL表达式输出key的值是：${key1}
```

EL 表达式的格式是：${表达式} 

**EL 表达式在输出 null 值的时候，输出的是空串。jsp 表达式脚本输出 null 值的时候，输出的是 null 字符串**



EL表达式本质上就是一个表达式，一般情况表示打印结果到页面，特殊情况表示取该表达式结果（参见JSTL的core核心库）



## EL 表达式搜索域数据的顺序

EL 表达式主要是在 jsp 页面中输出数据

主要是输出域对象中的数据

当四个域中都有相同的 key 的数据的时候，EL 表达式会按照四个域的从小到大的顺序去进行搜索，找到就输出

  pageContext ====>>> request ====>>> session ====>>> application

## EL 表达式输出属性 （值、数组、List、Map）

主要包括Bean的普通属性、数组属性、List 集合属性、map 集合属性

Person类

```java
public class Person {
    private String name;
    private String[] phones;
    private List<String> cities;
    private Map<String, Object> map;

    public Person() {
    }

    public Person(String name, String[] phones, List<String> cities, Map<String, Object> map) {
        this.name = name;
        this.phones = phones;
        this.cities = cities;
        this.map = map;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String[] getPhones() {
        return phones;
    }

    public void setPhones(String[] phones) {
        this.phones = phones;
    }

    public List<String> getCities() {
        return cities;
    }

    public void setCities(List<String> cities) {
        this.cities = cities;
    }

    public Map<String, Object> getMap() {
        return map;
    }

    public void setMap(Map<String, Object> map) {
        this.map = map;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name=" + name +
                ", phones=" + Arrays.toString(phones) +
                ", cities=" + cities +
                ", map=" + map +
                '}';
    }
}
```

EL表达式输出

```jsp
    <%
        Person person=new Person();
        person.setName("国哥好帅");
        person.setPhones(new String[]{"15521172641","13790963279","18699998888"});
        List<String> list=new ArrayList<>();
        list.add("北京");
        list.add("上海");
        list.add("深圳");
        person.setCities(list);
        Map<String,Object> map=new HashMap<>();
        map.put("key1","value1");
        map.put("key2","value2");
        map.put("key2","value2");
        person.setMap(map);

        pageContext.setAttribute("p",person);
    %>
    输出person：${ p } <br>

	<%--普通属性--%>

    输出person的name属性：${ p.name } <br>

    <%--数组属性--%>

    输出person的phones数组属性：${ p.phones } <br><%--由它的toString决定的--%>

    输出person的phones数组个别元素：${ p.phones[0] } <br>

    <%--List属性--%>
    输出person的List属性：${ p.cities } <br><%--由它的toString决定的--%>

    输出person的List的个别元素：${ p.cities.get(1) } <br>

	输出person的List的个别元素：${ p.cities[1] } <br>

    <%--Map属性--%>
    输出person的Map属性：${ p.map } <br><%--由它的toString决定的--%>

	输出person的cities的Map的个别元素：${ p.map.get("key1") } <br>

    输出person的cities的Map的个别元素：${ p.map.key1 } <br>

	输出person的cities的Map的个别元素：${ p.map["key1"] } <br>

```

相比java表达式，EL表达式里：List多了 [] 运算、Map多了 [] 运算与 . 运算。**注意Map的.与[]元素只有在key是String类型的时候才可以用，否则无效。更确切地说，EL表达式里，只能对key为String类型的map能正常使用，其他map无法在EL表达式里正常使用**

另外：

​		通过key获取到的对象，使用 *对象.属性* 去调用属性时，实质是调用该属性的 getter，即使不存在对应属性，但是存在对应的getter，也能够正常使用getter得到值；反之，即使存在对应属性，但没有对应的getter，也无法访问

​		这对于EL表达式里的任何对象都适用，访问属性本质是调用getter、而访问方法则直接访问该方法即可。最终的得到的数据将会打印在客户端页面上，没有数据就是空字符串



## EL 表达式——运算

${ 运算表达式 } ， EL 表达式支持如下运算符：

### 1）关系运算

![image-20210129132904006](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210129132904006.png)

```jsp
${ 12==12 }  //页面显示true
```

### 2）逻辑运算

![image-20210129133337860](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210129133337860.png)

```jsp
    ${ 12==12 && 13==12 }  //页面显示false
```

### 3）算数运算

![image-20210129133540139](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210129133540139.png)

```jsp
    ${ 12+13 }  //页面显示25
```

### i. empty 运算

empty 运算可以判断一个数据是否为空，如果为空，则输出 true,不为空输出 false

以下几种情况为空： 

​	1、值为 null 值的时候，为空 

​	2、值为空串的时候，为空 

​	3、值是 Object 类型数组，长度为零的时候 

​	4、list 集合，元素个数为零 

​	5、map 集合，元素个数为零

```jsp
    <%
        //值为null
        request.setAttribute("emptyNull",null);
        //字符串为空
        request.setAttribute("emptyStr","");
        //数组长度为0
        request.setAttribute("emptyArr",new Object[0]);
        //List没有元素
        List<String> list=new ArrayList<>();
        request.setAttribute("emptyList",list);
        //Map没有元素
        Map<String,Integer> map=new HashMap<>();
        request.setAttribute("emptyMap",map);
    %>
    ${ empty emptyNull } <br>
    ${ empty emptyStr } <br>
    ${ empty emptyArr } <br>
    ${ empty emptyList } <br>
    ${ empty emptyMap } <br>
```

以上都在页面输出 true



### ii. 三元运算

表达式 1？表达式 2：表达式 3 

如果表达式 1 的值为真，返回表达式 2 的值，如果表达式 1 的值为假，返回表达式 3 的值



### iii. “.”点运算 和 [] 中括号运算符

* .点运算，可以输出 Bean 对象中某个属性的值 、或调用对象里的方法、**或访问map的key对应的value**

* []中括号运算，可以输出有序集合（数组、List）中某个元素的值

* 并且[]中括号运算，**还可以输出 map 集合中 key 里含有特殊字符的 key 对应的value，[]里面写key可以用双引号或者单引号**

```jsp
    <%
        Map<String,Object> map=new HashMap<>();
        map.put("a.a.a","aaaValue");
        map.put("b+b+b","bbbValue");
        map.put("c-c-c","cccValue");

        request.setAttribute("m",map);

    %>

    ${ m["a.a.a"] } <br>
    ${ m['b+b+b'] } <br>
    ${ m['c-c-c'] } <br>
```

## EL表达式与Java表达式

可见EL表达式里，与常见的java表达式有很多共同点不同点

### 一、相同点

1、关系运算、逻辑运算、算术运算、三元运算表达式相同



### 二、不同点

1、EL表达式里面只能直接访问 特定对象（见下一节）、域map里的数据、常量数据（域map可以直接使用，参见下一节、域map里数据可以直接写key，也可以通过 域map.key或 域map["key"]  访问相应的value，不过前者将会按优先级遍历域对象，后者是指定域对象）

2、.  运算在java中，即访问对象属性和方法。**但在EL表达式中，除了访问对象属性方法之外，还可以用在Map上面获取key对象的值** 如 map.username 表示获取map中username对应的value

2、[] 运算，在Java表达式只能用在数组上面。**但在EL表达式里，除了可以用在数组上外，还可以用在List与Map上，在List上面功能与数组类似，在Map上面则表示key**

3、EL有empty运算



## EL 表达式的 11 个隐含对象

EL 个达式中 11 个隐含对象，是 EL 表达式中自己定义的，可以直接使用

![image-20210129201241067](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210129201241067.png)

![image-20210129201257690](D:%5Ctypora%5Cmarkdown%E5%9B%BE%E7%89%87%5Cimage-20210129201257690.png)



### i. EL 获取四个特定域中的属性

pageScope 	====== 	pageContext 域 

requestScope 	====== 	Request 域 

sessionScope 	====== 	Session 域 

applicationScope 	====== 	ServletContext 域



示例：

```jsp
    <%
        pageContext.setAttribute("key","pageContext");
        request.setAttribute("key","request");
        session.setAttribute("key","session");
        application.setAttribute("key","application");
    %>

    ${ pageScope.key } <br>
    ${ requestScope.key } <br>
    ${ sessionScope.key } <br>
    ${ applicationScope.key } <br>
```



### ii. pageContext 对象的使用

通过pageContext可以访问request、response、session、ServletContext、out等jsp九大内置对象

1. 协议： 
2. 服务器 ip： 
3. 服务器端口： 
4. 获取工程路径： 
5. 获取请求方法： 
6. 获取客户端 ip 地址： 
7. 获取会话的 id 编号：

```jsp
    <%
        request.getScheme(); //获取协议，如http
        request.getServerName(); //获取服务器 ip，如localhost
        request.getServerPort(); //获取服务器端口，如8080
        request.getContextPath(); //获取工程路径,如 /book
        request.getMethod(); //获取请求方法
        request.getRemoteHost(); //获取客户端 ip 地址
        session.getId(); //获取会话的 id 编号

    %>


    1. 协议：${pageContext.request.scheme } <br>
    2. 服务器 ip：${pageContext.request.serverName} <br>
    3. 服务器端口： ${pageContext.request.serverPort} <br>
    4. 获取工程路径： ${pageContext.request.contextPath}<br>
    5. 获取请求方法： ${pageContext.request.method}<br>
    6. 获取客户端 ip 地址： ${pageContext.request.remoteHost}<br>
    7. 获取会话的 id 编号： ${pageContext.session.id}<br>
```



### iii. EL 表达式其他隐含对象的使用

param 					Map<String,String> 		它可以获取请求参数的值 

paramValues 		Map<String,String[]> 		它也可以获取请求参数的值，获取多个值的时候使用

```jsp
    输出请求参数username的值：${ param.username } <br>
    输出请求参数password的值：${ param.password } <br>
    输出请求参数username的第一个值: ${ paramValues.username[0] } <br>
    输出hobby的值：${ paramValues.hobby[0] } <br>
    输出hobby的值：${ paramValues.hobby[1] } <br>
```

访问地址：http://192.168.0.101:8080/09_EL_JSTL/other_el_obj.jsp?username=zwg168&password=888&hobby=java&hobby=cpp



header 				Map<String,String> 		它可以获取请求头的信息 

headerValues 	Map<String,String[]> 		它可以获取请求头的信息，它可以获取多个值的情况

```jsp
    输出请求头User-Agent的值：${ header["User-Agent"] } <br>
    输出请求头connection的值：${ header.Connection } <br>
    输出请求头User-Agent的值：${ headerValues["User-Agent"][0] } <br>
    输出请求头connection的值：${ headerValues.Connection[0] } <br>
```



cookie 		Map<String,Cookie> 		它可以获取当前请求的 Cookie 信息 

```jsp
    获取cookie：${ cookie.JSESSIONID } <br>
    获取cookie的名称：${ cookie.JSESSIONID.name } <br>
    获取cookie的值：${ cookie.JSESSIONID.value } <br>
```



initParam 	Map<String,String> 	它可以获取在 web.xml 中配置的<context-param>上下文参数

```jsp
    获取initParam：${ initParam } <br>
    输出initParam的username值：${ initParam.username } <br>
    输出initParam的url值：${ initParam.url } <br>
```



# JSTL标签库

## 定义

​		JSTL 标签库 全称是指 JSP Standard Tag Library JSP 标准标签库。是一个不断完善的开放源代码的 JSP 标 签库

​		EL 表达式主要是为了替换 jsp 中的表达式脚本，而标签库则是为了替换代码脚本。这样使得整个 jsp 页面 变得更佳简洁

​		**所谓标签库，就是告诉jsp如何翻译这些标签到servlet中去**

JSTL 由五个不同功能的标签库组成

![image-20210129223051185](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210129223051185.png)



在 jsp 标签库中使用 taglib 指令引入标签库

CORE 标签库 ：

```jsp
<%@ taglib prefix=*"c"* uri=*"http://java.sun.com/jsp/jstl/core"* %> 
```

XML 标签库：

```jsp
<%@ taglib prefix=*"x"* uri=*"http://java.sun.com/jsp/jstl/xml"* %> 
```

FMT 标签库 ：

```jsp
<%@ taglib prefix=*"fmt"* uri=*"http://java.sun.com/jsp/jstl/fmt"* %> 
```

SQL 标签库 ：

```jsp
<%@ taglib prefix=*"sql"* uri=*"http://java.sun.com/jsp/jstl/sql"* %> 
```

FUNCTIONS 标签库 ：

```jsp
<%@ taglib prefix=*"fn"* uri=*"http://java.sun.com/jsp/jstl/functions"* %>
```



## JSTL 标签库的使用步骤

1、先导入 jstl 标签库的 jar 包

​		taglibs-standard-impl-1.2.1.jar 

​		taglibs-standard-spec-1.2.1.jar

2、第二步，使用 taglib 指令引入标签库

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```



## core 核心库使用

### i. <c:set />（使用很少）

作用：set 标签可以往域中保存数据 

```jsp
    <%--
    <c:set/>
        作用：set 标签可以往域中保存数据

        域对象.setAttribute(key,value)
        scope属性 设置保存到哪个域
            page表示PageContext域(默认值)
            request表示Resquest域
            session表示Session域
            application表示ServletContext域
        var 属性设置key是多少
        value属性设置value是多少

    --%>
    保存之前的值：${requestScope.abc} <br>
    <c:set scope="request" var="abc" value="abcValue" />
    保存之后的值：${requestScope.abc} <br>
```



### ii. <c:if />

if 标签用来做 if 判断

```jsp
    <%--
    <c:if />
        if 标签用来做 if 判断

        test属性表示判断的条件(使用EL表达式输出)
    --%>
    <c:if test="${ 12==12 }">
        <h1>
            12等于12
        </h1>
    </c:if>

    <c:if test="${ 12!=12 }">
        <h1>
            12不等于12
        </h1>
    </c:if>
```



### iii. < c:choose> < c:when> < c:otherwise>标签

作用：多路判断。跟 switch ... case .... default 非常接近

```jsp
    <%--
    iii. < c:choose> < c:when> < c:otherwise>标签
        作用：多路判断。跟 switch ... case .... default 非常接近

        choose标签开始选择判断
            when标签表示每一种判断情况
                test属性表示当前这种情况的值
            otherwise表示剩下的情况

        < c:choose> < c:when> < c:otherwise>标签使用时注意的点：
            0、choose里面的when和otherwise标签将从上到下检查、符合时就输出里面的内容。但不会输出后面的标签里的内容，这是与switch/case的不同之处
            1、标签里不要使用html注释，只能用jsp注释
            2、when标签的父标签一定要是choose标签
            3、多路判断里面可以嵌套多路判断，具体就是choose的子标签when/otherwise里还可以嵌套choose
    --%>
    <hr>
    <%
        request.setAttribute("height",198);
    %>
    <c:choose>
        <!-- 这是html注释，执行时将会报错，这里只能用户jsp注释 -->
        <c:when test="${ requestScope.height>190 }">
            <h2>小巨人</h2>
             <!-- 这是html注释，这里用没问题 -->
        </c:when>
        <c:when test="${ requestScope.height>180 }">
            <h2>很高</h2>
        </c:when>
        <c:when test="${ requestScope.height>170 }">
            <h2>还可以</h2>
        </c:when>
        <c:otherwise>
            <h2>剩下小于170的情况</h2>
        </c:otherwise>

    </c:choose>
```

注意：

 		0、choose里面的when和otherwise标签将从上到下检查、符合时就输出里面的内容。但不会输出后面的标签里的内容，这是与switch/case的不同之处
 	     1、标签里不要使用html注释，只能用jsp注释
 	     2、when标签的父标签一定要是choose标签
 	     3、多路判断里面可以嵌套多路判断，具体就是choose的子标签when/otherwise里还可以嵌套choose



### iv. <c:forEach />

作用：遍历输出使用

#### 1. 遍历 1 到 10，输出

```jsp
<%--
1. 遍历 1 到 10，输出
    begin属性设置开始的索引
    end属性设置结束的索引
    var属性表示循环的变量(也是当前正在遍历到的数据)
--%>
    <c:forEach begin="1" end="10" var="i">
        <h1>${ i } </h1>
    </c:forEach>

    <table>
        <c:forEach begin="1" end="10" var="i">
            <tr>
                <td>第 ${ i } 行</td>
            </tr>
        </c:forEach>
    </table>
```



#### 2. 遍历 Object 数组

```jsp
<%--
2. 遍历 Object 数组
    items表示遍历的数据源(遍历的集合)
    var表示当前遍历到的数据
--%>
    <%
        request.setAttribute("arr",new String[]{"15521172641","13790963279"});
    %>
    <c:forEach items="${ requestScope.arr }" var="item">
        ${ item }
    </c:forEach>
```



#### 3. 遍历 Map 集合

```jsp
    <%
        Map<String,Object> map=new HashMap<>();
        map.put("key1","value1");
        map.put("key2","value2");
        map.put("key3","value3");
        request.setAttribute("map",map);

        
    %>
    <c:forEach items="${ requestScope.map }" var="entry">
        <h1>${ entry.key } = ${ entry.value }</h1>
    </c:forEach>
```

```jsp
            <%--
        items 表示遍历的集合
        var 表示遍历到的数据
        begin 表示遍历开始的索引
        end 表示遍历结束的索引
        step表示遍历时的索引步长(默认是1)
        varStatus 表示当前遍历到的数据状态
        --%>
	<c:forEach begin="0" end="2" step="2" varStatus="status" items="${ requestScope.map }" var="entry">
        <h1>${ entry.key } = ${ entry.value }</h1>
    </c:forEach>
```



#### 4.遍历 List 集合

list 中存放 Student 类，有属性：编号，用户名，密码，年龄， 电话信息

```jsp
    <%
        List<Student> studentList=new ArrayList<>();
        for (int i = 0; i < 10; i+=1) {
            studentList.add(new Student(1,"username"+i,"pass"+i,18+i,"phone"+i));
        }
        request.setAttribute("stus",studentList);
    %>
    <c:forEach items="${ stus }" var="student">
        ${ student }
    </c:forEach>
```

```jsp
        <%--
        items 表示遍历的集合
        var 表示遍历到的数据
        begin 表示遍历开始的索引
        end 表示遍历结束的索引
        step表示遍历时的索引步长(默认是1)
        varStatus 表示当前遍历到的数据状态
        --%>
        <c:forEach begin="2" end="6" step="2" varStatus="status" items="${ stus }" var="student">
            <tr>
                <td>${student.id}</td>
                <td>${student.username}</td>
                <td>${student.password}</td>
                <td>${student.age}</td>
                <td>${student.phone}</td>
                <td>删除/修改</td>
            </tr>
        </c:forEach>
```

#### 关于varStatus

![varStatus](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/varStatus.PNG)



### core核心库与EL表达式的联系

​		由上面的core核心库的标签可以看出，core核心库的标签属性需要表达式赋值时只能用EL表达式，且EL表达式能够访问这些标签的var属性（参考c：forEach标签），或者可以这么说，**core核心库的标签扩展了EL表达式的功能**（EL表达式常见的功能时打印到页面，core核心库能翻译其EL为取值功能从而把值赋予其标签属性，且能创建EL表达式可访问的变量）

​		jsp本身是为了指导servlet输出页面的，EL表达式一般是取值输出到页面；遇到core核心库的标签时，属性赋值的EL表达式表示取值，之后再把core的标签翻译成相关java代码（判断，多路选择、循环等），core标签的var属性设置的变量底层原理是在EL可见域里面设置的，这样EL表达式也可以访问



# 文件的上传和下载

文件的上传和下载，是非常常见的功能。很多的系统中，或者软件中都经常使用文件的上传和下载

比如：QQ 头像，就使用了上传 

邮箱中也有附件的上传和下载功能 

OA 系统中审批有附件材料的上传

## 上传

文件上传时是通过二进制流的形式上传的，服务端 request.getInputStream（）才能获取上传的信息

### 前端上传的格式

1、要有一个 form 标签，method=post 请求 

2、form 标签的 encType 属性值必须为 multipart/form-data 值 。**只有post请求可以处理此类多段数据，如果是get请求，即使声明为 encType=multipart/form-data，实际上也不会是多段数据**

3、在 form 标签中使用 input type=file 添加上传的文件 

4、编写服务器代码（Servlet 程序）接收，处理上传的数据

 

encType=multipart/form-data 表示提交的数据，以多段（每一个表单项一个数据段）的形式进行拼 

接，然后以二进制流的形式发送给服务器。**在服务器里，信息不再在参数map里面，而是存放在inputStream里。相反，如果按照非多段数据传递时，信息是在参数map里面，而非inputStream里面**

### 上传的http协议

![image-20210203125950624](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210203125950624.png)

### 后端解析上传的文件

已经有很多第三方的jar包可以供我们用来解析这些上传的文件，例如 **commons-fileupload.jar** ，这个jar包需要依赖 **commons-io.jar** 这个jar包



**第一步，就是需要导入两个** **jar** **包：** 

​	commons-fileupload-1.2.1.jar 

​	commons-io-1.4.jar 



**commons-fileupload.jar 和 commons-io.jar 包中，我们常用的类有哪些？**

​	ServletFileUpload 类，用于解析上传的数据

​	FileItem 类，表示每一个表单项



* boolean ServletFileUpload.*isMultipartContent*(HttpServletRequest request); 

判断当前上传的数据格式是否是多段的格式



* public List<FileItem> parseRequest(HttpServletRequest request)  

解析上传的数据 （多段数据格式，每一段对应一个表单项，处理后对应一个FileItem）



* boolean FileItem.isFormField() 

判断当前这个表单项，是否是普通的表单项。还是上传的文件类型

true 表示普通类型的表单项 

false 表示上传的文件类型 



* String FileItem.getFieldName()  

获取表单项的 name 属性值String FileItem.getString() 

获取当前表单项的值 



* String FileItem.getName(); 

获取上传的文件名 



* void FileItem.write( file ); 

将上传的文件写到 参数 file 所指向的硬盘位置 



**第二步：解析上传的文件:**

```java
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //先判断数据是否是多段数据
        if(ServletFileUpload.isMultipartContent(request)) {

            //创建FileItemFactory工厂实现类的实例
            FileItemFactory fileItemFactory=new DiskFileItemFactory();

            //创建用于解析数据的工具类
            ServletFileUpload servletFileUpload=new ServletFileUpload(fileItemFactory);
            servletFileUpload.setHeaderEncoding("UTF-8");//参数utf-8解决文件名乱码问题

            //解析上传的数据，得到每一个表单项FileItem
            try {
                List<FileItem> list=servletFileUpload.parseRequest(request);
                //循环判断每一个表单项是普通类型还是上传的文件
                for (FileItem fileItem : list) {
                    if(fileItem.isFormField()){//普通表单项

                        System.out.println("表单项的name属性值："+fileItem.getFieldName());
                        //参数utf-8解决乱码问题
                        System.out.println("表单项的value属性值："+fileItem.getString("utf-8"));

                    }else {//上传的文件

                        System.out.println("表单项的name属性值："+fileItem.getFieldName());
                        System.out.println("上传的文件名："+fileItem.getName());
                        fileItem.write(new File("F:\\fileTest\\"+fileItem.getName()));

                    }
                }
            } catch (FileUploadException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```

注意：

* servletFileUpload.setHeaderEncoding("xxx")用于解决上传的文件的文件名中文乱码问题，保证fileItem.getName()可以得到正确的文件名；
* fileItem.getString("utf-8")用于得到正确的表单项value属性，解决中文乱码问题



## 下载

### 下载逻辑与示例

![文件下载逻辑](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/文件下载逻辑.jpg)

其中，服务器的第2、3步是设置好相关格式，一定要在response获取输出流之前设置好，否则无效。因此把他们安排在回传前面

示例如下：

```java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.获取要下载的文件名
        String downloadFilename="2.jpg";

        //2.通过响应头告诉客户端返回的数据类型
        ServletContext servletContext = getServletContext();
        /**
         * 斜杠/ 被服务器解读为 http://ip:port/工程名/  映射到源码工程的web目录
         */
        String mimeType = servletContext.getMimeType("/file/" + downloadFilename);//获取文件类型，如image/jpeg

        response.setContentType(mimeType);//设置了响应头，也设置了内容编码格式

        //3.通过响应头告诉客户端数据是用来下载的
        //Content-Disposition表示收到的数据怎么处理
        //attachment表示附件，表示下载使用
        //filename = 表示指定下载的文件名,这里可以自定义，不用和原文件名相同
        response.setHeader("Content-Disposition","attachment;filename="+downloadFilename);

        //4.读取要下载的文件内容(通过ServletContext对象可以读取)
        /**
         * 斜杠/ 被服务器解读为 http://ip:port/工程名/  映射到源码工程的web目录
         */
        InputStream resourceAsStream = servletContext.getResourceAsStream("/file/" + downloadFilename);

        //3.把下载的文件回传给客户端
        IOUtils.copy(resourceAsStream,response.getOutputStream());//读取输入流中的数据，复制给输出流输出给客户端

        
    }
```



### 下载文件的文件名中文乱码

​		下载文件的文件名是由服务端指定的（上面实例的第3步），如果该文件名有中文，客户端可能会出现乱码，因此需要对文件名的中文进行相应的编码转换（把中文字符串变成其他字符串）

​		普通的相应数据，我们在响应头里设置字符集即可，响应头的字符集告诉客户端如何解读响应体的数据。而下载文件名由于就在响应头设置，因此只有通过编码转换把中文字符串转换成其他字符串客户端才能正确识别



#### 方案1：

​		url编码本质是：找到字符对应编码（如utf-8或GB2312等等）的值，取该值的16进制xxyy...（如E6 98 A5），转化为%xx%yy...的字符串格式，如 %E6%98%A5 。参见百度

```java
    public static void main(String[] args) throws IOException {
        String content="这是需要url编码的内容";
        
        //url编码
        String urlEncodeStr=URLEncoder.encode(content,"utf-8");
        
        //url解码
        String urlDecodeStr= URLDecoder.decode(urlEncodeStr,"utf-8");
        
    }
```



<a href="https://www.cnblogs.com/hannover/p/4657463.html">关于url汉字编码问题</a>



​		如果客户端浏览器是 IE 浏览器 或者 是谷歌浏览器。我们需要使用 URLEncoder 类先对中文名进行 UTF-8 的编码 操作。因为 IE 浏览器和谷歌浏览器收到含有编码后的字符串后会以 UTF-8 字符集进行解码显示

```java
		//3.通过响应头告诉客户端数据是用来下载的
        //Content-Disposition表示收到的数据怎么处理
        //attachment表示附件，表示下载使用
        //filename = 表示指定下载的文件名
        //url编码是把汉子转化为 %xx%yy...的格式
        response.setHeader("Content-Disposition","attachment;filename="+ URLEncoder.encode("中国.jpg","utf-8"));

```





#### 方案2：

BASE64 编解码操作

不同于url编码的 **字符 -- 编码** 模式，BASE64 编解采用的是 **字节数组 -- 编码** 的模式，且编码解码的规则是固定不变的，用户可操作的就只有 字符与字节数组之间的转换

```java
    public static void main(String[] args) throws IOException {
        String content="这是需要base64编码的内容";
        
        //创建一个base64编码器
        BASE64Encoder base64Encoder=new BASE64Encoder();
        //执行base64编码操作
        String encodeStr = base64Encoder.encode(content.getBytes("utf-8"));

        
        //创建一个base64解码器
        BASE64Decoder base64Decoder=new BASE64Decoder();
        //执行base64解码操作
        byte[] bytes = base64Decoder.decodeBuffer(encodeStr);
        
        //把字节数组转换会字符
        String tt=new String(bytes,"utf-8");

    }
```



如果客户端浏览器是火狐浏览器或谷歌浏览器。 那么我们需要对中文名进行 BASE64 的编码操作

这时候需要把请求头 Content-Disposition: attachment; filename=中文名 

编码成为：Content-Disposition: attachment; filename==?charset?B?xxxxx?= 



=?charset?B?xxxxx?= 现在我们对这段内容进行一下说明： 

​		=?					表示编码内容开始		

​		charset 		表示字符集（ 如utf-8 ），用于BASE64解码得到字节数组后恢复原字符串

​		B					表示BASE64编码

​		xxxx 				表示编码后的内容

​		?=					表示编码内容结束

```java
        //3.通过响应头告诉客户端数据是用来下载的
        //Content-Disposition表示收到的数据怎么处理
        //attachment表示附件，表示下载使用
        //filename = 表示指定下载的文件名
        //url编码是把汉子转化为 %xx%xx...的格式
        response.setHeader("Content-Disposition","attachment;filename==?utf-8?B?"+new BASE64Encoder().encode("中国.jpg".getBytes("utf-8"))+"?=");
```



#### 两个编码的比较

![URL编码与BASE64编码](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/URL编码与BASE64编码.jpg)

综合上述，谷歌浏览器的兼容性最强，ie只能url编码，firefox只能BASE64编码。而chrome则两种编码都支持



#### 不同浏览器解决方案的兼容（User-Agent）

​		在代码里，服务端如果对于不同浏览器的请求采用不同的编码来解决文件名中文乱码问题，那么就能适应不同的浏览器。关键的一点就是需要知道发起请求的浏览器，可以通过请求头的 User-Agent 项来判断，具体代码如下：

```java
        if(request.getHeader("User-Agent").contains("Firefox")){//火狐浏览器
            
            //使用BASE64编码下载文件的中文名
            
        }else {//谷歌或ie浏览器
            
            //使用url编码下载文件的中文名
            
        }
```





# web中文乱码问题

![中文乱码问题](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/中文乱码问题.jpg)

* 客户端发送数据给服务器时，对于出现的中文都采用url编码，服务器接收数据后进行url解码取出数据（这都是客户端浏览器和服务器自动进行，不需要程序员动手，最多只需要设置相关字符集即可）
* 而服务器发送数据给客户端时，一般响应体的数据不会采用url或BASE64编码，而是直接在响应头指定字符集；只有响应头含有中文时，需要服务器把中文进行url或BASE64编码转换（这一步需要程序员写转换代码）

**解决办法：**

​		普通数据传输的中文乱码见Servlet中的request与response。文件名的中文乱码见文件的上传和下载



# Cookie与Session

## Cookie

### 定义

​	1、Cookie 翻译过来是饼干的意思

​	2、Cookie 是服务器通知客户端保存键值对的一种技术 

​	3、客户端有了 Cookie 后，每次请求都发送给服务器

​	4、每个 Cookie 的大小不能超过 4kb

### cookie的创建

![image-20210212214653611](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210212214653611.png)

如果存在相同key的键值对，就用新的value覆盖旧的value



servlet中的程序：cookie可以同时创建多个，例如下面的代码

```java
    protected void createCookie(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //1、创建Cookie对象
        Cookie cookie1 = new Cookie("key1", "value1");
        Cookie cookie2=new Cookie("key2","value2");

        //2、通知客户端保存Cookie
        response.addCookie(cookie1);
        response.addCookie(cookie2);

        //3、带话
        response.setContentType("text/html;charset=utf-8");

        response.getWriter().write("cookie创建两个成功");
    }
```

注意：如果客户端已有相同key和path的cookie键值对，那么新的value将会替换旧的value



### 服务器获取 Cookie

​		服务器获取客户端的 Cookie 只需要一行代码：req.getCookies() 得到Cookie[]

![image-20210212221735683](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210212221735683.png)

servlet中的程序：

```java
    protected void getCookie(HttpServletRequest request, HttpServletResponse response) throws IOException {
        Cookie[] cookies = request.getCookies();
        //cookie.getName()返回cookie的key(名称)
        //cookie.getValue()返回cookie的value(值)
        response.setContentType("text/html;charset=utf-8");
        for (Cookie cookie : cookies) {
            response.getWriter().write("cookie["+cookie.getName()+" = "+cookie.getValue()+"] <br/>");
        }
    }
```

​		注意：response.setContentType("text/html;charset=utf-8") 设置内容格式为html，客户端才能把< br> 解析为换行

### Cookie 值的修改

#### 方案一： 

​	1、先创建一个要修改的同名（指的就是 key）的 Cookie 对象 

​	2、在构造器，同时赋于新的 Cookie 值

​	3、调用 response.addCookie( Cookie );

#### 方案二

​	1、先查找到需要修改的 Cookie 对象 

​	2、调用 setValue()方法赋于新的 Cookie 值 

​	3、调用 response.addCookie()通知客户端保存修改

### 浏览器如何查看cookie

谷歌浏览器

![image-20210212231959313](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210212231959313.png)



火狐浏览器

![image-20210212232022181](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210212232022181.png)



### Cookie 生命控制

​		Cookie 的生命控制指的是如何管理 Cookie 什么时候被销毁（删除）

setMaxAge() 

​	正数，表示在指定的秒数后过期 （即使浏览器被关闭也不影响时间计数，当然浏览器不能设置退出清空cookie，否则退出浏览器就没了）

​	负数，表示浏览器一关，Cookie 就会被删除（默认值是-1） 

​	零，表示马上删除 Cookie 



```java
    protected void defaultLife(HttpServletRequest request, HttpServletResponse response) throws IOException {

        Cookie cookie=new Cookie("defaultLife","defaultLife");
        cookie.setMaxAge(-1); //设置存活时间
        response.addCookie(cookie);

    }

    protected void deleteNow(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //找到要删除的Cookie对象
        Cookie cookie=CookieUtils.findCookie("key4",request.getCookies());
        if(cookie!=null){
            cookie.setMaxAge(0); //设置存活时间
            response.addCookie(cookie);

            response.setContentType("text/html;charset=utf-8");
            response.getWriter().write("key4的cookie已经被删除");
        }
    }

    protected void life3600(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //找到要删除的Cookie对象
        Cookie cookie=new Cookie("lefe3600","lefe3600");

        //设置cookie一小时之后被删除
        cookie.setMaxAge(60*60);

        response.addCookie(cookie);

        response.setContentType("text/html;charset=utf-8");
        response.getWriter().write("已经创建了一个存活一小时的cookie");

    }
```



### Cookie 有效路径Path的设置

Cookie 的 path 属性可以有效的过滤哪些 Cookie 可以发送给服务器。哪些不发 

path 属性是通过请求的地址来进行有效的过滤。 只有访问的地址是**Path或者是Path的子路径**才会发送该cookie

​	CookieA		path=/工程路径 

​	CookieB 		path=/工程路径/abc 

请求地址如下： 

​		http://ip:port/工程路径/a.html 

​				CookieA 发送 

​				CookieB 不发送 

​		http://ip:port/工程路径/abc/a.html 

​				CookieA 发送 

​				CookieB 发送 



示例代码如下：

```java
 cookie.setPath("/13_cookie_session/cookie.html");
```



**注意**：由于path中不含有ip与port，因此这两个元素不影响cookie的发送，即不同的ip、port只要工程路径及后面能满足 path即可发送cookie。即cookie的path要等于页面地址或者是页面地址的父地址才能通过当前页面发送给服务器



### 免输入用户名登录

![image-20210213122619689](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210213122619689.png)



### 注意

* 同一个浏览器的不同网页窗口可以共享cookie，但一般浏览器为了安全一个网页窗口只显示path对应（等于网址的path或者是网址父地址的path）的cookie

* 即使名字相同的cookie也不一定是相同的cookie，因为path可能不同。即名字相同但path不同的cookie不是一个cookie，不会产生覆盖问题，只有名字相同且path相同的cookie才会覆盖



## Session

### 定义

​	1、Session 就一个接口（HttpSession）

​	2、Session 就是会话。它是用来维护一个客户端和服务器之间关联的一种技术

​	3、每个客户端都有自己的一个 Session 会话

​	4、Session 会话中，我们经常用来保存用户登录之后的信息



### Session的创建和获取

如何创建和获取 Session。它们的 API 是一样的

**request.getSession()** 

​		第一次调用是：创建 Session 会话 

​		之后调用都是：获取前面创建好的 Session 会话对象

​		注意：getSession是从浏览器发送过来的 [ path=/工程名 && name=JSESSION ] 的cookie中获取session的id的，如果不存在这个cookie或者是不存在相应id的session都会创建一个session，并创建 [ path=/工程名 && name=JSESSION ] 的cookie返回给客户端



**isNew()**：判断到底是不是刚创建出来的（新的） 

​		true 表示刚创建 

​		false 表示获取之前创建 



每个会话都有一个身份证号。也就是 ID 值。而且这个 ID 是唯一的

**getId()**： 得到 Session 的会话 id 值



总结：session是与服务端与客户端的会话，存放在服务端全局位置上，通过request可以创建或获取。request一地刺是创建，后面都是获取。当然，如果客户端浏览器关闭了，服务端的session也会消除



### Session 域数据的存取

代码如下：

```java
    /**
     * 往session中保存数据
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    protected void setAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        req.getSession().setAttribute("key1","value1");
        resp.setContentType("text/html;charset=utf-8");
        resp.getWriter().write("已经往session中保存了数据");


    }

    /**
     * 获取session域中的数据
     * @param req
     * @param resp
     * @throws ServletException
     * @throws IOException
     */
    protected void getAttribute(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        Object key1 = req.getSession().getAttribute("key1");

        resp.setContentType("text/html;charset=utf-8");
        resp.getWriter().write("从session中获取出key1的数据是："+key1);

    }
```



### Session 生命周期控制

* public void setMaxInactiveInterval(int interval) ：设置 Session 的超时时间（以秒为单位），超过指定的时长，Session 就会被销毁

  ​		值为正数的时候，设定 Session 的超时时长

  ​		负数表示永不超时（极少使用）

  **注意：不能设置为0**

  

* public int getMaxInactiveInterval()：获取 Session 的超时时间（默认30分钟）



* public void invalidate() ：让当前 Session 会话马上超时无效



Session 默认的超时时间长为 30 分钟 

​		因为在 Tomcat 服务器的配置文件 web.xml中默认有以下的配置，它就表示配置了当前 Tomcat 服务器下所有的 Session 

超时配置默认时长为：30 分钟

```xml
<session-config> 
	<session-timeout>30</session-timeout> 
</session-config>
```



​		如果说。你希望你的 web 工程，默认的 Session 的超时时长为其他时长。你可以在你自己的 web.xml 配置文件中做以上相同的配置。就可以修改你的 web 工程**所有 Seession 的默认超时时长**

```xml
<session-config> 
    <session-timeout>20</session-timeout> 
</session-config>
```



​		如果你想只修改个别 Session 的超时时长。就可以使用上面的 API。setMaxInactiveInterval(int interval)来进行单独的设置

​		session.setMaxInactiveInterval(int interval)  单独设置超时时长



Session 超时的概念介绍：

![image-20210214132653076](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210214132653076.png)

就是说：session的超时指的是**客户端两次请求的最大时间间隔**（而非从当下开始计时），如果超过就销毁，再请求的时候只能新建



### 浏览器和 Session 之间关联的技术内幕

Session 技术，底层其实是基于 Cookie 技术来实现的

![image-20210214134958414](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210214134958414.png)



即客户端每次发送请求给服务端：

* 请求没有session的id，这时，服务端的**getSession()**会创建新的session并把其id以cookie的形式发送给客户端
* 请求含有session的id：
* * 该id可以获取相应session，服务端的**getSession()**将会返回获取到的session
  * 该id获取不到相应session，服务端的**getSession()**将会创建新的session并以cookie形式返回其id给客户端



​		**注意**：getSession是从浏览器发送过来的 [ path=/工程名 && name=JSESSION ] 的cookie中获取session的id的，如果不存在这个cookie或者是不存在相应id的session都会创建一个session，并创建 [ path=/工程名 && name=JSESSION ] 的cookie返回给客户端



### Session与工程、浏览器的关系

![Session与工程、浏览器的关系](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Session与工程、浏览器的关系.jpg)



### 常用用法

* 用来保存用户登录后的User信息

* 用来保存购物车

* 用来保存最后一次添加的商品名称

  



# Filter过滤器

## 定义

​		1、Filter 过滤器它是 JavaWeb 的三大组件之一。三大组件分别是：Servlet 程序、Listener 监听器、Filter 过滤器 

​		2、Filter 过滤器它是 JavaEE 的规范。也就是接口 

​		3、Filter 过滤器它的作用是：**拦截请求**，过滤响应



拦截请求常见的应用场景有： 

​	1、权限检查 

​	2、日记操作 

​	3、事务管理 

​		……等等 



## Filter 的初体验

要求：在你的 web 工程下，有一个 admin 目录。这个 admin 目录下的所有资源（html 页面、jpg 图片、jsp 文件、等等）都必 须是用户登录之后才允许访问



### 不使用filter的拦截

思考：根据之前我们学过内容。我们知道，用户登录之后都会把用户登录的信息保存到 Session 域中。所以要检查用户是否 登录，可以判断 Session 中否包含有用户登录的信息即可！！！ 

```jsp
    <%
         Object user = session.getAttribute("user");
         if(user==null){//还没登录
             request.getRequestDispatcher("/login.jsp").forward(request,response);
             return;
         }
    %>
```



### Filter 的工作流程图

![image-20210220122739850](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210220122739850.png)



### Filter 的代码

```java
public class AdminFilter implements Filter {


    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        
    }

    /**
     * doFilter方法专门用于拦截请求。可以做权限检查
     * @param request
     * @param response
     * @param chain
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("进入adminFilter拦截器了");

        HttpServletRequest httpServletRequest= (HttpServletRequest) request;

        HttpSession session=httpServletRequest.getSession();

        Object user = session.getAttribute("user");
        if(user==null){//还没登录
            request.getRequestDispatcher("/login.jsp").forward(request,response);
            return;
        }else {//已经登录
            //让程序往下继续访问用户的目标资源
            chain.doFilter(request,response);
        }
    }

    @Override
    public void destroy() {

    }

}
```



## Filter 的生命周期

Filter 的生命周期包含几个方法 

​		1、构造器方法 

​		2、init 初始化方法 

​					第 1，2 步，在 web 工程启动的时候执行（Filter 已经创建） 

​		3、doFilter 过滤方法 

​					第 3 步，每次拦截到请求，就会执行 

​		4、destroy 销毁 

​					第 4 步，停止 web 工程的时候，就会执行（停止 web 工程，也会销毁 Filter 过滤器） 

## FilterConfig 类

FilterConfig 类见名知义，它是 Filter 过滤器的配置文件类

Tomcat 每次创建 Filter 的时候，也会同时创建一个 FilterConfig 类，这里包含了 Filter 配置文件的配置信息

​	FilterConfig 类的作用是获取 filter 过滤器的配置内容 ：

​			1、获取 Filter 的名称 filter-name 的内容 

​			2、获取在 Filter 中配置的 init-param 初始化参数 

​			3、获取 ServletContext 对象

具体代码：

```java
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("2.filter的init(filterConfig)初始化");

//      1、获取 Filter 的名称 filter-name 的内容
        System.out.println("filter-name的值是："+filterConfig.getFilterName());

//		2、获取在 Filter 中配置的 init-param 初始化参数
        System.out.println("初始化参数username的值是："+filterConfig.getInitParameter("username"));
        System.out.println("初始化参数url的值是："+filterConfig.getInitParameter("url"));

//		3、获取 ServletContext 对象
        System.out.println("servletContext: "+filterConfig.getServletContext());

    }
```

web.xml中的配置

```xml
    <filter>
        <!--filter的别名-->
        <filter-name>AdminFilter</filter-name>
        <!--配置filter的全类名-->
        <filter-class>com.atguigu.filter.AdminFilter</filter-class>
        
        <init-param>
            <param-name>username</param-name>
            <param-value>root</param-value>
        </init-param>
        <init-param>
            <param-name>url</param-name>
            <param-value>jdbc:mysql://localhost3306/test</param-value>
        </init-param>
    </filter>
    <!--filter-mapping配置filter过滤器的拦截路径-->
    <filter-mapping>
        <!--表示给哪个filter配置-->
        <filter-name>AdminFilter</filter-name>
        <!--配置拦截路径
            /：表示请求地址为 http://ip:port/工程路径/  映射到idea的web目录
            /admin/* ：表示请求地址为 http://ip:port/工程路径/admin/*
        -->
        <url-pattern>/admin/*</url-pattern>
    </filter-mapping>
```



## FilterChain 过滤器链

Filter 			过滤器 

Chain 			链，链条 

FilterChain 	就是过滤器链（多个过滤器如何一起工作）

![image-20210220210120900](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210220210120900.png)



## Filter 的拦截路径



### 精确匹配

```xml
 <url-pattern>/target.jsp</url-pattern>
```

以上配置的路径，表示请求地址必须为：http://ip:port/工程路径/target.jsp 



### 目录匹配

```xml
 <url-pattern>/admin/*</url-pattern>
```

以上配置的路径，表示请求地址必须为：http://ip:port/工程路径/admin/*



### 后缀名匹配

```xml
<url-pattern>*.html</url-pattern>
```

以上配置的路径，表示请求地址必须以.html 结尾才会拦截到

**注意：后缀名匹配不能以斜杠打头**



### 多个路径

```xml
    <filter>
        <filter-name>ManagerFilter</filter-name>
        <filter-class>com.atguigu.filter.ManagerFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>ManagerFilter</filter-name>
        <url-pattern>/pages/manager/*</url-pattern>
        <url-pattern>/manager/bookServlet</url-pattern>
    </filter-mapping>
```

一个filter可以设置多个url-pattern拦截路径



Filter 过滤器它只关心请求的地址是否匹配，不关心请求的资源是否存在！！！比如请求http://ip:port/book/a.abc 是会被匹配的Filter拦截的，即使不存在.abc这种后缀名的文件



## ThreadLocal 的使用

### 定义

ThreadLocal 的作用，它可以解决多线程的数据安全问题



ThreadLocal 它可以给当前线程关联一个数据（可以是普通变量，可以是对象，也可以是数组，集合）



ThreadLocal 的特点： 

​		1、ThreadLocal 可以为当前线程关联一个数据。（它可以像 Map 一样存取数据，key 为当前线程） 



​		2、每一个 ThreadLocal 对象，只能为当前线程关联一个数据，如果要为当前线程关联多个数据，就需要使用多个 ThreadLocal 对象实例



​		3、每个 ThreadLocal 对象实例定义的时候，一般都是 static 类型 



​		4、ThreadLocal 中保存数据，在线程销毁后。会由 JVM 虚拟自动释放

### 原理

![线程本地存储原理](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/线程本地存储原理.jpg)





## 使用 Filter 和 ThreadLocal 组合管理事务

###  一、同一个 Connection 对象中完成事务（ThreadLocal）

原理分析图：

![image-20210222011851436](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210222011851436.png)

关键在于：

​	1、Connection对象必须同一个线程内共享，不同线程有不同的Connection对象。这可以通过ThreadLocal对象实现

​	2、xxxService 整个过程出现的异常都抛出

​	3、在Servlet中对xxxService进行try/catch，如果没异常就【提交事务关闭连接清空ThreadLocal】，如果有异常就【回滚事务关闭连接清空ThreadLocal】



### 二、统一给所有service方法加上try/catch（Filter）

​		如果在每一处使用service都 try/catch 就太麻烦了，而我们可以通过使用 Filter 来间接给所有的service方法加上 try/catch 

​		即直接在 Filter 里面对后面的访问加上 try/catch ，如果没异常就【提交事务关闭连接清空ThreadLocal】，如果有异常就【回滚事务关闭连接清空ThreadLocal】

​		这要求这个 Filter 后面的所有访问出现异常都抛出而非catch内部消化

![image-20210222013124532](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/image-20210222013124532.png)

这样就不用给所有调用service方法的地方一一加上 try/catch 了。在filter加上一个就相当于给所有访问加上try/catch 。这样不好的地方就是如果后面出现非数据库异常，即事务全部正常执行但出现其他错误，依然会回滚事务

**Filter的Java代码如下：**

```java
public class TransactionFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        try {
            chain.doFilter(request,response);
            JdbcUtils.commitAndClose();//提交事务
        } catch (Exception e) {
            JdbcUtils.rollbackAndClose();//回滚事务
            e.printStackTrace();
        }
    }

    @Override
    public void destroy() {

    }
}
```





### 三、将所有异常都统一交给 Tomcat，让 Tomcat 展示友好的错误信息页面

在 web.xml 中我们可以通过错误页面配置来进行管理。这要求出现错误要抛出去，一直抛给tomcat。即上面所说的Filter也要抛出错误

**xml配置**

```xml
    <!-- error-page标签配置 服务器出错后，自动跳转的页面 -->
    <error-page>
        <!--error-code  表示错误类型-->
        <error-code>500</error-code>
        <!--location  表示要跳转去的页面路径-->
        <location>/pages/error/error500.jsp</location>
    </error-page>

    <!-- error-page标签配置 服务器出错后，自动跳转的页面 -->
    <error-page>
        <!--error-code  表示错误类型-->
        <error-code>404</error-code>
        <!--location  表示要跳转去的页面路径-->
        <location>/pages/error/error404.jsp</location>
    </error-page>
```

**Filter代码：**

```java
public class TransactionFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        try {
            chain.doFilter(request,response);
            JdbcUtils.commitAndClose();//提交事务
        } catch (Exception e) {
            JdbcUtils.rollbackAndClose();//回滚事务
            e.printStackTrace();
            throw new RuntimeException(e);// 把异常抛给tomcat统一展示友好的错误页面
        }
    }

    @Override
    public void destroy() {

    }
}
```



体现了，web.xml除了可以配置 JavaWeb 三大组件（Servlet、Listener、Filter）之外，还可以配置 Tomcat 对该工程的错误处理< error-page>



# JSON、AJAX、i18n



## JSON

​		**JSON** (JavaScript Object Notation) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。JSON 采用完全独立于语言的文本格式，而且很多语言都提供了对 json 的支持（包括 C, C++, C#, Java, JavaScript, Perl, Python 等）。 这样就使得 JSON 成为理想的数据交换格式

​	json 是一种轻量级的数据交换格式 

​	轻量级指的是跟 xml 做比较

​	数据交换指的是客户端和服务器之间业务数据的传递格式

### json在javaScript中的使用



​	一般我们要在客户端和服务器之间进行数据交换的时候，使用 json 字符串。 以下是Json字符串与对象、数组之间的转换：

​		JSON.stringify() 	把 json 对象转换成为 json 字符串 

​		JSON.parse() 		把 json 字符串转换成为 json 对象

### json在java中的使用

我们利用谷歌提供的 `gson-2.2.4.jar` 这个工具包

#### javaBean与json

```java
        Person person=new Person(1,"国哥好帅");

        //创建Gson实例
        Gson gson=new Gson();

        //toJson(Object)方法可以把java对象转成字符串
        String personJsonStr = gson.toJson(person);


        //fromJson(String,Class<T>)可以把json字符串转换成javaBean对象
        Person person1 = gson.fromJson(personJsonStr, Person.class);
```



#### List与json

```java
        List<Person> personList=new ArrayList<>();
        personList.add(new Person(1,"国哥"));
        personList.add(new Person(2,"康师傅"));

        //创建Gson实例
        Gson gson=new Gson();

        //把List集合转化成json字符串
        String personListJsonStr = gson.toJson(personList);


        //把json字符串转换成List集合
        //由于要转换为List<Person>类型，不能通过Class对象的方式，只能通过Type对象的方式才能包含泛型信息
        //取得包含List<Person>信息的Type对象：XX extends TypeToken<List<Person>> ，其中TypeToken是gson包提供的一个泛型类
        // 利用 TypeToken<List<Person>> 对象的getType方法即可
        //另外也可以使用匿名类(不能直接new，只能匿名类继承后new)，new TypeToken<List<Person>>(){}.getType()
        List<Person> list = gson.fromJson(personListJsonStr, new PersonListType().getType() );


```



#### Map与json

```java
        Map<Integer,Person> personMap=new HashMap<>();
        personMap.put(1,new Person(1,"国哥好帅"));
        personMap.put(2,new Person(2,"康师傅好帅"));

        //创建Gson实例
        Gson gson=new Gson();

        //把Map集合转化成json字符串
        String personMapJsonStr = gson.toJson(personMap);


        //把json字符串转换成Map集合
        //由于要转换为Map<Integer,Person>类型，不能通过Class对象的方式，只能通过Type对象的方式才能包含泛型信息
        //取得包含Map<Integer,Person>信息的Type对象：XX extends TypeToken<Map<Integer,Person>> ，其中TypeToken是gson包提供的一个泛型类
        // 利用 TypeToken<List<Person>> 对象的getType方法即可
        //另外也可以使用匿名类(不能直接new，只能匿名类继承后new)，new TypeToken<Map<Integer,Person>>(){}.getType()
        Map<Integer,Person> personMap1 = gson.fromJson(personMapJsonStr, new PersonMap().getType());

```

 

#### 注意

普通对象、Map 的 json 字符串形式为   {  key1：val1，key2：val2，...   }

List的 json 字符串是   [  val1，val2，...   ]





## AJAX

​		AJAX 即“Asynchronous Javascript And XML”（异步 JavaScript 和 XML），是指一种创建交互式网页应用的网页开发技术

​		ajax 是一种浏览器通过 js 异步发起请求，局部更新页面的技术。 

​		Ajax 请求的局部更新，浏览器地址栏不会发生变化 ，局部更新不会舍弃原来页面的内容。即有js对象发起请求，也由这个js对象接受响应内容。而非通常的由浏览器发起请求与接受响应

​		另外，ajax请求的同异步：同步表示只有对象收到响应后才能继续往下执行。异步表示不必等待对象接收到响应也可以继续往下执行，而是一边对象接受响应，一边继续往下执行



### 原生javaScript的ajax请求

主要是使用 **XMLHttpRequest** 对象来发起请求和接受返回数据

```html
		<script type="text/javascript">
			//在这里使用javaScript语言发起Ajax请求，访问服务器AjaxServlet中的javaScriptAjax
			function ajaxRequest() {
				
// 				1、我们首先要创建XMLHttpRequest
				var xmlHttpRequest=new XMLHttpRequest();

// 				2、调用open方法设置请求参数(请求方式， url地址，是否异步)，true异步，false同步
				xmlHttpRequest.open("GET","http://localhost:8080/16_json_ajax_i18n/ajaxServlet?action=javaScriptAjax",true);

// 				3、在send方法前绑定onreadystatechange事件，处理请求完成后的操作。
				xmlHttpRequest.onreadystatechange=function () {
					if(xmlHttpRequest.readyState==4&&xmlHttpRequest.status==200){
						var person = JSON.parse(xmlHttpRequest.responseText);
						$("#div01").text("编号："+person.id+"  姓名："+person.name);
					}
				}

// 				4、调用send方法发送请求
				xmlHttpRequest.send();


			}
		</script>
```

数据传递是这样的：

1、对象发起请求

2、服务器返回数据给对象的responseText或responseXML属性

3、我们可以通过实现设置的事件响应函数来操作返回的数据



### jquery的ajax请求



#### $.ajax 方法 

* url 		表示请求的地址  

  

* type 		表示请求的类型 GET 或 POST 请求 

  

* data 		表示发送给服务器的数据 

​							格式有两种： 

​										一：name1=value1&name2=value2 

​										二：{key1:value2，key2:value2} 

​							如果为数组，将自动为不同值对应同一个名称。如 {foo:["bar1", "bar2"]} 转换为 

​							"&foo=bar1&foo=bar2"



* success 	请求成功，响应的回调函数，注意回调函数需要一个参数接收响应数据 

  

* dataType 	响应的数据类型 

​							常用的数据类型有： 

​									text 表示纯文本 ，对响应数据不做处理直接调用success的响应函数

​									xml 表示 xml 数据 ，

​									json 表示 json 对象，对响应数据转换为 js 对象后再调用success的响应函数，无需手动转换

**使用模型：**$.ajax(  {key1:val1, key2:val2, ... }  )     即通过键值对配置参数

示例代码：

```javascript
    $.ajax({
        url:"http://localhost:8080/16_json_ajax_i18n/ajaxServlet",
        data:{action:"jqueryAjax"},
        type:"GET",
        success:function (data) {
            // var person = JSON.parse(data);
            // alert("服务器返回的数据是："+data);
            $("#msg").html("编号："+data.id+", 姓名："+data.name);
        },
        dataType:"json"

    });
```



#### $.get、$.post

* url 	请求的 url 地址 

* data 	发送的数据 

* callback 	成功的回调函数 

* type 	返回的数据类型

**使用模型：**$.ajax(  url, data , callback, type  )     即直接传递参数，不能使用 { key1：val1，... } 的键值对传参数，这与$.ajax不同

示例代码如下：

```javascript
					$.get("http://localhost:8080/16_json_ajax_i18n/ajaxServlet",{action:"jqueryGet"},function (data) {
						$("#msg").html("get 编号："+data.id+", 姓名："+data.name);
					},"json");



					$.post("http://localhost:8080/16_json_ajax_i18n/ajaxServlet",{action:"jqueryPost"},function (data) {
						$("#msg").html("post 编号："+data.id+", 姓名："+data.name);
					},"json");
```



#### $.getJSON

​	url 	请求的 url 地址 

​	data 	发送给服务器的数据 

​	callback 	成功的回调函数

**使用模型：**$.getJSON(  url, data , callback )     即直接传递参数，不能使用 { key1：val1，... } 的键值对传参数，这与$.ajax不同

示例代码：

```javascript
					$.getJSON("http://localhost:8080/16_json_ajax_i18n/ajaxServlet",{action:"jqueryGetJson"},function (data) {
						$("#msg").html("JSON 编号："+data.id+", 姓名："+data.name);
					});
```



#### 小结

* $.ajax 需要5个参数（url，type ，data，success，dataType），前3个参数确定发送，后面2个参数确定接收
* $.get与$.post省略了 type参数，因为方法默认使用GET/POST请求
* $.getJSON省略了 type 、dataType 参数，默认 type=“GET”、dataType=“json”
* 只有$.ajax是用 {key1:val1 ,key2:val2, ....}的键值对确定参数的，其他方法直接传参



注意：jquery的ajax请求的回调函数的参数不是直接使用服务端传递过来的数据，而是根据设置的返回数据类型做出相应的处理或不处理后再调用回调函数



### 表单序列化 serialize() 

​		serialize()可以把表单中所有表单项（设置了disabled属性为true或"disabled"的表单项除外）的内容都获取到，并以 name=value&name=value 的形式进行拼接

示例代码：

```javascript
var serialize = $("#form01").serialize();
```



## i18n国际化

稍作了解即可



# Jedis

是java用来连接redis的工具

## 使用前提

需要导入 `jedis-3.5.0.jar` 包

## 用法

普通

```java
        //连接redis
        Jedis jedis=new Jedis("127.0.0.1",6379);
        //设置密码
        jedis.auth("12345");
        //测试连接
        System.out.println(jedis.ping());
		//普通的set、get
        jedis.set("k1","v1");
        String k1 = jedis.get("k1");
```

事务

```java
 		Transaction transaction = jedis.multi();
        transaction.set("balance","90");
        transaction.set("debt","10");
        transaction.exec();
```

带有watch的事务( 如果提交时，watch的内容发生改变，就提交不成功 )

```java
        jedis.watch("balance");//监控balance

        Transaction transaction = jedis.multi();
        transaction.set("balance","90");
        transaction.set("debt","10");
        transaction.exec();
```

设置主从

```java
        //连接redis
        Jedis master=new Jedis("127.0.0.1",6379);
        Jedis slaver=new Jedis("127.0.0.1",6380);
        //设置密码
        master.auth("12345");
        slaver.auth("12345");

        //设置主从关系
        slaver.slaveof("127.0.0.1",6379);

        master.set("class","1122gg");

        String aClass = slaver.get("class");

        System.out.println(aClass);
```

## Jedis 连接池

池化代码如下:

主要是通过 JedisPoolConfig 对象配置好相关参数后，直接 new JedisPool 即可得到 redis 连接池

```java
public class JedisPoolUtil {

    private static volatile JedisPool jedisPool=null;

    public static Jedis getJedisInstance(){
        if(jedisPool==null){
            synchronized (JedisPoolUtil.class){
                if(jedisPool==null){
                    JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
                    jedisPoolConfig.setMaxActive(1000);
                    jedisPoolConfig.setMaxIdle(32);
                    jedisPoolConfig.setMaxWait(100*1000);
                    jedisPoolConfig.setTestOnBorrow(true);

                    jedisPool=new JedisPool( jedisPoolConfig, "127.0.0.1", 6379);
                }
            }
        }
        return jedisPool.getResource();
    }

    public void release(JedisPool jedisPool, Jedis jedis){
        jedisPool.returnResource(jedis);
    }


    private JedisPoolUtil(){}

}
```

