#Maven常用命令

mvn archetype：create 创建Maven项目
mvn compile 编译源代码
mvn deploy 发布项目
mvn test-compile 编译测试源代码
mvn test 运行应用程序中的单元测试
mvn site 生成项目相关信息的网站
mvn clean 清除项目目录中的生成结果
mvn package 根据项目生成的jar
mvn install 在本地Repository中安装jar
mvn eclipse:eclipse 生成eclipse项目文件
mvnjetty:run 启动jetty服务
mvntomcat:run 启动tomcat服务
mvn clean package -Dmaven.test.skip=true:清除以前的包后重新打包，跳过测试类
