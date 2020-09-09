# JSTL(JavaServer Pages Tag Library)

## 概述
    1. 概念：JSP标准标签库，是由Apache组织提供的开源的免费的jsp标签。

    2. 作用：用于简化或替换jsp页面上的Java代码

    3. 使用步骤：
        <1> 导入jstl的相关jar包
        
        <2> 在jsp中引入标签库：taglib指令 ： <%@ taglib %>
            <%@taglib  prefix="c"  uri="http://java.sun.com/jsp/jstl/core" %>
        
        <3> 使用标签

## 常用的JSTL标签
    1.if：相当于Java代码中的if语句
            <1> 属性：
                test属性（必须写），接收boolean表达式
                如果表达式为true，则显示if标签体的内容，如果为false则不会显示标签体的内容
                一般情况下，test属性会结合el表达式来使用
            <2> 注意：c:if标签没有else情况，如果想使用，那么可以重新定义一个if标签

```jsp

<!-- 举例一 -->
    <c:if test="true">
        <h3>真，显示内容</h3>
    </c:if>

<!-- 举例二 -->
     <%
        //判断request域中list集合是否为空，如果不为null则显示遍历集合
        List list = new ArrayList();
        list.add("aaa");
        request.setAttribute("list",list);
        request.setAttribute("number",4);

    %>
    <c:if test="${not empty list}">
        遍历集合。。。
    </c:if>
    <br>

<!-- 举例三 -->
     <!-- 判断request域中number的奇偶。 -->
    <c:if test="${number % 2 == 0}">
        number为偶数
    </c:if>
    <c:if test="${number % 2 == 1}">
        number为奇数
    </c:if>

```

    2. choose：相当于Java代码中的switch语句

```jsp
     <!--
        完成数字编号对应星期几的案例
            1.在域中存储一个数字
            2.使用choose标签取出数字            相当于switch的声明
            3.使用when标签对数字进行判断         相当于case
            4.otherwise标签用于做其他情况的声明  相当于default
    -->

    <c:choose>
        <c:when test="${number == 1}">星期一</c:when>
        <c:when test="${number == 2}">星期二</c:when>
        <c:when test="${number == 3}">星期三</c:when>
        <c:when test="${number == 4}">星期四</c:when>
        <c:when test="${number == 5}">星期五</c:when>
        <c:when test="${number == 6}">星期六</c:when>
        <c:when test="${number == 7}">星期日</c:when>
        <c:otherwise>输入的数字有误</c:otherwise>
    </c:choose>

```
    3.foreach：相当于Java代码中的for循环

```jsp
 <!-- 
     foreach:相当于java代码中的for语句
            1.完成重复的操作
                属性：
                    begin:开始值
                    end：结束值
                    var：临时变量
                    step：步长
                    varStatus：循环状态对象
                    index:容器中元素的索引，从0开始
                    count：循环次数，从1开始 
-->

<!--完成从1打印到10-->
    <c:forEach begin="1" end="10" var="i" step="1" varStatus="s">
        ${i} ${s.count}
    </c:forEach>


 <!-- 
            2.遍历容器(相当于java中foreach)
                属性：
                    items：容器对象
                    var：容器中元素的临时变量
                    varStatus：循环状态对象
                    index:容器中元素的索引，从0开始
                    count：循环次数，从1开始
 -->
    <%
        List list = new ArrayList();
        list.add("111");
        list.add("222");
        list.add("333");
        request.setAttribute("list",list);
    %>

    <!--遍历list-->
    <c:forEach items="${list}" var="i" varStatus="s">
        ${i}${s.index}${s.count}
    </c:forEach>

```

