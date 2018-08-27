# IOC 容器

把 Laravel 容器方面所了解的知识总结了下，如有错误的、不恰当的地方还请@我。注：并未讲到全部，后续更新

## 入口文件 index.php

```
1、加载框架依赖
2、创建 app 容器
3、解析 Http\Kernel.php 内核
4、执行请求操作
5、返回结果
6、终止应用程序
```
![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/kzuImouMTL.png?imageView2/2/w/1240/h/0)

## app.php 文件（require_once __DIR__.'/../bootstrap/app.php'）

```
1、创建容器
2、绑定 Http\Hernel.php 到容器
3、绑定 Console\Kernel.php 到容器
4、绑定 Exception\Hadnler.php 到容器
5、返回 app 容器
```
![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/muipVNnQMe.png?imageView2/2/w/1240/h/0)

## app 容器

### 打印 app 容器

```
1、serviceProviders 属性
    用于存放所有已经注册完毕的服务提供者

2、loadedProviders 属性
    跟 serviceProviders 数组类似，标记方式不同

3、deferredServices 属性
    用来储存所有延迟加载服务提供者

4、resolved 属性
    服务解析完毕，都会在 resolved 属性里面存入一条记录，表示绑定已经解析过

5、bindings 属性
    记录所有绑定

6、instances 属性
    在第一次解析绑定时，如果 shared 为 true则都会往 instances 属性里面存入一条记录

7、aliases 属性
    服务绑定的别名，别名数量无限制

```
![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/VtCKhcrcyC.png?imageView2/2/w/1240/h/0)

## 注册服务提供者

注册服务提供者只需要继承 ServiceProvider 抽象类即可（放在那儿都可以），并在 config/app.php providers 数组里面注册
或使用 app()->register(TestProvider::class)（此方法 $defer 属性无效） 即可

```
1、继承 ServiceProvider 抽象类

2、创建 register 方法并使用 app 容器绑定服务（可绑定任意数量服务）

3、创建 boot 方法，初始化所绑定的服务

4、$defer 属性为是否延迟加载，为 true 会存入到 deferredServices 属性，程序注册时会调用 ServiceProvider 抽象类
	的 isDeferred() 方法 『必须在config/app.php providers中注册』

5、事件触发注册服务提供者，ServiceProvider 抽象类的 when 方法返回一个数组，数组里面包含事件名称 
	如：[TestEvent::class]，『$defer 必须为 true』

注意：
    PHP 在处理请求前，都会从入口文件把所有 PHP 文件都扫描一遍。Laravel 为了性能考虑，会在第一次初始化的时候，
	把所有服务提供者都缓存到 bootstrap/cache/service.php 文件里面，所以有时候在改或增了服务提供者，刷新可能不生
	效，这有可能是缓存所致，这时把 service.php 与 packages.php 删除再重试

```

## 注册服务提供者别名

app()->alias('testProvider','test')  别名数量无限制

![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/8D6amdy1ZF.png?imageView2/2/w/1240/h/0)

## app 容器绑定

```
1、app()->bind($abstract, $concrete = null, $shared = false)
    第一个参数是服务绑定名称，第二个参数是绑定结果（参数类型：\Closure|string|null），第三个参数是否共享（类似
	单例），默认为 false，第二个参数，如果是非闭包，内部会包裹上闭包，好处是延迟加载，节约空间

    ![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/4cieFAedyN.png?imageView2/2/w/1240/h/0)

2、app()->singleton($abstract, $concrete = null)
    第一个参数是服务绑定名称，第一个参数是绑定结果，在内部是调用了 app()->bind($abstract, $concrete = null, true)，
	第三个参数为 true

    ![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/Qer6WAKPqD.png?imageView2/2/w/1240/h/0)

3、app()['test'] = function(){ return 'test' }
    使用了 PHP ArrayAccess 接口，在内容调用了 app()->bind() 方法，第三个参数为 false

    ![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/HKUVw8Y5RP.png?imageView2/2/w/1240/h/0)
	![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/d82lF7Nhrz.png?imageView2/2/w/1240/h/0)
	
4、app()->instance('test',实例)
    绑定一个实例，跟上面三个比，就是少了一个闭包

```

## app 容器解析

```
1、app($abstract, $parameters = [])
    第一个参数是服务绑定的名称或别名，第二个参数为上下文名称或别名
    
2、app()->make($abstract, $parameters = [])
    同上

3、app()[$abstract]
    只有第一个参数，使用了 PHP ArrayAccess 接口
	
```


## DI 依赖注入

DI 是 IOC 的一种实现，DI 是一种设计模式，IOC 是一种设计思想，DI 实现原理是使用 PHP 反射机制来反射出相应依赖对
象名称，通过 aliases 属性得到服务名称，然后从容器解析出服务实例，最后传入对应方法，这个过程就是所谓的依赖注入

![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/AihlK26IDc.png?imageView2/2/w/1240/h/0)
![file](https://lccdn.phphub.org/uploads/images/201808/23/23047/VwWEUNfPIv.png?imageView2/2/w/1240/h/0)

## 总结

Laravel 是一个组件式框架，实现了高度解耦，再使用 IOC 容器来管理解耦后的组件，我把这个容器理解为使用了对象的高级注册树，当组件越来越多时，那么这个容器对象不是越来越大吗？所以 Laravel 使用了闭包来延迟加载，但是有个问题，如果每次获取都去 new 一次，不是很浪费时间与空间吗？在能满足需求的情况下，能不能只 new 一次（类似单例），所以 Laravel 又引入了 shared 来实现单例，用 instances 属性来保存单例（服务在生命周期内，shared 不可更改，而且 Laravel 大部分服务 shared 都为 true），现在又有个需求，当某个动作发生时，就触发服务加载？所以 Laravel 又加入了 when 方法来绑定事件，前提是这个服务 shared 必须为 true。












