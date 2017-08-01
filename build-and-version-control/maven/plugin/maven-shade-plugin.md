# maven-shade-plugin 
**用于打可执行jar包**
 
1.参考文档 

https://maven.apache.org/plugins/maven-shade-plugin/ 
 
2.目标 

The Shade Plugin has a single goal: 

shade:shade is bound to the package phase and is used to create a shaded jar. 
 
 
3.Example 
 
Executable JAR 
 
```xml
<build> 
    <plugins> 
        <plugin> 
            <!--以下三个必须--> 
            <groupId>org.apache.maven.plugins</groupId> 
            <artifactId>maven-shade-plugin</artifactId> 
            <version>1.2.2</version> 
 
            <!----> 
            <executions> 
                <execution> 
                    <phase>package</phase> 
                    <goals> 
                        <goal>shade</goal> 
                    </goals> 
                    <configuration> 
                        <transformers> 
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer"> 
 
                                <!--指明main方法所在类，需要带包名--> 
                                <mainClass>HelloWorld</mainClass> 
                            </transformer> 
                        </transformers> 
                    </configuration> 
                </execution> 
            </executions> 
        </plugin> 
    </plugins> 
</build> 
```
