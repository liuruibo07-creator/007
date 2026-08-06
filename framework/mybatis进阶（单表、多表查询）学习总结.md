# MyBatis 进阶学习总结（Day 49）

> 本节核心内容：**条件查询、动态 SQL、增删改与批量操作、注解开发，以及一对多关联查询**。

## 目录

- [一、条件查询与多参数传递](#一条件查询与多参数传递)
- [二、动态 SQL](#二动态-sql)
- [三、新增与主键回填](#三新增与主键回填)
- [四、动态修改](#四动态修改)
- [五、删除与批量删除](#五删除与批量删除)
- [六、批量新增与批量修改](#六批量新增与批量修改)
- [七、注解开发](#七注解开发)
- [八、多表查询与 VO](#八多表查询与-vo)
- [九、一对多查询](#九一对多查询)
- [十、关联查询方案对比](#十关联查询方案对比)
- [十一、重点速记](#十一重点速记)

---

## 一、条件查询与多参数传递

Mapper 方法需要接收多个查询条件时，常用两种方式。

### 1. 使用多个独立参数

建议通过 `@Param` 明确每个参数在 SQL 中的名称：

```java
List<Brand> selectByCondition(
        @Param("brandName") String brandName,
        @Param("companyName") String companyName,
        @Param("status") Integer status
);
```

XML 中的名称对应 `@Param` 的 `value`：

```xml
<select id="selectByCondition" resultType="Brand">
    select *
    from tb_brand
    where brand_name = #{brandName}
      and company_name = #{companyName}
      and status = #{status}
</select>
```

> `#{brandName}` 指的是 `@Param("brandName")` 中指定的名称。

### 2. 使用对象封装参数

当条件较多时，可以将参数封装成实体类或查询条件对象：

```java
List<Brand> selectByCondition(Brand condition);
```

XML 中使用对象属性名：

```xml
brand_name = #{brandName}
```

此时 `#{brandName}` 对应参数对象的 `brandName` 属性。

### 3. 两种方式对比

| 方式 | 适用场景 | SQL 中名称来源 |
|---|---|---|
| 多个参数 + `@Param` | 参数少且彼此独立 | `@Param` 的值 |
| 对象传参 | 参数较多、条件成组 | Java 对象属性名 |

---

## 二、动态 SQL

动态 SQL 可以根据参数是否存在，决定是否拼接对应的 SQL 片段。

### 1. `<if>` 标签

基本语法：

```xml
<if test="条件表达式">
    SQL 片段
</if>
```

字符串通常需要同时判断 `null` 和空字符串：

```xml
<if test="brandName != null and brandName != ''">
    and brand_name like concat('%', #{brandName}, '%')
</if>
```

数值、日期、数组或集合一般至少判断是否为 `null`；集合还常判断是否为空：

```xml
<if test="status != null">
    and status = #{status}
</if>

<if test="ids != null and ids.size() > 0">
    <!-- 批量条件 -->
</if>
```

### 2. `<where>` 标签

`<where>` 的作用：

1. 条件成立时自动拼接 `WHERE`。
2. 自动去除开头多余的 `AND` 或 `OR`。
3. 所有条件都不成立时，不生成 `WHERE`。

```xml
<select id="selectByCondition" resultType="Brand">
    select id, brand_name, company_name, ordered,
           description, status
    from tb_brand
    <where>
        <if test="brandName != null and brandName != ''">
            and brand_name like concat('%', #{brandName}, '%')
        </if>
        <if test="companyName != null and companyName != ''">
            and company_name like concat('%', #{companyName}, '%')
        </if>
        <if test="status != null">
            and status = #{status}
        </if>
    </where>
</select>
```

---

## 三、新增与主键回填

### 1. 普通新增

```java
int insert(Brand brand);
```

```xml
<insert id="insert">
    insert into tb_brand(
        brand_name, company_name, ordered, description, status
    )
    values (
        #{brandName}, #{companyName}, #{ordered},
        #{description}, #{status}
    )
</insert>
```

返回值通常表示受影响的行数。

### 2. 获取数据库生成的主键

前提：数据库主键支持自动增长，例如 MySQL 的 `AUTO_INCREMENT`。

```xml
<insert id="insert"
        useGeneratedKeys="true"
        keyProperty="id">
    insert into tb_brand(
        brand_name, company_name, ordered, description, status
    )
    values (
        #{brandName}, #{companyName}, #{ordered},
        #{description}, #{status}
    )
</insert>
```

| 属性 | 作用 |
|---|---|
| `useGeneratedKeys="true"` | 开启数据库生成主键回填 |
| `keyProperty="id"` | 将生成的主键写入参数对象的 `id` 属性 |

执行新增后可直接读取：

```java
mapper.insert(brand);
System.out.println(brand.getId());
```

### 3. 事务提交

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
```

默认情况下，`openSession()` 不会自动提交事务。执行新增、修改、删除后需要：

```java
sqlSession.commit();
```

也可以开启自动提交：

```java
SqlSession sqlSession = sqlSessionFactory.openSession(true);
```

> 实际业务中通常显式控制事务，以保证多个数据库操作要么全部成功，要么全部回滚。

---

## 四、动态修改

### 1. `<set>` 标签

`<set>` 用于动态生成更新语句：

1. 自动拼接 `SET` 关键字。
2. 自动去除末尾多余的逗号。

```xml
<update id="updateById">
    update tb_brand
    <set>
        <if test="brandName != null and brandName != ''">
            brand_name = #{brandName},
        </if>
        <if test="companyName != null and companyName != ''">
            company_name = #{companyName},
        </if>
        <if test="ordered != null">
            ordered = #{ordered},
        </if>
        <if test="description != null and description != ''">
            description = #{description},
        </if>
        <if test="status != null">
            status = #{status},
        </if>
    </set>
    where id = #{id}
</update>
```

> 必须保证至少有一个字段参与修改，否则可能生成缺少 `SET` 内容的无效 SQL。更新条件也必须谨慎，避免漏写 `WHERE` 导致全表更新。

---

## 五、删除与批量删除

### 1. 单条删除

```java
int deleteById(Integer id);
```

```xml
<delete id="deleteById">
    delete from tb_brand where id = #{id}
</delete>
```

### 2. 使用 `<foreach>` 批量删除

Mapper 接口：

```java
int deleteBatch(@Param("ids") List<Integer> ids);
```

XML：

```xml
<delete id="deleteBatch">
    delete from tb_brand
    where id in
    <foreach collection="ids"
             item="id"
             open="("
             separator=","
             close=")">
        #{id}
    </foreach>
</delete>
```

`<foreach>` 常用属性：

| 属性 | 作用 |
|---|---|
| `collection` | 要遍历的集合参数 |
| `item` | 当前循环元素的临时变量名 |
| `index` | 当前下标或 Map 的键，可选 |
| `open` | 整体内容左侧拼接文本 |
| `separator` | 元素之间的分隔符 |
| `close` | 整体内容右侧拼接文本 |

当 Mapper 参数是集合时，建议使用 `@Param` 明确名称，避免依赖默认参数名。

---

## 六、批量新增与批量修改

### 1. 批量新增

Mapper 接口：

```java
void insertBatch(@Param("brands") List<Brand> brands);
```

XML：

```xml
<insert id="insertBatch">
    insert into tb_brand(
        brand_name, company_name, ordered, description, status
    )
    values
    <foreach collection="brands"
             item="brand"
             separator=",">
        (
            #{brand.brandName},
            #{brand.companyName},
            #{brand.ordered},
            #{brand.description},
            #{brand.status}
        )
    </foreach>
</insert>
```

最终 SQL 类似：

```sql
insert into tb_brand(...) values (...), (...), (...)
```

### 2. 使用多条 SQL 批量修改

```xml
<update id="updateBatch">
    <foreach collection="brands"
             item="brand"
             separator=";">
        update tb_brand
        <set>
            <if test="brand.brandName != null
                      and brand.brandName != ''">
                brand_name = #{brand.brandName},
            </if>
            <if test="brand.companyName != null
                      and brand.companyName != ''">
                company_name = #{brand.companyName},
            </if>
            <if test="brand.ordered != null">
                ordered = #{brand.ordered},
            </if>
            <if test="brand.description != null
                      and brand.description != ''">
                description = #{brand.description},
            </if>
            <if test="brand.status != null">
                status = #{brand.status},
            </if>
        </set>
        where id = #{brand.id}
    </foreach>
</update>
```

MySQL 驱动使用分号一次执行多条语句时，需要在连接 URL 中启用：

```properties
jdbc.url=jdbc:mysql://localhost:3306/mybatis?allowMultiQueries=true
```

> `allowMultiQueries=true` 会扩大多语句执行能力，应确保 SQL 使用安全参数绑定，并谨慎评估安全风险。大批量更新还可以考虑 JDBC Batch、分批执行或按业务设计单条批量更新 SQL。

---

## 七、注解开发

简单 SQL 可以直接写在 Mapper 接口注解中。

### 1. 常用注解

| 注解 | 作用 |
|---|---|
| `@Select` | 查询 |
| `@Insert` | 新增 |
| `@Update` | 修改 |
| `@Delete` | 删除 |
| `@Options` | 配置生成主键等执行选项 |

### 2. 示例

```java
public interface BrandMapper {

    @Select("select * from tb_brand where id = #{id}")
    Brand selectById(Integer id);

    @Insert("""
            insert into tb_brand(
                brand_name, company_name, ordered,
                description, status
            ) values (
                #{brandName}, #{companyName}, #{ordered},
                #{description}, #{status}
            )
            """)
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(Brand brand);

    @Update("update tb_brand set status = #{status} where id = #{id}")
    int updateStatus(@Param("id") Integer id,
                     @Param("status") Integer status);

    @Delete("delete from tb_brand where id = #{id}")
    int deleteById(Integer id);
}
```

### 3. 注解与 XML 的选择

| 方式 | 适用场景 |
|---|---|
| 注解 | 简短、固定、结构清晰的 SQL |
| XML | 复杂动态 SQL、多表映射、可读性要求高的 SQL |

---

## 八、多表查询与 VO

多表查询的结果可能来自多个数据库表，也可能组合多个实体类的信息。此时可定义 VO 对象封装最终结果。

VO（View Object）中的属性取决于接口最终需要返回的字段，而不是必须与某一张表完全对应。

例如订单与订单项：

```java
@Data
public class OrderVo {
    private Long id;
    private BigDecimal payment;
    private Integer paymentType;
    private Integer status;
    private List<OrderItem> orderItems;
}
```

```text
OrderVo
├─ 订单基本信息
└─ List<OrderItem> 订单项集合
```

---

## 九、一对多查询

以“一张订单对应多个订单项”为例，可采用联表查询或嵌套查询。

### 1. 方案一：一条 SQL 联表查询

使用 `<collection>` 将多行子表数据封装到集合属性。

```xml
<resultMap id="orderVoMap" type="OrderVo">
    <id column="order_id" property="id"/>
    <result column="payment" property="payment"/>
    <result column="payment_type" property="paymentType"/>
    <result column="order_status" property="status"/>

    <collection property="orderItems" ofType="OrderItem">
        <id column="item_id" property="id"/>
        <result column="goods_name" property="goodsName"/>
        <result column="goods_price" property="goodsPrice"/>
        <result column="item_count" property="count"/>
        <result column="item_order_id" property="orderId"/>
    </collection>
</resultMap>

<select id="selectWithItemsById" resultMap="orderVoMap">
    select o.id           as order_id,
           o.payment,
           o.payment_type,
           o.status       as order_status,
           i.id           as item_id,
           i.goods_name,
           i.goods_price,
           i.count        as item_count,
           i.order_id     as item_order_id
    from tb_order o
    inner join tb_order_item i
            on o.id = i.order_id
    where o.id = #{id}
</select>
```

关键属性：

| 属性 | 含义 |
|---|---|
| `property` | 主对象中的集合属性名 |
| `ofType` | 集合元素的 Java 类型 |
| 子标签 `<id>` | 子对象主键映射，帮助 MyBatis 合并重复父对象 |
| 子标签 `<result>` | 子对象普通字段映射 |

> 若希望即使没有订单项也返回订单，可根据业务需要考虑使用 `LEFT JOIN`。

### 2. 方案二：两条 SQL 嵌套查询

先查询订单，再根据订单 ID 查询订单项。

订单映射：

```xml
<resultMap id="orderVoMap2" type="OrderVo">
    <id column="id" property="id"/>
    <result column="payment" property="payment"/>
    <result column="payment_type" property="paymentType"/>
    <result column="status" property="status"/>

    <collection
        property="orderItems"
        select="com.edu.mapper.OrderItemMapper.selectByOrderId"
        column="id"/>
</resultMap>

<select id="selectWithItemsById2" resultMap="orderVoMap2">
    select id, payment, payment_type, status
    from tb_order
    where id = #{id}
</select>
```

子查询：

```xml
<select id="selectByOrderId" resultType="OrderItem">
    select id, goods_name, goods_price, count, order_id
    from tb_order_item
    where order_id = #{orderId}
</select>
```

`<collection>` 中：

- `select`：要调用的另一条 Mapper SQL 的完整标识。
- `column`：从当前结果中取哪个字段，作为子查询实参。

### 3. 嵌套映射的自动映射

使用 `<collection>` 等嵌套结果映射时，为保证结果明确可靠，建议为父对象和子对象显式配置 `<id>` 与 `<result>`。

也可以配置完整自动映射：

```xml
<setting name="autoMappingBehavior" value="FULL"/>
```

但复杂联表查询中可能存在同名列或映射歧义，因此使用清晰的列别名和显式 `resultMap` 通常更安全。

---

## 十、关联查询方案对比

| 对比项 | 联表查询 + 嵌套结果 | 嵌套查询 |
|---|---|---|
| SQL 数量 | 通常 1 条 | 主查询 1 条，再执行子查询 |
| 映射方式 | `<collection ofType="...">` | `<collection select="...">` |
| 懒加载 | 不适用嵌套查询式懒加载 | 可配置懒加载 |
| N+1 风险 | 通常没有 | 查询多个父对象时可能出现 |
| 数据重复 | 联表结果中父表列会重复 | 主、子结果分别查询 |
| 适用场景 | 一次性需要完整数据 | 子数据不一定使用、适合按需加载 |

### 1. N+1 问题

假设先查询出 `N` 个订单：

```text
1 次：查询所有订单
N 次：分别查询每个订单的订单项
总计：N + 1 次查询
```

这会增加数据库往返次数，数据量大时可能产生明显性能问题。

### 2. 懒加载

嵌套查询可以配合懒加载：只有真正访问 `orderItems` 属性时，才执行查询订单项的 SQL。

核心配置示例：

```xml
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

> 懒加载能避免不必要的子查询，但若循环访问大量父对象的子集合，仍可能触发 N+1。应根据数据量、访问方式和性能要求选择方案。

---

## 十一、重点速记

> [!TIP]
> **动态查询用 `if + where`，动态修改用 `if + set`，批量参数用 `foreach`，一对多集合用 `collection`。**

- 多个独立参数建议使用 `@Param`。
- 对象传参时，`#{}` 中填写对象属性名。
- `<if>`：根据条件决定是否拼接 SQL。
- 字符串常判断 `!= null` 且 `!= ''`。
- `<where>`：自动添加 `WHERE`，并去掉开头多余的 `AND/OR`。
- `<set>`：自动添加 `SET`，并去掉结尾多余的逗号。
- `<foreach>`：遍历集合，常用于 `IN`、批量新增和批处理。
- `useGeneratedKeys="true"`：开启数据库生成主键回填。
- `keyProperty="id"`：将主键写入参数对象的 `id` 属性。
- `openSession()` 默认不自动提交，DML 后记得 `commit()`。
- 简单 SQL 可使用 `@Select`、`@Insert`、`@Update`、`@Delete`。
- `@Options` 可配置生成主键回填。
- 多表结果可定义 VO，属性由最终返回数据决定。
- 一对多集合使用 `<collection>` 映射。
- 单条联表查询通常没有 N+1 问题。
- 嵌套查询支持按需加载，但批量查询父对象时要警惕 N+1。
- 复杂多表查询建议使用列别名和显式 `resultMap`，避免同名字段映射错误。

---

> 本文根据思维导图 **《mybatis进阶（单表、多表查询）》** 整理，对动态 SQL、批量操作和一对多关联映射进行了结构化归纳。
