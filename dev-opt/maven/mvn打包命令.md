# mvn打包

- mvn package：打包到本项目，一般在项目target目录下。 

- mvn install：打包到本地仓库，如果没设置Maven本地仓库，一般在用户/.m2目录下。 

- mvn deploy：打包上传到远程仓库，如：私服nexus等，需要配置pom文件。 

## 打包过程 

- mvn clean package 依次执行：clean、resources、compile、testResources、testCompile、test、jar(打包)。 
- mvn clean install 依次执行：clean、resources、compile、testResources、testCompile、test、jar(打包)、install。 
- mvn clean deploy 依次执行：clean、resources、compile、testResources、testCompile、test、jar(打包)、install、deploy。