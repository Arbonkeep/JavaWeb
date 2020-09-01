# RequestAndResponse


## HTTP
    1. 概念：Hyper text Yransfer Protocol 超文本传输协议
        <1> 传输协议：定义了客户端和服务器通信时，发送数据的格式
        
        <2> 特点：
            * 基于TCP/IP的高级协议
            * 默认端口号：80
            * 基于请求响应模型：一次请求对应一次响应
            * 无状态的：每次请求之间相互独立,不能交互数据
        
        <3> 历史版本：
            1.0：每次请求都会创立新的连接
            1.1：复用连接

    2. 请求消息数据格式
        <1> 请求行
            * 请求方式 请求url 请求协议/版本
            * GET /login.html HTTP/1.1
            * 请求方式：
                HTTP中的请求方式有7种，其中常用的有2种
                    * GET:
                        1) 请求参数在请求行中
                        2) 请求的url长度有限制
                        3) 不太安全(参数在请求行中)
                    * POST：
                        1) 请求参数在请求体中
                        2) 请求的url长度没有限制
                        3) 相对安全

        <2> 请求头:客户端浏览器
            请求头名称: 请求头值
            常见的请求头：
                1) User-Agent:浏览器告诉服务器，我访问你使用的浏览器版本信息
                    可以在服务器端获取该头 的信息，解决浏览器的兼容性问题

                2) Referer:http://localhost/login.html
                    告诉服务器，我（当前请求）来自于哪里
                        作用：
                            * 防盗链（在技术层次来防止盗取链接）
                            * 统计工作

        <3> 请求空行
            空行，就是用于分割POST请求的请求头和请求体的
        
        <4>请求体(正文)
            用于封装POST请求消息的请求参数

    3. 响应消息：服务器端发送给客户端的数据
        <1>响应消息数据格式
            1) 响应行
                    组成:协议/版本 响应状态码 状态码描述
                    1> 状态码都是3位数字

                    2> 状态码分类：
                        1XX:服务器接收客户端消息，但是没有接收完成，等一段时间后发送1XX状态码
                        2XX:成功。代表：200
                        3XX:重定向。代表：301（代表永久移动）302（重定向，代表暂时移动）304（访问缓存）
                        4XX:客户端错误。
                            代表：
                                1. 404：请求路径没有对应的资源
                                2. 405：请求方式没有对应的doXxx方法（即doget，或者dopost）
                        5XX:服务器端错误。代表：500（服务器内部出现异常）

            2) 响应头
                <1> 格式  头名称: 值

                <2> 常见的响应头：
                    Content-Type：服务器告诉客户端本次响应体数据格式以及编码格式
                    Content-disposition:服务器告诉客户端以什么格式打开响应体数据
                        值：
                            in-line:默认值，在当前页面打开
                            attachment;filename=xxx：以附件形式打开响应体，文件下载

            3) 响应空行
            4) 响应体：传输的数据
        
        <2> 字符串格式：
            响应行

            响应头
                Content-Length: 90
                Content-Type: text/html;charset=UTF-8
                Date: Fri, 19 Jul 2019 01:20:46 GMT
            响应空行

            响应体
                html的内容

## Request
    
    1. Request对象和Response对象的原理
        <1> request和response对象是由tomcat服务器创建的，我们负责使用
        <2> request对象是用来获取请求消息的，response对象是用来设置响应消息的

    2. request对象继承体系结构
        ServletRequest        接口
            | 继承
        HttpServletRequest    接口
            | 实现
        org.apache.catalina.connector.RequestFacade 类（tomcat）

    3. request的功能
        <2> 获取请求行消息数据
            请求消息的格式：GET /day08/demo1?name=zhangsan HTTP/1.1
            方法：
                1) 获取请求方式:GET
                    String getMethod()
                2) 获取虚拟目录：/day08
                    String getContextPath()
                3) 获取Servlet路径：/demo1
                    String getServletPath()
                4) 获取get方式请求参数：name=zhangsan
                    String getQueryString()
                5) 获取请求URI：/day08/demo1
                    String getRequestURI():/day08/demo1
                    StringBuffer getRequestURL():http://localhost/day08/demo1

                    URL:统一资源定位符        http://localhost/day08/demo1 (这里有限定)
                    URI：统一资源标识符（范围比较广）/day08/demo1


                6) 获取协议以及版本：HTTP/1.1
                    String getProtocol()
                7)获取客户机IP地址
                    String getRemoteAddr()

        2. 获取请求头数据
            方法：
                String getHeader(String name):通过请求头的名称获取请求头的值
                Enumeration<String> getHeaderNames():获取所有请求头的名称

        3. 获取请求体数据
            请求体：只有POST请求方式，才有请求体，在请求体中封装了POST请求的请求参数
                1、获取流对象
                    BufferedReader getReader():获取字符输入流，只能操作字符流
                    ServletInputStream getInputStream():获取字节输入流，可以操作所有类型数据
                2、再从流对象中拿数据

        4、其他功能：
            <1> 获取请求参数通用方式：不论是get还是post都能通过下面方法获取数据
                1) String getParameter(String name):根据参数获取参数值
                2) String[] getParameterValues():根据参数名称获取参数值的数组
                3) Enumeration<string> getParameterNames():获取所有请求数据名称
                4) Map<String,String[]> getparameterMap():获取所有参数的Map集合

                * 中文乱码问题：
                    get方式：tomcat8已经将get方式的乱码问题解决了
                    post方式：会乱码
                        解决方案：在获取参数前，设置request的编码request.setCharacterEncoding("utf-8")

            <2> 请求转发:一种在资源内部的资源跳转方式
                1) 步骤：
                    1. 通过request对象获取请求转发器对象：RequestDispatcher getResquestDispatcher(String path)
                    2. 使用ResquestDispatcher对象进行转发：forward(ServletResquest resquest, ServletResponse response)
                2) 特点：
                    1. 浏览器地址栏路径不会发生改变
                    2. 只能转发到当前服务器内部资源中
                    3. 转发是一次请求

            3. 共享数据:
                域对象：一个有作用范围的对象，可以在范围内共享数据
                request域：代表一次请求范围，一般用去请求转发多个资 源中共享数据
                方法：
                    1、void setAttribute(String name,Object obj):存储数据
                    2、Object getAttribute(String name):通过键获取值
                    3、void removeAttribute(String name):通过键移除键值对

                注意：只有在转发的情况下才能使用request域来共享数据


            4. 获取ServletContext：
                ServletContext getServletContext()

## BeanUtils工具类使用
    BeanUtils工具类，简化数据封装,用于封装JavaBean的

            1. JavaBean：标准的Java类
                <1> 要求：
                    1) 必须被public修饰
                    2) 必须提供空参的构造方法
                    3) 成员变量必须用private修饰
                    4) 提供公共的setter和getter方法
                <2>功能：
                    封装数据

            2. 概念：
                成员变量：
                属性：setter和getter方法截取后的产物
                如：getUsername()----> Username -------->username

            3. 方法
                1) setProperty():设置属性值
                2) getProperty()：获取属性值
                3) populate(Object obj，Map map):将map集合的键值对信息封装到对应的JavaBean对象中


## Response
    1. 功能：设置响应消息
        <1> 设置响应行
            格式：http/1.1 200 ok
            设置状态码：setStatus(int sc)

        <2> 设置响应头:setHeader(String name, String value)

        <3> 设置响应体:
            使用步骤：
                1) 获取输出流：
                    字符输出流：PrintWriter getWriter()
                    字节输出流：ServletOutputStream getOutputStream()

                2) 使用输出流，将数据输出到客户端浏览器

    2. 重定向：资源跳转的方式
        <1> 实现：
            方式一: 1.设置状态码为302（重定向）通过setStatus()
                    2.设置响应头location通过setHeader()

            方式二：直接调用sendRedirct():指定资源路径

        <2> 重定向（redirect）的特点：
                1. 重定向的地址栏该变
                2. 重定向能访问其他站点（服务器）下的资源
                3. 重定向是两次请求。不能使用request对象来共享数据

        <3> 请求转发（forward）的特点：
            1. 转发的地址栏不变
            2. 转发只能访问当前服务器下的资源
            3. 转发是一次请求，可以共享数据


        <4> 判断定义的路径是给谁使用的？判断请求从哪出发？
            * 给客户端浏览器使用：需要加虚拟目录（项目的访问路径）
                建议虚拟目录通过动态获取：request.getContextPath() //可用于解决后期修改了下虚拟目录的问题
                <a>,<form>,重定向等都是给浏览器使用

            * 给服务器使用：不需要加虚拟目录
                请求转发路径

    <3> 输出字符数据到浏览器的乱码问题
        * 原因：浏览器解码与我们编码的字符集不同
                
        * 解决方案
                <1> 设置该流的默认编码
                response.setCharacterEncoding("utf-8");

                <2> 告诉浏览器响应体使用的编码
                    1) response.setHeader("context-type","text/html;charset=utf-8");
                    2) response.setContextType("text/html;charset=utf-8")

## ServletContext
    1、概念：代表整个web应用，可以和程序的容器（服务器来通信）

        <1> 获取：
            1) 通过request对象获取
                request.getServletContext();
            2) 通过httpServlet获取
                this.getServletContext()

        <2>功能：
            <1> 获取MIME类型
                MIME类型：在互联网通信过程中定义的一种文件数据类型
                    格式：大类型/小类型      如：text/html
                    获取：String getMimeType(String file)

            <2> 域对象：共享数据
                1) setAttribute(String name,Object obj)
                2) getAttribute(String name)
                3) removeAttribute(String name)
                4) ServletContext对象范围：所有用户所有请求的数据

            <3> 获取文件的真实（服务器）路径
                1) String getRealPath(String path)

```java
        //1.获取a.txt（WEB目录下）的资源路径
        String realPath = context.getRealPath("/b.txt");
        System.out.println(realPath);

        File file = new File(realPath);//将真实路径封装成File对象
        //2.获取b.txt(WEB-INF目录下)的资源路径
        String b = context.getRealPath("/WEB-INF/c.txt");
        System.out.println(b);

        //3.获取c.txt（src目录下）的资源路径
        String c = context.getRealPath("/WEB-INF/classes/a.txt");

```

## Request对象与Response对象的原理
    1. tomcat服务器会根据用户请求的url的资源路径，创建对应的Servlet对象(tomcat会将字节码文件加载进内存，然后创建对象)
    2. tomcat服务器会创建request对象和response对象，request对象中会封装对应的请求消息
    3. tomcat会将request和response两个对象传递给service方法，并且会调用service方法
    4. 程序员通过request对象来获取到对应的请求消息数据，并通过response对象来设置响应消息数据
    5. 服务器在给浏览器做出响应之前，会从response对象中获取到程序员设置的响应消息数据