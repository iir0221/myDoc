# reporting

## pom中格式 
```xml 
 
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
``` 
## Goals 
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
 
 
## site
```xml
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
 ```
 
命令：mvn site 
 
 
## 用处 
 
排除依赖 
```xml 
 
<reporting> 
    <plugins> 
        <plugin> 
<groupId>org.apache.maven.plugins</groupId> 
            <artifactId>maven-project-info-reports-plugin</artifactId> 
            <version>2.7</version> 
        </plugin> 
    </plugins> 
</reporting> 
``` 
命令：mvn project-info-reports:dependencies 
 
可用于查看依赖信息从而排除有问题的依赖 
 
 