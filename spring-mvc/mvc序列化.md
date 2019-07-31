# springmvc序列化

> 使用SpringBoot为例，SpringBoot默认使用Jackson作为序列化工具。通过修改Jackson配置即可自定义序列化规则。

## @Valid

> Spring MVC采用的校验是hibernate-validate

```properties
@NotNull  值不能为空
@Null     值必须为空
@Pattern(regex=) 字符串必须匹配正则表达式
@Size(min=,max=)  集合的元素数量必须在min和max之间
@CreditNumber(ignoreNonDigitCharacters=) 字符串必须是信用卡号(按美国的标准验证)
@Email    字符串必须是Email地址
@Length(min=，max=) 检查字符串的长度
@NotBlank   字符串必须有字符
@Range(min=,max=) 数字必须大于min，小于等于max
@SafeHtml   字符串是安全的html
@URL   字符串是合法的URL
@AssertFalse   值必须是false
@AssertTrue    值必须是true
@DecimalMax(value,inclusive=) 值必须小于等于(inclusive=true)/小于(inclusive=false) value属性指定的值，可以注解在字符串类型的属性上
@DecimalMin(value,inclusive=) 值必须大于等于(inclusive=true)/大于(inclusive=false) value属性指定的值，可以注解在字符串类型的
属性上
@Digits(integer=,fraction=)  数字格式检查，integer指定整数部分的最大长度，fraction指定小数部分的最大长度
@Future  值必须是未来的日志
@Past  值必须是过去的日期
@Max(value=) 值必须小于等于value指定的值，不能注视在字符串类型的属性上
@Min(value=) 值必须大于等于value指定的值，不能注视在字符串类型的属性上
```

## Jackson 常用注解

* @JsonNaming(SnakeCaseStrategy.class)

  指定Json字段名映射策略为蛇形大小写策略。缺省则直接使用Bean属性名

 可用的命名映射策略还有：
KebabCaseStrategy: 肉串策略 - 单词小写，使用连字符'-'连接
SnakeCaseStrategy: 蛇形策略 - 单词小写，使用下划线'_'连接；即老版本中的LowerCaseWithUnderscoresStrategy
LowerCaseStrategy: 小写策略 - 简单的把所有字母全部转为小写，不添加连接符
UpperCamelCaseStrategy: 驼峰策略 - 单词首字母大写其它小写，不添加连接符；即老版本中的PascalCaseStrategy

* @JsonIgnoreProperties({"id", "created", "steps", "copy", "stepList"})

​         类注解，指定序列化时忽略这些属性，可以用于覆盖超类中默认输出的属性

* @JsonInclude(Include.NON_EMPTY)

  仅在属性不为空时序列化此字段，对于字符串，即null或空字符串

* @JsonIgnore

  序列化时忽略此字段

* @JsonProperty(value = "user_name")

  指定序列化时的字段名，默认使用属性名

* @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")

  指定Date类字段序列化时的格式

* @JsonUnwrapped(prefix = "user_")

  private User user;
  把成员对象中的属性提升到其容器类，并添加给定的前缀，比如上例中: User类中有name和age两个属性，不使用此注解则序列化为：
  ... "user": { "name": "xxx", "age": 22 } ...
  使用此注解则序列化为：
  ... "user_name": "xxx", "user_age": 22, ...

* @JsonIgnoreType

  类注解，序列化时忽略此类

* @JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class,property = "id") 

  作用于类或属性上，被用来在序列化/反序列化时为该对象或字段添加一个对象识别码，通常是用来解决循环嵌套的问题



序列化器：

* 实现JsonSerializer<String>
* 对应注解上加注解
* 需求是：json 字符串不想直接转给前段，而是json对象返回。

```java
@Data
public class Test01 {
    @JsonSerialize(using = JsonStringSerilizer.class)
    private String jsond = "[{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/eQEj1SYtXmOdTqNWZlETR4CpVJieMJQI.jpg\"},{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/d5ClnyPCHTuWhBqglVDtIllF2wViDteC.jpg\"},{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/ZGSQX6iAxZ0tZhenH0IIvDlagYloSZLn.jpg\"},{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/KFOqZenayBSNHykp546iEa5Fus6PmxMs.jpg\"}]";

    private Integer age;
    private Date time;
}
```

```java
/**
 * <p>DESC: </p>
 * <p>DATE: 2019/6/27</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
public class JsonStringSerilizer extends JsonSerializer<String> {

    @Override
    public void serialize(String s, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        Object o = JSONUtil.toList(new JSONArray(s),Object.class );
        jsonGenerator.writeObject(o);
    }
}
```







## 前段数据提交：

* from-data表单提交
* application/json 提交



### 1、 from-data

springmvc默认转换类：Convert

可以自定义实现Convert<in,out> 接口，并且注册到spring中，即可生效

```java
package com.leyou.auth.service.config;

import org.apache.commons.lang3.StringUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * <p>DESC: </p>
 * <p>DATE: 2019/6/27</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Bean
    public Converter<String, Date> stringToDate(){
        return new Converter<String, Date>() {
            @Override
            public Date convert(String s) {
                if(StringUtils.isNotBlank(s)){
                    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    try {
                        System.out.println("进入convert");
                        Date parse = simpleDateFormat.parse(s);
                        return parse;
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                }
                return null;
            }
        };
    }
}

```

### 2、application/json 

因为传入的时json,springmvc默认参数转换使用的是：facson反序列化器 JsonDeserializer<T>

注意controller中参数 注解：==@requestBody==

```java
@RequestMapping("hello")
@RestController
public class MvcController {

    @PostMapping("/hello")
    public ResponseEntity hello(@RequestBody Test01 test){
        Test01 test01 = new Test01();
        test01.setAge(test.getAge());
        test01.setTime(test.getTime());
        test01.setJsond("[{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/eQEj1SYtXmOdTqNWZlETR4CpVJieMJQI.jpg\"},{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/d5ClnyPCHTuWhBqglVDtIllF2wViDteC.jpg\"},{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/ZGSQX6iAxZ0tZhenH0IIvDlagYloSZLn.jpg\"},{\"url\":\"https://dftc-temps.oss-cn-shenzhen.aliyuncs.com/KFOqZenayBSNHykp546iEa5Fus6PmxMs.jpg\"}]");
        return ResponseEntity.ok(test01);
    }
}
```



第一步，实现并且，重写反序列化方法。

```java
package com.leyou.auth.service.serializer;

import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;

import java.io.IOException;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * <p>DESC: </p>
 * <p>DATE: 2019/6/27</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: ZhengYong</p>
 */
public class JsonStringDesSerilizer extends JsonDeserializer<Date> {

    @Override
    public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
        String value = jsonParser.getValueAsString();
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            return dateFormat.parse(value);
        } catch (ParseException e) {
            System.out.println("反序列化");
            e.printStackTrace();
        }
        return null;
    }
}
```



第二步，对应属性添加注解：@

```java
@Data
public class Test01 {

    private String jsond = "";
   
    private Integer age;

    @JsonDeserialize(using = JsonStringDesSerilizer.class)
    private Date time;
}
```

