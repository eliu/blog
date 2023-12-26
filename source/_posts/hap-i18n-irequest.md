---
title: 在非页面入口处理多语言环境
date: 2018-10-15 16:11:09
categories:
- HAP
tags:
- i18n
- iRequest
- locale
---

## 概述

在 HAP 中，从浏览器打开的页面或者调用的 Rest API 都必须经过控制器层，而控制器层可以很容易地获取 `HttpServletRequest` 对象，进而构造 `IRequest` 对象来存储环境上下文信息，这里就包括当前的语言环境。

举例说明：

```java
public ResponseData processRequest(HttpServletRequest request) {
    IRequest iRequest = this.createRequestContext(request);
    // 调用服务层方法并传入 IRequest 进行业务处理
    return new ResponseData(service.someMethod(iRequest));
}
```

因此在控制器层获取当前设置的语言环境是轻而易举的事情。不过，HAP 开发过程中会有很多其他的场景不会将控制器作为入口进入。在`任务管理`、`定时任务`、`工作流`和`UReport2报表`中，核心程序的入口都是通过向 Spring 注册一些通用的服务组件(`@Component`) 并实现特定的接口来实现的。因此这种场景下，如何构建一个正确的 IRequest 对象就成为一个很有技巧性的问题了。

<!-- more -->

### 基本原则

构造一个 IRequest 对象的通用方法一般是通过 RequestHelper 帮助类进行辅助完成的。同时，我们要根据不同场景来获取当前环境的语言代码。然后将构建好的 IRequest 通过参数形式传递到 Service 层的方法。

> **注意：** HAP 框架的最佳实践要求我们一定要将 IRequest 经由 Service 层调用才可以，因为框架内部已经为 Service 层增加了 AOP 处理，在调用前后会自动判断 Service 方法是否存在 IRequest 类型的参数。如果有，那么会对其自动进行管理。

### RequestHelper

HAP 框架内部提供了一个帮助类 RequestHelper 来辅助开发者获取当前设置的 IRequest 实例或者新建一个全新的 IRequest。该类定义在 `com.hand.hap.core.impl`包下，下表列出常见的方法：

| 方法名称             | 解释                                      |
| -------------------- | ----------------------------------------- |
| createServiceRequest | 根据 HttpServletRequest 构建 IRequest     |
| newEmptyRequest      | 创建一个全新的 IRequest 实例              |
| clearCurrentRequest  | 从 ThreadLocal 中清除当前的 IRequest 实例 |
| setCurrentRequest    | 将一个 IRequest 对象设置为当前的生效      |
| getCurrentRequest    | 获取当前上下文的 IRequest 对象            |

> **注意：**`getCurrentRequest` 的唯一的 bool 类型参数 `returnEmptyForNull`的含义为，若从当前上下文中无法获取有效的 IRequest 对象时：
>
> `true`: 调用 `newEmptyRequest` 创建一个新的请求对象
>
> `false`:  直接返回给调用者 **null**

接下来我们利用 RequestHelper 类来处理不同的应用场景下的多语言需求。

### en_US vs en_GB

当系统无法从上下文（ThreadLocal\<IRequest\>）获取 IRequest 实例时，通常我们会使用 newEmptyRequest 来新建一个空的 IRequest 对象。该方法会使用 ServiceRequest 类实例化 IRequest 对象，而此时 locale 属性则是使用 `Locale.getDefault().toString()` 方法来初始化默认值的。这在开发者本机一般是没有问题，然而在服务器运行时，它会返回服务器默认的语言代码 **en_US**，而不是 HAP 的 en_GB 。

```java
import java.util.Locale;

public class ServiceRequest implements IRequest {
    private String locale = Locale.getDefault().toString();
    // ...
}
```

这个现象通常会在定时任务或者 Rest API 等等场景下，在服务层无法正确查询到多语言记录。接下来我们针对不同的应用场景来分别讨论解决方法。



## 任务管理

对于任务管理，我们可以通过 `RequestHelper.getCurrentRequest` 来得到当前的 IRequest 实例。但在 Linux 服务器下，它的 locale 属性并不是 `en_GB`，而是 `en_US`，这与 HAP 的语言代码不一致，导致无法正确获取多语言记录。

因此任务管理场景下的实现思路是根据任务的提交者找到对应的系统用户ID，进而找到这个用户在首选项中的语言设置，之后替换 IRequest 的 locale 属性，最后传入 Service 层。实现代码大致如下：

```java
import com.hand.hap.account.service.IUserService;
import com.hand.hap.core.IRequest;
import com.hand.hap.core.impl.RequestHelper;
import com.hand.hap.system.dto.SysPreferences;
import com.hand.hap.system.service.ISysPreferencesService;
import com.hand.hap.task.info.ExecutionInfo;
import com.hand.hap.task.service.ITask;
import org.springframework.beans.factory.annotation.Autowired;
import com.hand.hap.account.dto.User;

import demo.service.IDemoService;

public class DemoTask implements ITask {
    @Autowired
    IDemoService demoService;

    @Autowired
    ISysPreferencesService preferencesService;
    @Autowired
    IUserService userService;

    @Override
    public void execute(ExecutionInfo executionInfo) throws Exception {
        // 获取当前任务的提交者
        User executioner = userService.selectByUserName(executionInfo.getUsername());
        // 根据提交者的用户ID得到用户首选项中设置的语言代码
        SysPreferences preference = preferencesService.selectUserPreference("locale", executioner.getUserId());
        // 得到当前的 IRequest
        IRequest request = RequestHelper.getCurrentRequest(true);
        // 设置IRequest的语言上下文
        request.setLocale(preference.getPreferencesValue());
        // 将IRequest传入Service层的方法
        // Service的AOP机制将会自动处理IRequest的生命周期
        demoService.serviceMethod(request);
    }
}
```

## 工作流

工作流场景下的实现思路是获取当前工作流的提交者（Initiator），进而定位至该员工的系统用户，最后获取首选项的语言代码构建 IRequest。实现代码大致如下：

```java
@Component
public class ActivityDemoBean implements IActivitiBean, JavaDelegate {

    @Autowired
    IUserService userService;
    @Autowired
    ISysPreferencesService preferencesService;
    @Autowired
    IDemoService demoService;

    @Override
    public void execute(DelegateExecution execution) {
        String initiator = execution.getVariable("initiator", String.class);
        List<User> users = userService.selectUserNameByEmployeeCode(initiator);

        if(CollectionUtils.isNotEmpty(users)) {
            User user = users.get(0);
            SysPreferences preference = 
                preferencesService.selectUserPreference("locale", user.getUserId());
            // 新建IRequest
            IRequest request = RequestHelper.getCurrentRequest(true);
            // 设置IRequest的语言上下文
            request.setLocale(preference.getPreferencesValue());
            // 将IRequest传入Service层的方法
            // Service的AOP机制将会自动处理IRequest的生命周期
            demoService.serviceMethod(request);
        }
    }
}
```

## UReport2 Beans

UReport2 本身提供了一个方法叫 RequestHolder 可以用来获取当前的 IRequest，我们可以借助这个方法来达到目的：

```java
@Component
public class UreportDataSourceBean {

    @Autowired
    IDemoService demoService;

    public List<Demo> loadDemoData(String dsName, 
                                   String datasetName, 
                                   Map<String, Object> parameters) {
        // 初始化请求上下文
        HttpServletRequest request = RequestHolder.getRequest();
        IRequest requestContext = RequestHelper.createServiceRequest(request);

        // 执行查询
        List<Demo> result = demoService.method(requestContext);
        return result;
    }
}
```

## 定时任务

定时任务由于其后台自动调度的特点，导致其上下文无法获取用户信息。所以针对定时任务，我们需要定义一个系统配置项来预设置一个默认的语言代码：

```java
public class Ora2062DemoJob extends AbstractJob {

    @Autowired
    IOra2062DemoService demoService;
    @Autowired
    IProfileService profileService;

    @Override
    public void safeExecute(JobExecutionContext context) throws Exception {
        // 新建一个空的 IRequest
        IRequest request = RequestHelper.getCurrentRequest(true);
        // 从系统配置项 SYSTEM_LANG 中获取默认的语言代码
        request.setLocale(profileService.getProfileValue(request, "SYSTEM_LANG"));
        // 调用服务方法
        demoService.method(request);
    }
}

```

> **提示：**SYSTEM_LANG 是一个全局系统配置项，可设置默认值为 en_GB

## Rest API Controller

HAP 通过在控制器层映射特定前缀的路径来向外部提供 Rest API 服务。一般情况下，Rest API 会结合 OAuth2 进行客户端授权管理来完成身份的验证。当我们采用用户名密码方式进行授权的时候，客户端账户会与这个系统用户进行关联和初始化，所以在控制器层创建 IRequest 的时候是可以正确取到上下文信息的。

不过这种方式有个弊端是系统用户信息会暴露给外界。在某些场景下，这是不太安全的做法，这时我们通常会改用 `client_credentials` 的授权方式。不过这种方式下，控制器层是没办法定位当前执行的上下文是哪个用户，进而 locale 也是没法获取上下文值。

## 优化服务层AOP切面类

你可以采用与`定时任务`类似的做法来解决此问题，不过这种方式需要对每一个定时任务入口类和控制器进行额外的编码处理，带来了不少的工作量。这里有一种一劳永逸的方法，就是修改服务层的切面类 `ServiceExecutionAdvice`，直接处理 en_US 的情况。这样一来，无论请求是从定时任务还是 Rest API 过来，只要遵循服务的处理都是通过调用服务接口方法并且**传入 IRequest 参数实例**，我们就会始终可以获取正确的 locale 值，实现如下：

```java
package com.hand.hap.core.impl;
public class ServiceExecutionAdvice implements MethodInterceptor {
    private final String SERVER_DEFAULT_LANG = "en_US";
    
    @Autowired
    private IProfileService profileService;
    
    public void before(Method method, Object[] args, Object target) throws Throwable {
        // ...
        IRequest requestContext = (IRequest) args[idx];
        if (requestContext != null) {

            if(SERVER_DEFAULT_LANG.equals(requestContext.getLocale())) {
                requestContext.setLocale(
                    profileService.getProfileValue(null, "SYSTEM_LANG"));
            }
            
            RequestHelper.setCurrentRequest(requestContext);
            initMDC(requestContext);
        }
        // ...
    }
}
```

