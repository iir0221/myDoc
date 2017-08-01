# install

http://maven.apache.org/plugins/maven-install-plugin/


Install Plugin 有 3个 goals:

install:install 

install:install-file 

install:help 

## install:install-file 


mvn install:install-file -Dfile=/path/mojarra~git/jsf-ri/build/lib/javax.faces.jar -DgroupId=org.glassfish -DartifactId=javax.faces -Dversion=2.2.8-19-SNAPSHOT -Dpackaging=jar