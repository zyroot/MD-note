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