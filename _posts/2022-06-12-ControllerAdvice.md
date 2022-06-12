---
layout: post
read_time: true
show_date: true
title: "SpringBoot 全局化消息返回与错误处理，兼容swagger"
date: 2022-06-12
tags: [ControllerAdvice, RestControllerAdvice, swagger]
category: opinion
author: 荆凉凉
github: stpJing/stpJing.web
img: posts/20220612/title.jpg
description: "使用@ControllerAdvice注解处理信息返回是方便的，在适用它之前，仍然有许多内容要知道。"

---

依据[网络请求 - Ant Design Pro](https://pro.ant.design/zh-CN/docs/request/)中提出的后端接口规范，后端返回的数据至少应遵循以下模板

```json
{
"success": true,
"data": {},
"errorMessage": "error message"
}
```

为了帮助前端更好地完成工作，后端应强化接口规范。

首先创建一个工具包，再设想中，只要导入工具包，就会自动完成接口的转化。

pom中我们只引入spring-boot-starter-web：

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

然后我们定义要统一返回的Message类，这里使用泛型：

```java
public class Message<T> implements Serializable {
    boolean success;
    String errorMessage;
    T data;
    Timestamp timestamp;
    public Message(boolean success, String errorMessage, T data){
        this.success = success;
        this.errorMessage = errorMessage;
        this.data = data;
        this.timestamp = new Timestamp(System.currentTimeMillis());
    }
    public static Message failureMessage(String err){
        return new Message(false, err, null);
    }
    public static <T> Message<T> successMessage(T data){
        return new Message(true, null, data);
    }
    //省略getter, setter以及无参构造
}
```

可以看出，我们用方法failureMessage来返回一个失败信息，用方法successMessage来返回一个成功信息。

接下来我们我们分别构造错误处理与成功信息处理。

错误信息的处理是容易的，按照以上标准，只需要返回成功与否，一切错误都可视作失败，因此我们定义一个全局错误处理器，对所有错误进行统一的处理：

```java
@ControllerAdvice
public class ExceptionAdviceConfig {
    @ResponseBody
    @ExceptionHandler(Throwable.class)//处理所有异常
    public Message exceptionHanlder(Exception e){
        return Message.failureMessage(e.getMessage());
    }
}
```

之后我们去定义一个全局信息处理器吧，处理所有信息:

```java
@RestControllerAdvice
public class ResponseBodyAdivceConfig implements ResponseBodyAdvice {
    @Autowired
    private ObjectMapper objectMapper;

    /**
     * 过滤器
     * @param methodParameter
     * @param aClass
     * @return 避免重复处理Message
     */
    @Override
    public boolean supports(MethodParameter methodParameter, Class aClass) {
        return !methodParameter.getParameterType().getName().equals("org.example.utils.Message");
    }


    /**
     *进行预处理
     */
    @Override
    public Object beforeBodyWrite(Object returnValue, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        String returnClassType = returnType.getParameterType().getName();
        if ("java.lang.String".equals(returnClassType)){
            try {
                return objectMapper.writeValueAsString(Message.successMessage(returnValue));
            } catch (JsonProcessingException e) {
                throw new RuntimeException(e);
            }
        }
        return Message.successMessage(returnValue);
    }
}
```

第一个函数是过滤器，第二个是处理规则。在过滤器中，我们把自己写的Message泛型类给过滤掉了，这样能防止我们再次处理已经处理过的Message，因为我们之前已经设置了错误处理器，错误处理器会将封装的结果再次返回全局信息处理器。

这样我们用错误处理器处理失败信息，用全局信息处理器处理成功信息。

在全局信息处理器中，我们需要对String进行特殊处理，否则会报错：在Spring中，String对象不会被转换为json，所以我们需要手动转换。

当然，出于代码的美观性考虑，也可以将所有的输入信息在beforeBodyWrite中统一处理为json，这样就不需要对String单独处理了。

这样处理后，在项目中同时引入以上项目和swagger/openapi，会发现swagger/openapi无法发现接口。

我们以openapi为例，解决这个问题。

解决这个问题有两种思路，白名单与黑名单：

白名单，我们指定一个扫描范围@RestControllerAdvice(basePackages = "org.example")，这样处理器只会处理包名下面的包，但是缺少了普适性

黑名单，在过滤器中禁止掉openapi的输出类，这样openapi无法被输出，缺点是随着依赖的增加，维护成本会变大。

我们有一个更优的处理办法：

我们继承一个restcontroller注解类，让advice去扫描它，只处理用这个注解标记过的controller。

注解如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RestController
public @interface MessageController {
    @AliasFor(
            annotation = RestController.class
    )
    String value() default "";
}

```

用@MessageController取代@RestController，并修改advice标记：

```java
@RestControllerAdvice(annotations = MessageController.class)//我们用自己的注释
```

我们这样处理controller：

```java
@MessageController

@Tag(name = "demo", description = "测试")

public class DemoController {
    ...
}

```
大功告成！
