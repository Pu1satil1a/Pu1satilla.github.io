---
title: JDBC事务
description: MySQL事务处理，使用MySQL，JDBC，DbUtils操作事务以及事务相关概念性问题
categories:
 - MySQL
 - Web
 - Java

tags: [MySQL,Web,Java,JDBC]
---

# 事务概述
- 事务指的是逻辑上一组操作，组成这组操作的各个单元要么全都成功，要么全部失败
- 事务作用：保证一个事务中多次操作要么全都成功，要么全都失败

## MySQL事务操作

sql语句 | 操作
----- | --------  
start transaction; | 开启事务  
commit; | 提交事务  
rollback; | 回滚事务  

MySQL默认的事务：一条sql语句就是一个事务 默认开启事务并提交事务  
手动提交事务：
- 显示地开启一个事务：strat transaction;
- 事务提交：commit代表从startsaction 到commit中的sql语句合并为一个事务，commit表示提交这个事务
- 事务回滚：rollback代表事务的回滚，从开启事务到事务回滚中间所有sql操作都认为无效，数据库没有被更新

# JDBC事务操作

Connection对象的方法名 | 描述
----|----
conn.setAutoCommit(false) | 开启事务
conn.commit() | 提交事务
conn.rollback() | 回滚事务

示例：
```java
public class MyJDBC {
    public static void main(String[] args) throws ClassNotFoundException {

        //        1.加载驱动
        Class.forName("com.mysql.jdbc.Driver");

        //        2.获取连接对象
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/databaseName?useUnicode=true&characterEncoding=utf8",
			"root", "password");
            //        3.开启事务,取消自动提交
            conn.setAutoCommit(false);

            //        4.获取执行平台
            Statement statm = conn.createStatement();

            //        5.执行sql语句
            statm.executeUpdate("INSERT INTO account VALUES (NULL ,'mark',1000);");
            statm.executeUpdate("INSERT INTO account VALUES (NULL ,'jack',10001);");

            //        6.提交事务
            conn.commit();

            //        7.关闭连接对象
            statm.close();
            conn.close();
        } catch (SQLException e) {
            try {
                assert conn != null;
                e.printStackTrace();
                conn.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }
    }
}
```
**注意**：执行与开启事务的连接对象`conn`必须是同一个才能对事务进行控制。

# DBUtils事务操作
QueryRunner构造方法传递参数限制是否能够进行事务操作
## 有参构造
```java
public class MyDbUtils {
    public static void main(String[] args) {
        QueryRunner qr = new QueryRunner(DbUTils.get_connection_pool());
    }
}
```
QueryRunner传入的参数为连接池对象，没有对连接对象`conn`进行操作，所以无法操作事务。

## 无参构造
```java
public class MyDbUtils {
    public static void main(String[] args) {
        try {

            //            1.载入数据库驱动
            Class.forName("com.mysql.jdbc.Driver");

            //            2.获取dbutils执行对象
            QueryRunner qr = new QueryRunner();

            //            3.获取数据库连接对象
            Connection conn = DriverManager.getConnection("
			jdbc:mysql://localhost:3306/xxx?useUnicode=true&characterEncoding=utf8","root", "xxx");

            //            4.开启事务，取消自动提交
            conn.setAutoCommit(false);

            //            5.执行sql语句
            qr.update(conn,"UPDATE account SET money=money-100 WHERE name='jack';");
            qr.update(conn,"UPDATE account SET money=money+100 WHERE name='rose';");

            //            6.提交或回滚事务
            conn.commit();

            //            7.关闭连接
            conn.close();
        } catch (ClassNotFoundException|SQLException e) {
            e.printStackTrace();
        }
    }
}
```
`QueryRunner`构造器没有传入参数，在其类对象的`update()`或者`query()`方法里面传入连接对象，sql语句，那么可以在使用`QueryRunner`对象操作数据库时使用连接对象`conn`调用事务操作的方法。

通过连接池获取的连接对象，进行事务处理：
```java
public class MyDbUtils {
    public static void main(String[] args) {
        Connection conn = null;
        try {
            //            1.获取dbutils执行对象
            QueryRunner qr = new QueryRunner();

            //            2.获取数据库连接对象
            DataSource pool = DbUTils.get_connection_pool();
            conn = pool.getConnection();

            //            3.开启事务，取消自动提交
            conn.setAutoCommit(false);

            //            4.执行sql语句
            qr.update(conn, "UPDATE account SET money=money-100 WHERE name='jack';");
            qr.update(conn, "UPDATE account SET money=money+100 WHERE name='rose';");

            //            5.提交或回滚事务
            conn.commit(); 
        } catch (SQLException e) {
            try {
                assert conn != null;
                conn.rollback();
                e.printStackTrace();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }finally{
			//            6.关闭连接
            conn.close();
		}
    }
}
```

# 案例(转账操作)
## web层  
servlet获取请求并且将请求传入到service层：
```java
public class TransferServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        doGet(request, response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {


        //        获取请求参数，并且传递给service层进行处理
        Map<String, String[]> params = request.getParameterMap();
        TransferService tS = new TransferService();
        boolean tS_boolean = tS.service(params);

        //        进行用户回显
        response.setContentType("text/html;charset=UTF-8");
        if (tS_boolean) {
            response.getWriter().write("转账成功！");
        } else {
            response.getWriter().write("转账失败！");
        }
    }
}
```
## service层  
进行事务处理，连接dao层：
```java
public class TransferService {
    public boolean service(Map<String, String[]> params) {

        //        Dao层对象进行转账操作
        TransferDao tsDao = new TransferDao();
        double amount = Double.parseDouble(params.get("transfer_amount")[0]);

        Connection conn = null;
        boolean flag = true;

        DataSource pool = DbUTils.get_connection_pool();
        try {
            //        获取事务处理的连接对象
            conn = pool.getConnection();

            //        开启手动事务
            conn.setAutoCommit(false);

            //        转出操作
            tsDao.out(params.get("tsPeople"), amount, conn);

            //            自设异常
            //            int i = 1 / 0;

            //        转入操作
            tsDao.in(params.get("payee"), amount, conn);
        } catch (Exception e) {

            //            事务回滚
            try {
                assert conn != null;
                conn.rollback();
                flag = false;
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            e.printStackTrace();
            return flag;
        } finally {

            //            事务提交,关闭连接对象
            try {
                if (conn != null) {
                    conn.commit();
                    conn.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        return flag;
    }
}
```
## dao层  
与数据库进行连接
```java
public class TransferDao {
    private QueryRunner qr = new QueryRunner();
    public void out(String[] tsPeople, double amount, Connection conn) throws SQLException {

        //        转出操作
        String sql = "UPDATE account SET money=money-? WHERE name=?";
        qr.update(conn, sql, amount, tsPeople[0]);
    }

    public void in(String[] payees, double amount, Connection conn) throws SQLException {
        //        转入操作
        String sql = "UPDATE account SET money=money+? WHERE name=?";
        qr.update(conn, sql, amount, payees[0]);
    }
}
```
# 使用ThreadLocal绑定连接资源
在上述事例中，参数必须通过方法传递才能完成整个事务的操作。`ThreadLocal`类可以在一个线程中共享数据。

## 相关内容
`java.lang.ThreadLocal`该类提供了线程局部`(thread-local)`变量，用于在当前线程中共享数据。`ThreadLocal`工具类底层是一个`Map`，`key`为当前线程，`value`存放需要共享的数据。
示意图：  
![](/assets/images/JDBC/ThreadLocal.PNG)

## DbUtils改进
使用DbUtils，Dbcp以及ThreadLocal完成连接池的自建工具类，包含的功能：
- 从当前线程存储连接对象conn
- 开启事务
- 提交事务
- 回滚事务
- 关闭连接

```java
public class DbUTils {

    //    1.建立连接池对象
    static private BasicDataSource dataSource = new BasicDataSource();

    static {
        Properties pro = get_properties();
        dataSource.setDriverClassName(pro.getProperty("driver_name"));
        dataSource.setUsername(pro.getProperty("user"));
        dataSource.setUrl(pro.getProperty("url"));
        dataSource.setPassword(pro.getProperty("pwd"));
        dataSource.setMaxActive(10);
        dataSource.setMinIdle(1);
        dataSource.setMaxIdle(5);
        dataSource.setInitialSize(1);
    }

    //    2.获取配置文件键值对
    static private Properties get_properties() {
        Properties pro = new Properties();

        //        2.1获取文件对象
        try {
            pro.load(DbUTils.class.getClassLoader().getResourceAsStream("com/company/config/DbUtils.properties"));
        } catch (IOException e) {
            throw new RuntimeException("连接池读取配置文件出错");
        }
        return pro;
    }

    //    3.返回连接池对象
    static public DataSource get_connection_pool() {
        return dataSource;
    }

    //    4.获取新的连接对象
    static private Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    //    5.将连接对象传入ThreadLocal中
    private static ThreadLocal<Connection> TLC = new ThreadLocal<>();

    static private void setConnection() throws SQLException {
        TLC.set(getConnection());
    }

    //    6.获取当前线程中的连接对象
    private static Connection getCurrentConnection() throws SQLException {
        Connection conn = TLC.get();
        if (conn == null) {
            setConnection();
            return TLC.get();
        }
        return conn;
    }

    //    7.开启事务
    static public void startTransaction() throws SQLException {

        //        7.1获取当前线程中连接对象
        Connection currentConn = getCurrentConnection();

        //        7.2开启事务，并且设置为手动事务
        currentConn.setAutoCommit(false);
    }

    //        当前线程中连接对象
    public static Connection currentConn;

    static {
        try {
            currentConn = getCurrentConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    //      提交事务
    static public void commitTransaction() throws SQLException {

        //        将连接对象从ThreadLocal中移除
        TLC.remove();

        //        事务提交
        currentConn.commit();

        //        关闭连接
        currentConn.close();
    }

    //    回滚事务
    static public void rollTransaction() throws SQLException {
        currentConn.rollback();
    }

    //    关闭连接
    static public void closeConnection() throws SQLException {
        currentConn.close();
    }
}
```
## 案例改进
**service层**
```java
public class TransferService {
    public boolean service(Map<String, String[]> params) {

        //        Dao层对象进行转账操作
        TransferDao tsDao = new TransferDao();
        double amount = Double.parseDouble(params.get("transfer_amount")[0]);

        boolean flag = true;

        try {
            //        DbUTils开启手动事务

            DbUTils.startTransaction();

            //        转出操作
            tsDao.out(params.get("tsPeople"), amount);

            //            自设异常
            //            int i = 1 / 0;

            //        转入操作
            tsDao.in(params.get("payee"), amount);
        } catch (Exception e) {
            //            出现异常，事务回滚
            try {
                DbUTils.rollTransaction();
                flag = false;
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            e.printStackTrace();
        } finally {
            //            事务提交,关闭连接对象
            try {
                DbUTils.commitTransaction();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        return flag;
    }
}
```
**Dao层**
```java
public class TransferDao {

    //    当前线程的连接对象
    private Connection conn = DbUTils.currentConn;
    private QueryRunner qr = new QueryRunner();
    public void out(String[] tsPeople, double amount) throws SQLException {

        //        转出操作
        String sql = "UPDATE account SET money=money-? WHERE name=?";
        qr.update(conn, sql, amount, tsPeople[0]);
    }

    public void in(String[] payees, double amount) throws SQLException {
        //        转入操作
        String sql = "UPDATE account SET money=money+? WHERE name=?";
        qr.update(conn, sql, amount, payees[0]);
    }
}
```
此案例通过`ThreadLocal`，`set()`,`get()`连接对象，将连接对象放置在当前线程中，防止了Dao层数据库操作污染到service层，最主要还是掌握ThreadLocal用法以及特性

# ㊕事务特性
## 事务的特性 ACID
- 原子性：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生
- 一致性：事务前后数据的完整必须保持一致
- 隔离性：多个用户并发访问数据库时，一个用户的事务不能被其它用户的事务所干扰，多个并发事务之间数据要相互隔离
- 持久性：一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接卸来及时数据库发生故障也不应该对其有任何影响

## 并发访问出现的问题
- 脏读：B事务读取到A事务尚未提交的数据       
- 不可重复读：一个事务读到了另一个事务已经提交(update)的数据。引发另外一个事务，在事务中的多次查询结果不一致
- 虚读/幻读：一个事务读到了另一个事务已经提交(insert,delect)的数据。导致另一个事务，在事务中多次查询的结果不一致

## 隔离级别
数据库规范规定4中隔离级别，分别用于描述两个事务并发的所有情况。
1. **read uncommitted**：读取尚未提交的数据，一个事务读到另一个事务没有提交的数据
- 存在：3个问题
- 解决：0个问题
2. **read commited**：读取提交的数据，一个事务读取另外一个提交的事务(Oracle默认)
- 存在：2个问题
- 解决：1个问题(脏读)
3. **repeatable read**：可重复读，在一个事务中读到的数据始终保持一致，无论另一个事务是否提交(MySQL默认)
- 存在：1个问题(虚读/幻读)
- 解决：2个问题
4. **serializable**：串行化，同时只能执行一个事务，相当于事务中的单线程
- 存在：0个问题
- 解决：3个问题

- 安全与性能成反比(能解决更多的问题，性能越低)
- 常见数据库隔离级别(Oracle:read commited, MySQL:repeatable read)












