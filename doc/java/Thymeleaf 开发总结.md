[TOC]

## HTML5 中 Thymleaf 头约束信息

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
</html>
```

## Spring Boot 1.x 下替换 Thymleaf  版本

```xml
<!--  替换 thymeleaf 的版本  -->
<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
<thymeleaf-layout-dialect.version>
    2.0.4
</thymeleaf-layout-dialect.version>
```

## Spring Boot 与 Thymleaf 整合坐标

```xml
<!--Spring boot 与 thymeleaef 整合-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## thymeleaf 基础知识

1. 在 html 页面中引入thymeleaf命名空间，即`<html xmlns:th=http://www.thymeleaf.org></html>`，此时在html模板文件中动态的属性使用th:命名空间修饰 

2. 引用静态资源文件，比如CSS和JS文件，语法格式为`@{}`，如`@{/js/blog/blog.js}`会引入/static目录下的`/js/blog/blog.js`文件 

3. 访问spring-mvc 中model的属性，语法格式为`${}`，如`${user.id}`可以获取model里的user对象的id属性 

4. 循环，在html的标签中，加入`th:each="value:${list}"`形式的属性，如`<span th:each="user:${users}"></span>`可以迭代users的数据 

5. 判断，在html标签中，加入`th:if=”表达式”`可以根据条件显示 HTML元素 

    ```html
    <span th:if="${not #lists.isEmpty(blog.publishTime)}"> 
    <span 
          id="publishtime" 
          th:text="${#dates.format(blog.publishTime, 'yyyy-MM-dd HH:mm:ss')}">
        </span> 
    </span> 
    ```

    以上代码表示若 blog.publishTime 时间不为空，则显示时间 


6. 时间的格式化 

    表示将时间格式化为` yyyy-MM-dd HH:mm:ss`格式化写法与Java格式化Date的写法是一致的。 

    ```html
    ${#dates.format(blog.publishTime,'yyyy-MM-dd HH:mm:ss')} 
    ```

7. 字符串拼接，有两种形式 
   比如拼接这样一个`URL:/blog/delete/{blogId} `
   第一种：`th:href="'/blog/delete/' + ${blog.id }" `
   第二种：`th:href="${'/blog/delete/' + blog.id }" `

## Thymeleaf 传值问题

### URL 传值

```html
th:href="@{/admin/goodsList?pageNum={num}(num=${goodsGrid.current-1})}"

<a th:href="@{/params(id=1,name="zwz")}"></a>
```

### Thymleaf 页面中 js 中获取后台的值

使用两对方括号 `[[]]`

```js
var msg= [[${msg}]];
```

### Thymleaf 页面中事件传值

```html
th:onclick="'javascript:getGoodsDetail('+${item.key.id}+')'"

<input type="button"  value="收&nbsp&nbsp&nbsp&nbsp货"                                                   th:onclick="'javascript:ordersReceipt('+${items.orderNum}+','+${items.orderPrice}+','+${items.goods.id}+')'"
                                                                                class="btn btn-info" th:if="${items.orderState==2}"/>
```


##  Thymleaf 引入 HTML 页面
```html
th:include="html 页面的位置"

<span th:include="/admin/main_top" />
```
## href 路径拼接

```html
 <a th:href="@{'/goods/goodsId/'+${goods.id}+'.html'}" class="btn btn-danger">
     取消支付
</a>
```

## 其他

 Themeleaf 遍历最好使用 span 标签，而不要图省事直接在 div 写，防止样式出现问题

```html
<span th:each="items : ${goodsExtendList}" th:if="${goodsExtendList != null}">
 	  <div class="story">省略  </div>  <!-- 循环结束-->
</span>
```

#### thymeleaf if 多条件判断

```html
<img th:src="@{{imgUrl}(imgUrl=${item.value.imgUrl})}" 
	 th:if="${item.value != null and  #strings.contains(item.value.imgUrl,'sysecondhandmarket')}"/>
```

#### thymleaf 非空判断使用 

```
<span th:text="${cur_user.?name}"></span>
```

