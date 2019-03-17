# Mybatis Plus实际运用

# 一、介绍

官方文档：https://mp.baomidou.com/

使用版本为mybatis 2.x

# 二、公共字段自动填充

将公共字段抽取到一个类中：

```java
package com.dftcmedia.tckk.microservice.common.model.dos;

import com.baomidou.mybatisplus.annotations.TableField;
import com.baomidou.mybatisplus.enums.FieldFill;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

/**
 * <p>DESC: 其他Do去继承该类即拥有该类的抽取来的公共字段</p>
 * <p>DATE: 2019/1/30</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: DengC</p>
 */
@Data
public class BaseDO<ID extends Serializable> implements Serializable {
    private static final long serialVersionUID = 1794049233259189385L;
    private ID id;

    /**
     * 本条数据的备注信息
     */
    private String memo;

    /**
     * 数据创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;

    /**
     * 数据修改时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
}

/**  FieldFill 说明
*public enum FieldFill {
    /**
     * 默认不处理
     */
    DEFAULT,
    /**
     * 插入填充字段
     */
    INSERT,
    /**
     * 更新填充字段
     */
    UPDATE,
    /**
     * 插入和更新填充字段
     */
    INSERT_UPDATE
}
*/
```



```java
package com.dftcmedia.tckk.microservice.common.conf;

import com.baomidou.mybatisplus.mapper.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * <p>DESC: mybatisPlus参数填充处理器</p>
 * <p>DATE: 2019/11/30</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: DengC</p>
 */
@Component
public class ParamFillingHandler extends MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        Object createTime = getFieldValByName("createTime", metaObject);
        if (createTime == null) {
            setFieldValByName("createTime", new Date(), metaObject);
        }
        updateFill(metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        Object updateTime = getFieldValByName("updateTime", metaObject);
        if (updateTime == null) {
            setFieldValByName("updateTime",new Date(), metaObject);
        }
    }

}

```

# 三、逻辑删除

## 1、创建实体类，加相关注解

```java
package com.eim.dos;

import com.baomidou.mybatisplus.annotation.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

/**
 * DESC: *
 * Created by zhengyong on 2019/3/9.
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder(toBuilder = true)
@TableName("tbl_employee")
public class JavaBean {

//        @TableId(type = IdType.ID_WORKER)
        private Long id ;

        private String lastName;

        private String email ;

        private Integer gender;

        private Integer age;

        @Version //乐观锁机制
        private Integer version;

        @TableLogic //逻辑删除注解
        private Integer delFlag;

        @TableField(fill = FieldFill.INSERT)
        private Date createTime;

        @TableField(fill = FieldFill.INSERT_UPDATE)
        private Date updateTime;

}
```

## 2、全局配置逻辑删除规则：

```yaml
# 配置slq打印日志
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  #sql打印配置
  global-config:
    db-config:
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
      id-type: id_worker  # 主键生成策略
```

==当调用delete方法的时候，会使用逻辑删除，将删除编辑字段修改为  1  即为删除；==

==且查询时，会自动加上逻辑删除字段做查询条件；==  

# 四、乐观锁

## 主要适用场景

意图：

当要更新一条记录的时候，希望这条记录没有被别人更新

乐观锁实现方式：

- 取出记录时，获取当前version
- 更新时，带上这个version
- 执行更新时， set version = newVersion where version = oldVersion
- 如果version不对，就更新失败

==操作：具体参考官方文档即可；==  

# 五、通用枚举

# 六、xml动态sql语句

##  # {}和${}区别

```properties
#{}表示一个占位符号，通过#{}可以实现preparedStatement向占位符中设置值，自动进行java类型和jdbc类型转换。#{}可以有效防止sql注入。 #{}可以接收简单类型值或pojo属性值。 如果parameterType传输单个简单类型值，#{}括号中可以是value或其它名称。

${}表示拼接sql串，通过${}可以将parameterType 传入的内容拼接在sql中且不进行jdbc类型转换， ${}可以接收简单类型值或pojo属性值，如果parameterType传输单个简单类型值，${}括号中只能是value。
```

通过mybatis提供的各种标签方法实现动态拼接sql。

需求：根据性别和名字查询用户

查询sql：

SELECT id, username, birthday, sex, address FROM `user` WHERE sex = 1 AND username LIKE '%张%'

```sql
SELECT id, username, birthday, sex, address FROM `user`
	WHERE sex = #{sex} AND username LIKE '%${username}%'
```

加入动态sql:

## 1、if标签

```sql
	SELECT id, username, birthday, sex, address FROM `user`
	WHERE 1=1
	<if test="sex != null and sex != ''">
		AND sex = #{sex}
	</if>
	<if test="username != null and username != ''">
		AND username LIKE
		'%${username}%'
	</if>
```

注意字符串类型的数据需要要做不等于空字符串校验。

## 2、Where标签

上面的sql还有where 1=1 这样的语句，很麻烦

可以使用where标签进行改造

```sql
SELECT id, username, birthday, sex, address FROM `user`
<!-- where标签可以自动添加where，同时处理sql语句中第一个and关键字 -->
	<where>
		<if test="sex != null">
			AND sex = #{sex}
		</if>
		<if test="username != null and username != ''">
			AND username LIKE
			'%${username}%'
		</if>
	</where>
```

## 3、foreach标签

```sql
	SELECT * FROM `user`
	<where>
		<!-- foreach标签，进行遍历 -->
		<!-- collection：遍历的集合，这里是QueryVo的ids属性 -->
		<!-- item：遍历的项目，可以随便写，，但是和后面的#{}里面要一致 -->
		<!-- open：在前面添加的sql片段 -->
		<!-- close：在结尾处添加的sql片段 -->
		<!-- separator：指定遍历的元素之间使用的分隔符 -->
		<foreach collection="ids" item="item" open="id IN (" close=")"
			separator=",">
			#{item}
		</foreach>
	</where>
```

## 4、mybatis整合springboot项目实操



```java
    /**
     * 连表查询 shop_info  shopcerf
     */
    @Select("<script>"
            + "SELECT c.id,c.certification_status,c.shop_id ,s.shop_name,s.logo,s.staff_size,s.province_id,s.city_id, "
            +"s.district_id,s.vip_status,s.create_time,s.tel "
            +"FROM dftc_app_shop_certification c "
            +"join dftc_app_shop_info s "
            +"on c.shop_id = s.id "
            + "<where>"
            + "<if test='sv != null and sv.auditState !=null  '>AND c.certification_status = #{sv.auditState}</if> "
            + "<if test='sv != null and sv.addressProvinceId !=null and sv.addressProvinceId != 100000  '>AND s.province_id = #{sv.addressProvinceId}</if> "
            + "<if test='sv != null and sv.addressCityId !=null and sv.addressCityId !=\"\" '>AND s.city_id = #{sv.addressCityId}</if> "
            + "<if test='sv != null and sv.addressDistrictId !=null and sv.addressDistrictId !=\"\" '>AND s.district_id = #{sv.addressDistrictId}</if> "
            + "<if test='sv != null and sv.shopName !=null and sv.shopName !=\"\" '>AND s.shop_name LIKE '%${sv.shopName}%'  </if> "
            + "<if test='sv != null and sv.tel !=null and sv.tel !=\"\" '>AND s.tel  LIKE '%${sv.tel}%' </if> "
            + "<if test='sv != null and sv.vip !=null and sv.vip !=\"\" '>AND s.vip_status = #{sv.vip} </if> "
            + "<if test='sv != null and sv.createTimeStart !=null and sv.createTimeEnd !=null '>AND s.create_time  between #{sv.createTimeStart} and #{sv.createTimeEnd}</if> "
            + "</where>"
            +"order by s.create_time desc"
            + "</script> ")
    List<ShopManagerRepastListVO> getListByShopInfo(@Param("sv") ShopManagerRepastLikeVo shopManagerRepastLikeVo);
```

==接受结果集的包装类，如下，字段，默认是以驼峰命名规则。==

==注意：连表查询中，两个表中相同字段，请用别名，或者，写出查询字段一一对应。== 

```java
package com.dftcmedia.tckk.microservice.oss.model.vos;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

/**
 * <p>DESC: </p>
 * <p>DATE: 2019/3/15</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ShopManagerRepastListVO {
    /**
     * 认证id
     */
    @JsonSerialize(using = ToStringSerializer.class)
    private Long id;

    /**
     * 店铺id
     */
    @JsonSerialize(using = ToStringSerializer.class)
    private Long shopId;

    /**
     * 店铺名称
     */
    private String shopName;

    /**
     * logo
     */
    private String logo;

    /**
     * staff_size 规模
     */
    private Integer staffSize;

    /**
     * 省id
     */
    private Integer provinceId;

    /**
     * 市id
     */
    private Integer cityId;

    /**
     * 区id
     */
    private Integer districtId;

    /**
     * 地区包装
     */
    private RegionVO regionVO;

    /**
     * 审核状态
     */
    private Integer certificationStatus;

    /**
     * 电话
     */
    private String tel;


    /**
     * vip状态
     */
    private Integer vipStatus;

    /**
     * 创建时间
     */
    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss", timezone="GMT+8")
    private Date createTime;
}
```

利用@Param("sv")将参数对象传入sql语句中

利用，sv.属性名 做判断和 传值

利用<script></script> 可以写动态sql语句（标签判断）

注意：字符类型应该判断非空

## 5、choose 标签的使用

有时候我们并不想应用所有的条件，而只是想从多个选项中选择一个。MyBatis提供了choose 元素，按顺序判断when中的条件出否成立，如果有一个成立，则choose结束。当choose中所有when的条件都不满则时，则执行 otherwise中的sql。类似于Java 的switch 语句，choose为switch，when为case，otherwise则为default。

         ==if是与(and)的关系，而choose是或（or）的关系。== 

```mysql
<select id="getStudentListChooseEntity" parameterType="StudentEntity" resultMap="studentResultMap">     
    SELECT * from STUDENT_TBL ST      
    <where>     
        <choose>     
            <when test="studentName!=null and studentName!='' ">     
                    ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')      
            </when>     
            <when test="studentSex!= null and studentSex!= '' ">     
                    AND ST.STUDENT_SEX = #{studentSex}      
            </when>     
            <when test="studentBirthday!=null">     
                AND ST.STUDENT_BIRTHDAY = #{studentBirthday}      
            </when>     
            <when test="classEntity!=null and classEntity.classID !=null and classEntity.classID!='' ">     
                AND ST.CLASS_ID = #{classEntity.classID}      
            </when>     
            <otherwise>     
                      
            </otherwise>     
        </choose>     
    </where>     
</select>     
```

## 6、set标签

当在update语句中使用if标签时，如果前面的if没有执行，则或导致逗号多余错误。使用set标签可以将动态的配置SET 关键字，和剔除追加到条件末尾的任何不相关的逗号。

 使用set+if标签修改后，如果某项为null则不进行更新，而是保持数据库原值。如下示例：

```sql
<!-- 更新学生信息 -->     
<update id="updateStudent" parameterType="StudentEntity">     
    UPDATE STUDENT_TBL      
    <set>     
        <if test="studentName!=null and studentName!='' ">     
            STUDENT_TBL.STUDENT_NAME = #{studentName},      
        </if>     
        <if test="studentSex!=null and studentSex!='' ">     
            STUDENT_TBL.STUDENT_SEX = #{studentSex},      
        </if>     
        <if test="studentBirthday!=null ">     
            STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday},      
        </if>     
        <if test="classEntity!=null and classEntity.classID!=null and classEntity.classID!='' ">     
            STUDENT_TBL.CLASS_ID = #{classEntity.classID}      
        </if>     
    </set>     
    WHERE STUDENT_TBL.STUDENT_ID = #{studentID};      
</update>   
```

## 7、**trim**标签

trim是更灵活的去处多余关键字的标签，他可以实践where和set的效果。

 where例子的等效trim语句：

```sql
<!-- 查询学生list，like姓名，=性别 -->     
<select id="getStudentListWhere" parameterType="StudentEntity" resultMap="studentResultMap">     
    SELECT * from STUDENT_TBL ST      
    <trim prefix="WHERE" prefixOverrides="AND|OR">     
        <if test="studentName!=null and studentName!='' ">     
            ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}),'%')      
        </if>     
        <if test="studentSex!= null and studentSex!= '' ">     
            AND ST.STUDENT_SEX = #{studentSex}      
        </if>     
    </trim>     
</select>     
```

set例子的等效trim语句：

```sql
<!-- 更新学生信息 -->     
<update id="updateStudent" parameterType="StudentEntity">     
    UPDATE STUDENT_TBL      
    <trim prefix="SET" suffixOverrides=",">     
        <if test="studentName!=null and studentName!='' ">     
            STUDENT_TBL.STUDENT_NAME = #{studentName},      
        </if>     
        <if test="studentSex!=null and studentSex!='' ">     
            STUDENT_TBL.STUDENT_SEX = #{studentSex},      
        </if>     
        <if test="studentBirthday!=null ">     
            STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday},      
        </if>     
        <if test="classEntity!=null and classEntity.classID!=null and classEntity.classID!='' ">     
            STUDENT_TBL.CLASS_ID = #{classEntity.classID}      
        </if>     
    </trim>     
    WHERE STUDENT_TBL.STUDENT_ID = #{studentID};      
</update>     
```

## 8、经典案例：

```java
package com.kingboy.repository.user;

import com.kingboy.domain.user.Sex;
import com.kingboy.domain.user.Status;
import com.kingboy.domain.user.User;
import com.kingboy.dto.user.UserQueryDTO;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * @author kingboy--KingBoyWorld@163.com
 * @date 2017/12/31 上午12:01
 * @desc 用户仓储.
 */
public interface UserRepository {

    /**
     * 添加用户
     * @Options返回在数据库中主键，自动赋值到user的id字段中
     * keyProperty = "id"的默认值为id,可以省略
     */
    @Insert("INSERT INTO `user` VALUES (#{id}, #{nickName}, #{phoneNumber}, #{sex}, #{age}, #{birthday}, #{status})")
    @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
    void saveUser(User user);

    /**
     * 批量保存用户
     * @param users
     */
    @Insert("<script>" +
            "INSERT INTO `user` VALUES " +
            "<foreach item = 'item' index = 'index' collection='list' separator=','>" +
            "(#{item.id}, #{item.nickName}, #{item.phoneNumber}, #{item.sex}, #{item.age}, #{item.birthday}, #{item.status})" +
            "</foreach>" +
            "</script>")
    @Options(useGeneratedKeys = true)
    void saveUserList(List<User> users);

    //使用set标签进行动态set，要注意条件判断：没被删除的用户才可以更新数据
    @Update("<script>"
            + "UPDATE `user` "
            + "<set>"
            + "<if test='nickName != null'>nick_name = #{nickName}, </if>"
            + "<if test='age != null'>age = #{age}, </if>"
            + "<if test='phoneNumber != null'>phone_number = #{phoneNumber}, </if>"
            + "<if test='birthday != null'>birthday = #{birthday}, </if>"
            + "<if test='status != null'>status = #{status}, </if>"
            + "<if test='sex != null'>sex = #{sex}, </if>"
            + "</set>"
            + "WHERE id = #{id} AND status != 'DELETE';"
            + "</script>")
    void updateUser(User user);


    //删除用户，软删除
    @Update("UPDATE `user` SET status = #{status} WHERE id = #{id}")
    void remove(@Param(value = "id") Long id, @Param(value = "status") Status status);

    //删除用户，硬删除
    @Delete("DELETE FROM `user` WHERE id = #{id}")
    void delete(@Param(value = "id") Long id);

    /**
     * 查询用户
     * 单个参数时，@Param注解可以省略
     * 在配置中指定了驼峰映射，所以@Results的结果映射可以省略，不是驼峰类型的仍然需要写结果映射。
     */
    @Select("SELECT * FROM `user` WHERE id = #{id}")
    @Results({
            @Result(column = "nick_name", property = "nickName"),
            @Result(column = "phone_number", property = "phoneNumber")
    })
    User get(Long id);

    //分页查询用户
    @Select("SELECT * FROM `user`")
    List<User> listUser();

    /**
     * 通过id集合查询用户
     * @param ids
     * @return
     */
    @Select("<script>"
            + "SELECT * FROM `user` WHERE id in "
            + "<foreach item='item' collection='list' open='(' close=')' separator=','>"
            + "#{item}"
            + "</foreach>"
            + "</script>")
    List<User> listUserByIds(List<Long> ids);

    /**
     * 根据条件查询用户
     * 注意其中nickName模糊查询的处理方法
     * 注意其中关于生日的区间判断
     * @param userQueryDTO
     * @return
     */
    @Select("<script>"
            + "SELECT * FROM `user`"
            + "<where>"
            + "<bind name='nickName' value=\"'%' + nickName + '%'\" />"
            + "<if test='nickName != null'>AND nick_name like #{nickName}</if>"
            + "<if test='phoneNumber !=null'>AND phone_number = #{phoneNumber}</if>"
            + "<if test='sex !=null'>AND sex = #{sex}</if>"
            + "<if test='age !=null'>AND age = #{age}</if>"
            + "<if test='fromBirthday !=null'>AND birthday > #{fromBirthday}</if>"
            + "<if test='toBirthday !=null'>AND birthday < #{toBirthday}</if>"
            + "<if test='status !=null'>AND status = #{status}</if>"
            + "</where>"
            + "</script>")
    List<User> queryByCondition(UserQueryDTO userQueryDTO);


    /**
     * 如果age有值，通过age查询
     * 如果age没有值，则通过sex查询
     * 如果age和sex都没值，则查询所有status为UNLOCK的用户
     * @param age
     * @param sex
     * @return
     */
    @Select("<script>"
            + "SELECT * FROM `user`"
            + "<where>"
            + "<choose>"
            + "<when test='age != null'>AND age = #{age}</when>"
            + "<when test='sex != null'>AND sex = #{sex}</when>"
            + "<otherwise>AND status = 'UNLOCK'</otherwise>"
            + "</choose>"
            + "</where>"
            + "</script>")
    List<User> getByOrderCondition(@Param("age") Integer age, @Param("sex") Sex sex);
}
```

