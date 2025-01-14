
## 前言


说到 MyBatis，很多小伙伴都会用，但未必用得“惊艳”。


实际上，这个轻量级的持久层框架还有很多隐藏的“宝藏技巧”。


如果你能掌握这些技巧，不但能让开发更高效，还能避免掉入一些常见的“坑”。


今天就从浅入深，分享 10 个让人眼前一亮的 MyBatis 开发技巧，每一个都配上具体的场景和代码示例，务求通俗易懂，希望对你会有所帮助。


(我最近开源了一个基于 SpringBoot\+Vue\+uniapp 的商城项目，欢迎访问和star。)\[[https://gitee.com/dvsusan/susan\_mall](https://github.com)]


## 1\. 灵活使用动态 SQL


很多小伙伴在写 SQL 的时候，喜欢直接用拼接字符串的方式，比如：



```
String sql = "SELECT * FROM user WHERE 1=1";
if (name != null) {
    sql += " AND name = '" + name + "'";
}

```

这种写法不仅麻烦，而且安全性很差（容易引发 SQL 注入）。


MyBatis 的动态 SQL 是专门为解决这种问题设计的，你可以用 `if`、`choose`、`foreach` 等标签来动态构造 SQL。


**示例：动态条件查询**



```
<select id="findUser" resultType="User">
    SELECT * FROM user
    WHERE 1=1
    <if test="name != null and name != ''">
        AND name = #{name}
    if>
    <if test="age != null">
        AND age = #{age}
    if>
select>

```

这个代码的好处是，SQL 逻辑清晰，不会因为某个参数为空就导致整个 SQL 报错。


## 2\. 善用 `resultMap` 自定义结果映射


有些小伙伴会遇到这样的问题：数据库表字段是下划线命名，但 Java 对象是驼峰命名。比如 `user_name` 对应 `userName`。如果直接用默认的 `resultType`，MyBatis 是无法自动映射的。


这个时候，用 `resultMap` 就能完美解决。


**示例：自定义结果映射**



```
<resultMap id="userResultMap" type="User">
    <id column="id" property="id"/>
    <result column="user_name" property="userName"/>
    <result column="age" property="age"/>
resultMap>

<select id="getUserById" resultMap="userResultMap">
    SELECT id, user_name, age FROM user WHERE id = #{id}
select>

```

有了 `resultMap`，再复杂的字段映射都可以轻松搞定。


## 3\. 利用 `foreach` 实现批量操作


有些小伙伴可能会遇到这种需求：传入一个 ID 列表，查询所有匹配的用户信息。如果用拼接字符串的方式生成 `IN` 条件，不但代码丑，还容易踩坑。


MyBatis 提供了 `foreach` 标签，可以优雅地处理这种场景。


**示例：批量查询**



```
<select id="findUsersByIds" resultType="User">
    SELECT * FROM user WHERE id IN
    <foreach item="id" collection="idList" open="(" separator="," close=")">
        #{id}
    foreach>
select>

```

传入的 `idList` 是一个 `List` 或数组，MyBatis 会自动帮你展开为 `IN (1, 2, 3)` 这样的格式，完全不用担心语法问题。


## 4\. MyBatis\-Plus 的分页功能


很多小伙伴在做分页的时候，习惯自己写 `LIMIT` 的 SQL，这样不仅麻烦，还容易出错。


其实，用 MyBatis\-Plus 的分页插件能省不少事。


**示例：MyBatis\-Plus 分页功能**



```
Page page = new Page<>(1, 10); // 第 1 页，每页 10 条
IPage userPage = userMapper.selectPage(page, null);
System.out.println("总记录数：" + userPage.getTotal());
System.out.println("当前页数据：" + userPage.getRecords());

```

只需引入分页插件，就能轻松完成分页操作，简直不要太爽。


## 5\. 使用 `@Mapper`的接口代理


有些小伙伴觉得 XML 文件太多太麻烦，其实 MyBatis 支持纯注解的开发模式，尤其是对于简单的 SQL，非常方便。


**示例：注解方式查询**



```
@Mapper
public interface UserMapper {

    @Select("SELECT * FROM user WHERE id = #{id}")
    User getUserById(int id);

    @Insert("INSERT INTO user(name, age) VALUES(#{name}, #{age})")
    void addUser(User user);
}

```

用这种方式，可以完全省掉 XML 配置，代码更加简洁。


## 6\. 二级缓存


MyBatis 内置了一级缓存（SqlSession 范围内），但对于多次查询的场景，可以开启二级缓存来提升性能。


**示例：开启二级缓存**



```
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
    settings>
configuration>

<mapper namespace="com.example.mapper.UserMapper">
    <cache/>
    <select id="getUserById" resultType="User">
        SELECT * FROM user WHERE id = #{id}
    select>
mapper>

```

开启二级缓存后，同一个 Mapper 下的查询会自动命中缓存，大幅提高性能。


## 7\. 动态表名切换


有些多租户系统需要在运行时切换表名，比如按租户分表。这种情况下，可以用 MyBatis 的动态 SQL 特性来实现。


**示例：动态表名**



```
<select id="getDataFromDynamicTable" resultType="Map">
    SELECT * FROM ${tableName} WHERE id = #{id}
select>

```

在调用时传入 `tableName` 参数，MyBatis 会动态替换表名。


## 8\. 用 `typeHandler` 自定义类型处理


有些小伙伴可能遇到过这种场景：数据库存的是 `1/0`，但在代码里想用 `true/false` 表示。


这种情况可以通过自定义 `typeHandler` 来实现。


**示例：自定义 TypeHandler**



```
@MappedTypes(Boolean.class)
@MappedJdbcTypes(JdbcType.INTEGER)
public class BooleanTypeHandler extends BaseTypeHandler {

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Boolean parameter, JdbcType jdbcType) throws SQLException {
        ps.setInt(i, parameter ? 1 : 0);
    }

    @Override
    public Boolean getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getInt(columnName) == 1;
    }
}

```

在 MyBatis 配置中注册这个 `typeHandler`，就可以实现自动类型转换了。


## 9\. 日志调试，快速排查问题


开发中经常需要排查 SQL 执行的问题，这时 MyBatis 的日志功能非常好用。


通过配置，可以轻松打印出完整的 SQL 和参数。


**示例：开启日志**



```
<configuration>
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    settings>
configuration>

```

日志会输出类似下面的内容：



```
==>  Preparing: SELECT * FROM user WHERE id = ?
==> Parameters: 1(Integer)
<==      Total: 1

```

有了这些日志，排查问题再也不头疼了。


## 10\. 多数据源支持


当系统需要连接多个数据库时，可以通过 MyBatis 的多数据源配置轻松搞定。


**示例：配置多数据源**



```
@Configuration
@MapperScan(basePackages = "com.example.mapper", sqlSessionTemplateRef = "sqlSessionTemplate1")
public class DataSourceConfig1 {
    @Bean(name = "dataSource1")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "sqlSessionFactory1")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource1") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean.getObject();
    }

    @Bean(name = "sqlSessionTemplate1")
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("sqlSessionFactory1") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```

通过类似的配置，就可以轻松切换多个数据源。


## 总结


MyBatis 的魅力在于简单、高效，但很多时候我们用得太“基础”，没有发挥它的全部潜力。


希望这 10 个技巧能帮你更高效地使用 MyBatis，也让你的代码看起来更“惊艳”。


如果觉得有帮助，记得收藏分享！


## 最后说一句(求关注，别白嫖我)


如果这篇文章对您有所帮助，或者有所启发的话，帮忙关注一下我的同名公众号：苏三说技术，您的支持是我坚持写作最大的动力。


![](https://img2024.cnblogs.com/blog/2238006/202501/2238006-20250110110205598-1778300774.png)


求一键三连：点赞、转发、在看。


关注公众号：【苏三说技术】，在公众号中回复：进大厂，可以免费获取我最近整理的10万字的面试宝典，好多小伙伴靠这个宝典拿到了多家大厂的offer。


 本博客参考[milou云加速器官网](https://www.milo333.com)。转载请注明出处！
