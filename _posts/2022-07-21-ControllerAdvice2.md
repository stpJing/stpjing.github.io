---
layout: post
read_time: true
show_date: true
title: "SpringBoot 全局化消息返回与错误处理，兼容swagger(二)"
date: 2022-06-12
tags: [ControllerAdvice, RestControllerAdvice, swagger]
category: opinion
author: 荆凉凉
github: stpJing/stpJing.web
img: posts/20220612/title.jpg
description: "在上一篇中提供的解决方案是不完善的，事实上，我们还有更多东西要知道。"

---

在[上一篇](https://stpjing.github.io/ControllerAdvice.html)中提到的解决方案是不完善的。很多情况下，你需要面对的接口文档是不可理喻的，他们并不知道什么是OpenAPI，甚至连我们上一篇中提到的统一格式返回也不愿意接受，这一篇中，我会为您展示Message定制化的小技巧。

我们以一下应用场景为例：

仍然，我们需要返回这样的一个对象：

```json
{
"success": true,
"data": {},
"message": "error message or success message"
}
```

如果您看过我的上一篇博客，就会发现两者的区别，我们需要返回的字符串，现在并不是"errorMessage"，而是"message"。

无疑，这个改变是灾难性的，这就意味着，我们需要真对不同的接口，返回不同的message。

毫无疑问，通过controllerAdvice对所有的返回处理是无法做到这一点的，当然，有些情况下也可以，比如说，您可以创建一个超类，使他具有一个message属性，再让所有的返回都继承这个超类，进而，在controllerAdvice中，我们去判断返回体的类型，进而读取这一属性，将其设置为message的内容。

我相信您读到这里的时候一定和我一样倒吸了一口凉气，因为这样的写法不仅会使您的程序变得无比的复杂，更会破坏了您代码的优雅，更要命的是，这种写法更破坏了解耦，这无疑是对一个后端程序员尊严的挑战！

幸好，天无绝人之路，我为您提供了一种方法，是您不需要那么难看地去做这件事。

我们需要定义一个注解，像这样：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface MessageBody {
    String value();
}
```

如您所见，这是一个普通到不能再普通的注解，我们能够用它来做什么呢？

答案是我们用它来标注方法，像这样：

```java
    @MessageBody("登出成功！")
    @PostMapping("/logout")
    public void logout(){

        userService.logoutProcess();
    }
```

我想您已经猜出我要做什么了，在接下来的处理中，我们只需要对这个注解进行检测，并将其注入message即可。

```java
    @Override
    public Object beforeBodyWrite(Object returnValue, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        String returnClassType = returnType.getParameterType().getName();
        //必须对String单独处理，否则报错
        Message message = Message.successMessage(returnValue);
        MessageBody annotation = AnnotationUtils.findAnnotation(returnType.getMethod(), MessageBody.class);
        if (annotation!=null)
            message.setInfo(annotation.value());


        if ("java.lang.String".equals(returnClassType)){
            try {
                return objectMapper.writeValueAsString(message);
            } catch (JsonProcessingException e) {
                throw new RuntimeException(e);
            }
        }
        return message;
    }
```

大功告成！新的代码无疑既保住了您的尊严，也十分恰当地符合了解耦的标准。

稍后我会更新github中的内容。
