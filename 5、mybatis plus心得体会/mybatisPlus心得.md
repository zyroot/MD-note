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