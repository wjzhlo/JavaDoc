# Java基础

## 一、字符串篇

### 1.1 String、StringBuffer、StringBuilder的区别

| &emsp;&emsp; | String                                                 | StringBuffer                                                 | StringBuilder                  |
| -------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------ |
| **是否可变**&emsp; | String 是不可变的，每次操作字符<br />串哦都会导致新的String对象生成，不仅会造成效率低下，还会浪费大量内存 | StringBuffer是可变类，任何对其指向的字符串的操作都不会产生新的对象。<br/>这是由于在创建时会自动分配一定的缓冲区，在字符串大小超出容量时，会自动增加容量 | 与StringBuffer一样，但速度更快&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; |
| **线程安全**&emsp;&emsp;&emsp;&emsp;&emsp; |  | 线程安全 | 线程不安全，故效率更高 |
| **使用场景**&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; |                                                              | 多线程操作字符串中使用 | 单线程操作字符串中使用 |
| **拓展功能** | | 支持更强大，允许替换某个字符 | 与 StringBuffer 相同 |



### 1.2 未定







## 二、SpringMvc

### 2.1 @RequestBody、@RequestParam、@PathVariable

#### 2.1.1 @RequestBody

@RequestBody 用于接收请求编码格式为 application/json、application/xml 类型的数据。如：

```java
@RequestMapping("/register")
@ResponseBody
public R doReigster(@RequestBody Map<String, Object> params) {
    User user = new User();
    user.setAccount((String) params.get("account"));
    user.setPassword((String) params.get("password"));
    user.setAvatar((String) params.get("avatar"));
    user.setUsername((String) params.get("username"));
    user.setState(StateConstant.ACTIVE);
    user.setCreated(new Date());
    user.setUpdated(new Date());
    user.setUserId(String.valueOf(idWorker.nextId()));
    R r = userService.doRegister(user);
    return r;
}
```

若前面没有加 @RequestBody，则是接收格式为 application/x-www-form-urlencoded 的数据

```Java
@RequestMapping("/register")
@ResponseBody
public R doReigster(String account, String password, String avatar, String username) {
    User user = new User();
    user.setAccount((String) params.get("account"));
    user.setPassword((String) params.get("password"));
    user.setAvatar((String) params.get("avatar"));
    user.setUsername((String) params.get("username"));
    user.setState(StateConstant.ACTIVE);
    user.setCreated(new Date());
    user.setUpdated(new Date());
    user.setUserId(String.valueOf(idWorker.nextId()));
    R r = userService.doRegister(user);
    return r;
}
```



#### 2.1.2 @RequestParam

@RequestParam 用于接收请求编码格式为 application/x-www-form-urlencoded 的数据，也可以接收 application/json 的数据。实际识别到的是

url 的 ? 后面的部分，如 http://aaa.com?id=5&isParent=0

可以通过 value 匹配到具体的哪个参数，通过 defaultValue 设置默认值

##### 示例

```java
@RequestMapping("/list")
@ResponseBody
public List<TreeNode> getCatList(@RequestParam(value = "id", defaultValue = "0") long parentId) {
    List<TreeNode> treeList = itemCatService.getCatList(parentId);
    return treeList;
}
```



#### 2.2.3 @PathVariable

@PathVariablee 实际使用的是 url 中带来的参数，要求名称必须一致

##### 示例

```java
@RequestMapping("/item/{itemId}")
@ResponseBody
public TbItem findItemById(@PathVariable Long itemId) {
    TbItem tbItem = itemService.getItemById(itemId);
    return tbItem;
}
```



### 2.2 controller 返回 JSON 数据中文乱码

#### 2.2.1 单个接口的解决方案

@ResponseBody 用于指定返回类型为： application/json

但也可以在 @RequestMapping 中添加参数 produces

```
@RequestMapping(value="/list", produces= MediaType.APPLICATION_JSON_VALUE+";charset=utf-8")
@ResponseBody
public String getItemCatList(String callback) {
    xxxxxxx
}
```

produces 参数的作用是：

1. 浏览器查看方便（json自动格式化，带搜索）

2. 指定编码格式，防止中文乱码

   

#### 2.2.2 全局解决方案

**引入  jackjson 的依赖**

```
<dependency>
    <groupId>org.codehaus.jackson</groupId>
    <artifactId>jackson-mapper-asl</artifactId>
    <version>1.9.13</version>
</dependency>
<dependency>
    <groupId>org.codehaus.jackson</groupId>
    <artifactId>jackson-core-asl</artifactId>
    <version>1.9.13</version>
</dependency>
```

**在 springmvc.xml 中配置**

```
<!-- 处理请求返回json字符串的乱码问题 -->
<!-- SpringMVC注解驱动 -->
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>text/plain;charset=utf-8</value>
                    <value>text/html;charset=UTF-8</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

**注意：** 配置要放在注解驱动前面！！

```
<!--  配置注解驱动  -->
<mvc:annotation-driven></mvc:annotation-driven>
```


























