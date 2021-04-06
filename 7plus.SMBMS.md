# SMBMS

Small/Mid-sized Business Management System 中小型交易管理系统（超市管理系统）

<img src="images/67.jpg" alt="67" style="zoom:45%;" />

数据库

<img src="images/68.jpg" alt="68" style="zoom:50%;" />

**项目如何搭建?**

考虑使用不使用Maven? 依赖，jar

**架构要先理清，MVC三层架构：**

<img src="images/62.jpg" alt="62" style="zoom:45%;" />

**Filter的作用：**

<img src="images/63.jpg" alt="63" style="zoom:40%;" />

**Servlet运行原理：**

<img src="images/52.jpg" alt="52" style="zoom:45%;" />

**可用于存储的内置对象：**

```java
pageContext.setAttribute("name1","神州1号");  //保存的数据只在一个页面中有效
request.setAttribute("name2","神州2号");   //保存的数据只在一次请求中有效，请求转发会携带这个数据
session.setAttribute("name3","神州3号");   //保存的数据只在一个会话中有效，从打开浏览器到关闭浏览器
application.setAttribute("name4","神州4号"); //保存的数据在服务器中有效，从打开服务器到关闭服务器
//从底层到高层（作用域）：page-->request-->session-->application
```



## 项目搭建准备工作

1. 搭建一个Maven Web项目

2. 配置Tomcat

3. 测试项目是否能够跑起来

4. 导入项目中会遇到的jar包

   * Jsp, Servlet, mysql驱动, jstl, standard...

5. 创建项目包结构

   <img src="images/69.jpg" alt="69" style="zoom:50%;" />

6. 连接数据库

7. 编写实体类

   * ORM映射：表-类映射

8. 编写基础公共类

   1. 数据库配置文件

   ```properties
   driver=com.mysql.cj.jdbc.Driver
   url=jdbc:mysql://localhost:3306/smbms?userUnicode=true&characterEncoding=utf-8&useSSL=true
   username=root
   password=123456
   ```

   2. 编写数据库的公共类

   ```java
   package com.jin.dao;
   
   import java.io.IOException;
   import java.io.InputStream;
   import java.sql.*;
   import java.util.Properties;
   
   //操作数据库的公共类
   public class BaseDao {
     private static String driver;
     private static String url;
     private static String username;
     private static String password;
   
     //静态代码块，类加载的时候就初始化了
     static {
       Properties properties = new Properties();
       //通过类加载器读取对应的资源
       InputStream is = BaseDao.class.getClassLoader().getResourceAsStream("db.properties");
   
       try {
         properties.load(is);
       } catch (IOException e) {
         e.printStackTrace();
       }
   
       driver = properties.getProperty("driver");
       url = properties.getProperty("url");
       username = properties.getProperty("username");
       password = properties.getProperty("password");
     }
     //获取数据库的连接
     public static Connection getConnection() {
       Connection connection = null;
       try {
         Class.forName(driver);
         connection = DriverManager.getConnection(url, username, password);
       } catch (Exception e) {
         e.printStackTrace();
       }
       return connection;
     }
   
     //编写查询公共方法, 提出所有参数方便统一初始化和关闭
     public static ResultSet execute(Connection connection, String sql, Object[] params, ResultSet resultSet, PreparedStatement preparedStatement) throws SQLException {
       //预编译的sql，在后面直接执行就可以了
       preparedStatement = connection.prepareStatement(sql);
   
       for (int i = 0; i < params.length; i++) {
         //setObject，占位符从1开始，但是数组从0开始！
         preparedStatement.setObject(i+1, params[i]);
       }
       resultSet = preparedStatement.executeQuery();
       return resultSet;
     }
   
     //编写增删改公共方法
     public static int execute(Connection connection, String sql, Object[] params, PreparedStatement preparedStatement) throws SQLException {
       preparedStatement = connection.prepareStatement(sql);
   
       for (int i = 0; i < params.length; i++) {
         //setObject，占位符从1开始，但是数组从0开始！
         preparedStatement.setObject(i+1, params[i]);
       }
       int updateRows = preparedStatement.executeUpdate();
       return updateRows;
     }
   
     //释放资源
     public static boolean closeResource(Connection connection, PreparedStatement preparedStatement, ResultSet resultSet) {
       boolean flag = true;
       if (resultSet != null) {
         try {
           resultSet.close();
           //GC回收
           resultSet = null;
         } catch (SQLException e) {
           e.printStackTrace();
           flag = false;
         }
       }
       if (preparedStatement != null) {
         try {
           preparedStatement.close();
           //GC回收
           preparedStatement = null;
         } catch (SQLException e) {
           e.printStackTrace();
           flag = false;
         }
       }
       if (connection != null) {
         try {
           connection.close();
           //GC回收
           connection = null;
         } catch (SQLException e) {
           e.printStackTrace();
           flag = false;
         }
       }
   
       return flag;
     }
   }
   ```

   3. 编写字符编码过滤器

9. 导入静态资源（css, js, images, calendar)

## 登录功能实现

<img src="images/70.jpg" alt="70" style="zoom:40%;" />

1. 编写前端页面

2. 设置首页

```xml
<!--设置欢迎界面-->
<welcome-file-list>
  <welcome-file>login.jsp</welcome-file>
</welcome-file-list>
```

3. 编写dao层登录用户登录的接口

```java
//得到登录的用户
    public User getLoginUser(Connection connection, String userCode) throws SQLException;
```

4. 编写dao接口的实现类

```java
public class UserDaoImpl implements UserDao {
  @Override
  public User getLoginUser(Connection connection, String userCode) throws SQLException {
    PreparedStatement pstm = null;
    ResultSet rs = null;
    User user = null;
    if (connection != null) {
      String sql = "select * from smbms_user where userCode=?";
      Object[] params = {userCode};

      rs = BaseDao.execute(connection, pstm, rs, sql, params);
      if (rs.next()) {
        user = new User();
        user.setId(rs.getInt("id"));
        user.setUserCode(rs.getString("userCode"));
        user.setUserName(rs.getString("userName"));
        user.setUserPassword(rs.getString("userPassword"));
        user.setGender(rs.getInt("gender"));
        user.setBirthday(rs.getDate("birthday"));
        user.setPhone(rs.getString("phone"));
        user.setAddress(rs.getString("address"));
        user.setUserRole(rs.getInt("userRole"));
        user.setCreatedBy(rs.getInt("createdBy"));
        user.setCreationDate(rs.getTimestamp("creationDate"));
        user.setModifyBy(rs.getInt("modifyBy"));
        user.setModifyDate(rs.getTimestamp("modifyDate"));
      }
      //连接可能存在事务，在业务层调事务时再处理
      BaseDao.closeResource(null, pstm, rs);

    }
    return user;
  }
}
```

5. 业务层接口

```java
   public interface UserService {
       //用户登录
       public User login(String userCode, String password);
   }
```

6. 业务层实现类

```java
   public class UserServiceImpl implements UserService{
       //业务层都会调dao层，所以我们要引入dao层；
       private UserDao userDao;
       public UserServiceImpl() {
           userDao = new UserDaoImpl();
       }
   
       @Override
       public User login(String userCode, String password) {
           Connection connection = null;
           User user = null;
   
           try {
               connection = BaseDao.getConnection();
               //通过业务层调用对应的具体的数据库操作
               user = userDao.getLoginUser(connection,userCode);
           } catch (SQLException throwables) {
               throwables.printStackTrace();
           } finally {
               BaseDao.closeResource(connection, null,null);
           }
           return user;
       }
   }
```

7. 编写Servlet

```java
public class LoginServlet extends HttpServlet {
    //Servlet: 控制层，调用业务层代码
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("LoginServlet---start...");
        //获取用户名和密码，根据前端login.jsp编写
        String userCode = req.getParameter("userCode");
        String userPassword = req.getParameter("userPassword");

        //和数据库中的密码进行对比，调用业务层
        UserService userService = new UserServiceImpl();
        User user = userService.login(userCode, userPassword); //这里已经把登录的人给查出来了

        if (user != null) { //查有此人，可以登录
            //将用户的信息放到session中
            req.getSession().setAttribute(Constants.USER_SESSION, user);
            //跳转到主页
            resp.sendRedirect("jsp/frame.jsp");
        } else { //查无此人，无法登录
            //转发回登录页面，顺带提示它，用户名或者密码错误
            req.setAttribute("error", "用户名或者密码不正确");
            req.getRequestDispatcher("login.jsp").forward(req, resp);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

8. 注册Servlet

```xml
<!--Servlet-->
<servlet>
  <servlet-name>loginServlet</servlet-name>
  <servlet-class>com.jin.servlet.user.LoginServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>loginServlet</servlet-name>
  <url-pattern>/login.do</url-pattern>
</servlet-mapping>
```

9. 测试访问，确保以上功能成功！

## 登录功能优化

1. 注销功能：

   思路：移除Session，返回登录页面

```java
public class LogoutServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //移除用户的Constants.USER_SESSION
        req.getSession().removeAttribute(Constants.USER_SESSION);
        resp.sendRedirect("/login.jsp"); //返回登录页面
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

2. 注册xml

```xml
<servlet>
  <servlet-name>logoutServlet</servlet-name>
  <servlet-class>com.jin.servlet.user.LogoutServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>logoutServlet</servlet-name>
  <url-pattern>/jsp/logout.do</url-pattern>
</servlet-mapping>
```

3. **登录拦截优化**

编写一个过滤器，并注册

```java
public class SysFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;

        //过滤器，从session中获取用户
        User user = (User) request.getSession().getAttribute(Constants.USER_SESSION);
        if (user==null) { //已经被移除或者注销了，或者未登录
            response.sendRedirect("/smbms/error.jsp");
        } else {
            chain.doFilter(req, resp);
        }
    }
    @Override
    public void destroy() {}
}
```

```xml
<!--用户登录过滤器-->
<filter>
  <filter-name>sysFilter</filter-name>
  <filter-class>com.jin.filter.SysFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>sysFilter</filter-name>
  <url-pattern>/jsp/*</url-pattern>
</filter-mapping>
```

测试，登录，注销，权限，都要保证OK!

## 密码修改

1. 导入前端素材

```jsp
<li><a href="${pageContext.request.contextPath }/jsp/pwdmodify.jsp">密码修改</a></li>
```

2. 写项目，建议从底层向上写

<img src="images/98.jpg" alt="98" style="zoom:50%;" />

3. UserDao接口

```java
//修改当前用户密码
    public int updatePwd(Connection connection, int id, int password) throws SQLException;
```

4. UserDao接口实现类

```java
//修改当前用户密码
@Override
public int updatePwd(Connection connection, int id, int password) throws SQLException {
  PreparedStatement pstm = null;
  int execute = 0;
  if (connection != null) {
    String sql = "update smbms_user set userPassword = ? where id = ?";
    Object params[] = {password, id};
    execute = BaseDao.execute(connection, pstm, sql, params);
    BaseDao.closeResource(null, pstm,null);
  }
  return execute;
}
```

5. UserService层

```java
//根据用户ID修改密码
public boolean updatePwd(int id, String pwd);
```

6. UserService实现类

```java
@Override
public boolean updatePwd(int id, String pwd) {
  Connection connection = null;
  boolean flag = false;
  //修改密码
  try {
    connection = BaseDao.getConnection();
    if (userDao.updatePwd(connection, id, pwd) > 0) {
      flag = true;
    }
  } catch (SQLException e) {
    e.printStackTrace();
  } finally {
    BaseDao.closeResource(connection, null, null);
  }
  return flag;
}
```

7. Servlet记得实现复用，需要提取出方法！

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  String method = req.getParameter("method");
  // "savepwd".equals(method)
  if (method != null && method.equals("savepwd")) {
    this.updatePwd(req,resp);
  }
}

public void updatePwd(HttpServletRequest req, HttpServletResponse resp) {
  //从Session里面拿id
  Object o = req.getSession().getAttribute(Constants.USER_SESSION);
  String newpassword = req.getParameter("newpassword");
  boolean flag = false;

  if (o != null && newpassword != null) {
    UserService userService = new UserServiceImpl();
    flag = userService.updatePwd(((User) o).getId(), newpassword);
    if (flag) {
      req.setAttribute("message", "修改密码成功，请退出，使用新密码登录");
      //密码修改成功，移除当前session
      req.getSession().removeAttribute(Constants.USER_SESSION);
    } else {
      //密码修改失败
      req.setAttribute("message", "密码修改失败");
    }
  } else {
    req.setAttribute("message", "新密码有问题");
  }

  try {
    req.getRequestDispatcher("pwdmodify.jsp").forward(req, resp);
  } catch (ServletException | IOException e) {
    e.printStackTrace();
  }
}
```

8. 注册servlet

```xml
<servlet>
  <servlet-name>userServlet</servlet-name>
  <servlet-class>com.jin.servlet.user.UserServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>userServlet</servlet-name>
  <url-pattern>/jsp/user.do</url-pattern>
</servlet-mapping>
```

9. 测试