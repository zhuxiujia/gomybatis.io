## 乐观锁 主要适用场景
意图：

当要更新一条记录的时候，希望这条记录没有被别人更新

乐观锁实现方式：

* 取出记录时，获取当前version
* 更新时，带上这个version
* 执行更新时， set version = newVersion where version = oldVersion
* 如果version不对，就更新失败

### 逻辑删除 主要适用场景
意图：
0、需要保证数据绝对不丢失的
1、如果真的删除，会对其他数据带来严重的后果的。
2、金融应用，大数据分布式应用，无所谓容量只要确保数据安全的。

逻辑删除实现方式：
将对应数据中代表是否被删除字段状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录。

## 乐观锁和逻辑删除 配置使用步骤

> 1 确保mapper 内容中含有resultMap：
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
>2 确保使用的是模板标签 <*Templete>，而不是<insert><update><delete><select>.例如：
```
    <insertTemplete tables="biz_activity" />
    <insertTemplete tables="biz_activity" id="InsertTempleteBatch"/>
    <selectTemplete tables="biz_activity" wheres="name?name = #{name}" columns=""/>
    <updateTemplete tables="biz_activity" sets="name?name = #{name}" wheres="name?name = #{name}"/>
    <deleteTemplete tables="biz_activity" wheres="name?name = #{name}"/>
```

>相关配置完成后，updateTemplete生成的sql例如以下内容：
```
update set name = 'rs168',version = 1 from biz_activity where name = 'rs168' and delete_flag = 1 and version = 0
```