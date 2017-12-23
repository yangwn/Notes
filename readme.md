## 第一章：Maven的奇技淫巧（坑）
Maven的pom文件详解  
	1：<modelVersion>4.0.0</modelVersion>描述这个项目遵从那个版本,最低是4.0.0,maven官方要求  
	2：Maven项目聚合做法 
		假设现在有项目A,项目A1,项目A2，要求项目A是负责聚合项目A1和A2，实现多个模块联合编译，实现起来很简单
		只需要在A的pom文件中，添加这么一段配置
	
			<modules> 
   				<module>A1</module>
   				<module>A2</module>
			</modules>

3：比较坑的一点是，项目的maven依赖视图能找到某依赖，但是在其pom文件找不到。  
	   这样编译时总是找不到某程序包，还以为出了什么鬼的bug，结果万万没想到，居然是没写依赖。。。

注：A1通过A项目的pom文档的artifactId而确定
		这样，编译A项目，就会把A1和A2项目一起编译
		接下来有个需求，项目A1和A2使用同一个依赖，难道要各自使用各自的依赖包吗？  
		以上叫做模块聚合，接下来就是模块间的继承，这继承，第一个就是能子项目继承父项目引用的依赖包  
		假设父项目的pom文件是这样的
		
		<modelVersion>4.0.0</modelVersion>  
		<groupId>com.Example.main</groupId>              
		<artifactId>Parent-Moduel</artifactId>       
		<version>1.0.2</version>            
		<packaging>pom</packaging>  
		<name>Simple-main</name>

那么子项目就可以这么写：

		<parent>
   			<groupId>com.Example.main</groupId>
   			<artifactId>Parent-Moduel</artifactId>
   			<version>1.0.2</version>
   			<relativePath>../pom.xml</relativePath>  <!--本例中此处是可选的-->
	   </parent>

值得注意的是`<relativePath>`标签，如果pom的层次关系就像本例中的那样只隔一层，则可以省略这个。  
maven同样可以找到子pom。  
子pom中引入<parent>标签后，就会从父pom继承<version>等属性了  
父类添加这样的依赖：


     <dependencyManagement>
    		   <dependencies>
    		      <dependency>
    		           <groupId>javax.servlet</groupId>
    		          <artifactId>servlet-api</artifactId>
    		          <version>2.5</version>
    		      </dependency>
    		   </dependencies>
    	</dependencyManagement>

子pom如果需要引用该jar包，则直接引用即可！不需要加入<version>，便于统一管理。当然如果你加version的话，表明这个依赖是子项目特有的
		然后插件也可以这么管理
		主项目：
	
	<build>
		   <pluginManagement>
		      <plugins>
		          <plugin>
		               <groupId>org.apache.maven.plugins</groupId>
		               <artifactId>maven-source-plugin</artifactId>
		               <version>2.1.1</version>
		          </plugin>
		      </plugins>
		   </pluginManagement>
		</build>

子项目：

		<build>   
		   <plugins>
		      <plugin>
		           <groupId>org.apache.maven.plugins</groupId>
		           <artifactId>maven-source-plugin</artifactId>
		      </plugin>
		   </plugins>
		</build>

不用加version了，便于管理。  
### 2017-5-30日6月1日bug:
1:在Eclipse运行Maven命令没反应或者爆这个错:  
`-Dmaven.multiModuleProjectDirectory system property is not set. Check $M2_HOME environment variable and mvn script match.`
理由。安装的Maven太高级了，Eclipse不认识他，安装个旧版本的  
或者进入eclipse的Window->Preference->Java->Installed JREs->Edit，在Default VM arguments中设置-Dmaven.multiModuleProjectDirectory=$M2_HOME  
告诉Eclipse去哪里找这个高版本的Maven..前提一定要设置M2_HOME这个环节变量啊，别告诉我你不会设置。  

2:有时候下载依赖失败后再次下载就不行了。你得手动去删除依赖的那个文件夹的_maven.repositories和_lastUpdate....这两个文件，然后重新update项目。  
还有，仓库有时候依赖是个pom文件，建议不要引用该依赖，要下那些有jar包的该依赖

### 2017-8-28 晚BUg：
无论clean install多少次，resource文件夹里面的配置文件都没有发布到war包中。
最后发现是resource写错了，应该是resources

### 2017-8-29 晚BUg：
老实说，这个BUG早知道了，也有应对之策。  
可是应对之策忘了，所以特此补充  
具体BUg的描述就是用Maven打包的项目后，包里面没有Mybatis的实体类映射文件  
众所周知，这个映射文件一般是跟着实体类放在一起的，也就是`src/main/java`文件夹里面，  
但是Maven打包只打包`src/main/java`文件夹里面的java文件，xml配置文件不会打包。  
所以我们需要这个配置  

	<resource>
	          <directory>src/main/java</directory>
	          <includes>
	              <include>**/*.properties</include>  
	                <include>**/*.xml</include>  
	          </includes>
	          <filtering>false</filtering>
	      </resource>
	      <resource>  
	        <directory>src/main/resources</directory>  
	            <includes>  
	                <include>**/*.properties</include>  
	                <include>**/*.xml</include>  
	            </includes>  
	            <filtering>false</filtering>  
	        </resource>   
 这下就可以把实体类的映射文件一起打包了，爽不？ 

### maven全局或者局部设置java编译版本
在setting文件中，补上这份代码  
​	
	<profile>   
	    <id>jdk1.6</id>
	    <activation>   
	    <activeByDefault>true</activeByDefault>
	    <jdk>1.6</jdk>   
	    </activation>
	    <properties>   
	        <maven.compiler.source>1.6</maven.compiler.source>
	        <maven.compiler.target>1.6</maven.compiler.target>
	        <maven.compiler.compilerVersion>1.6</maven.compiler.compilerVersion>   
	    </properties>   
	</profile>  
保存即可  
如果全局设置失败，试下局部设置  
在pom 文件中补上这个代码  

	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-compiler-plugin</artifactId>
	    <configuration>
	        <source>1.6</source>
	        <target>1.6</target>
	        <encoding>${project.build.sourceEncoding}</encoding>
	    </configuration>
	</plugin>


## 第二章：杂项

### 1.BigDecimal的学习  
		构造函数  
		BigDecimal(double val);  
		BigDecimal(int val);  
		BigDecimal(String val);  
		分别可以将double，int，String代表的数字转成BigDecimal对象  
### 2:HttpClient分两个阶段版本，有些时候下错了就悲剧了  
		目前最新阶段的最新版本的依赖  

		<dependency>
  		 	<groupId>org.apache.httpcomponents</groupId>
    		<artifactId>httpclient</artifactId>
    		<version>4.5.2</version>
    	</dependency>

其HttpPost对象可以设置实体内容。而get不行！  
其HttpClient对象这么创建：CloseableHttpClient httpclient = HttpClients.createDefault();   

### 写接口注意事项
写接口时切记沟通发送报文和接受报文的编码！
接口一定要用HTTPClient测试一下！


### 写方法或者整理前人的方法时：
1. 注意把逻辑分为几部分，在方法里面分由其他几个私有方法来处理。程序看起来会更清晰
2. 把主要的分支判断逻辑放在

### 今天教下大家，怎么把一个文件夹打包成war包  
其实很简单，就跟打包成jar包一样  
`jar -cvf  web.war  .  `
最后一个小点不要去掉，这表示当前目录下（不包括该目录）下的所有目录都归档。  
什么，最后一个小点你要换成path。你的想法很好，可惜，这样打成的包会把路径信息也保留下来。。。  
什么玩意啊。。  

## 第三章：NOSQL
NOSQL意为非关系型数据库，分为几种  
1：基于键值对的-redis最强  
2：列存储，Hbase、Cassandra这种  
3：文档存储：MongoDB  
4：图片存储：Neo4j、Versant  
5：xml存储：Berkeley DB Xml还有XBASE， 
很早已经支持这种存储方式了  
### redis的基础运用场景 
1：多web项目中共享一个session  
2：分布式缓存，由于redis提供了几大语言的接口，比如java，.net,c等语言。  
		因此对于异质平台间进行数据交换起到了作用，因此它可以用作大型系统的分布式缓存，  
		并且其setnx的锁常被用于”秒杀“，”抢红包“这种电商活动场景中。  



### 第四章：SPring的奇技淫巧
1：Springmvc控制器方法上的produces 要注意一下，仅当Accept请求头和produces注解指明的媒体类型一致时，
比如说produces=text/plain   发来请求的请求头中Accept=text/plain  ，该请求才会被接受！否则报错！  

2：@ResponseBody  其返回的数据类型主要看你的classpath有没有相关的第三方jar包，比如说jackson.jar  
这样的话返回的数据就会转成json格式输出  
一般情况下开发的框架都是加了json相关的jar包，也就是说@ResponseBody返回的都是json数据。  
所以要返回一个xml格式的。。。你还是老老实实的用响应流来输出吧。。  
注：就算@ResponseBody返回一个字符串，就平台来说，也会给你字符串加上两个隐形的翅膀（字符串首尾加括号）
什么，你想setContentType("text/plain; charset=utf-8");  
@ResponseBody到最后还是会变成application/javascript;charset=utf-8的。。。



经测试可以成功的本地依赖。。。
 <dependency>
            <groupId>jdk.tools</groupId>
            <artifactId>jdk.tools</artifactId>
            <version>1.7</version>
            <scope>system</scope>
            <systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
</dependency>


@RequestParam(value = "custom", required = false)可以非必传参数。
多个Springmvc参数，字符串形式，必须加@RequestParam("factoryCode")标注参数名
一个参数就不需要


平台的特性？ajax不加这个东西就给我返回一个字符串dataType : "jsonp"？
嗯？要获得json对象只能加这个属性

----------


在任务交付之前，一定要检查一遍代码
包括但不限于
代码的健壮性（为空判断，空指针检查）
代码的隐患检查（没小心判断为空就直接调用）
代码的语义规范检查：包括变量名大小写规范，类名规范
功能性检查，能否完成功能。。
特别注意：拷贝后修改的代码更是要仔细检查一遍！	
代码的类名方法名是否注释是否过时！	
------------------------------------------注意分割线---------------------------------------------------	

------------------------------------------马云家代码规范笔记-----------------------------------
前记：只记我还不知道的部分，一些烂大街的规范大家都懂得吧，一个好程序员不仅程序写的6，规范也要6
1：
	常量命名全部大写（很少用到，权且一记）
2：
	抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类命名以它要测试的类的名称开始，以Test结尾。
3：
	数组定义请用 String []  args 而不要用 String args [] 
	为什么？第一个这样写很明显的，前者是字符串数组类型，后者是它的应用。第二个这么写，前者很明显是字符串类型，后者是其数组类型的一个引用。
	单从语句执行情况没有区别，但是前者把数据类型和引用分开了，更清晰。
	况且，使用后者，万一出现这种情况，String args[],str ;也许你写的时候知道str是字符串类型，但是后来的维护人员想骂街，一行定义两个数据类型，坑！
4：
	包名统一小写啊（开发很少需要写包名，自己写项目的时候就要注意了）
5：
	接口类中的方法和属性不要加任何修饰符号（public 也不要加）
	个人倒觉得无可厚非，加了更明显的标注接口的功能，再者，接口本来代码就不多。简洁嘛，也没多大必要
	标注是为了开发时注意点，个人写就随意了
6：
	自己添加，代码里面的公有常量最好封装在一个工具包的枚举中，
	枚举出现的原因就是为了服务公有变量，枚举不能实例化符合工具类不能实例化这个需求。
7：
	接口类后缀名是Service，实现类后缀名是ServiceImpl
8:
	枚举类名建议带上Enum后缀，枚举成员名称需要全大写，单词间用下划线隔开。
9:
	业务层数据层方法规范
	获取单个对象用getxxx
	获取列表对象用listxxxx
	获取总数什么的用countxxxx
	插入方法的用insert或者save
	删除方法用remove或者delete
	修改方法用update
	注意：不是说阿里云叫你这么做你就非得怎么做，遵守项目组的规范，看看前人写的方法名，
	照犬画狗即可，当然如果项目组没规范可讲，你就根据上面的规范吧。。。。
10：抛异常时想清楚这个异常是什么情况的，不要直接在方法名上throws Exception 就行了

11：额，那个，把不需要的引入去除掉。
12:接手或者修改别人代码之前注意看别人是怎么实现了。。。！
13:抛异常把异常信息说清楚点。。。还有封装代码时思考一下这么做值得吗？
14:如果一个对象已经是注入的了，就不要用程序来再次修改它的状态
	比如说有一个对象通过ioc注入，在使用时为了方便还在程序设置了几个对象属性。这是不合理的。
***
*git的readme.md的编写*
1. 3个*号作为一行即可创建一个分割线
2. 数字加英语句点即可创建一个有序列表
3. 虽然 *``*可以写入代码，但是结构上是不换行的，还是pre标签里面写代码比较好
  *EGIT的全局忽略文件*
  只要在你在eclispe有安装EGIT，就可以在用户文件夹里面发现一个.gitconfig文件，里面就是关于git的全局配置，配置全局忽略的文件也在这里配，那么怎么配呢？  
4.  在用户文件夹里面再创建一个文件，就叫.gitignore 
5.  在新建的文件写上要过滤的文件，eg：
  `.setting
  .project
  target
  bin`  
  一行一个。#表示注释
6. 在.gitconfig文件写上
   `[user]
   name = XiaoHang
   email = 1083594261@1-PC
   [core]
   excludesfile = .gitignore
   `

   ***
7.   JSON数据其实也有数据格式。别看它是js对象还是字符串。
  什么意思呢？
  16:修改maven全局配置的jdk版本，setting.xml添加
   <profile>   
   <id>jdk1.6</id>    
   <activation>   
       <activeByDefault>true</activeByDefault>    
       <jdk>1.6</jdk>   
   </activation>    
   <properties>   
       <maven.compiler.source>1.6</maven.compiler.source>    
       <maven.compiler.target>1.6</maven.compiler.target>    
       <maven.compiler.compilerVersion>1.6</maven.compiler.compilerVersion>   
   </properties>   
  </profile> 
8. 局部配置jdk版本的做法，在项目的pom文件加上
  <build>  
   <plugins>  
       <plugin>  
           <groupId>org.apache.maven.plugins</groupId>  
           <artifactId>maven-compiler-plugin</artifactId>  
           <configuration>  
               <source>1.6</source>  
               <target>1.6</target>  
           </configuration>  
       </plugin>  
   </plugins>  
  </build>  
  18:看来最新版本的SpringMVC不是默认为你添加Json支持，只是说，你在mvc的配置文件加上<mvc:annotation-driven />
  自动为你配置
  DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter两个bean，配置一些messageconverter。
  当然也包括Json支持
9. 为什么一般情况下，服务器都不允许接受跨域请求呢？
  这其实是为了安全
  	设想一下。你在xx银行登录了个人信息，然后xx银行服务器保存了你的Cookie
  	接着你被骗子引诱到了一个山寨网站。你进入这个网站的那一刻，这个山寨网站就向xx银行发送了一个ajax请求。
  	这就是跨域请求。这种请求不被阻止的话，银行就会发回你的个人信息。你的个人信息就泄漏了。
  20 :接下来的学习步骤  ：Eclipse插件的开发-能方便修改jar包源码   Lambda表达式的学习
  21. 用cglib作为代理类的生成，生成Controller代理类。如果原来的Controller的requestMapping方法有private的话，那么cglib无法代理这个方法，造成的结果就是当前端访问到这个接口时，无法读取到IOC注入的变量。其他public方法就可以造成访问。其中缘由，
     22:
     Arrays.asList的方法返回的不是我们常见的java.util.ArrayList，而是java.util.Arrays.ArrayList。两种名字一样实际可大不同。后者仅仅是对数组的一种封装而不能进行修改操作，执行修改操作都报UnsupportedOperationException异常
     然后Arrays.asList方法里面参数只能放引用类型，不能放基本类型，否则都会解析成数组类型。
     例如Arrays.asList(1,2,4);会把1，2，4当做整体，即数组。长度为1.
     两个坑记住了  
10. 今日坑，如果要传一个数组给后端，ajax请求要加一个参数，traditional: true
  RT:
  <pre>
  $.ajax({
  					url: 'xxxx',  
  					traditional: true,
  					type: "get",  
  					dataType: "json",
  					data:{
  						parameter_1 : id,
  						parameter_2 : arr_1,
  						parameter_2 : arr_2 
  					}
  </pre>
  这样才能传到后端，当然了，arr_1和arr_2都是js的数组对象。
  原因：阻止jquery对参数的深度序列化
11.  如何传一个复杂对象到Springmvc的控制器中？
   其实这部分应该是Spring知识，下次有空将其补上
   回到正题，如何传呢？
   url地址 用对象的属性.内部属性来传，ajax 如下	
   	$.ajax({
   	  			type:"get",
   	  			url:"xxxx",
   	  			data : {
   	  				    "requestBody.user ": "admin:10000",
   	  					"requestBody.requestType" :"01100061",
   	  					"requestBody.password" : "10086",
   	  					"requestBody.serverVersion" : "00000000",
   	  					"requestBody.uuid" : "UUID" ,
   	  					"requestBody.flowintime" : "2017/6/7 22:22:22",
   	  					},
   	  ......
   	就酱

##第三章：Springmvc
1.给Springmvc的控制器传对象我想大家都知道了吧，不就是对象.属性=xxx传键值对嘛....<br/>
	要换个说法，传个json字符串让控制器接受并自觉实例化成对象，怎么做？
	别瞎比比了，亮代码
<pre>@RequestMapping("/xxxx")
 		     public String  action (@RequestBody DsspRequestVo vo,
 		 																		HttpServletRequest request,
 		 																		HttpServletResponse response,
 		 																		ModelMap model) throws IOException{return null}  </pre>
 		 后台代码酱紫，重要的是那个@RequestBody  
 		后台代码很简单，重要的是前台代码
<pre> 	
			$.ajax({
			type:"post",
			url:"xxxx",
			traditional: true,
			contentType: 'application/json',
			data :JSON.stringify(obj),
				success:function(data){
					 xxxx
				},
				error : function(data){
					xxxx
				}
			}); 
 </pre>  
 		关键点：  
 		contentType: 'application/json' 没加，报类型不匹配，毕竟加了@RequestBody   
 		mvc就很智能的帮你找这部分请求了，不合请求的 当然就踢走了
 		traditional: true,阻止jquery将你的字符串序列化成键值对。
 		JSON.stringify(obj)  毕竟真正在网络传输的其实是字符串，传个对象是不科学的。
 		js的对象组合也是很重要的，不过说白了也就是对象.属性.属性=xxxx  酱紫。。。
施工完毕

##第四章：基础不扎实篇
1. java传对象给方法时，传的是对象的地址，方法体内用另一个参数来接受这个地址，若是方法体内操作这个地址
  亦即操作对象，set一个属性什么的。方法体外仍可以得知这个改动。。<br/>
  但是，如果方法体内另一个参数接受到传进来的对象的地址后，擅自把该参数指向另一个地址，也就是new。<br/>
  那么。方法体外的对象和对象引用不会受任何影响<br/>
  地址本身就是一个值，java把地址拷贝了一份发给方法，所以也就是说为什么java只有值传递的原因了。
  要是引用传递，还拷贝个毛。
  偷来的一个截图为例<br/>
  ![吔屎啦，图片显示不出来](https://github.com/XHang/Node/blob/master/java%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92%E7%A4%BA%E4%BE%8B.png)

2. 泛型篇---上界通配符和下界通配符详解
  Plate<？ extends Fruit>  上界通配符，?表示Fruit类以及Fruit的派生类（子类）  
  代码示例

  public class Plate<T>{
  	private T item;
  	public Plate(T item){
  		this.item = item;
  	}
  	public void set(T item){
  		this.item = item;
  	}
  	public T get(){
  		return this.item;
  	}
  }

这样的话，Plate<? extends Fruit> p 这个引用变量就可以接受所有Fruit以及Fruit的子类对象了  
如: `Plate<? extends Fruit> p = new Plate<Apple>(new Apple())`   水果盘子可以放苹果，很科学  
不过，这个水果盘子只能放一次水果，就是在它new的时候，你再执行set方法就不行了。  
虽然编译器知道水果盘子里面放的是水果，可是它并不知道是什么水果，所以你放什么水果进去它都不知道能不能匹配，当然就不让你放了。
取水果是有效的，因为编译器虽然不知道你放的是什么水果，可它至少是水果，并且还是对象。
只要用这两种类型来接受就OK了

Plate<？ super Fruit> 下界通配符，'？'表示水果以及水果类的父类。
换句话说，这个盘子可以放水果和食物。。
​	
	`PS：虽然说可以放食物，你可不能往里面放猪肉。它可以放的是食物的实例，因为食物是水果的父类。
	        猪肉是水果的父类吗?显然不对`
下界通配符 可以这么使用： `Plate<？ super Fruit> plate = new Plate<Food>(new Food())`
这样的话编译器知道你搞了一个盘子，里面可以放水果或者水果以上的食物，但具体是什么它也不清楚。  
所以放苹果是合适的，苹果既是水果，又是食物。  
但是你不能放水果的父类，因为可能这个盘子一开始new的时候只允许放水果。  
说的再直白一点就是编译器虽然不知道你葫芦里卖的什么药，可它起码是药，放药进去就行了。  
其实不太科学的说，我new了一个放食物的盘子，居然只能放水果，够憋屈的。  
更憋屈的还有，你取出来的话，只能用Object来接受...还是那句话，编译器不知道你放进去的是什么东西。  
所以只能推断为最顶层的基类：Object。你再怎么放东西，它最终都是一个对象  
以上。
​	
4. Error会回滚吗？
  此问题将重定向到Error错误类可会捕捉到？
  因为回滚依赖的是catch子句，如果错误不能被catch子句捕捉到，那就不用谈了  
  答案：能捕捉到
  所以：
5. Error的异常能捕捉到
6. Error的异常能使事务回滚，前提是设置好回滚异常

7. 类的静态变量和静态代码块什么时候执行？  
  初始化。。。那么什么时候进行初始化呢？    
  可不是类加载时，比如说，Class class= A.Class  
  这个时候类加载了，但却没有初始化  
  类的初始化主要由下面几个因素触发  

    当创建某个类的新实例时（如通过new或者反射，克隆，反序列化等）
    当调用某个类的静态方法时
    当使用某个类或接口的静态字段时
    当调用Java API中的某些反射方法时，比如类Class中的方法，或者java.lang.reflect中的类的方法时
    当初始化某个子类时
    当虚拟机启动某个被标明为启动类的类（即包含main方法的那个类）

这是一般的情况，但是从web环境下运行，类的静态成员也是由以上的因素触发初始化吗？    
某人认为：调用前就初始化了，因为静态成员是作为一个类变量存在，在这个类被加载到虚拟机里面时(或者tomcat运行容器)就已经被初始化了    
实验下。。。  
实验条件：标准的SSM项目，三个测试  
1. 仅写好类  
2. 调用一下类.class  
  3.调用类的静态属性  
  结果只有调用类的静态属性才会触发类的初始化，结论，百度都是骗人的  

但是，比较特殊的是，一旦类初始化失败，在下面的web应用中，将不能再使用这个类了。  
除非重启并正确地初始化该类  


7. 定义接口的错误信息时，不要总是想自定义错误信息。一旦你的程序需要事务管理的话，你返  回错误信息，相当于程序不报错，不抛异常。事务管理自然失效。。  
  正确的做法是既能抛异常，又能把错误传递给对方。。  

8. 如何在Maven项目中，用相对路径取得resources里面的文件呢？
  用这个代码  `类名.class.getResourceAsStream("/文件名");`
  具体原理不是很清楚。大概就是从classpath路径下搜索所有符合条件的文件吧

9.
jdk7引入了一个工具类，专门是Object的工具类：Objects
原有的equals方法都可以使用Objects类

10.
怎么把经过url编码转换成中文？  
假设现在通过Socket协议连接时客户还往连接的url地址加一些中文参数，那么怎么在服务端把这些文字解码成中文呢？  
说来也挺简单的  
`new String(URLDecoder.decode(要转换的字符串, "utf-8")`  

11.
绝大多数dao框架，都不允许写这种sql语句  
`select * from user where userName like '%?%'`  
然后setString。  
这样会报错说栏目数不对，实际栏目数1，预期栏目数0   
这估计是因为占位符`?`写到了字符串里面  

### 反射
1. 反射中，`class.getFields()`方法只能获取public的字段，`private`方法的字段获取不到，这个方法才可以 `clazz.getDeclaredFields();`
2. ​


##第五章：Eclipse的坑
1. eclipse的构建路径中的order and export 作用是 <br/>
   order就是使用class的顺序(因为可能出现class同名的情况)<br/>
   export就是把用到的一些的lib和project同时发布.<br/>
2. 问题描述 ：改了项目的jre和编译jdk版本，但是一执行maven update 全都打回解放前了。  
     经查，settting.xml配置了jre版本，pom文件也设置了jre版本，仍是不见效。
     已知：本人用的Eclipse不支持jdk1.8.所以pom文件即使追加了jre版本，也依然无效		
     最后改了jdk版本为1.7.重新update。版本恢复为1.7了。。		
     付代码
     `     <plugin>    
            <groupId>org.apache.maven.plugins</groupId>    
            <artifactId>maven-compiler-plugin</artifactId>    
            <configuration>    
                <source>1.7</source>    
                <target>1.7</target>    
            </configuration>    
        </plugin>   
        <plugin>    
            <groupId>org.apache.maven.plugins</groupId>    
            <artifactId>maven-compiler-plugin</artifactId>    
            <configuration>    
                <source>1.7</source>    
                <target>1.7</target>    
            </configuration>    
        </plugin>   </pre>`
       未知：setting文件改jdk版本为1.7仍然无效。这个小妖精。。。。

3. 这个问题也不是Eclipse的坑，但却是Eclipse的svn插件，所以也一并放到这里。
  在svn，如果你想恢复的文件夹的父文件夹不存在那么复原是失败的。
  比如说你想复原/web/server 的server文件夹，但是本地系统中web文件夹已经消失了。
  那么复原是失败的，你得先复原web文件夹，才能往下继续复原。。  
  这个特性真糟糕。  

##第六章：正则表达式和java的坑
1.  目前测试得知，java1.7 不支持捕获组里面写无限匹配量词，也就是'+'和'{2,}' 不能用
   eg:(?<=package\\s{1,})  or  (?<=package\\s+)  可以匹配'package   '后面的位置，但是java不支持这种写法
   	考虑可以换成(?<=package\\s{1,10000})
   	顺便贴上msg：Look-behind group does not have an obvious maximum length near index {num}

## 第七章：消息队列
### 何谓消息队列？  
消息队列首先是个队列，有入队，也有出队。  
那么就有    
一个程序专门做生产者来入队。    
一个程序专门做消费者来出队。    
出入对顺序可以是先进先出，也可以先进后出，很像压栈和出栈对吧  
### 使用场景？  
当你的网站并发量无法支撑太多请求，并且这些请求都不是需要立即得到结果的情况下。  
你可以用下消息队列，将请求入队，然后一个一个出队去解决它，减少并发压力。  
其实现实不也是有这种情况吗，你发一个请求，然后服务器告诉你说你的请求已接受，请等到3-5个工作日后查看结果。  
### 常用的解决方案有哪些
像这种东西其实实现起来特别简单，但不要轻易总是想实现它，毕竟业界已经给我们提供了很多轮子了  
比如说RabbitMQ或者zmq甚至redis（这货可以吗）  
先试用几个再考虑要不要造轮子。  
### 最后
什么？这么轻易就结束了？那倒不是，只不过接下来的示例程序在这个文件无法开展，去看看messagequeue项目吧  
什么?没有，那可能是作者挖坑又不填了       
## 第八章：linux的技能
1. openssh安装后如何启动？  
  命令：/etc/init.d/ssh start  必须在su权限下运行

2. 怎么用主机访问VirtualBox呢？
  见图。用PuTTY即可访问

3. vin编辑器怎么使用？  
  用vi  文件路径  即可打开一个文本文件  
  初次进入是以命令行模式打开的，要编辑此文件，敲入i即可进入编辑模式。  
  编辑模式下按esc回退到命令行模式  
  将光标移动到某处，按dd即可删除该行  
   「x」：每按一次，删除光标所在位置的“后面”一个字符。  
  「#x」：例如，「6x」表示删除光标所在位置的“后面”6个字符。  
  「X」：大写的X，每按一次，删除光标所在位置的“前面”一个字符。  
  「#X」：例如，「20X」表示删除光标所在位置的“前面”20个字符。  
  按下：号  
  输入wq!强制保存并退出。。    

4. 利用putt传文件到远程服务器  

putt客户端下载下来一般都有那个pscp.exe文件。进入该文件对应的文件夹，敲入cmd命令  
pscp 发送的源文件   服务器用户名@服务器地址:home 敲入后输入密码，即可发送！  
eg:pscp jdk-8u131-linux-x64.rpm cxh@192.168.21.248:/home  
注意：有时候发送过去但是找不到文件或者发送时提示permission denied  
就是访问被拒绝了，这时候你得手动更改远程服务器的文件夹为可读可写  chmod 777 xxxx  
注：pscp -r 后面指定文件夹名可以远程传输文件夹  
5. 安装，删除命令：  
  Ubuntu版：  
  dpkg -l 可以查看安装的软件列表    
  apt-get remove --purge 名字    可以删除软件    

centos版：
yum list installed
yum remove 软件名

mv [选项] 源文件或目录 目标文件或目录  
6. 删除文件夹命令
  rm -rf  文件夹路径  

7. 解压命令
  tar -xzvf file.tar.gz  

8. 设置linux的环境变量  
    vi  /etc/proifle 在其末尾添加这几句  
    export JAVA_HOME="xxx"
    export PATH="$PATH:$JAVA_HOME/bin"
    export JRE_HOME="$JAVA_HOME/jre"
    export CLASSPATH=".:$JAVA_HOME/lib:$JRE_HOME/lib"
      即可设置jdk的环境变量。  

source /etc/profile更新一下。。  

9. pwd命令    
  pwd命令可以查看当前所在的路径（centos）  


10. 获取java安装目录  
  利用`which java`得到路径1  which java是打印出java命令文件的路径    
  `ls -lrt+路径1`得到 箭头后的路径2  
  `ls -lrt+路径2`就ok了  
  默认情况下，用rpm安装后的java在`/usr/java/jdk1.8.0_144/`  

11. 
   centos查看网络端口占用:` firewall-cmd --zone=public --list-ports`  
    开启或者关闭firewalld（centos7的防火墙）:`systemctl start firewalld`and`systemctl stop firewalld`    
   永远禁用centos防火墙:`systemctl disable firewalld.service `  

12. 
   centos修改主机名:`hostnamectl set-hostname 主机名`
   hostnamectl --static 可以查看主机名

13.
centos关机命令
reboot  重启
poweroff 立刻关机

14.
在linux文件系统路径中
~代表用户目录
如：~/就是/home/make/

15. 
   wget是一个在控制台可以从各个协议上下载东西的工具
   如这条命令
   `wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.4/hadoop-2.7.4.tar.gz`
   直接在控制台执行，就可以从镜像网站下载hadoop压缩包

16：
看下在环境变量配置这里
`export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin`
这种配法有什么用？

17. 更新yum源  
   有时候自带的yum版本落后了，下载的软件都是老版本的，这时候就要考虑更换下yum的版本了  
   首先备份旧的yum源`/etc/yum.repos.d/CentOS-Base.repo`
   然后在/etc/yum.repos.d/目录下用wget命令下载镜像的repo文件，确保下载下来的文件重命名  为：`CentOS-Base.repo`  
   `yum clean all`  
   `yum makecache`
   运行以上两个命令生成yum的缓存  

18. 为wget设置代理，其实很简单
   修改其配置文件` vi /etc/wgetrc  `  里面有教你如何设置代理，将其设置为有运行ss软件的机子的ip地址，端口设置为ss的端口即可。
   当然ss要开：允许局域网连接，并且要设置代理的机子和开ss的机子在同一个网段上

19. centos添加用户
20. 首先登录 root 账号
21. 执行： `useradd 用户名`命令创建一个新用户
22. 执行`passwd username` 为新用户设置新密码并激活

23. centos删除用户 ：`userdel -rf grid` 删除用户的所有信息，不加参数的仅仅只是删除用户，用户的信息没有被删除。  

24. 切换用户登录，centos：login -f username 
   注：1. 加f参数不用输入密码  
25. 在ssh客户端切换登录会退出哦

26. 查看用户所在的组`groups username` 一般说来
27. 添加组 `groupadd name`
28. 将某文件或者文件夹的
29. 归属到某一个组中
   `chown groupname /var/run/httpd.pid`  将/var/run/httpd.pid此文件的所有权归属到groupname这个组中

### 文件描述符
含义：在linux中，文件描述符是linux为了高效管理已打开的文件而创建的索引，其值是一个非负整数，用于指代被打开的文件。  
虽说系统有多少内存就可以打开多少个文件，但是实际运行最大文件打开数是系统内存的10%，此外，系统还会为单一线程能打开的文件数做限制，（用户级限制），查看该用户级限制命令：`ulimit -a`  PS：`-aH`  是查看硬的限制  
怎么增加系统的文件描述符
执行`vi /etc/security/limits.conf `
输入内容

	* soft nofile 65536

	* hard nofile 131072

	* soft nproc 2048

	* hard nproc 4096

解释：* 代表所有用户  
            soft或者hard代表为其限制的是硬配置还是软配置（软配置超过警告，硬配置超过会fail）
            nofile(可打开的文件描述符的最大数)  
            nproc(单个用户可用的最大进程数量)


​            
### sysctl命令
这个命令可以修改内核的运行参数
参数 -a 可以查看所有可读的变量
参数 key  可以查看这个key的值  eg:`sysctl vm.max_map_count`。 
参数  key=value  可以为运行参数设置值


### Linux线程
技能1： 查看端口占用的线程


## 第⑨章：前端技能
1. bootstrap的弹窗功能怎么关闭？官方有个示例性文档，在创建窗口过程中预定义几个按钮，可以实现关闭功能。
    或者把创建窗口那个代码用变量接受，该变量实际是一个对象，可调用该对象的close方法关闭窗口。
    另：窗口的html代码可以写在主html文件里面，也可以写在新的html文件里面。
    但是取窗口里面的输入框的值，必须能获取窗口里面的元素对象，然后再取值。\

## 第十章 windows的命令
netstat -ano|findstr "8080"  查看占用该端口的piD
taskkill /pid 2472 -t -f; 

## 第十一章 PostgreSQL数据库的知识
以下命令在PostgreSQL 9.6.2 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-3), 64-bit  测试通过  
1. 如何查看pg数据库的版本  
  `select version();`   
2. 如何查看pg数据的连接资源  
  `SELECT * FROM pg_stat_activity WHERE datname='postgres';`
  几个字段说下  
  state:运行状态，可以为几种值：  
  active:正在执行；  
  idle:等待新的命令    
  idle in transaction:后端是一个事务，但是尚未执行查询；  
  idle in transaction(aborted):和idle in transaction类似，除了事务执行出错。



3. 连接资源太多，怎么kill
  `SELECT pg_terminate_backend(PID);`  
  pid的值需要从第二项的sql语句查询结果来寻找  


条件的postgres指的数据库名  
另注：你没看错，就是datname，而不是dataname  

4.  死锁了怎么搞？
  什么情况下会死锁？当你发现一个DML语句总是卡住时，一个简单的select一次性通过的话，你就得考虑死锁的可能性了
  用这个sql语句可以看到死锁的进程

  WITH t_wait AS (
  SELECT
  	A .locktype,
  	A . DATABASE,
  	A .relation,
  	A .page,
  	A .tuple,
  	A .classid,
  	A .objid,
  	A .objsubid,
  	A .pid,
  	A .virtualtransaction,
  	A .virtualxid,
  	A,
  	transactionid,
  	b.query,
  	b.xact_start,
  	b.query_start,
  	b.usename,
  	b.datname
  FROM
  	pg_locks A,
  	pg_stat_activity b
  WHERE
  	A .pid = b.pid
  AND NOT A . GRANTED
  ),t_run AS (
  SELECT
  	A . MODE,
  	A .locktype,
  	A . DATABASE,
  	A .relation,
  	A .page,
  	A .tuple,
  	A .classid,
  	A .objid,
  	A .objsubid,
  	A .pid,
  	A .virtualtransaction,
  	A .virtualxid,
  	A,
  	transactionid,
  	b.query,
  	b.xact_start,
  	b.query_start,
  	b.usename,
  	b.datname
  FROM
  	pg_locks A,
  	pg_stat_activity b
  WHERE
  	A .pid = b.pid
  AND A . GRANTED
  ) SELECT
  r.locktype,
  r. MODE,
  r.usename r_user,
  r.datname r_db,
  r.relation :: regclass,
  r.pid r_pid,
  r.xact_start r_xact_start,
  r.query_start r_query_start,
  r.query r_query,
  w.usename w_user,
  w.datname w_db,
  w.pid w_pid,
  w.xact_start w_xact_start,
  w.query_start w_query_start,
  w.query w_query
  FROM
  t_wait w,
  t_run r
  WHERE
  r.locktype IS NOT DISTINCT
  FROM
  w.locktype
  AND r. DATABASE IS NOT DISTINCT
  FROM
  w. DATABASE
  AND r.relation IS NOT DISTINCT
  FROM
  w.relation
  AND r.page IS NOT DISTINCT
  FROM
  w.page
  AND r.tuple IS NOT DISTINCT
  FROM
  w.tuple
  AND r.classid IS NOT DISTINCT
  FROM
  w.classid
  AND r.objid IS NOT DISTINCT
  FROM
  w.objid
  AND r.objsubid IS NOT DISTINCT
  FROM
  w.objsubid
  ORDER BY
  r.xact_start

看到r_pid没，kill it！

其实吧，最重要的是建立死锁超时机制，死锁的线程一旦超时，就强制关闭，释放锁。
死锁并发数高的话，很容易发生。

5. 什么情况下会死锁
  A线程拿着表A的锁，要访问表B.  
  B线程拿着表B的锁，要访问表A  


6 . 教训
1. 不要太依赖工具，比如说Navicat 。这货会把太大的数据变成科学计数法。实测，用pg数据库的shell工具查出来的数据是正常的。。
  但是，比较坑的是，pg数据库支持用科学计数法表示的数字来作为查询语句的条件来查询数据。。
2. 机器永远是对的，**未测试代码永远是错的**，别人写的代码不要轻易相信
  写接口的调用者也要清楚对方的接口的业务逻辑是怎么跑的。这样才能磨炼自己的接口更健壮


## oracle 数据库知识
一般如果在执行DML语句出现这个错误：ora-00054:resource busy and acquire with nowait specified  
简而言之，就是数据库正在忙，待会再试吧。。
..... 开玩笑，怎么可能待会再试，这种情况一般是数据库正在执行事务甚至死锁，这时候就要查询哪里正在执行事务了
​	
	select t2.username,t2.sid,t2.serial#,t2.logon_time
	from v$locked_object t1,v$session t2
	where t1.session_id=t2.sid order by t2.logon_time;

这个语句可以查出数据库有哪些锁


## 通用sql知识
### Exists的特点
使用exists的sql子句只会返回true或者false。例如  `select * from user where id=? and exists (select * from vip where user_id=?)`      
这样的话，会先查找vip表里面是否有指定的用户id,如果有的话，把该用户从user表查出来.    
所以exists是子查询如果包含行，返回true，否则返回false  
注:值的一提的是，如果sql子句是:`where exists (null)` 那么这个子句会返回true。  

## 读某规范手册有感
1：
实体类属性中基本数据类型最好设置为包装数据类型，这样如果这些数据缺失的情况下，不会设置为默认值。而是报空指针。
避免设置为默认值后掩盖了原有的错误。
## 刷初级题的要点
1. 单个字符的‘+’号连接并不是字符串连接，而是用ascll码相加
2. 方法名可以类名一致，和构造方法唯一区别是，构造方法没有返回值
3. 实例方法不单单指public方法，private方法亦是  
4. long d = 2 是正确的，虽然单单写一个2，其实是int。这个应该用了隐式类型提升
5. 原生类就是java的基本数据类型  。  玩文字游戏啊。。

##  ELK日志收集系统
ELK日志收集系统是一个分布式的，收集服务器上面的日志，并将其展示的一款服务器软件  
ELK由三个组件组成  
1. E表示Elasticsearch，是日志分析引擎
2. L表示Logstash 是日志搜集器，负责搜集服务器上面的日志，备选的其他搜集器还有Beats
3. K表示Kibana 就是负责web页面展示的一款组件，支持扩展
  建议安装顺序为Elasticsearch，kibana，Logstash

### Elasticsearch快速入门
1. 下载Elasticsearch组件
2. 解压
  3.运行
  注意点1：
  Elasticsearch开启过程中，你可以看到这个日志重复出现`][INFO ][o.e.e.NodeEnvironment    ] [1xBUNqo]`
  告诉你，`1xBUNqo`就是自动生成的Elasticsearch节点名，嘛，你当然自定义一个节点名
  用这个命令启动`./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name`  
  注意点2：
  后台运行Elasticsearch后，可以键入`curl http://localhost:9200/` 查看运行结果  
  注意3：  
  Elasticsearch启动后，默认开启的是9200来提供服务  
  注意点4：
  开启Elasticsearch过程中，注意这一行
  `publish_address {127.0.0.1:9300}`它告诉你哪个ip地址和端口可以访问到Elasticsearch的服务。  
  后面还有一个bound address。没设置的情况不能从其他主机访问Elasticsearch的服务（经验之谈）  

#### 问题
1. 运行时报`failed error='Cannot allocate memory' (errno=12)`,可能的原因是elasticsearch运行要分配的内存太大，java虚拟机扛不住  
  解决办法： 修改elasticsearch的`jvm.options`,将

  -Xms2g
  -Xmx2g
   修改成

  -Xms512m
  -Xmx512m

2. 运行时报org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root  
  意思就是说，不能用root用户来运行elasticsearch服务器。所以我们要为其创建一个用户和用户组。关于这个内容，请参考上面的linux部分

3. 切换了用户运行报：
  `Could not register mbeans java.security.AccessControlException: access denied ("javax.management.MBeanTrustPermission" "register")`
  这大概是因为你现在执行的时候用的用户对这个文件没有所有权，所以挂了，怎么办，诶，递归操作一下，把这个文件夹里面的文档的所有权全交给你当前登录用户的组就行了。  

4. 改了elasticsearch配置文件后报错了  
  1 . `max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]`  
  这个的意思是说用于elasticsearch的文件描述符过低，现在是4096，要求增加到65536.
  文件描述符的意思和含义请参照上面linux的知识
  2. `max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`
    这个是虚拟内存的`vm.max_map_count [65530]`太低了，至少设置到262144
    用`sysctl`命令即可实现


### kibana快速入门
1. 解压
2. 修改config里面的kibana配置文件，将
  `elasticsearch.url: "http://192.168.21.254:9200/`修改为elasticsearch的服务器地址。
3. 如果远程主机要访问kibana的服务，请把`#server.host: "localhost"`此行取消注释，并设置为非回环地址
4. 运行bin里面的可执行文件  
5. 如果一切正常的话，访问kibana的ip地址+端口，可以看到kibana为你展现的前端页面，不过，在食用之前，你需要在页面设置一个模式。。


这么简单的吗？当然不是，接下来让我们加载一些简单的数据集。。。。。

####  加载一些简单的数据集

**前提，kibana和elasticelasticsearch能互通有无**

步骤1 ：请先下载这三个文件

[jsonshakespeare.json](https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json)

[accounts.zip](https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip)

 [logs.jsonl.gz](https://download.elastic.co/demos/kibana/gettingstarted/logs.jsonl.gz)

步骤2： 解压其中的两个压缩文件，并把这三个文件放在elasticelasticsearch根目录下

步骤3： 为字段设置映射

在`http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/kibana/current/snippets/tutorial-load-dataset/1.json`

页面下有一个控制台，将以下代码复制到控制台并点击运行按钮

```
PUT /shakespeare
{
 "mappings": {
  "doc": {
   "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
   }
  }
 }
}
PUT /logstash-2015.05.18
{
  "mappings": {
    "log": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}
PUT /logstash-2015.05.19
{
  "mappings": {
    "log": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}
PUT /logstash-2015.05.20
{
  "mappings": {
    "log": {
      "properties": {
        "geo": {
          "properties": {
            "coordinates": {
              "type": "geo_point"
            }
          }
        }
      }
    }
  }
}
```

在本机的linux控制台运行这三个命令

```
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl
```

最后，在kibana的控制台执行该命令查看是否操作成功`GET /_cat/indices?v`

结果应该如下所示

```
health status index               pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank                  5   1       1000            0    418.2kb        418.2kb
yellow open   shakespeare           5   1     111396            0     17.6mb         17.6mb
yellow open   logstash-2015.05.18   5   1       4631            0     15.6mb         15.6mb
yellow open   logstash-2015.05.19   5   1       4624            0     15.7mb         15.7mb
yellow open   logstash-2015.05.20   5   1       4750            0     16.4mb  

```

解释：

为什么在加载数据之前，要为字段设置映射

答： 

Mapping divides the documents in the index into logical groups

并且指定字段的特征，比如字段的可搜索性，是否被标记,哪些字段包含数字，日期，地理位置信息。。。

#### 定义你的索引模式

加载到Elasticsearch 的每一组数据都有索引模式

> 索引模式是一串带有可选通配符的字符串，可以匹配多个索引
>
> 例如，在之前的例子中，有一些索引名称就包含YYYY.MM.DD格式的日期，比如说`logstash-2015.05 *`



### Logstash 快速入门

1. 解压  
2. 准备logstash.conf文件  
3. 运行 `bin/logstash -f logstash.conf` 命令  


#### Logstash日志存储管道
先不管`管道`是什么鬼，总之，它是Logstash存储日志的一个过程。这个过程需要二个必须的元素，还有一个可选的元素
1. input：这个其实是一个插件，负责从源收集，消费数据  
2. filter：这个可以根据你自己的指定来修改数据  
3. output：将数据写到目的地，一般是Elasticsearch  

#### hello world
首先要看下你的logstash是否安装成功了，简单的话，执行这个命令  
`bin/logstash -e 'input { stdin { } } output { stdout {} }'`  
可以发现控制台先打印一串鬼画符，然后出现光标了。这时候，你输入什么，敲回车，就回显什么  
SUCCESS！这表示你的logstash已经能正常工作了  
解释：  
1. -e 可以让你以命令行的方式给logstash指定配置。  
2. 该示例的管道是从标准输入获取数据，并以结构化的数据输出到标准输出  

> PS： 小贴士，按ctrl-d 可以断开shell哦  
> ps:  如果执行以上命令抛了一个异常，说找不到这个文件`logstash-plain.log`请尝试用超级管理员执行命令  
#### 在logstash中分析日志
在现代社会中，日志的处理可不是上面的简单hello world,现实的日志的处理可能有更多的input,filter,output插件
在下面的教程中，将教你怎么从web项目中取出日志并送进管道，解析这些日志，用来创建指定的特点fields，并将这些已经解析好的数据传到Elasticsearch集群中。当然，你不用在从控制台输入配置了，多累啊，现在开启配置文件时代
步骤
1. 下载一个日志文件[https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz](https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz "下载")，把它解压  
2. 配置Filebeat以将日志每行的发送到Logstash
> 简单介绍下Filebeat，这个是一个轻量的，资源友好的工具，它从服务器上面搜集日志，并将日志转发给Logstash实例进行处理
>   占用资源极少，`Beats input plugin`最大限度的减少了logstash实例的资源需求。  
> logstash在安装时就默认集成了Beats input plugin了
> Beats input plugin 使logstash能从Elastic Beats框架接受事件  
> PS：对于 Filebeat在另一个章节讲述，这里不说了



####  logstash.conf文件格式
这个文件其实就是logstash的配置文件
作用：  
1. 指定要使用的插件以及插件的配置
2. 可以引用配置的事件字段，处理符合条件的事件。  
  注： 运行logstash可执行文件时 用 -f参数来指定配置文件。
  logstash配置文件的格式大概如下所示，这个配置主要是配置logstash的每一个插件。。   
  包括：输入插件，输出插件，筛选器插件

      input {
        ...
      }
      
      filter {
        ...
      }
      
      output {
        ...
      }

3. 输入插件的配置格式

        input {
          file {
            path => "/var/log/messages"
            type => "syslog"
          }
       
          file {
            path => "/var/log/apache/access.log"
            type => "apache"
          }
        }

RT 酱紫的话，就为这个输入插件配置了两个文件输入
每一个文件输入都有路径和类型信息。
4. 插件的值
  插件的值，指的就是类似于上文的`type => "apache"`的值，这种值的支持一下几种类型
5. 数组
6. Lists
7. Boolean 
8. Bytes 
9. Codec  
  .....   
  要看更多的话，请点击[plugin-value-types](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html#plugin-value-types)



#### 使用Grok Filter Plugin来解析log日志

以上的例子中，虽然我们成功的将日志信息事件打印到控制台了但是格式太乱了，我们有必要整治这货，怎么办呢？
这时候就需要Grok Filter Plugin来帮忙过滤日志信息了  
先推销下Grok Filter Plugin，它的特色如下  

1. 能解析日志信息并从日志从创建指定的命名字段  

2. Grok Filter Plugin是logstash默认可用的几个插件之一  

3. 能使非结构化的日志信息解析成可结构化的，可查询的日志信息，听起来是不是很心动？  

4. Grok Filter Plugin通过传过来的日志数据中查找模式，你需要决定哪种模式用来识别日志数据，以便取出你感兴趣的数据  
   **怎么使用Grok Filter Plugin？**   

5. 分析日志数据，确定使用哪种模式  
   开始分析上次实验的日志，可以分析：开头的ip和括号的时间戳很容易识别，适合用`％{COMBINEDAPACHELOG}`模式。  
   如果你不知道怎么构建这个模式的话，可以使用`Grok Debugger.`可以免费安装使用哦  

6. 在logstash的配置文件中，为你的管道写一个过滤器。上面的例子中，我们用的配置文件是first-pipeline.conf。  
   这次我们仍然来改它，只需要在过滤器部分补上。

   filter {

   ```
   grok {
       match => { "message" => "%{COMBINEDAPACHELOG}"}
   }

   ```

   }
   保存你的改变，不用重启logstash，因为在上次启动时我们已经用了自动加载配置文件的方式来启动了  
   但是你还是需要强制让filebeat重头开始读取日志文件，要做到这一点    

7. 关掉上次运行的filebeat，删除filebeat的注册表文件:`sudo rm data/registry`,这个注册表文件保存了filebeat收集的每一个文件的状态  删除这个文件将强制让filebeat从头开始读取它所搜集的所有文件  

8. 接下来，重启filebeat吧.再等待几分钟等filebeat自动加载配置文件，就可以把新的日志结构体打印出来了，厉害吧。  
   然而我还是感觉像鬼画符  

#### Geoip过滤器插件为你的数据补充细节

除了分析日志以达到更好的搜索结果外，还有一个插件可以在已存在的数据中获取补充信息
比如说，从现有数据获取ip地址信息，再从地址信息获取地理位置，并将该位置信息添加到日志中  

1. 怎么做呢？把以下代码复制到`first-pipeline.conf`文件里，关于过滤器部分

   geoip {

   ```
     source => "clientip"

   ```

     }

这个过滤器要求你指定包含ip地址的原字段的名称，在这个例子中，clientip 字段就包含了ip地址信息。
由于过滤器是按顺序进行过滤的，所以你得确保geoip过滤器位于Grok过滤器的后面，先让Grok过滤器创建字段，然后geoip过滤器才能根据字段获取ip地址信息  

1. 删除注册文件，然后重启Filebeat 。等logstash重新加载配置文件完毕，你就可以看到控制台打印出来的日志信息了
2. 这一步什么都不用做，看下打印出来的日志json数据，里面有一个`city_name`或者`country_name`就OK了

#### 将你的数据编入Elasticsearch

**前提：Elasticsearch必须，在9200端口上运行，处于运行状态**  
在上面的几步中，我们已经将web日志分割成几个特定的字段，但是仅仅只是把数据输出到控制台是不是有点不爽，嗯，接下来，我们就要把数据送到
Elasticsearch了
步骤  

1. 编辑你的logstahs的配置文件`first-pipeline.conf`，改动output的插件，使其如下所示  

   output {

   ```
   elasticsearch {
       hosts => [ "localhost:9200" ]
   }

   ```

   }
   logstash使用http协议和elasticsearch连接，在这个例子中，我们假设logstash和elasticsearch在同一个机器上运行，所以使用了localhost，实际上，localhost也可以换成其他主机名

2. 保存你的改变，关闭filebeat，删除filebeat的注册文件，重启filebeat（秘技：重复操作）

3. 最后尝试用浏览器访问以下地址`localhost:9200/logstash-$DATE/_search?pretty&q=response=200`  
   或者在本机上运行此命令`localhost:9200/logstash-$DATE/_search?pretty&q=response=200`

> 注意:如果执行以上命令报了此错误：`index_not_found_exception`  
> 请确保$DATE代表的是索引的实际名称。要查看可用的索引的列表  
> 执行`curl 'localhost:9200/_cat/indices?v`
> 在这个地址发来的反馈中，有一列是index，其中内容大致如此`logstash-2017.12.15`  
> 这样的话，查看日志的地址就要变成酱紫：`/logstash-2017.12.15/_search?pretty&q=response=200`  

解释：索引使用的日期格式是基于UTC的，使用$DATE可以让显示出来的日期格式变为`YYYY.MM.DD`这种格式的  
最后结果还是在控制台上看日志，但是这次是在elasticsearch上面看日志的，换了一种方式，会感觉比较厉害  
而且不仅如此，观察url，会发现它带了`q=response=200`,这大概是查找日志中response字段等于200的日志    
基于这种考虑，我们可以执行另一个命令  
`curl -XGET 'localhost:9200/logstash-$DATE/_search?pretty&q=geoip.city_name=Buffalo'`  结果应该是查找日志中`geoip.city_name=Buffalo`的日志  

> 如果你学会为`Filebeat`加载Kibana索引模式,你可以在kibana的web页面中看到日志信息，有关信息，请参阅filebeat的入门教程

#### 在logstash使用多个输入和输出插件

在实际开发中，你的日志来源通常有多个地方，要输出的目的地也可能有多个，这个时候，单个input和output肯定不能满足需求。
在这一节中。你将创建一个logstash的管道，该管道从推特简讯和filebeat的client获取输入，也就是说，有两个输入。  
然后将信息发送到Elasticsearch集群，并且还会写入到文件。也就是说，有两个输出。  
那么问题来了，在中国，你怎么能访问到推特简讯？？？WTF！



#### logstash 是如何工作的？

logstash处理事件有三个阶段：输入生成事件，过滤修改事件，输出将事件运往其他地方.

输入和输出插件支持编码和解码，你不必使用单独的过滤器来处理这些事情。

来简单介绍下输入，输出，过滤，编解码。

##### 输入插件

输入插件将数据传到logstash，一些常见的输入插件是

1. **file** 从文件系统中读取文件，这个过程就像linux的命令`tail -0F`
2. **syslog**  监听众所周知的514端口来分析 syslog messages ，根据RFC3164格式进行解析
3. **redis**  略
4. **beats** 处理由Filebeat发送的事件。
5. log4j插件  将开一个章节来讲解log4j插件如何使用

##### 过滤器

过滤器是logstash管道的中间处理设备，如果符合条件，你可以组合一些条件过滤器，对事件进行操作。

一些有用的过滤器如下

1. **grok** 分析和构造任意文本，可将非结构化的日志分析成可结构化和可查询的最佳方法，lagstash内置了120中模式，慢慢找吧骚年
2. **mutate** 在事件字段上执行一般转换，你可以在事件上重命名，移除，覆盖，修改字段
3. **drop**  完全的drop一个事件，比如说，debug事件
4. **clone** 复制一个事件，可能会添加或者修改字段
5. **geoip** 从ip地址获取地理信息添加到数据中

##### 输出插件

输出是logstash管道的最后阶段，一个事件可以有多个输出，但是一旦所有输出处理完成，这个事件就结束了，一些常见的输出插件如下  

1. **elasticsearch** 如果你想将数据便捷，高效，且便于查询的保存起来，那么 Elasticsearch是你不错的选择
2. **file** 在磁盘中写入事件数据
3. **graphite** 将事件数据发送给graphite，这是一个开源的工具，可以存储和绘制指标
4. **statsd** 将事件数据发送给statsd

##### 编解码器

编解码器一般是作为输出或者输出过程的一部分，作为流过滤器而存在的

编解码器可以让你把消息从序列化过程中分离开来，常用的编解码器包括json，msgpack，plain (text)

- **json** 使用json格式编解码数据
- **multiline** 将多行文本事件（例如java的堆栈异常）合并到单个事件中



#### log4j的输入插件

注：logstash的log4j插件其实已经过时了，现在推荐使用的是搭配filebeat来使用

使用方式

1. 配置你的日志输出方式为写入文件。

2. 安装并配置filebeat来收集这些日志并发送到logstash

3. 配置logstash来使用beats的输出插件

   ​

   其中1不解释，看log4j2的官方文档教你如何将服务器产生的日志写到文件中

   2的配置如下

```yaml
filebeat:
  prospectors:
    -
      paths:
        - /var/log/your-app/app.*.log
      input_type: log
output:
  logstash:
    hosts: ["your-logstash-host:5000"]
```

3 的配置，请查看上文

### filebea

#### filebeat快速入门
1. 下载，解压，配置`filebeat.yml`

  配置代码如下
      filebeat.prospectors:
       - type: log
           paths:
          - /home/logstash-5.6.4/testData/logstash-tutorial.log
              output.logstash:
          hosts: ["localhost:5044"]    

要求清空原来`filebeat.yml`的内容，然后把以上代码复制上去。

解释： `- /home/logstash-5.6.4/testData/logstash-tutorial.log`要根据具体平台而定，指向实际的日志文件路径。  
2. 运行`sudo ./filebeat -e -c filebeat.yml -d "publish"` 
> 第一次运行时可能会出现两个问题  
> 1. 抛出错误，只能用所有权用户来登陆系统，执行命令  
> 2. 抛出`more then one namespace configured accessing 'output' (source:'filebeat.yml')`
> 3. 运行是正常的，但是没有出现预想中的端口连接5044失败信息，一派祥和，风平浪静。  
>   叫你Y的清空`filebeat.yml`的内容，你没清空吧，不清空也行，这个错误大概是说只能配置一个output的配置，把其他多余的配置注释掉即可  
>   运行过程中，你可能会看到filebeat在试图连接5044这个端口，直到logstash开启一个Beats plugin之前，这个端口都不会有任何反馈。
>   所以你看到任何关于这个端口连接失败的信息都是正常的。  
>   笔者记：我怎么没看到呢？ 诡异，真诡异  
>   这其实是不正常的，你应该用将filebeat安装目录的所有权归于root用户，然后再执行以上命令。我猜你之前应该用的是非root用户执行这条命令对吧。。。。
#### 为  Filebeat Input 配置logstash
接下来，我们要创建一个Logstash的pipeline配置，该pipeline可以从Beats input plugin接受来自Beats的事件
以下文本就是配置pipeline的基本格式
​	
	input {
	}
	# filter {
	#
	# }
	output {
	}
以上代码其实没有实用性，因为输入输出都没有有效的选项定义
接下来这个代码就给了输入输出几个选项定义

	input {
	    beats {
	        port => "5044"
	    }
	}
	# filter {
	#
	# }
	output {
	    stdout { codec => rubydebug }
	}

解释：input部分是配置了一个beats的输入插件，output部分是配置了一个输出，将输出打印到stdut  
3. 把以上的代码，创建一个`first-pipeline.conf`文件，并复制进去。  
  然后执行： `bin/logstash -f config/first-pipeline.conf --config.test_and_exit &`  
  如果打出的日志没有带fail字样，恭喜你，配置是有效的。  顺便告诉你`--config.test_and_exit`是专门用来校验配置文件是否有效的  
  那么，真正运行Logstash的命令应该是下面那个
  `bin/logstash -f config/first-pipeline.conf --config.reload.automatic &`
> `--config.reload.automatic`是自动配置重新加载。改了配置文件，无需重启即可生效  
> 在启动过程中，你可能会看到多个关于`pipelinelines.yml 被忽略的警告`，你可以大胆的无视它，因为这个文件其实是用于在单个Logstash实例中运行多个管道，在刚才的操作中，你只是在运行一个管道而已。  
> 笔者注：这个警告我没碰上，真是太幸运了？

**最后，当filebeat和logstash工作正常的时候，你应该能看到一系列事件写入控制台，恭喜你，本章节完成...才怪，还要优化下**
​	
	"@timestamp" => 2017-11-09T01:44:20.071Z,
	        "offset" => 325,
	      "@version" => "1",
	          "beat" => {
	            "name" => "My-MacBook-Pro.local",
	        "hostname" => "My-MacBook-Pro.local",
	         "version" => "6.0.0"





## YAML文件格式
###  what is YAML?
YAML是新一代的配置文件，其文件名的后缀是`yml`  
### 格式特点  
1. 大小写敏感  
2. 使用缩进表示层级关系  
3. 缩进时不允许使用Tab键，只允许使用空格。  
4. 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可  
5. 不允许莫名奇妙的空行  
6. 用# 号表示注释


