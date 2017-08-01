# ant

## ant 构建方式
ant在分析依赖树与执行目标之前，会读入完整的构建文件。

ant在执行任何目标之前，都会先执行目标的前驱目标（depends）。

出现环依赖，ant会中止并报错。

构建文件中的所有任务都会检查它们的依赖性，如果没有必要，它们是不会做任何事的。比如说，&lt;javac&gt;只有在源代码文件比相应的.class文件更新时，才进行编译。

```xml
<?xml version="1.0"?>
<project name="structured" default="execute">
    <description>Compile and run a program</description>

    <property name="build.dir" location="build" />
    <property name="build.classes.dir" location="${build.dir}/classes" />
    <property name="dist.dir" location="dist" />
    <property name="src.dir" location="src" />

    <path id="compile.classpath">
        <pathelement location="lib/junit-4.6.jar" />
    </path>

    <target name="init" description="create dir">
        <mkdir dir="${build.classes.dir}" />
        <mkdir dir="${dist.dir}" />
    </target>

    <target name="compile" depends="init" description="compile source code">
        <javac srcdir="${src.dir}" destdir="${build.classes.dir}" debug="true">

            <classpath refid="compile.classpath" />
        </javac>
        <echo>compilation complete!</echo>
    </target>

    <target name="archive" depends="compile" description="create jar file">
        <jar destfile="${dist.dir}/project.jar" basedir="${build.classes.dir}" />
    </target>

    <target name="execute" depends="compile" description="run the program">
        <echo level="warning" message="running"/>
        <java classname="com.zxy.Hello" classpath="${build.classes.dir}">
            <arg value="hello" />
            <arg value="world" />
            <!--表示build文件所在目录绝对地址-->
            <arg file="." />
        </java>
    </target>

    <target name="clean" depends="init" description="removes the temporary dir">
        <delete dir="${build.dir}" />
        <delete dir="${dist.dir}" />
    </target>

</project>
```
和maven不同，上面例子中，如果执行ant compile archive，其真实的执行顺序为init compile init compile archive。不过由于依赖性检查的存在，会阻止其重新构建现有输出文件的行为。

另外，ant执行依赖时，按照依赖的顺序执行，因此，不要写错依赖的顺序。

## ant 命令行选项

### ant -help
查看命令行选项

### ant -buildfile（或者-f） 

ant -buildfile（或者-f） build.xmlPath compile：当前目录并不是build.xml文件所在目录时，可以通过这种方式执行

### ant -verbose和ant -debug
输出更详细的信息

### ant -emacs和ant -quiet
减少输出信息

### ant -keep-going
当失败时，继续执行剩下的目标

### ant -projecthelp
获取项目信息










