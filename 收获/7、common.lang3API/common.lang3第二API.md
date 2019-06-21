# Java第二API之

# apache的commons-lang3

简介：apache提供的众多commons工具包，号称Java第二API，而common里面lang3包更是被我们使用得最多的。因此本文主要详细讲解lang3包里面几乎每个类的使用，希望以后大家使用此工具包，写出优雅的代码



```xml
 <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.8</version>
</dependency>
```

包结构展示：

```properties
org.apache.commons.lang3
org.apache.commons.lang3.builder
org.apache.commons.lang3.concurrent
org.apache.commons.lang3.event
org.apache.commons.lang3.exception
org.apache.commons.lang3.math
org.apache.commons.lang3.mutable
org.apache.commons.lang3.reflect
org.apache.commons.lang3.text
org.apache.commons.lang3.text.translate
org.apache.commons.lang3.time
org.apache.commons.lang3.tuple
```



## ArrayUtils：

`用于对数组的操作，如添加、查找、删除、子数组、倒序、元素类型转换等`

- toString：功能基本同java自己的Arrays.toString方法
- hashCode：相同个数、相同顺序的数组hashCode会是一样的

```java
  public static void main(String[] args) {
        Integer[] inArr = new Integer[]{1, 2, 3};
        Integer[] inArr2 = new Integer[]{1, 2, 3};
        System.out.println(ArrayUtils.hashCode(inArr)); //862547
        System.out.println(ArrayUtils.hashCode(inArr2)); //862547

        inArr = new Integer[]{1, 2, 3};
        inArr2 = new Integer[]{1, 3, 3};
        System.out.println(ArrayUtils.hashCode(inArr)); //862547
        System.out.println(ArrayUtils.hashCode(inArr2)); //862584
    }
```

- isEquals：该方法已经被废弃。取代的为java自己的java.util.Objects.deepEquals(Object, Object)

```java
public static void main(String[] args) {
        Integer[] inArr = new Integer[]{1, 2, 3};
        Integer[] inArr2 = new Integer[]{1, 2, 3};
        System.out.println(Objects.deepEquals(inArr, inArr2)); //true

        inArr = new Integer[]{1, 2, 3};
        inArr2 = new Integer[]{1, 3, 3};
        System.out.println(Objects.deepEquals(inArr, inArr2)); //false
    }
```

- toArray：可以简便的构建一个数组。但是注意下面的区别：

```java
Integer[] integers = ArrayUtils.toArray(1, 2, 3);
        Serializable[] serializables = ArrayUtils.toArray(1, 2, "3");

```

- toObject/toPrimitive：这两个方法很有用 可以实现比如int[]和Integer[]数组之间的互转

```java
Integer[] inArr = new Integer[]{1, 2, 3};
        int[] ints = ArrayUtils.toPrimitive(inArr);
        Integer[] integers = ArrayUtils.toObject(ints);

```

toStringArray：同上。这个方法是将Object数组转换成String数组。

```java
public static void main(String[] args) {
        Integer[] inArr = new Integer[]{1, 2, 3};
        int[] ints = new int[]{1,2,3};
        String[] strings = ArrayUtils.toStringArray(inArr);
        //ArrayUtils.toStringArray(ints); //编译报错哟
    }

```

注意：

```java
 public static void main(String[] args) {
        Integer[] inArr = new Integer[]{1, 2, null};
        //String[] strings = ArrayUtils.toStringArray(inArr);
        
        //如果里面有null元素，会报错的，所以我们可以用下面这个方法 把null转成指定的值即可
        String[] strings = ArrayUtils.toStringArray(inArr,"");
        
    }

```

- getLength、isSameLength：有时候建议使用。因为它是对null安全的。null的length为0

## CharEncoding：

过时。被Java自己的java.nio.charset.StandardCharsets取代

## CharUtils – 用于操作char值和Character对象

toCharacterObjec/toChart：把char或者String转为一个Character对象。互转。Character,valueOf()很多时候也能达到这个效果
toIntValue：把char和Character转为对应的int值

isAscii系列：判断该字符是否是Ascii码！

## ClassPathUtils：处理类路径的一些工具类

- toFullyQualifiedName(Class<?> context, String resourceName) 返回一个由class包名+resourceName拼接的字符串 	

```java
public static void main(String[] args) {

        String fullPath = ClassPathUtils.toFullyQualifiedName(Integer.class, "");
        System.out.println(fullPath); //java.lang.
        //fullPath = ClassPathUtils.toFullyQualifiedName(Integer.class.getPackage(), "Integer.value");
        fullPath = ClassPathUtils.toFullyQualifiedName(Integer.class, "Integer.value");
        System.out.println(fullPath); //java.lang.Integer.value
    }

```

toFullyQualifiedName(Package context, String resourceName) 返回一个由class包名+resourceName拼接的字符串
toFullyQualifiedPath(Class<?> context, String resourceName) 返回一个由class包名+resourceName拼接的字符串

toFullyQualifiedPath(Package context, String resourceName) 返回一个由class包名+resourceName拼接的字符串



## ClassUtils 

`用于对Java类的操作（有很多方法还是挺有用的）` 

- getShortClassName：

```java
public static void main(String[] args) {
        System.out.println(int[].class.getSimpleName()); //int[]
        System.out.println(ClassUtils.getShortClassName(int[].class)); //int[]
        System.out.println(ClassUtils.getShortClassName(String.class)); //String
        System.out.println(ClassUtils.getShortClassName(ArrayList.class)); //ArrayList
        System.out.println(ClassUtils.getShortClassName("List")); //List
    }
```

- getPackageName：获取包名

```java
 public static void main(String[] args) {
        System.out.println(ClassUtils.getPackageName(int[].class)); //""
        System.out.println(ClassUtils.getPackageName(String.class)); //java.lang
    }

```

- getAllSuperclasses：获取到该类的所有父类 注意：只是父类 不包含接口

```java
 public static void main(String[] args) {
        List<Class<?>> allSuperclasses = ClassUtils.getAllSuperclasses(ArrayList.class);
        System.out.println(ArrayUtils.toString(allSuperclasses)); //[class java.util.AbstractList, class java.util.AbstractCollection, class java.lang.Object]
    }

public static void main(String[] args) {
        List<Class<?>> allSuperclasses = ClassUtils.getAllSuperclasses(ArrayList.class);
        System.out.println(ArrayUtils.toString(allSuperclasses)); //[class java.util.AbstractList, class java.util.AbstractCollection, class java.lang.Object]
        allSuperclasses = ClassUtils.getAllSuperclasses(Object.class);
        System.out.println(ArrayUtils.toString(allSuperclasses)); //[]
    }

```

- getAllInterfaces：同上。但此方法指的是接口
- convertClassNamesToClasses/convertClassesToClassNames 见名知意

```java
public static void main(String[] args) {
        List<Class<?>> classes = ClassUtils.convertClassNamesToClasses(Arrays.asList("java.lang.Integer","java.lang.int"));
        System.out.println(classes); //[class java.lang.Integer, null]
    }

```

- isPrimitiveOrWrapper、isPrimitiveWrapper 、primitiveToWrapper、primitivesToWrappers、wrapperToPrimitive判断是基本类型还是包装类型

```java
public static void main(String[] args) {
        System.out.println(ClassUtils.isPrimitiveOrWrapper(Integer.class)); //true
        System.out.println(ClassUtils.isPrimitiveOrWrapper(int.class)); //true

        //检测是否是包装类型
        System.out.println(ClassUtils.isPrimitiveWrapper(Object.class)); //false 注意 此处是false
        System.out.println(ClassUtils.isPrimitiveWrapper(Integer.class)); //true
        System.out.println(ClassUtils.isPrimitiveWrapper(int.class)); //false

        //检测是否是基本类型
        System.out.println(Object.class.isPrimitive()); //false 注意 此处也是false
        System.out.println(Integer.class.isPrimitive()); //false
        System.out.println(int.class.isPrimitive()); //true
    }

```

isAssignable：是否是相同的class类型 支持class、数组等等 挺实用的

isInnerClass：检查一个类是否是内部类或者静态内部类等

getClass：加强版的Class.forName() 可以指定值是否要马上初始化该类

hierarchy：获取到该类的继承结构

```java
public static void main(String[] args) {
        Iterable<Class<?>> hierarchy = ClassUtils.hierarchy(ArrayList.class);
        hierarchy.forEach(System.out::println);
        //输出了类的层级结构（默认是不包含接口的）
        //class java.util.ArrayList
        //class java.util.AbstractList
        //class java.util.AbstractCollection
        //class java.lang.Object
        hierarchy = ClassUtils.hierarchy(ArrayList.class,ClassUtils.Interfaces.INCLUDE);
        hierarchy.forEach(System.out::println);
        //class java.util.ArrayList
        //interface java.util.List
        //interface java.util.Collection
        //interface java.lang.Iterable
        //interface java.util.RandomAccess
        //interface java.lang.Cloneable
        //interface java.io.Serializable
        //class java.util.AbstractList
        //class java.util.AbstractCollection
        //class java.lang.Object
    }

```

## EnumUtils：

辅助操作枚举的一些工具

getEnum(Class enumClass, String enumName) 通过类返回一个枚举，可能返回空
getEnumList(Class enumClass) 通过类返回一个枚举集合
getEnumMap(Class enumClass) 通过类返回一个枚举map

isValidEnum(Class enumClass, String enumName) 验证enumName是否在枚举中，返回true false

```java
//枚举类
public enum ImagesTypeEnum {
    JPG,JPEG,PNG,GIF;
}

//测试
        ImagesTypeEnum imagesTypeEnum = EnumUtils.getEnum(ImagesTypeEnum.class, "JPG");
        System.out.println("imagesTypeEnum = " + imagesTypeEnum);
        System.out.println("--------------");
        List<ImagesTypeEnum> imagesTypeEnumList = EnumUtils.getEnumList(ImagesTypeEnum.class);
        imagesTypeEnumList.stream().forEach(
                imagesTypeEnum1 -> System.out.println("imagesTypeEnum1 = " + imagesTypeEnum1)
        );
        System.out.println("--------------");
        Map<String, ImagesTypeEnum> imagesTypeEnumMap = EnumUtils.getEnumMap(ImagesTypeEnum.class);
        imagesTypeEnumMap.forEach((k, v) -> System.out.println("key：" + k + ",value：" + v));
        System.out.println("-------------");
        boolean result = EnumUtils.isValidEnum(ImagesTypeEnum.class, "JPG");
        System.out.println("result = " + result);
        boolean result1 = EnumUtils.isValidEnum(ImagesTypeEnum.class, null);
        System.out.println("result1 = " + result1);

输出：
imagesTypeEnum = JPG
--------------
imagesTypeEnum1 = JPG
imagesTypeEnum1 = JPEG
imagesTypeEnum1 = PNG
imagesTypeEnum1 = GIF
--------------
key：JPG,value：JPG
key：JPEG,value：JPEG
key：PNG,value：PNG
key：GIF,value：GIF
-------------
result = true
result1 = false

```

## RandomStringUtils :

` 需要随机字符串的时候，它或许能帮上忙`

```java
 public static void main(String[] args) {
        //随便随机一个字  所以有可能是乱码
        String random = RandomStringUtils.random(10);
        //在指定范围内随机
        String randomChars = RandomStringUtils.random(3,'a','b','c','d','e');
        //随便随机10个Ascii
        String randomAscii = RandomStringUtils.randomAscii(10);
        //注意这里不是5到10内随机,而是随机一个长度的数字
        String randomNumeric = RandomStringUtils.randomNumeric(5,10);
        System.out.println(random); //?ᣒ?⍝?䆃ぬ
        System.out.println(randomChars); //dac
        System.out.println(randomAscii); //hpCQrtmUvi
        System.out.println(randomNumeric); //2580338
    }

```

## RandomUtils：

`这个不解释，如果你需要随机数字，用它吧。int、long、flort都是ok的`

## RegExUtils：处理字符串用正则替换等

- removeAll
- removeFirst
- removePattern
- replaceAll
- replaceFirst

## SerializationUtils：

`对象的序列化工具。` 

在Json流行的时代，这个工具使用的几率就较小了。

clone：采用字节数组ByteArrayInputStream来拷贝一个一模一样的对象
serialize(final Serializable obj, final OutputStream outputStream) ：可以把对象序列化到输出流里
byte[] serialize(final Serializable obj)：直接序列化成字节数组

deserialize(final InputStream inputStream)、deserialize(final byte[] objectData)

# StringUtils的使用详解

## isEmpty

public static boolean isEmpty(CharSequence cs)

这个可能用得是非常多的，null和空串都被定义为empty了哟

```java
StringUtils.isEmpty(null)      = true
 StringUtils.isEmpty("")        = true
 StringUtils.isEmpty(" ")       = false  //注意这里是false
 StringUtils.isEmpty("bob")     = false
 StringUtils.isEmpty("  bob  ") = false

```

## isAnyEmpty

public static boolean isAnyEmpty(CharSequence… css)

任意一个参数为空的话，返回true。如果这些参数都不为空的话返回false。**在写一些判断条件的时候，这个方法还是很实用的。** 

```java
StringUtils.isAnyEmpty(null)             = true
 StringUtils.isAnyEmpty(null, "foo")      = true
 StringUtils.isAnyEmpty("", "bar")        = true
 StringUtils.isAnyEmpty("bob", "")        = true
 StringUtils.isAnyEmpty("  bob  ", null)  = true
 StringUtils.isAnyEmpty(" ", "bar")       = false //注意这个是false哦
 StringUtils.isAnyEmpty("foo", "bar")     = false 

```

## isNoneEmpty

public static boolean isNoneEmpty(CharSequence… css) 和isAnyEmpty取返

## isBlank

public static boolean isBlank(CharSequence cs)

判断字符对象是不是空字符串，注意与isEmpty的区别。相当于深度的isEmpty

```java
StringUtils.isBlank(null)      = true
 StringUtils.isBlank("")        = true
 StringUtils.isBlank(" ")       = true //注意此处是null哦  这和isEmpty不一样的
 StringUtils.isBlank("bob")     = false
 StringUtils.isBlank("  bob  ") = false

```

## isAnyBlank、isNoneBlank

这里就不再解释了

public static String trim(String str)

移除字符串两端的空字符串，制表符char <= 32如：\n \t 如果为null返回null

```java
StringUtils.trim(null)          = null
 StringUtils.trim("")            = ""
 StringUtils.trim("     ")       = ""
 StringUtils.trim("abc")         = "abc"
 StringUtils.trim("    abc    ") = "abc"

```

变体有如下：
public static String trimToNull(String str) //如果是null就返回null，否则trim之后返回
public static String trimToEmpty(String str)

## containsAny

public static boolean containsAny(CharSequence cs,char… searchChars)

是否包含后面数组中的任意对象，返回true（和List里的containsAll有点像）

```java
StringUtils.containsAny(null, *)                = false
 StringUtils.containsAny("", *)                  = false
 StringUtils.containsAny(*, null)                = false
 StringUtils.containsAny(*, [])                  = false
 StringUtils.containsAny("zzabyycdxx",['z','a']) = true
 StringUtils.containsAny("zzabyycdxx",['b','y']) = true
 StringUtils.containsAny("aba", ['z'])           = false

```

## substring

public static String substring(String str,int start)

这个系列有的时候很有用，特别是下面的衍生方法

```java
//从左边开始截取指定个数
public static String left(String str,int len)
//从右边开始截取指定个数
public static String right(String str,int len)
//从中间的指定位置开始截取  指定个数
public static String mid(String str,int pos,int len)

```

## join

public static String join(T… elements)、

public static String join(Object[] array,char separator)

默认使用空串Join

```java
StringUtils.join(null)            = null
 StringUtils.join([])              = ""
 StringUtils.join([null])          = ""
 StringUtils.join(["a", "b", "c"]) = "abc"
 StringUtils.join([null, "", "a"]) = "a"

```

自定义符号：特定字符串连接数组，很多情况下还是蛮实用，不用自己取拼字符串

```java
StringUtils.join(null, *)               = null
 StringUtils.join([], *)                 = ""
 StringUtils.join([null], *)             = ""
 StringUtils.join(["a", "b", "c"], ';')  = "a;b;c"
 StringUtils.join(["a", "b", "c"], null) = "abc"
 StringUtils.join([null, "", "a"], ';')  = ";;a" //注意这里和上面的区别

```

## deleteWhitespace

public static String deleteWhitespace(String str)

删除空格 这个方法还挺管用的。比trim给力

```java
StringUtils.deleteWhitespace(null)         = null
 StringUtils.deleteWhitespace("")           = ""
 StringUtils.deleteWhitespace("abc")        = "abc"
 StringUtils.deleteWhitespace("   ab  c  ") = "abc"

```

## removeStart

public static String removeStart(String str,String remove)

删除以特定字符串开头的字符串，如果没有的话，就不删除。
这个方法有时候很管用啊

```java
StringUtils.removeStart(null, *)      = null
 StringUtils.removeStart("", *)        = ""
 StringUtils.removeStart(*, null)      = *
 StringUtils.removeStart("www.domain.com", "www.")   = "domain.com"
 StringUtils.removeStart("domain.com", "www.")       = "domain.com"
 StringUtils.removeStart("www.domain.com", "domain") = "www.domain.com" //注意这个结果哟  并没有删除任何东西
 StringUtils.removeStart("abc", "")    = "abc"

```

## rightPad

public static String rightPad(String str,int size,char padChar)

这个方法还是蛮管用的。对于生成统一长度的字符串的时候。
比如生成订单号，为了保证长度统一，可以右边自动用指定的字符补全至指定长度

```java
StringUtils.rightPad(null, *, *)     = null
 StringUtils.rightPad("", 3, 'z')     = "zzz"
 StringUtils.rightPad("bat", 3, 'z')  = "bat"
 StringUtils.rightPad("bat", 5, 'z')  = "batzz"
 StringUtils.rightPad("bat", 1, 'z')  = "bat"
 StringUtils.rightPad("bat", -1, 'z') = "bat"

```

对应的：leftPad 左边自动补全

## capitalize

public static String capitalize(String str)、uncapitalize

首字母大、小写

## swapCase

public static String swapCase(String str)

去返大小写 大变小 小变大

## wrap

高级用法：可以自定义缩略的部分内容角标

public static String wrap(String str,char wrapWith)

包装，用后面的字符串对前面的字符串进行包装
其实相当于前后拼了相同的串

```java
StringUtils.wrap(null, *)        = null
 StringUtils.wrap("", *)          = ""
 StringUtils.wrap("ab", '\0')     = "ab"
 StringUtils.wrap("ab", 'x')      = "xabx"
 StringUtils.wrap("ab", '\'')     = "'ab'"
 StringUtils.wrap("\"ab\"", '\"') = "\"\"ab\"\""

```

## isAllBlank、isAllEmpty

这些都不解释了。处理数组可变参数而已

## isAllLowerCase、isAllUpperCase

判断字符串所有字符是否都是大、小写