# 设计目标
* 可以通过一系列可复用的UI，容易的构造新的UI
* 容易将数据传递给UI
* 管理UI状态
* 提供事件模型，允许应用编写服务端handlers处理客户端产生的事件
* validating and error reporting
* conversion
* navigation

# FactoryFinder
```
public final class FactoryFinder extends java.lang.Object
```
FactoryFinder implements the standard discovery algorithm for all factory objects specified in the JavaServer Faces APIs. For a given factory class name, a corresponding implementation class is searched for based on the following algorithm.

# Application
```
public abstract class Application extends java.lang.Object
```
an application-wide singletons and provide functionality required by JavaServer Faces