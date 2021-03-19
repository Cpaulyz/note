# MVC

## SpringMVC对请求的处理

![image-20210311201424892](https://cyzblog.oss-cn-beijing.aliyuncs.com/image-20210311201424892.png)

1. 请求到DispatcherServlet
2. 通过HandleMapping，找到对应的Controller
3. 向controller请求处理
4. 返回model和view
5. 根据view找到视图解析器，把model传过去
6. 视图解析器渲染页面
7. 返回给前端

> DispatcherServlet是核心
>
> Controller是处理请求的核心

**考试：根据这个图解释一下一个请求的处理过程**