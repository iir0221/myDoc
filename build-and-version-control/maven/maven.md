# maven



 
 
 

 
 
 
 
 
3.dependency 
3.1依赖范围与classpath的关系 
依赖范围(Scope) 
对于编译有效 
对于测试有效 
对于运行有效 
例子 
备注 
compile 
Y 
Y 
Y 
spring-core 
在编译，测试，运行时spring-core的jar都要加入到classpath中 
test 
- 
Y 
- 
Junit 
只在测试阶段junit的jar要加入到classpath中 
provided 
Y 
Y 
- 
servlet-api 
只在编译，测试中要加入servlet-api的jar,但运行时，容器会提供这个jar，所以运行期不要加入 
runtime 
- 
Y 
Y 
jdbc驱动 
在编译时，只需要 sun提供的jdbc接口即可,在测试和运行期则要这个驱动. 
POMsystem 
Y 
Y 
- 
本地的,Maven仓库之外的类库文件 
与provided依赖范围一样。但使用system范围的依赖必须通过systemPath元素显式指定依赖文件的路径。因为这个依赖不是由Maven仓库解析的，而且都与本机系统绑定，可能造成构建不可移植，慎用. 
 
3.2依赖传递性影响依赖范围 
 
最左侧一列代表了直接依赖范围，最顶层一行代表了传递性依赖的范围，行与列的交叉单元格就表示最终的传递性依赖范围。表中的“-“表示该传递性依赖将会被忽略。 
 
 
compile 
provided 
runtime 
test 
compile 
compile(*) 
– 
runtime 
– 
provided 
provided 
– 
provided 
– 
runtime 
runtime 
– 
runtime 
– 
test 
test 
– 
test 
– 
 
 
3.3依赖调解 
 
两个原则： 
1.路径最近者优先原则 
2.第一声明者优先原则 
 
 
3.4排除依赖 
 
<dependencies> 
   <dependency> 
     <groupId>sample.ProjectB</groupId> 
     <artifactId>Project-B</artifactId> 
     <version>1.0-SNAPSHOT</version> 
     <exclusions> 
       <exclusion> 
         <groupId>sample.ProjectE</groupId>             <!-- Exclude Project-E from Project-B --> 
         <artifactId>Project-E</artifactId> 
       </exclusion> 
     </exclusions> 
   </dependency> 
 </dependencies> 
 
3.5properties 
 
<!--用于定义变量--> 
<properties> 
    <springframework.version>4.2.6.RELEASE</springframework.version> 
</properties> 
 
<!--所以依赖--> 
<dependencies> 
     
    <dependency> 
        <groupId>org.springframework</groupId> 
        <artifactId>spring-core</artifactId> 
        <!--使用了前面定义的变量--> 
        <version>${springframework.version}</version> 
    </dependency> 
 
    <dependency> 
        <groupId>org.springframework</groupId> 
        <artifactId>spring-beans</artifactId> 
        <version>${springframework.version}</version> 
    </dependency> 
 
</dependencies> 
 
3.6从命令行调用dependency插件目标 
mvn dependency:analyze 
used undeclared dependencies 
unused declared dependencies 
 
mvn dependency:tree 
 
mvn dependency:list 
 
 
4.reporting 
4.1pom中格式 
 
 
<project> 
  <reporting> 
        <plugins> 
            <plugin> 
                <!--必要--> 
                <!--通过命令project-info-reports:一个具体的Goal，来生成相应的info--> 
                <!--example--> 
                <!--project-info-reports:dependencies --> 
                <groupId>org.apache.maven.plugins</groupId> 
                <artifactId>maven-project-info-reports-plugin</artifactId> 
                <version>2.7</version> 
 
                <!--用于一次性生成多种info--> 
                <!--命令为:--> 
                <!--mvn:site--> 
                <reportSets> 
                    <reportSet> 
                        <reports> 
                            <report>dependencies</report> 
                            <report>project-team</report> 
                            <report>mailing-list</report> 
                            <report>cim</report> 
                            <report>issue-tracking</report> 
                            <report>license</report> 
                            <report>scm</report> 
                            <report>summary</report> 
                            <report>index</report> 
                        </reports> 
                    </reportSet> 
                </reportSets> 
            </plugin> 
        </plugins> 
    </reporting> 
 
</project> 
 
4.2Goals 
The Project Info Reports Plugin has the following goals: 
project-info-reports:cim is used to generate the Project Continuous Integration Management report. 
project-info-reports:dependencies is used to generate the Project Dependencies report. 
project-info-reports:dependency-convergence is used to generate the Project Dependency Convergence report for (reactor) builds. 
project-info-reports:dependency-info is used to generate code snippets to be added to build tools. 
project-info-reports:dependency-management is used to generate the Project Dependency Management report. 
project-info-reports:distribution-management is used to generate the Project Distribution Management report. 
project-info-reports:help is used to display help information on the Project Info Reports Plugin. 
project-info-reports:index is used to generate the Project index page. 
project-info-reports:issue-tracking is used to generate the Project Issue Management report. 
project-info-reports:license is used to generate the Project Licenses report. 
project-info-reports:mailing-list is used to generate the Project Mailing List report. 
project-info-reports:modules is used to generate the Project Modules report. 
project-info-reports:plugin-management is used to generate the Project Plugin Management report. 
project-info-reports:plugins is used to generate the Project Plugins report. 
project-info-reports:project-team is used to generate the Project Team report. 
project-info-reports:scm is used to generate the Project Source Code Management report. 
project-info-reports:summary is used to generate the Project Summary report. 
 
注：每一个都是一个命令 
 
 
4.3site 
<project> 
... 
<reporting> 
<plugins> 
<plugin> 
<groupId>org.apache.maven.plugins</groupId> 
<artifactId>maven-project-info-reports-plugin</artifactId> 
<version>2.9</version> 
<reportSets> 
<reportSet> 
<reports> 
<report>dependencies</report> 
<report>project-team</report> 
<report>mailing-list</report> 
<report>cim</report> 
<report>issue-tracking</report> 
<report>license</report> 
<report>scm</report> 
</reports> 
</reportSet> 
</reportSets> 
</plugin> 
... 
</plugins> 
</reporting> 
... 
</project> 
 
 
命令：mvn site 
 
 
4.4用处 
 
排除依赖 
 
 
<reporting> 
    <plugins> 
        <plugin> 
<groupId>org.apache.maven.plugins</groupId> 
            <artifactId>maven-project-info-reports-plugin</artifactId> 
            <version>2.7</version> 
        </plugin> 
    </plugins> 
</reporting> 
 
命令：mvn project-info-reports:dependencies 
 
可用于查看依赖信息从而排除有问题的依赖 
 
 
5.仓库 
5.1 快照 
 
默认情况下，maven每天检查一次快照的更新 
可以使用mvn clean install-U，强制检查更新 
 
6. 
