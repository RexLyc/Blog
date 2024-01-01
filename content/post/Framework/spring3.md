---
title: "Spring框架：关键模块分析"
date: 2022-08-24T15:29:45+08:00
categories:
- 计算机科学与技术
- Spring
tags:
- Spring
- 滚动更新
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/spring.jpg
# draft: true
---
Spring框架中有一些常用的模块，这里对其原理进行一定的分析，学习一下其中的设计思路。
<!--more-->
## SpringWebMVC
1. 基本流程
    - 概述
        - 核心类型：DispatchServlet
        - 术语：HandlerMapping处理器映射器、HandlerExecutionChain执行链、HandlerAdaptor处理器适配器、ViewResolver视图解析器、View视图
1. 扩展能力：
    - 添加请求拦截器：用于权限校验、建立会话、流量限制、跨域处理等各方面
        ```java
        // 自定义拦截器做CORS处理
        public class CorsInterceptor  implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
                response.setHeader("Access-Control-Allow-Origin", request.getHeader(
                        "Origin"));
                response.setHeader("Access-Control-Allow-Methods",request.getHeader(
                        "Access-Control-Request-Method"));
                response.setHeader("Access-Control-Allow-Headers",request.getHeader(
                        "Access-Control-Request-Headers"));
                response.addHeader("Access-Control-Max-Age","3600");
                response.setHeader("Access-Control-Allow-Credentials", "true");
                String method = request.getMethod();
                if (method.equals("OPTIONS")) {
                    response.setStatus(200);
                    return false;
                }
                return true;
            }
        }

        // 自定义WebMVC配置类
        @Configuration
        public class MySessionConfig implements WebMvcConfigurer {
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                // 注册CORS拦截器
                InterceptorRegistration corsRegistration=
                        registry.addInterceptor(new CorsInterceptor());
                //所有路径都被拦截
                corsRegistration.addPathPatterns("/**");
                
                // 可以进一步添加更多的拦截器
                // 继承HandlerInterceptor自定义token权限拦截
                // 继承HandlerInterceptor自定义rateLimit限流拦截器
            }
        }

        ```
1. 一些坑：
    1. 拦截器、过滤器、跨域问题：
        - 此时相关的过程包括：预检请求（OPTIONS请求）、权限检查、跨域问题。直接继承WebMvcConfigurer，并重写addCorsMapping函数时，由于该函数执行顺序晚于用户在addInterceptors阶段添加的各类自定义拦截器，因此会出现跨域问题。解决的办法有很多种：使用过滤器做跨域处理（过滤器早于所有的拦截器）、放弃重写addCorsMapping并自定义跨域处理专用Interceptors。
        - 参考：[SpringMVC 与权限拦截器冲突导致的 Cors 跨域设置失效问题](https://ld246.com/article/1631700989149)