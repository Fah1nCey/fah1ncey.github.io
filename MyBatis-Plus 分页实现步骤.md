# ⭐ MyBatis-Plus 分页实现步骤

##### 1. 引入分页插件相关依赖（以Spring Boot 2，jdk8为例）

```java
		<dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.9</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-jsqlparser-4.9</artifactId>
        </dependency>
            
        <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-bom</artifactId>
                <version>3.5.9</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```

> 具体查看：https://mvnrepository.com/artifact/org.mindrot/jbcrypt

##### 2. 注入分页拦截器（MP 必须配置）

```java
@Configuration
@MapperScan("scan.your.mapper.package")
public class MybatisPlusConfig {

    /**
     * 添加分页插件, 拦截所有查询语句
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL)); 
        // 如果配置多个插件, 切记分页最后添加
        // 如果有多数据源可以不配具体类型, 否则都建议配上具体的 DbType
        return interceptor;
    }
}
```

> 具体查看：[分页插件 | MyBatis-Plus](https://baomidou.com/plugins/pagination/)

##### 3. 使用 Page<T> 对象发起分页查询

MP 的分页只需要 1 行：

```java
Page<User> page = userService.page(
        new Page<>(current, size),
        queryWrapper
);
```

或者传 null：

```java
Page<User> page = userService.page(new Page<>(current, size), null);
```

##### 4. Page<T> 常用字段，获取分页信息

Page 对象包含分页全部数据：

| 字段    | 说明               |
| ------- | ------------------ |
| records | 当前页数据 List<T> |
| total   | 总记录数           |
| size    | 每页条数           |
| current | 当前页             |
| pages   | 总页数             |

示例：

```java
long total = page.getTotal();        // 总记录数
List<User> list = page.getRecords(); // 当前页数据
long pages = page.getPages();        // 总页数
```

##### 5. Controller 分页示例

```
@PostMapping("/list/page")
public BaseResponse<Page<User>> listUsers(@RequestBody UserQueryRequest req) {
    long current = req.getCurrent();   // 当前页
    long size = req.getPageSize();     // 每页条数

    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like(StringUtils.isNotBlank(req.getName()), "name", req.getName());

    Page<User> pageResult = userService.page(new Page<>(current, size), queryWrapper);
    // 统一响应体
    return ResultUtils.success(pageResult);
}
```

返回内容示例：

```java
{
  "code": 0,
  "data": {
    "records": [ ... ],
    "total": 55,
    "current": 1,
    "size": 10,
    "pages": 6
  }
}
```