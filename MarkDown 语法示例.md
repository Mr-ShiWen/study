# MarkDown 语法示例

## *代码*

**1.1 江南的微博**



```java
public static void main(String[] args) throws Exception{
    //准备好驱动管理类（给它注册一个驱动类实例）、相关的连接信息
    Class.forName(driver);//加载驱动类，并初始化它。Driver的静态代码块会给DriverManager注册一个Driver实例
    String url="jdbc:mysql://localhost:3308/shop?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=GMT%2B8";
    String user="root";
    String passWord="199402160579shch";
    //创建连接、statement实例
    Connection connection=DriverManager.getConnection(url,user,passWord);
    Statement statement=connection.createStatement();
    ResultSet resultSet;
    //执行
    String sql="select product_name,product_type,sale_price ,sum(sale_price) over (partition by product_type order by sale_price)as sumPrice from product";
    resultSet=statement.executeQuery(sql);
    //输出
    int i=0;
    while (resultSet.next()){
        System.out.print(resultSet.getString("product_name")+",");
        System.out.print(resultSet.getString("product_type")+",");
        System.out.print(resultSet.getString("sale_price")+",");
        System.out.print(resultSet.getString("sumPrice"));
        System.out.println();
    }
    //关闭
    resultSet.close();
    statement.close();
    connection.close();
	//以下为了控制台执行完程序后不闪退
    Scanner in= new Scanner(System.in);
    in.next();
}
```

```java
String a=new String("Hello World");
System.out.println(a);
```

## **图片**

Spring 学习路线

![冰岛钻石沙滩风景4k壁纸_彼岸图网](C:\Users\施汶\Pictures\Saved Pictures\冰岛钻石沙滩风景4k壁纸_彼岸图网.jpg)

# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

###### 六级标题

**加粗字体**

*倾斜字体*

***加粗&倾斜字体***

~~带删除线字体~~

> 这是引用内容 

> > 这是引用的引用

---

傍晚风景图

![ alt showMe ](C:\Users\施汶\Pictures\Saved Pictures\26c5cae2ad671ecd98167ebd712a31a5.jpg)

这是一网图

![loading.png](http://upload-images.jianshu.io/upload_images/1503319-c696a9cd1495d68f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



> * 整理知识

* 这是一个点点
* * 这是两个点点

注意：点点可以通过 enter 键消除

`单行代码`



<a href="#代码">跳到代码</a>      //跳转只能跳到标题，如果href属性值不是某个标题，是跳不过去的。当然这里标题不限制级数（包含typora支持的1到6级标题），都可以跳转



[超链接](F:\研究生学习资料\流程图\JVM流程图\JVM的类加载器.jpg)