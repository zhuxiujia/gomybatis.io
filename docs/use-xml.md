# 使用教程,代码文件请移步Github文件夹https://github.com/zhuxiujia/GoMybatis/tree/master/example

准备好model层数据模型 和 xml 文件

Example_Activity.go
```
package example

import "time"
//模型Activity
type Activity struct {
	Id         string    `json:"id"`
	Uuid       string    `json:"uuid"`
	Name       string    `json:"name"`
	PcLink     string    `json:"pcLink"`
	H5Link     string    `json:"h5Link"`
	Remark     string    `json:"remark"`
	CreateTime time.Time `json:"createTime"`
	DeleteFlag int       `json:"deleteFlag"`
}
```

Example_ActivityMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://github.com/zhuxiujia/GoMybatis/blob/master/mybatis-3-mapper.dtd">
<mapper>
    <resultMap id="BaseResultMap" type="example.Activity">
        <id column="id" property="id"/>
        <result column="name" property="name" goType="string"/>
        <result column="pc_link" property="pcLink" goType="string"/>
        <result column="h5_link" property="h5Link" goType="string"/>
        <result column="remark" property="remark" goType="string"/>
        <result column="create_time" property="createTime" goType="time.Time"/>
        <result column="delete_flag" property="deleteFlag" goType="int"/>
    </resultMap>
    <!--List<Activity> selectByCondition(@Param("name") String name,@Param("startTime") Date startTime,@Param("endTime") Date endTime,@Param("index") Integer index,@Param("size") Integer size);-->
    <!-- 后台查询产品 -->
    <select id="selectByCondition" resultMap="BaseResultMap">
        select * from biz_activity where delete_flag=1
        <if test="name != ''">
            and name like concat('%',#{name},'%')
        </if>
        <if test="startTime != ''">
            and create_time >= #{startTime}
        </if>
        <if test="endTime != ''">
            and create_time &lt;= #{endTime}
        </if>
        order by create_time desc
        <if test="page >= 0 and size != 0">limit #{page}, #{size}</if>
    </select>
    <!--int countByCondition(@Param("name")String name,@Param("startTime") Date startTime, @Param("endTime")Date endTime);-->
    <select id="countByCondition">
        select count(id) from biz_activity where delete_flag=1
        <if test="name != ''">
            and name like concat('%',#{name},'%')
        </if>
        <if test="startTime != ''">
            and create_time >= #{startTime}
        </if>
        <if test="endTime != ''">
            and create_time &lt;= #{endTime}
        </if>
    </select>
    <!--List<Activity> selectAll();-->
    <select id="selectAll">
        select * from biz_activity where delete_flag=1 order by create_time desc
    </select>
    <!--Activity selectByUUID(@Param("uuid")String uuid);-->
    <select id="selectByUUID">
        select * from biz_activity
        where uuid = #{uuid}
        and delete_flag = 1
    </select>
    <select id="selectById">
        select * from biz_activity
        where id = #{id}
        and delete_flag = 1
    </select>
    <select id="selectByIds">
        select * from biz_activity
        where delete_flag = 1
        and id in
        <foreach collection="ids" item="item" index="index" open="(" close=")">
            #{item}
        </foreach>
    </select>
    <update id="deleteById">
        update biz_activity
        set delete_flag = 0
        where id = #{id}
    </update>
    <update id="updateById">
        update biz_activity
        <set>
            <if test="name != ''">name = #{name,jdbcType=VARCHAR},</if>
            <if test="pcLink != ''">pc_link = #{pcLink,jdbcType=VARCHAR},</if>
            <if test="h5Link != ''">h5_link = #{h5Link,jdbcType=VARCHAR},</if>
            <if test="remark != ''">remark = #{remark,jdbcType=VARCHAR},</if>
            <if test="createTime != ''">create_time = #{createTime,jdbcType=TIMESTAMP},</if>
            <if test="deleteFlag != ''">delete_flag = #{deleteFlag},</if>
        </set>
        where id = #{id} and delete_flag = 1
    </update>
    <insert id="insert">
        insert into biz_activity
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="id != ''">id,</if>
            <if test="name != ''">name,</if>
            <if test="pcLink != ''">pc_link,</if>
            <if test="h5Link != ''">h5_link,</if>
            <if test="remark != ''">remark,</if>
            <if test="createTime != ''">create_time,</if>
            <if test="deleteFlag != ''">delete_flag,</if>
        </trim>

        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="id != ''">#{id,jdbcType=VARCHAR},</if>
            <if test="name != ''">#{name,jdbcType=VARCHAR},</if>
            <if test="pcLink != ''">#{pcLink,jdbcType=VARCHAR},</if>
            <if test="h5Link != ''">#{h5Link,jdbcType=VARCHAR},</if>
            <if test="remark != ''">#{remark,jdbcType=VARCHAR},</if>
            <if test="createTime != ''">#{createTime,jdbcType=TIMESTAMP},</if>
            <if test="deleteFlag != ''">#{deleteFlag},</if>
        </trim>
    </insert>
</mapper>
```