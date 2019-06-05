# 常见springboot 配置

## (1)、springboot之jackson

```yaml
spring:
  jackson:
    #日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    serialization:
       #格式化输出 
      indent_output: true
      #忽略无法转换的对象
      fail_on_empty_beans: false
    #设置空如何序列化
    defaultPropertyInclusion: NON_EMPTY
    deserialization:
      #允许对象忽略json中不存在的属性
      fail_on_unknown_properties: false
    parser:
      #允许出现特殊字符和转义符
      allow_unquoted_control_chars: true
      #允许出现单引号
      allow_single_quotes: true
```

## (2) redis

```yaml
spring:
  redis:
#    host: 192.168.2.34
    host: 47.102.115.60
    password: 123456
    port: 60010
```

## (3) mysql

```yaml
spring: 
   datasource:
    url: jdbc:mysql://192.168.2.33:3306/dftc_ots?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: Tckk@2019_mysql
```

## (4) mybatisPlus

```yaml
mybatis-plus:
  configuration:
    cache-enabled: false
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    id-type: 2
    db-column-underline: true
    table-prefix: dftc_oss_
    refresh-mapper: true
  type-handlers-package: com.dftcmedia.tckk.microservice.ots.common.typehandler
```

## (5) eureka

```yaml
eureka:
  client:
    register-with-eureka: false # 是否注册自己的信息到EurekaServer，默认是true
    fetch-registry: false # 是否拉取其它服务的信息，默认是true
    service-url: # EurekaServer的地址，现在是自己的地址，如果是集群，需要加上其它Server的地址。
      defaultZone: http://127.0.0.1:${server.port}/eureka
  instance:
    prefer-ip-address: true # 当调用getHostname获取实例的hostname时，返回ip而不是host名称
    ip-address: 127.0.0.1 # 指定自己的ip信息，不指定的话会自己寻找    
```

## (6) pageHelper

选项讲解：

```xml
<!-- 该参数默认为false -->
        <!-- 设置为true时，会将RowBounds第一个参数offset当成pageNum页码使用 -->
        <!-- 和startPage中的pageNum效果一样-->
        <property name="offsetAsPageNum" value="true"/>
        <!-- 该参数默认为false -->
        <!-- 设置为true时，使用RowBounds分页会进行count查询 -->
        <property name="rowBoundsWithCount" value="true"/>
        <!-- 设置为true时，如果pageSize=0或者RowBounds.limit = 0就会查询出全部的结果 -->
        <!-- （相当于没有执行分页查询，但是返回结果仍然是Page类型）-->
        <property name="pageSizeZero" value="true"/>
        <!-- 3.3.0版本可用 - 分页参数合理化，默认false禁用 -->
        <!-- 启用合理化时，如果pageNum<1会查询第一页，如果pageNum>pages会查询最后一页 -->
        <!-- 禁用合理化时，如果pageNum<1或pageNum>pages会返回空数据 -->
        <property name="reasonable" value="true"/>
        <!-- 3.5.0版本可用 - 为了支持startPage(Object params)方法 -->
        <!-- 增加了一个`params`参数来配置参数映射，用于从Map或ServletRequest中取值 -->
        <!-- 可以配置pageNum,pageSize,count,pageSizeZero,reasonable,不配置映射的用默认值 -->
        <property name="params" value="pageNum=start;pageSize=limit;pageSizeZero=zero;reasonable=heli;count=contsql"/>
```

其他五个参数说明：

增加dialect属性，使用时必须指定该属性，可选值为oracle,mysql,mariadb,sqlite,hsqldb,postgresql,sqlserver,没有默认值，必须指定该属性。

增加offsetAsPageNum属性，默认值为false，使用默认值时不需要增加该配置，需要设为true时，需要配置该参数。当该参数设置为true时，使用RowBounds分页时，会将offset参数当成pageNum使用，可以用页码和页面大小两个参数进行分页。

增加rowBoundsWithCount属性，默认值为false，使用默认值时不需要增加该配置，需要设为true时，需要配置该参数。当该参数设置为true时，使用RowBounds分页会进行count查询。

增加pageSizeZero属性，默认值为false，使用默认值时不需要增加该配置，需要设为true时，需要配置该参数。当该参数设置为true时，如果pageSize=0或者RowBounds.limit = 0就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是Page类型）。

增加reasonable属性，默认值为false，使用默认值时不需要增加该配置，需要设为true时，需要配置该参数。具体作用请看上面配置文件中的注释内容。

为了支持startPage(Object params)方法，增加了一个params参数来配置参数映射，用于从Map或ServletRequest中取值，可以配置pageNum,pageSize,count,pageSizeZero,reasonable,不配置映射的用默认值。



# bootstrap和application.yml

SpringBoot默认支持properties和YAML两种格式的配置文件。前者格式简单，但是只支持键值对。如果需要表达列表，最好使用YAML格式。SpringBoot支持自动加载约定名称的配置文件，例如application.yml。如果是自定义名称的配置文件，就要另找方法了。可惜的是，不像前者有@PropertySource这样方便的加载方式，后者的加载必须借助编码逻辑来实现。

## 一、执行顺序

**bootstrap.yml（bootstrap.properties）与application.yml（application.properties）执行顺序**

bootstrap.yml（bootstrap.properties）用来程序引导时执行，应用于更加早期配置信息读取，如可以使用来配置application.yml中使用到参数等

application.yml（application.properties) 应用程序特有配置信息，可以用来配置后续各个模块中需使用的公共参数等。

bootstrap.yml 先于 application.yml 加载

## 二、典型的应用场景如下：

- 当使用 Spring Cloud Config Server 的时候，你应该在 bootstrap.yml 里面指定 spring.application.name 和 spring.cloud.config.server.git.uri
- 和一些加密/解密的信息

技术上，bootstrap.yml 是被一个父级的 Spring ApplicationContext 加载的。这个父级的 Spring ApplicationContext是先加载的，在加载application.yml 的 ApplicationContext之前。

为何需要把 config server 的信息放在 bootstrap.yml 里？

当使用 Spring Cloud 的时候，配置信息一般是从 config server 加载的，为了取得配置信息（比如密码等），你需要一些提早的或引导配置。因此，把 config server 信息放在 bootstrap.yml，用来加载真正需要的配置信息

# 读取自定义的yml文件

## 一、自定义yml

```yaml
client:
 id: 95279527
 tel: 18290569081
 password: 123456
 photo: http://www.baidu.com
 sex: 1
 status: 0
 delFlag: 0
 name: uniqueUser
 post: 派单员
 onlineStatus: 0
 userNo: F9527
organization:
  id: 9527
  name: 9527组织
  vest:
  outerId:
  cityId: 510100
  level: A
  shopType: 1
  cityName: 9527派单员城市
```

## 二、实体配置类

```java
package com.dftcmedia.tckk.microservice.ots.orderdispatch.entity.constants;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

/**
 * <p>DESC: 替补派单业务员</p>
 * <p>DATE: 2019/6/5</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
@Data
@Component
@ConfigurationProperties(prefix="client")
@PropertySource(value = "classpath:uniuser.yml",
                ignoreResourceNotFound = true,
                encoding = "utf-8")
public class ClientUserConstants {

    private Long id;

    private String tel;

    private String password;

    private String photo;

    private Integer sex;

    private Integer status;

    private Integer delFlag;

    private String name;
  
    private String post;

    private Integer onlineStatus;

    private String userNo;
}

```

```java
package com.dftcmedia.tckk.microservice.ots.orderdispatch.entity.constants;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

/**
 * <p>DESC: 组织配置文件读取</p>
 * <p>DATE: 2019/6/5</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
@Data
@Component
@ConfigurationProperties(prefix = "organization")
@PropertySource(value = "classpath:uniuser.yml",
                ignoreResourceNotFound = true,
                encoding = "utf-8")
public class OrganizationConstants {

    private Long id;

    private String name;

    private Long vest;

    private Long outerId;

    /**
     * 办事处城市id
     */
    private Integer cityId;

    private String level;

    private Integer shopType;

    private String cityName;

}
```

## 三、解读

（1）@ConfigurationProperties(prefix = "organization")

读取前缀为organization的yml值

（2）@PropertySource

```java
 @PropertySource(value = "classpath:uniuser.yml",
 			   ignoreResourceNotFound = true,
            	encoding = "utf-8")
```
读取自定义配置文件

## 四：读取问题

> 将yml中的内容放入，application.yml文件中正常，自定义novellist.yml文件中无法找到。使用@ConfigurationProperties注解，只能用于properties文件。
>
> 解决方式：可以通过PropertySourcePlaceholderConfigurer来加载yml文件，暴露yml文件到spring environment，如下：

```java
package com.dftcmedia.tckk.microservice.ots.common.config;

import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.io.ClassPathResource;

/**
 * <p>DESC: </p>
 * <p>DATE: 2019/6/5</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
@Configuration
public class LoadFileConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer properties() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
        yaml.setResources(new ClassPathResource("uniuser.yml"));
        configurer.setProperties(yaml.getObject());
        configurer.setFileEncoding("utf-8");
        return configurer;
    }
}

```

## 五：中文乱码问题

解决：

（1）

@PropertySource(value = {"路径"},ignoreResourceNotFound = true,encoding = "utf-8")

 （2）

文件编码格式设置为（file encodeing = utf-8）

   (3)

configurer.setFileEncoding("utf-8");