# property
## properties文件
将properties定义到单独的特征文件build.properties，并在build.xml设置文件位置。

```xml
<property file="build.properties" />
```
<center>
  <font color=green>
    ********************************************************NOTE********************************************************
    <br/><br/>
    property一旦设定不再改变
    <br/><br/>
    ***********************************************************************************************************************
  </font>
 </center>

## 环境变量
```xml
<!--通过env访问系统设置的环境变量-->
<property environment="env" />

<target name="envproperty">
    <echo>
        ${env.PATH}
        ${env.JAVA_HOME}
    </echo>
</target>
```

