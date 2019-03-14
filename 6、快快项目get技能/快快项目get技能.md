# 实际项目开发：

# 一、全局异常处理

## 1、异常枚举对象创建

```java
package com.dftcmedia.tckk.microservice.common.enums;

import com.alibaba.druid.wall.Violation;
import lombok.Getter;

/**
 * <p>DESC: </p>
 * <p>DATE: 2019/2/13</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: DengC</p>
 */
@Getter
public enum ServiceCodeEnum {
    /**
     *
     */
    SUCCESS(0, "成功"),

    /**
     * 失败
     */
    FAIL(1, "失败"),

    /**
     * 无效请求
     */
    BAD_REQUEST(400, "无效请求"),

    /**
     * 无效鉴权信息
     */
    UNAUTHORIZED(401, "无效授权信息"),

    /**
     * 无权访问此资源
     */
    FORBIDDEN(403, "无权访问此资源"),

    /**
     * 无效的访问地址
     */
    INVALID_URL(404, "无效的访问地址"),

    /**
     * 访问控制
     */
    LIMIT(406, "访问过于频繁，请稍后再试"),

    /**
     * 服务器内部错误
     */
    SERVER_INTERNAL_ERROR(500, "服务器内部错误"),

    /**
     * 暂不可服务
     */
    SERVICE_UNAVAILABLE(503, "暂不可服务"),

    /**
     * 数据完整性异常 数据过长  过短
     */
    LENGTH_REQUIRED(411, "数据完整性异常"),

    /**
     * 数据格式错误
     */
    DATA_FORMAT_ERROR(416, "数据格式错误"),

    /**
     * 不支持的请求方式
     */
    NOT_SUPPORTED(415, "不支持的请求方式"),

    /**
     * 账户已锁定
     */
    LOCKED(423, "账户已锁定，请联系客服"),

    /**
     * 需要认证
     */
    AUTHENTICATION_REQUIRED(511, "需要认证"),

    /**
     * 不支持的文件类型
     */
    UNSUPPORTED_FILE_TYPE(601, "不支持的文件类型"),

    /**
     * 手机号已绑定
     */
    TEL_IS_NOT_BIND(603, "未绑定手机号"),

    /**
     * 手机号已绑定
     */
    TEL_IS_BIND(602, "手机号已绑定"),

    /**
     * 账号已存在
     */
    ACCOUNT_REGISTERED(604, "账号已存在"),

    /**
     * 无效验证码
     */
    INVALID_VERIFY_CODE(605, "无效验证码"),

    /**
     * 账户密码错误
     */
    ACCOUNT_PASSWORD_ERROR(606, "账户密码错误"),

    /**
     * 不支持的登录方式
     */
    NOT_SUPPORT_LOGIN_CHANNEL(607, "不支持的登录方式"),

    /**
     * token已过期
     */
    TOKEN_EXPIRED(608, "token已过期"),

    /**
     * 需要登录后操作
     */
    NEED_LOGIN(609, "需要登录后操作"),

    /**
     * 第三方登录失败
     */
    THIRD_LOGIN_ERROR(610, "第三方登录失败"),

    /**
     * 验证码获取失败
     */
    VERIFY_CODE_FAIL(611, "验证码获取失败"),

    /**
     * 注册失败
     */
    REGISTER_FAIL(612, "注册失败"),

    /**
     * 角色已存在
     */
    ROLE_EXIST(613, "角色已存在"),

    /**
     * 二维码获取失败
     */
    QR_CODE_GET_FAIL(614, "二维码获取失败"),
    /**
     * 该商铺已经认证
     */
    SHOP_IS_CERTIFICATION(615,"该商铺已经认证过该类型产品"),

    /**
     * 开启重复的代办流程
     */
    TASK_IS_OPEN(615,"开启重复的代办流程"),

    /**
     * 有尚未完成的任务
     */
    TASK_IS_NOT_OVER(616,"有尚未完成的任务"),

    /**
     * 参数错误
     */
    PARAM_ERROR(701, "参数错误，缺少必要参数"),

    /**
     * 无数据
     */
    NO_DATA(702, "无数据");


    ServiceCodeEnum(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    private Integer code;

    private String message;
}
```

## 2、自定义异常，继承于RuntimException

```java
package com.dftcmedia.tckk.microservice.common.exceptions;

import com.dftcmedia.tckk.microservice.common.enums.ServiceCodeEnum;
import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * <p>DESC: </p>
 * <p>DATE: 2019/2/13</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: DengC</p>
 */
@EqualsAndHashCode(callSuper = true)
@Data
public class ApiException extends RuntimeException {
    private static final long serialVersionUID = -3005819231762830260L;
    public ApiException(ServiceCodeEnum codeEnum) {
        super(codeEnum.getMessage());
        this.errorCode = codeEnum.getCode();
    }

    public ApiException(Integer code, String message) {
        super(message);
        this.errorCode = code;
        this.data = null;
    }

    public ApiException(Integer code, String message, Object data) {
        super(message);
        this.errorCode = code;
        this.data = data;
    }

    private Integer errorCode;

    private Object data;
}
```

## 3、配置全局异常捕获

```java
package com.dftcmedia.tckk.microservice.common.exceptions;

import com.dftcmedia.tckk.microservice.common.enums.ServiceCodeEnum;
import com.dftcmedia.tckk.microservice.common.model.vos.JsonResultVO;
import com.dftcmedia.tckk.microservice.common.util.ResponseHelper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.HttpStatus;
import org.springframework.validation.BindException;
import org.springframework.validation.FieldError;
import org.springframework.web.HttpMediaTypeNotSupportedException;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * <p>DESC: 统一异常处理 </p>
 * <p>DATE: 2019/2/13</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: DengC</p>
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * Exception异常类捕捉
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = Exception.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public JsonResultVO catchException(Exception e) {
        log.error("error={},详细信息={}", "Exception 抛出异常", e);
        return ResponseHelper.createJsonResult(ServiceCodeEnum.BAD_REQUEST);
    }

    /**
     * apiException异常类捕捉
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = ApiException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public JsonResultVO catchApiException(ApiException e) {
        log.error("error={},详细信息={}", "apiException抛出异常", e);
        return ResponseHelper.createJsonResult(e.getErrorCode(), e.getMessage(), e.getData());
    }

    /**
     * MethodArgumentNotValidException异常处理
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public JsonResultVO catchMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error("error={}", "MethodArgumentNotValidException 抛出");
        log.error("详细信息={}", e);
        FieldError fieldError = e.getBindingResult().getFieldError();
        String defaultMessage = null;
        if (fieldError != null) {
            defaultMessage = fieldError.getDefaultMessage();
        }
        return ResponseHelper.createJsonResult(ServiceCodeEnum.PARAM_ERROR.getCode(), defaultMessage);
    }

    /**
     * HttpMediaTypeNotSupportedException异常处理
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = HttpMediaTypeNotSupportedException.class)
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)
    public JsonResultVO catchHttpMediaTypeNotSupportedException(HttpMediaTypeNotSupportedException e) {
        log.error("error={}", "HttpMediaTypeNotSupportedException 抛出");
        log.error("详细信息={}", e);
        return ResponseHelper.createJsonResult(ServiceCodeEnum.NOT_SUPPORTED);
    }

    /**
     * IllegalStateException异常处理
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = IllegalStateException.class)
    @ResponseStatus(HttpStatus.NOT_ACCEPTABLE)
    public JsonResultVO catchIllegalStateException(IllegalStateException e) {
        log.error("error={}", "IllegalStateException 抛出");
        log.error("详细信息={}", e);
        return ResponseHelper.createJsonResult(ServiceCodeEnum.DATA_FORMAT_ERROR);
    }

    /**
     * IllegalStateException异常处理
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = HttpRequestMethodNotSupportedException.class)
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)
    public JsonResultVO catchHttpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) {
        log.error("error={}", "HttpRequestMethodNotSupportedException 抛出");
        log.error("详细信息={}", e);
        return ResponseHelper.createJsonResult(ServiceCodeEnum.NOT_SUPPORTED);
    }

    /**
     * DataIntegrityViolationException异常处理
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.LENGTH_REQUIRED)
    public JsonResultVO dataIntegrityViolationException(DataIntegrityViolationException e) {
        log.error("error={}", "DataIntegrityViolationException 抛出");
        log.error("详细信息={}", e);
        return ResponseHelper.createJsonResult(ServiceCodeEnum.LENGTH_REQUIRED);
    }

    /**
     * BindException异常处理
     *
     * @param e 异常信息
     * @return JsonResultVO
     */
    @ExceptionHandler(value = BindException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public JsonResultVO catchBindException(BindException e) {
        log.error("error={}", "BindException 抛出");
        log.error("详细信息={}", e);
        FieldError fieldError = e.getBindingResult().getFieldError();
        String defaultMessage = null;
        if (fieldError != null) {
            defaultMessage = fieldError.getDefaultMessage();
        }
        return ResponseHelper.createJsonResult(ServiceCodeEnum.PARAM_ERROR.getCode(), defaultMessage);
    }
}
```

# 二、日期工具类

## 1、Date日期格式化工具

```java
package com.dftcmedia.tckk.microservice.common.util;

import javax.xml.crypto.Data;
import java.sql.Timestamp;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.ZoneOffset;
import java.util.*;

/**
 * <p>DESC: 时间工具</p>
 * <p>DATE: 2019/2/12</p>
 * <p>VERSION:1.0.0</p>
 * <p>@AUTHOR: DengC</p>
 */
public class DateUtil {
    private final static String YEAR = "yyyy";
    private final static String Y_M_D = "yyyy-MM-dd";
    private final static String YMD = "yyyyMMdd";
    private final static String Y_M_D_HMS = "yyyy-MM-dd HH:mm:ss";
    private final static String YMDHMS = "yyyyMMddHHmmss";

    /**
     * 获取YYYY格式
     *
     * @return String
     */
    public static String getYear() {
        return new SimpleDateFormat(YEAR).format(new Date());
    }

    /**
     * 获取yyyy-MM-dd格式
     *
     * @return String
     */
    public static String getYMD() {
        return new SimpleDateFormat(Y_M_D).format(new Date());
    }

    /**
     * 获取yyyyMMdd格式
     *
     * @return string
     */
    public static String getYMDay(String date) {
        return new SimpleDateFormat(YMD).format(date);
    }

    /**
     * 获取yyyyMMdd格式
     *
     * @return String
     */
    public static String getYMDay() {
        return new SimpleDateFormat(YMD).format(new Date());
    }

    /**
     * 获取yyyy-MM-dd HH:mm:ss格式
     *
     * @return string
     */
    public static String getYMDHMSS() {
        return new SimpleDateFormat(Y_M_D_HMS).format(new Date());
    }

    /**
     * 获取yyyyMMddHHmmss格式
     *
     * @return string
     */
    public static String getYmdhms() {
        return new SimpleDateFormat(YMDHMS).format(new Date());
    }

    /**
     * 日期比较
     *
     * @param s 时间
     * @param e 市价
     * @return boolean
     */
    public static boolean compareDate(String s, String e) {
        return fomatDate(s) != null && fomatDate(e) != null && Objects.requireNonNull(fomatDate(s)).getTime() >= Objects.requireNonNull(fomatDate(e)).getTime();
    }

    /**
     * 格式化时间
     *
     * @param date date
     * @return Date
     */
    public static Date fomatDate(String date) {
        return format(date, Y_M_D);
    }

    public static Date fomatDateTime(String date) {
        return format(date, Y_M_D_HMS);
    }

    private static Date format(String date, String pattern) {
        DateFormat fmt = new SimpleDateFormat(pattern);
        try {
            return fmt.parse(date);
        } catch (ParseException e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 校验日期合法性
     *
     * @param date 日期
     * @return boolean
     */
    public static boolean isValidDate(String date) {
        DateFormat fmt = new SimpleDateFormat(Y_M_D);
        try {
            fmt.parse(date);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 得到间隔的天数
     *
     * @param beginDateStr 开始时间
     * @param endDateStr   结束时间
     * @return long
     */
    public static long getDaySub(String beginDateStr, String endDateStr) {
        long day = 0;
        SimpleDateFormat format = new SimpleDateFormat(Y_M_D);
        Date beginDate = null;
        Date endDate = null;

        try {
            beginDate = format.parse(beginDateStr);
            endDate = format.parse(endDateStr);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        assert endDate != null;
        day = (endDate.getTime() - beginDate.getTime()) / (24 * 60 * 60 * 1000);
        return day;
    }

    /**
     * 得到N天后的日期
     *
     * @param days days
     * @return string
     */
    public static String getAfterDayDate(String days) {
        int daysInt = Integer.parseInt(days);
        Calendar canlendar = Calendar.getInstance();

        // 日期减 如果不够减会将月变动
        canlendar.add(Calendar.DATE, daysInt);
        Date date = canlendar.getTime();

        SimpleDateFormat sdfd = new SimpleDateFormat(Y_M_D_HMS);
        return sdfd.format(date);
    }

    /**
     * 得到n天之后是周几
     *
     * @param days days
     * @return String
     */
    public static String getAfterDayWeek(String days) {
        int daysInt = Integer.parseInt(days);
        Calendar calendar = Calendar.getInstance();

        // 日期减 如果不够减会将月变动
        calendar.add(Calendar.DATE, daysInt);
        Date date = calendar.getTime();
        SimpleDateFormat sdf = new SimpleDateFormat("E");
        return sdf.format(date);
    }

    /**
     * 日期转时间戳
     *
     * @param date 时间日期
     * @return timestamp
     */
    public static Timestamp getTimestamp(Date date) {
        return new Timestamp(date.getTime());
    }

    /**
     * 获取美制GMT格式时间
     *
     * @return String
     */
    public static String getGMT() {
        Date d = new Date();
        DateFormat format = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z", Locale.US);
        format.setTimeZone(TimeZone.getTimeZone("GMT"));
        return format.format(d);
    }

    /**
     * 将时间字符串转化为GMT格式时间
     *
     * @param gmtString gmt字符串
     * @return GMT格式时间
     */
    public static Date toGMT(String gmtString) throws ParseException {
        DateFormat gmt = new SimpleDateFormat("EEE, d MMM yyyy HH:mm:ss z", Locale.US);
        return gmt.parse(gmtString);
    }

    /**
     * 把时间根据时、分、秒转换为时间段
     *
     * @param strDate 时间字符串
     */
    public static String getTimes(String strDate) {
        String resultTimes = "";
        SimpleDateFormat df = new SimpleDateFormat(Y_M_D_HMS);
        Date now;
        try {
            now = new Date();
            Date date = df.parse(strDate);
            long times = now.getTime() - date.getTime();
            long day = times / (24 * 60 * 60 * 1000);
            long hour = (times / (60 * 60 * 1000) - day * 24);
            long min = ((times / (60 * 1000)) - day * 24 * 60 - hour * 60);
            long sec = (times / 1000 - day * 24 * 60 * 60 - hour * 60 * 60 - min * 60);

            StringBuilder sb = new StringBuilder();
            if (hour > 0) {
                sb.append(hour).append("小时前");
            } else if (min > 0) {
                sb.append(min).append("分钟前");
            } else {
                sb.append(sec).append("秒前");
            }
            resultTimes = sb.toString();
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return resultTimes;
    }

    /**
     * 获取当前时间之后多少毫秒的时间
     *
     * @param times 毫秒数
     * @return date
     */
    public static Date getExpiresInDate(Long times) {
        Date date = null;
        try {
            // 获取当前毫秒数
            Long milliSecond = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
            SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            milliSecond = milliSecond + times;
            String d = format.format(milliSecond);
            date = format.parse(d);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }


    /**
     * 获取年龄
     *
     * @param birthDay 生日
     * @return 年龄
     */
    public static Integer getAge(Date birthDay) {
     	if (birthDay == null){
            return 0;
        }
        //获取当前系统时间
        Calendar cal = Calendar.getInstance();
        //如果出生日期大于当前时间，则抛出异常
        if (cal.before(birthDay)) {
            throw new IllegalArgumentException(
                    "The birthDay is before Now.It's unbelievable!");
        }
        //取出系统当前时间的年、月、日部分
        int yearNow = cal.get(Calendar.YEAR);
        int monthNow = cal.get(Calendar.MONTH);
        int dayOfMonthNow = cal.get(Calendar.DAY_OF_MONTH);

        //将日期设置为出生日期
        cal.setTime(birthDay);
        //取出出生日期的年、月、日部分
        int yearBirth = cal.get(Calendar.YEAR);
        int monthBirth = cal.get(Calendar.MONTH);
        int dayOfMonthBirth = cal.get(Calendar.DAY_OF_MONTH);
        //当前年份与出生年份相减，初步计算年龄
        int age = yearNow - yearBirth;
        //当前月份与出生日期的月份相比，如果月份小于出生月份，则年龄上减1，表示不满多少周岁
        if (monthNow <= monthBirth) {
            //如果月份相等，在比较日期，如果当前日，小于出生日，也减1，表示不满多少周岁
            if (monthNow == monthBirth) {
                if (dayOfMonthNow < dayOfMonthBirth) {
                    age--;
                }
            } else {
                age--;
            }
        }
        return age;
    }
  
      private final static int[] dayArr = new int[] { 20, 19, 21, 20, 21, 22, 23, 23, 23, 24, 23, 22 };
    private final static String[] constellationArr = new String[] { "摩羯座", "水瓶座", "双鱼座", "白羊座", "金牛座", "双子座", "巨蟹座", "狮子座", "处女座", "天秤座", "天蝎座", "射手座", "摩羯座" };


    /**
     * 根据日期计算属于哪个星座
     * @param  date
     * @return 星座
     */
    public static String getConstellation(Date date) {
        if(date == null){
            return "";
        }
        Calendar instance = Calendar.getInstance();
        instance.setTime(date);
        int month = instance.get(Calendar.MONTH)+1;
        int day = instance.get(Calendar.DAY_OF_MONTH);
        return day < dayArr[month - 1] ? constellationArr[month - 1] : constellationArr[month];
    }
  
      /**
     * 將当前天数+1
     * @param date
     * @return
     */
    public static Date addOneDay(Date date) {
        Calendar calendar = new GregorianCalendar();
        calendar.setTime(date);
        //把日期往后增加一天,整数  往后推,负数往前移动
        calendar.add(Calendar.DATE, 1);
        //这个时间就是日期往后推一天的结果
        date = calendar.getTime();
        return date;
    }

}
```

# 三、递归查询组织架构

建表

```sql
CREATE TABLE `tbl_dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `dept_name` varchar(255) DEFAULT NULL,
  `loc_add` varchar(255) DEFAULT NULL,
  `parent_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
```

核心代码

```java
/**
	 * @Description: 递归查询机构 
	 * @param @param departList
	 * @param @param departId    设定文件 
	 * @return void    返回类型 
	 * @throws
	 */
private void getDepartmentList(List<SysDepartment> departList, Integer departId) {
  try {
    List<SysDepartment> list = departmentService.getDListByParentId(departId);
    if (null != list && list.size()>0) {
      for (int i = 0; i < list.size(); i++) {
        SysDepartment department = list.get(i);
        departList.add(department);
        getDepartmentList(departList, department.getDepartId());//递归调用
      }
    }
  } catch (Exception e) {
    e.printStackTrace();
  }
}
```

调用：

```java
List<SysDepartment> departList = departmentService.getDListByParentId(Integer.parseInt(departId));
if (null != departList && departList.size() > 0) {
  SysDepartment department = departList.get(0);
  getDepartmentList(departList, department.getDepartId());
  returnCode = Const.RETURN_CODE_1;
  map.put("department", departList);
}
```

自己感悟写的递归：

```java
package com.eim.dos;

import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

/**
 * DESC: *
 * Created by zhengyong on 2019/3/14.
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@TableName("tbl_dept")
public class DepartmentDO {

    private Integer id;

    private String deptName;

    private String locAdd;

    private Integer parentId;

    @TableField(exist = false)
    private List<DepartmentDO> doList;

}

```

测试类：、

```java
	/**
	 * @Description: 递归查询机构
	 * @param @param departList
	 * @param @param departId    设定文件
	 * @return void    返回类型
	 * @throws
	 */
	private void getDepartmentList(DepartmentDO departmentDO) {
		try {

			QueryWrapper<DepartmentDO> wp = new QueryWrapper<>();
			wp.eq("parent_id",departmentDO.getId());
			List<DepartmentDO> list = departmentDao.selectList(wp);

			departmentDO.setDoList(list);
			if (null != list && list.size()>0) {
				for (int i = 0; i < list.size(); i++) {

					DepartmentDO department = list.get(i);

					getDepartmentList(department);//递归调用
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


	/**
	 * 测试递归方法
	 */
	@Test
	public void testDiGui(){
		QueryWrapper<DepartmentDO> wp = new QueryWrapper<>();
		wp.eq("parent_id",0);
		List<DepartmentDO> departList = departmentDao.selectList(wp);
		if (null != departList && departList.size() > 0) {
			departList.stream().forEach(departmentDO->{
				getDepartmentList(departmentDO);
			});
		}
	}
```

