### Welcome to my simple mvc.:smiley:

我觉得学习一个框架有两个方式，一是直接去一层层剖析源码，二是通过别人的学习总结，滤清思路，有的放矢的去学习，并参考已经研究的部分自己尝试去写个类似的实现。

我个人更喜欢第二种方式。所以这是一个简单的MVC实现的文档。

:japanese_goblin: 首先我们先搭建一个 Maven Web 项目，在 pom.xml 中加入下列依赖：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.machangchang</groupId>
  <artifactId>MyMVC</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<java.version>1.8</java.version>
	</properties>
	
	<dependencies>
	     <dependency>
  		   <groupId>javax.servlet</groupId> 
		   <artifactId>javax.servlet-api</artifactId> 
		   <version>3.0.1</version> 
		   <scope>provided</scope>
		</dependency>
    </dependencies>
</project>
```

:hear_no_evil: 接着我们的Web.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	<servlet>
		<servlet-name>MySpringMVC</servlet-name>
		<servlet-class>com.machangchang.servlet.MyDispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>application.properties</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>MySpringMVC</servlet-name>
		<url-pattern>/*</url-pattern>
	</servlet-mapping>
</web-app>
```

:frog: application.properties文件中配置了要扫描的包。

```
scanPackage=com.machangchang.core
```

:baby_chick: 创建自己的注解，Controller注解，它只能标注在类上面；RequestMapping注解，可以在类和方法上；RequestParam注解,只能注解在参数上。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyController {
	/**
     * 表示给controller注册别名
     * @return
     */
    String value() default "";
}
```

```java
@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyRequestMapping {
	/**
     * 表示访问该方法的url
     * @return
     */
    String value() default "";
}
```

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyRequestParam {
	/**
     * 表示参数的别名，必填
     * @return
     */
    String value();
}
```

:horse: 然后创建MyDispatcherServlet这个类，去继承HttpServlet，重写init方法、doGet、doPost方法。

```java
public class MyDispatcherServlet extends HttpServlet{
	
	private Properties properties = new Properties();
	
	private List<String> classNames = new ArrayList<>();
	
	private Map<String, Object> ioc = new HashMap<>();
	
	private Map<String, Method> handlerMapping = new  HashMap<>();
	
	private Map<String, Object> controllerMap  =new HashMap<>();
	
	@Override
	public void init(ServletConfig config) throws ServletException {
		
		//1.加载配置文件
		doLoadConfig(config.getInitParameter("contextConfigLocation"));
		
		//2.初始化所有相关联的类,扫描用户设定的包下面所有的类
		doScanner(properties.getProperty("scanPackage"));
		
		//3.拿到扫描到的类,通过反射机制,实例化,并且放到ioc容器中(k-v  beanName-bean) beanName默认是首字母小写
		doInstance();
		
		//4.初始化HandlerMapping(将url和method对应上)
		initHandlerMapping();
		
	}
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		this.doPost(req,resp);
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		try {
			//处理请求
			doDispatch(req,resp);
		} catch (Exception e) {
			resp.getWriter().write("500!! Server Exception");
		}

	}
	
	private void doDispatch(HttpServletRequest req, HttpServletResponse resp) throws Exception {
		if(handlerMapping.isEmpty()){
			return;
		}
		
		String url =req.getRequestURI();
		String contextPath = req.getContextPath();
		
		//拼接url并把多个/替换成一个
		url=url.replace(contextPath, "").replaceAll("/+", "/");
		
		if(!this.handlerMapping.containsKey(url)){
			resp.getWriter().write("404 NOT FOUND!");
			return;
		}
		
		Method method =this.handlerMapping.get(url);
		
		//获取方法的参数列表
		Class<?>[] parameterTypes = method.getParameterTypes();
	
		//获取请求的参数
		Map<String, String[]> parameterMap = req.getParameterMap();
		
		//保存参数值
		Object [] paramValues= new Object[parameterTypes.length];
		
		//方法的参数列表
        for (int i = 0; i<parameterTypes.length; i++){  
            //根据参数名称，做某些处理  
            String requestParam = parameterTypes[i].getSimpleName();  
                  
            if (requestParam.equals("HttpServletRequest")){  
                //参数类型已明确，这边强转类型  
            	paramValues[i]=req;
                continue;  
            }  
            if (requestParam.equals("HttpServletResponse")){  
            	paramValues[i]=resp;
                continue;  
            }
            if(requestParam.equals("String")){
            	for (Entry<String, String[]> param : parameterMap.entrySet()) {
         			String value =Arrays.toString(param.getValue()).replaceAll("\\[|\\]", "").replaceAll(",\\s", ",");
         			paramValues[i]=value;
         		}
            }
        }  
		//利用反射机制来调用
		try {
			method.invoke(this.controllerMap.get(url), paramValues);//obj是method所对应的实例 在ioc容器中
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	private void  doLoadConfig(String location){
		//把web.xml中的contextConfigLocation对应value值的文件加载到留里面
		InputStream resourceAsStream = this.getClass().getClassLoader().getResourceAsStream(location);
		try {
			//用Properties文件加载文件里的内容
			properties.load(resourceAsStream);
		} catch (IOException e) {
			e.printStackTrace();
		}finally {
			//关流
			if(null!=resourceAsStream){
				try {
					resourceAsStream.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		
	}
	
	private void doScanner(String packageName) {
		//把所有的.替换成/
		URL url  =this.getClass().getClassLoader().getResource("/"+packageName.replaceAll("\\.", "/"));
		File dir = new File(url.getFile());
		for (File file : dir.listFiles()) {
			if(file.isDirectory()){
				//递归读取包
				doScanner(packageName+"."+file.getName());
			}else{
				String className =packageName +"." +file.getName().replace(".class", "");
				classNames.add(className);
			}
		}
	}
	
	private void doInstance() {
		if (classNames.isEmpty()) {
			return;
		}	
		for (String className : classNames) {
			try {
				//把类搞出来,反射来实例化(只有加@MyController需要实例化)
				Class<?> clazz =Class.forName(className);
			   if(clazz.isAnnotationPresent(MyController.class)){
					ioc.put(toLowerFirstWord(clazz.getSimpleName()),clazz.newInstance());
				}else{
					continue;
				}	
				
			} catch (Exception e) {
				e.printStackTrace();
				continue;
			}
		}
	}

	private void initHandlerMapping(){
		if(ioc.isEmpty()){
			return;
		}
		try {
			for (Entry<String, Object> entry: ioc.entrySet()) {
				Class<? extends Object> clazz = entry.getValue().getClass();
				if(!clazz.isAnnotationPresent(MyController.class)){
					continue;
				}
				
				//拼url时,是controller头的url拼上方法上的url
				String baseUrl ="";
				if(clazz.isAnnotationPresent(MyRequestMapping.class)){
					MyRequestMapping annotation = clazz.getAnnotation(MyRequestMapping.class);
					baseUrl=annotation.value();
				}
				Method[] methods = clazz.getMethods();
				for (Method method : methods) {
					if(!method.isAnnotationPresent(MyRequestMapping.class)){
						continue;
					}
					MyRequestMapping annotation = method.getAnnotation(MyRequestMapping.class);
					String url = annotation.value();
					
					url =(baseUrl+"/"+url).replaceAll("/+", "/");
					//这里应该放置实例和method 
					handlerMapping.put(url,method);
					controllerMap.put(url,clazz.newInstance());
					System.out.println(url+","+method);
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 把字符串的首字母小写
	 * @param name
	 * @return
	 */
	private String toLowerFirstWord(String name){
		char[] charArray = name.toCharArray();
		charArray[0] += 32;
		return String.valueOf(charArray);
	}
}
```

上面完成之后部署项目到服务器。

访问：(http://localhost:8080/MyMVC/test/doTest?param=machangchang
)

输出：doTest method success! param:machangchang。

:chicken: 这样一个简单的MVC就完成了。
