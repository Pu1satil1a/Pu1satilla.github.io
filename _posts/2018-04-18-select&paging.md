---
title: JDBC案例
description: 使用JDBC完成分页以及条件查询案例
categories:
 - MySQL
 - Web
 - Java

tags: [MySQL,Web,Java,JDBC]
---

# 条件查询
## 流程
1. 编写查询表单
- 根据查询规则(关键字、分类、是否热门)建立表单
- post方式提交
2. 指定表单处理的Servlet(ConditionSelectServlet)
3. 获取表单数据，并且将数据封装到Condition对象中
4. 告知service层，传递condition对象，获取条件查询之后的productList，service层告知dao层，获取条件查询之后的productList
5. servlet获取查询之后的product集合productList，调用service层方法获取分类对象集合categoryList
6. 将productList、categoryList、Condition传递到request域中，用于转发jsp之后显示产品信息、分类信息、以及condition回显

## 前端查询表单
```html
<form action="${pageContext.request.contextPath}/Select" method="post">
    <table>
        <tr>
            <td>
                请输入关键字：<input type="text" value="${condition.keywords}" name="keywords">
            </td>
            <td>
                类别：
                <select name="cid">
                    <option value="">--请选择--</option>
                    <c:forEach items="${categories}" var="category">
                        <option value="${category.cid}">${category.cname}</option>
                    </c:forEach>
                </select>
            </td>
            <td>
                热门：
                <select name="is_hot">
                    <option value="2">--请选择--</option>
                    <option value="0">否</option>
                    <option value="1">是</option>
                </select>
            </td>
            <td>
                &nbsp;&nbsp;&nbsp;&nbsp;<input type="submit" value="确认">
            </td>
        </tr>
    </table>
</form>
```
## 条件查询servlet
将前端查询的表单传递给SelectServlet
```java
package com.company.web;

import com.company.domain.Condition;
import com.company.domain.Product;
import com.company.domain.ProductCategory;
import com.company.service.AdminService;
import org.apache.commons.beanutils.BeanUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.sql.SQLException;
import java.util.List;
import java.util.Map;

@WebServlet(name = "Select", urlPatterns = "/Select")
public class ConditionSelectServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException
            , IOException {
        doGet(request, response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException
            , IOException {

        //        0.设置请求编码方式
        request.setCharacterEncoding("utf-8");

        //        1.获取数据
        Map<String, String[]> map = request.getParameterMap();

        //        2.封装数据
        Condition condition = new Condition();
        try {
            BeanUtils.populate(condition, map);
        } catch (IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }

        //        3.将condition实体传递给service，获取查询到的product集合
        List<Product> products = null;
        try {
            products = new AdminService().getProductList(condition);
        } catch (SQLException e) {
            e.printStackTrace();
        }

        //        4.告知service，获取categoryList
        List<ProductCategory> Categories = null;
        try {
            Categories = new AdminService().getCategoryList();
        } catch (SQLException e) {
            e.printStackTrace();
        }

        //        5.将查询到的products、condition、categoryList回显给用户
        request.setAttribute("Categories", Categories);
        request.setAttribute("products", products);
        request.setAttribute("condition", condition);
        request.getRequestDispatcher(request.getContextPath() + "/admin/product/list.jsp").forward(request, response);

    }
}
```
## 封装条件，传递给dao层
**在domain包中建立condition类，用于封装用户条件查询的条件信息**
```java
package com.company.domain;

public class ProductCategory {
    private int cid;
    private String cname;

    public int getCid() {
        return cid;
    }

    public void setCid(int cid) {
        this.cid = cid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }
}
```

**将数据封装到condition对象中**
```java
//        2.封装数据
Condition condition = new Condition();
try {
	BeanUtils.populate(condition, map);
} catch (IllegalAccessException | InvocationTargetException e) {
	e.printStackTrace();
}
```

## dao层处理condition
**获取从service层获取的condition对象，根据对象内容编写相应的sql**
```java
public List<Product> getProductList(Condition condition) throws SQLException {

	//        1.获取sql语句执行对象
	QueryRunner queryRunner = new QueryRunner(DbUTils.get_connection_pool());

	//        2.根据condition编写sql语句
	String sql = "SELECT * FROM product WHERE 1=1";

	//        2.0要求建立一个数组，用于存储？对应参数
	List<Object> params = new ArrayList<>();

	//        2.1关键字不为空，添加pdesc
	if (condition.getKeywords() != null && !"".equals(condition.getKeywords().trim())) {
		sql += " AND pdesc LIKE ?";

		//            进行模糊匹配的条件查询
		params.add("%" + condition.getKeywords().trim() + "%");
	}

	//        2.2 cid不为空,添加cid作为条件
	if (condition.getCid() != null && !"".equals(condition.getCid())) {
		sql += " AND cid=?";
		params.add(condition.getCid());
	}

	//        2.3is_hot不为空，添加is_hot
	if (condition.getIs_hot() != 2) {
		sql += " AND is_hot=?";
		params.add(condition.getIs_hot());
	}

	//        3.执行queryRunner,获取product对象集合并且返回
	return queryRunner.query(sql, new BeanListHandler<>(Product.class), params.toArray());
}
```
**注意：**  
QueryRunner对象传递的是数组或者多个参数，不能是集合，所以需要先将集合转化成集合
# 分页
## 流程
1. 根据浏览器请求分析需求
- 请求页面产品信息
- 分页条(请求页面，总分页数)
2. Web层
- control: 告知service层，计算总页数，计算总条目数，获取对应页面产品集合 ---->第3步
- model: JavaBean对象，封装了第1步需要的信息 
- view: 将JavaBean对象设置在请求域中，并且转发给Jsp文件显示
3. Service层
- 告知Dao层，请求产品的所有信息
- 告知Dao层，请求对应页面的产品集合
4. Dao层
与数据库进行交互，获取Service层需要的数据，编写相应方法  
**Image**
![](/assets/images/case/Page.PNG)


# 细节
- `query`参数:`new ScalarHandler<>()`返回值为Long型，不能强转成int类型，但是调用`intValue()`方法可行
