# EL表达式（Expression Language）

## 概述
    1. 作用：替换或简化jsp页面中java代码的编写

    2. 语法：${表达式}

    3. 注意：
        jsp默认支持el表达式的。忽略el表达式的方式：
            1) 设置jsp中的page指令中：isELIgnored="true"  忽略当前jsp页面中所有的el表达式
            2) \${表达式}：忽略当前这个el表达式

## EL表达式的使用
    1、运算:
        <1> 算术运算符：+,-,*,/(可以用div),%(可以用mod)

        <2> 比较运算符：> < >= <= == !=

        <3> 逻辑运算符：&&(可以用and) ||(可以用or)  !(可以用not)

        <4> 空运算符：empty
            * 功能：用于判断字符串，集合，数组对象是否为null或者 长度是否为0
                ${empty list}:用于判断字符串，集合，数组对象是否为null或者长度是否为0
                ${not empty list}:表示判断字符串，集合，数组对象是否不为null或者长度大于0

    2、获取值
        <1> el表达式只能从域对象中获取值
        <2> 语法：
            1) ${域的名称.键名}：从指定域中获取指定的键值
                域名称：
                    pageScope       --> pageContext
                    requestScope    --> request
                    sessionScope    --> session
                    applicationScope--> application(ServlerContext)

                举例：在request域中存储了name=张三
                获取：${requestScope.name}

            2) ${键名}：表示依次从最小的域中查找是否有该键对应的键值，直到找到为止，如果没有就直接返回一个空字符串

        <3> 获取对象List集合，Map集合的值
            1.对象：${域名称.键名.属性名}     这里的属性名指的是getter方法后面的部分
                本质上会去调用对象的getter方法

            2.List集合：${域名称.键名[索引]}
                如果集合中存储的是对象那么可以通过${域名称.键名[索引].属性名}来获取对象的相关属性

            3.Map集合：
                第一种方式：${域名称.键名.key名称}
                第二种方式：${域名称.键名["key的名称"]
                
                * 获取map中对象的属性：${域名称.键名[索引].属性名}

    3.隐式对象
        el表达式中有其11个隐式对象(不用创建能调用的对象)
        pageContext:
            可以获取其他8个对象的内置对象
            ${pageContext.request.contextPath}:动态获取虚拟目录