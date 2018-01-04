---
title: http cookie session状态管理

---

#### web状态管理机制

状态管理的含义： 将浏览器与web服务器之间多次交互当做一个整体来看待，并且将多次交互所涉及的数据(即状态)保存下来。


如何进行状态管理：
 1. 将状态保存在客户端
      将状态存放到浏览器(cookie技术)。
 2. 将状态保存在服务器端
      将状态存放到web服务器(session技术)。



#### cookie

>属于客户端的状态管理技术。浏览器访问web服务器时，服务器会将少量的数据以set-cookie消息头的方式发送给浏览器，浏览器会将这些数据保存下来；
当浏览器再次访问服务器，会将之前保存的这些数据以cookie消息头的方式发送给服务器。

对 cookie 的操作包括如下

1. 名称(Name)
2. 值(Value)
3 .域(Domain)
4 .路径(Path)
5 .失效日期(Expires)
6 .安全标志(Secure)
7 .HttpOnly (仅服务器端)
注意，cookie 多数时候由服务器端创建，JS 也可以创建 cookie，但 HttpOnly 类型的 JS 无法创建。

cookie 类型

普通 cookie，服务器端和 JS 都可以创建，JS 可以访问

HttpOnly cookie，只能由服务端创建，JS 是无法读取的，主要基于安全考虑

安全的 cookie (仅https)，服务器端和 JS 都可以创建，JS 仅HTTPS下访问


1. 创建cookie
            （即让服务器发送set-cookie消息头给浏览器）
            Cookie c = new Cookie(String name,String value);
            name:cookie的名称
            value:cookie的值
            response.addCookie(c);

2. 查看cookie
            Cookie[]  request.getCookies();
            注意:
                该方法有可能返回null。
            String cookie.getName();
            String cookie.getValue();

3. 编码问题
        cookie只能存放合法的ascii字符，如果要保存中文，很显然需要将中文转换成合法的ascii字符。
        String URLEncoder.encode(String str,"utf-8");
        String URLDecoder.decode(String str,"utf-8");

4. 生存时间
        cookie.setMaxAge(int seconds);
        注意
           单位是秒
           seconds > 0 : 浏览器会将cookie保存在硬盘上，超过指定时间，会删除这个cookie。
           seconds = 0：删除cookie。
                      比如，要删除一个名称为user的cookie
                          Cookie c = new Cookie("user","");
                          c.setMaxAge(0);
                          response.addCookie(c);
           seconds < 0: 默认值,浏览器会将cookie保存在内存里面，只有浏览器不关闭，cookie会一直存在。

5. 路径问题
      什么是路径问题： 浏览器在向服务器发请求时，会比较cookie的路径与请求地址是否匹配，只有匹配的cookie才会发送给服务器。

      cookie的默认路径等于创建cookie的组件的路径。
       比如:
           /web06_3/sub01/addCookie.jsp,则该jsp创建的cookie的路径等于"/web06_3/sub01"
       匹配规则
           要访问的地址必须是cookie的路径或者其子路径,浏览器才会发送该cookie。
       设置cookie的路径
           cookie.setPath(String path);
       经常设置为
               cookie.setPath("/appname");
       这样，可以保证某个组件所创建的cookie可以被同一个应用内部其他的组件访问到。


6. cookie的限制
      1. cookie可以被用户禁止。
      2. cookie不安全，如果有敏感数据，最好不要以cookie方式来保存。如果一定要保存，一定要加密。
      3. cookie只能够保存少量的数据(大约4k)
      4. 浏览器保存的cookie的数量也有限制，大约是300个。
      5. cookie只能保存字符串。

--------------------------
#### session是什么
>一种服务器端的状态管理技术（将状态保存在服务器上）当浏览器访问web服务器时，服务器会创建一个session对象(该对象有一个唯一的id属性值，
类似于一个人的身份证号码一样，一般称之为sessionId)，接下来服务器在默认情况下，会将sessionId以set-cookie消息头的方式发送给浏览器;
当浏览器再次访问服务器时，会将sessionId以cookie消息头的方式发送过来，服务器依据sessionId就可以找到之前创建的session对象。

session不多做阐述！
