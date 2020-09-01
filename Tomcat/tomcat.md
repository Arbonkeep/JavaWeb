# tomcat

## windows安装
    1. 下载：官网
    2. 安装：直接解压（安装目录建议不要有中文和空格）
    3. 卸载：删除文件
    4. 启动：
        bin/startup.bat，双击运行
        访问：浏览器输入：http//:localhost:8080 访问本机
                        http//:ip地址:8080 访问其他

        可能遇到的问题：
            1. 在开启时窗口一闪而过：
                原因：配置jdk时,没有正确配置JAVA_HOME环境变量
                解决方案：正确配置JAVA_HOME环境变量

            2. 启动时报错：
                原因：端口号被占用
                解决方案：
                    <1> 找到占用的端口号，并找到对应的进程，关闭该进程
                        * 通过cmd命令行窗口输入netstat —ano查找pid,然后在任务管理器中关闭相应pid的进程

                    <2> 修改自身的端口号，在安装目录中conf文件下找到配置文件server.xml修改端口号

    5. 关闭
        1. 正常关闭
            <1> 在bin目录下运行shutdown.bat
            <2> ctrl + c
            
        2. 强制关闭：
            点击窗口x按钮

## tomcat安装目录解析
        * bin：可执行文件
        * conf：配置文件
        * lib：依赖jar包
        * logs：日志文件
        * temp：临时文件
        * webapps：存放web项目
        * work：存放运行时数据

## tomcate-配置
        1. 部署项目的方式：
            <1> 直接将项目放到webapps目录下
                简化部署：将项目打包成war包，再将war包放在webapps目录下，war包会自动解压缩

            <2> 配置conf/server.xml文件
                在<Host>标签体中配置
                <Context docBase="D:aaa/bbb" path="/ccc"/>
                docBase:项目存放路径
                path：虚拟目录

            <3> 在conf\Catalina\localhost创建任意名称的xml文件。在文件中编写
                <Context docBase="D:\aaa" />
                虚拟目录：xml文件的名称

        2. 静态项目和动态项目
            * 目录结构：
                java的动态项目的目录结构
                    --项目的根目录
                        -- WEB-INF目录(带有这个目录的是动态项目)
                            -- web.xml:web项目的核心配置文件
                            -- classes目录：放置字节码文件的目录
                            -- lib目录：放置依赖jar包
        3. 将Tomcat集成到idea中，并创建JavaEE项目，部署项目



    