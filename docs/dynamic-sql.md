> 动态sql

GoMyBatis 的强大特性之一便是它的动态 SQL。如果你有使用 JDBC 或其它类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句的痛苦。例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

     
虽然在以前使用动态 SQL 并非一件易事，但正是GoMyBatis 提供了可以被用在任意 SQL 映射语句中的强大的动态 SQL 语言得以改进这种情形。

     
动态 SQL 标签和 JSTL 或基于类似 XML 的文本处理器相似。在GoMyBatis 之前的版本中，有很多标签需要花时间了解。GoMyBatis 大大精简了标签种类，现在只需学习原来一半的标签便可。GoMyBatis 采用功能强大的基于expr,govaluate,gojee 的3种表达式来淘汰其它大部分标签。

## 注意 新版本 表达式“title != nil”和“title != null”都是支持的。以下都用nil代替

* if
* choose (when, otherwise)
* trim (where, set)
* foreach

## if
动态 SQL 通常要做的事情是根据条件包含 where 子句的一部分。比如：
```
<select id="findActiveBlogWithTitleLike">
  SELECT * FROM BLOG 
  WHERE state = ‘ACTIVE’ 
  <if test="title != nil">
    AND title like #{title}
  </if>
</select>
```
这条语句提供了一种可选的查找文本功能。如果没有传入“title”，那么所有处于“ACTIVE”状态的BLOG都会返回；反之若传入了“title”，那么就会对“title”一列进行模糊查找并返回 BLOG 结果（细心的读者可能会发现，“title”参数值是可以包含一些掩码或通配符的）。
如果希望通过“title”和“author”两个参数进行可选搜索该怎么办呢？首先，改变语句的名称让它更具实际意义；然后只要加入另一个条件即可。
```
<select id="findActiveBlogLike">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’ 
  <if test="title != nil">
    AND title like #{title}
  </if>
  <if test="author != nil and author.name != nil">
    AND author_name like #{author.name}
  </if>
</select>
```
## choose, when, otherwise 
有时我们不想应用到所有的条件语句，而只想从中择其一项。针对这种情况，GoMyBatis 提供了 choose 标签，它有点像 Golang / Java 中的 switch 语句。
还是上面的例子，但是这次变为提供了“title”就按“title”查找，提供了“author”就按“author”查找的情形，若两者都没有提供，就返回所有符合条件的 BLOG（实际情况可能是由管理员按一定策略选出 BLOG 列表，而不是返回大量无意义的随机结果）。
```
<select id="findActiveBlogLike"
     >
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != nil">
      AND title like #{title}
    </when>
    <when test="author != nil and author.name != nil">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```
## trim, where, set
前面几个例子已经合宜地解决了一个臭名昭著的动态 SQL 问题。现在回到“if”示例，这次我们将“ACTIVE = 1”也设置成动态的条件，看看会发生什么。
```
<select id="findActiveBlogLike"
     >
  SELECT * FROM BLOG 
  WHERE 
  <if test="state != nil">
    state = #{state}
  </if> 
  <if test="title != nil">
    AND title like #{title}
  </if>
  <if test="author != nil and author.name != nil">
    AND author_name like #{author.name}
  </if>
</select>
```
如果这些条件没有一个能匹配上会发生什么？最终这条 SQL 会变成这样：
```
SELECT * FROM BLOG
WHERE
```
这会导致查询失败。如果仅仅第二个条件匹配又会怎样？这条 SQL 最终会是这样:
```
SELECT * FROM BLOG
WHERE 
AND title like ‘someTitle’
```
这个查询也会失败。这个问题不能简单地用条件句式来解决，如果你也曾经被迫这样写过，那么你很可能从此以后都不会再写出这种语句了。
GoMyBatis 有一个简单的处理，这在 90% 的情况下都会有用。而在不能使用的地方，你可以自定义处理方式来令其正常工作。一处简单的修改就能达到目的：
```
<select id="findActiveBlogLike">
  SELECT * FROM BLOG 
  <where> 
    <if test="state != nil">
         state = #{state}
    </if> 
    <if test="title != nil">
        AND title like #{title}
    </if>
    <if test="author != nil and author.name != nil">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```
where 标签只会在至少有一个子标签的条件返回 SQL 子句的情况下才去插入“WHERE”子句。而且，若语句的开头为“AND”或“OR”，where 标签也会将它们去除。
如果 where 标签没有按正常套路出牌，我们可以通过自定义 trim 标签来定制 where 标签的功能。比如，和 where 标签等价的自定义 trim 标签为：
```
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
```
prefixOverrides 属性会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它的作用是移除所有指定在 prefixOverrides 属性中的内容，并且插入 prefix 属性中指定的内容。
类似的用于动态更新语句的解决方案叫做 set。set 标签可以用于动态包含需要更新的列，而舍去其它的。比如：
```
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != nil">username=#{username},</if>
      <if test="password != nil">password=#{password},</if>
      <if test="email != nil">email=#{email},</if>
      <if test="bio != nil">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```
这里，set 标签会动态前置 SET 关键字，同时也会删掉无关的逗号，因为用了条件语句之后很可能就会在生成的 SQL 语句的后面留下这些逗号。（译者注：因为用的是“if”标签，若最后一个“if”没有匹配上而前面的匹配上，SQL 语句的最后就会有一个逗号遗留）  
若你对 set 标签等价的自定义 trim 标签的代码感兴趣，那这就是它的真面目：
```
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```
注意这里我们删去的是后缀值，同时添加了前缀值。
## foreach
动态 SQL 的另外一个常用的操作需求是对一个集合进行遍历，通常是在构建 IN 条件语句的时候。比如：
```
<select id="selectPostIn" >
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```
foreach 标签的功能非常强大，它允许你指定一个集合，声明可以在标签体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及在迭代结果之间放置分隔符。这个标签是很智能的，因此它不会偶然地附加多余的分隔符。
注意 你可以将任何可迭代对象（如 slice、array 等）、Map 对象或者数组对象传递给 foreach 作为集合参数。当使用可迭代对象或者数组时，index 是当前迭代的次数，item 的值是本次迭代获取的标签。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。
到此我们已经完成了涉及 XML 配置文件和 XML 映射文件的讨论。下一章将详细探讨 Golang API，这样就能提高已创建的映射文件的利用效率。
## bind
bind 标签可以从 govaluate表达式中创建一个变量并将其绑定到上下文。比如：
```
<select id="selectBlogsLike" >
  <bind name="pattern" value="'%' + name + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```
## sql,include
sql 标签可以创建一段逻辑判断或者sql并将其绑定到上下文,从而复用许多重复的逻辑片段。比如：
```
 <sql id="links"> pc_link,h5_link </sql>
 <select id="selectLinks">
        select
        <include refid="links"/>
        from biz_activity where delete_flag = 1
    </select>
```
你也可以加入例如if，foreach,trim,choose, when, otherwise...等等 你需要的逻辑标签
```
 <sql id="links">
     pc_link,h5_link
     <!-- select ....  from 中,需要不启用TypeConvert的话，使用${} 而不是 #{},否则传入column 会被类型转换器转换为带有'符号的'column'而不是 column -->
     <if test="column != nil">,${column}</if>
 </sql>
 <select id="selectLinks">
        select
        <include refid="links"/>
        from biz_activity where delete_flag = 1
    </select>
```