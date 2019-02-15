> 有时我们即厌倦了mybatis繁琐的语法，仅仅需要CRUD增删改查和乐观锁以及逻辑删除。
于是 模板标签 就诞生了，它支持自动支持逻辑删除，和乐观锁，同时极大的简化代码行数

* go语言部分，定义我们需要的的mapper 函数
```
type ExampleActivityMapper struct {
	SelectTemplete      func(name string) ([]Activity, error) `mapperParams:"name"`
	InsertTemplete      func(arg Activity) (int64, error)
	InsertTempleteBatch func(args []Activity) (int64, error) `mapperParams:"args"`
	UpdateTemplete      func(arg Activity) (int64, error)    `mapperParams:"name"`
	DeleteTemplete      func(name string) (int64, error)     `mapperParams:"name"`
}
```
* xml部分，模板标签默认指向于resultMap标签（resultMap="BaseResultMap"），因此先定义resultMap标签
```
    <!--logic_enable 逻辑删除字段-->
    <!--logic_deleted 逻辑删除已删除字段-->
    <!--logic_undelete 逻辑删除 未删除字段-->
    <!--version_enable 乐观锁版本字段,支持int,int8,int16,int32,int64-->
    <resultMap id="BaseResultMap">
        <id column="id" property="id"/>
        <result column="name" property="name" langType="string"/>
        <result column="pc_link" property="pcLink" langType="string"/>
        <result column="h5_link" property="h5Link" langType="string"/>
        <result column="remark" property="remark" langType="string"/>
        <result column="version" property="version" langType="int" version_enable="true"/>
        <result column="create_time" property="createTime" langType="time.Time"/>
        <result column="delete_flag" property="deleteFlag" langType="int" logic_enable="true" logic_undelete="1" logic_deleted="0"/>
    </resultMap>
```

> insertTemplete 插入，支持逻辑删除，默认insert值绑定deleteFlag=1（在resultMap中指定）

```
插入的数据是单个只需定义 InsertTemplete  func(arg Activity) (int64, error)
批量插入只需把 参数部分改为 数组  InsertTemplete func(args []Activity) (int64, error) `mapperParams:"args"`
<insertTemplete tables="biz_activity" />
```

> UpdateTemplete 修改，支持乐观锁，逻辑删除绑定deleteFlag=1（在resultMap中指定）

```
sets值可以为*?*,那么生成的sql为 <if test="name!=null">#{name}</>
<updateTemplete tables="biz_activity" sets="name?name = #{name}" wheres="name?name = #{name}"/>
```
> SelectTemplete 查询

```
<selectTemplete tables="biz_activity" wheres="name?name = #{name}" columns=""/>
```
> deleteTemplete 删除

```
<deleteTemplete tables="biz_activity" wheres="name?name = #{name}"/>
```
