# @Valid与@Validated注解

> @Valid：常见用在方法，类中字段上进行校验

>  @Validated：是spring提供的对@Valid的封装，常见用在方法上进行校验



定义的校验类型

@Null       验证对象是否为null
@NotNull    验证对象是否不为null, 无法查检长度为0的字符串
@NotBlank 检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.
@NotEmpty 检查约束元素是否为NULL或者是EMPTY.
@CreditCardNumber信用卡验证
@Email  验证是否是邮件地址，如果为null,不进行验证，算通过验证。
@URL(protocol=,host=, port=,regexp=, flags=) ip地址校验

Booelan检查
@AssertTrue     验证 Boolean 对象是否为 true  
@AssertFalse    验证 Boolean 对象是否为 false  

长度检查
@Size(min=, max=)   验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  
@Length(min=, max=) Validates that the annotated string is between min and max included.

日期检查
@Past       验证 Date 和 Calendar 对象是否在当前时间之前  
@Future     验证 Date 和 Calendar 对象是否在当前时间之后  
@Pattern    验证 String 对象是否符合正则表达式的规则

数值检查，建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为""时无法转换为int，但可以转换为Stirng为"",Integer为null
@Min            验证 Number 和 String 对象是否大等于指定的值  
@Max            验证 Number 和 String 对象是否小等于指定的值  
@DecimalMax 被标注的值必须不大于约束中指定的最大值. 这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度
@DecimalMin 被标注的值必须不小于约束中指定的最小值. 这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度
@Digits     验证 Number 和 String 的构成是否合法  
@Digits(integer=,fraction=) 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度。





注意的几点：

（1）如果一个bean中包含第二个bean，这时要检验第二个bean中某个字段，即嵌套校验，必须要在第一个bean对象中使用@Valid标注到表示第二个bean对象的字段上，然后再第二个bean对象里面的字段上加上校验类型

（2）@Validated支持分组注解

1、先定义一个空接口：GroupA

2、对bean对象中校验类型，添加分组信息，这里我对version字段进行了分组校验信息添加

3、方法入参中进行分组信息添加



如上图所示，则shelveProject方法由于添加了分组信息会校验DeleteProjectRequest对象中的version字段是否为空，而offShelveProject方法没有添加分组信息，不会校验version是否为空。



博客原文：https://blog.csdn.net/zhenwei1994/article/details/81460419