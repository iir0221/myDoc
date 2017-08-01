# 路径与文件

## fileset patternset

```xml
<target name="copyJsp">
    <mkdir dir="newweb" />
    <copy todir="newweb">
        <!--we目录下的所有JSP文件-->
        <fileset dir="web" includes="**/*.jsp" />
    </copy>
</target>

<target name="copyJsp2">
    <mkdir dir="newweb" />
    <copy todir="newweb">
        <fileset dir="web">
            <include name="**/*.jsp" />
        </fileset>
    </copy>
</target>

<target name="copyJsp3">
    <mkdir dir="newweb" />
    <copy todir="newweb">
        <fileset dir="web">
            <patternset id="webpattern">
                <include name="**/*.jsp" />
            </patternset>
        </fileset>
    </copy>
</target>

<target name="copyJsp4">
    <mkdir dir="newweb" />
    <copy todir="newweb">
        <fileset dir="web">
            <patternset id="webpattern">
                <exclude name="**/*.jsp" />
            </patternset>
        </fileset>
    </copy>
</target>
```

### defaultexcludes

默认排除模式

```xml
<target name="defaultexcludesecho">
    <!--打印排除文件-->
    <defaultexcludes echo="true" />
    <!--添加默认排除文件-->
    <defaultexcludes add="**/*.iml" />
    <!--去掉默认排除文件-->
    <defaultexcludes remove="**/.svn" />
    <!--重置模式集合的设置-->
    <defaultexcludes default="true" />
</target>
```

## 选择器

```
<contains> - Select files that contain a particular text string

<date> - Select files that have been modified either before or after a particular date and time

<depend> - Select files that have been modified more recently than equivalent files elsewhere

<depth> - Select files that appear so many directories down in a directory tree

<different> - Select files that are different from those elsewhere

<filename> - Select files whose name matches a particular pattern. Equivalent to the include and exclude elements of a patternset.

<present> - Select files that either do or do not exist in some other location

<containsregexp> - Select files that match a regular expression

<size> - Select files that are larger or smaller than a particular number of bytes.

<type> - Select files that are either regular files or directories.

<modified> - Select files if the return value of the configured algorithm is different from that stored in a cache.

<signedselector> - Select files if they are signed, and optionally if they have a signature of a certain name.

<scriptselector> - Use a BSF or JSR 223 scripting language to create your own selector

<readable> - Select files if they are readable.

<writable> - Select files if they are writable.
```



