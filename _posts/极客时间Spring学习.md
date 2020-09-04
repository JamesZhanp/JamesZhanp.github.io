# 极客时间Spring学习

maven  打包的语句

> mvn clean package  -Dmaven.test.skip

不使用spring-boot-parent的时候

```xml
将对应spring boot版本的内容进行加载
<dependencyManagement>
	<dependencies>
        <dependency>
        	<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>


<build>
	<plugins>
    	<plugin>
        	<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.1.RELEASE</version>
            <executions>
            	<execution>
                	<goals>
                    	<goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



## JDBC数据源配置

> pom.xml

```xml

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>


	</dependencies>

```

添加依赖Actuator，进行对于整个Spring Boot项目的监控

当访问`http://localhost:8080/actuator/beans`的时候可能会弹出需要用户名和密码， 因为我们在上面导入了spring  的Security， 因此需要进行相关的配置, 配置文件如下

```properties
management.endpoints.web.exposure.include=*
spring.output.ansi.enabled=ALWAYS
# 基本配置
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=sa
spring.datasource.hikari.maximumPoolSize=5
spring.datasource.hikari.minimumIdle=5
spring.datasource.hikari.idleTimeout=600000
spring.datasource.hikari.connectionTimeout=30000
spring.datasource.hikari.maxLifetime=1800000

spring.security.user.name=james
spring.security.user.password=123456
```



### Spring Boot中的多数据源配置

```properties
#多数据源配置
foo.datasource.url=jdbc:h2:mem:foo
foo.datasource.username=sa
foo.datasource.password=

bar.datasource.url=jdbc:h2:mem:bar
bar.datasource.username=sa
bar.datasource.password=
```



```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class,
		DataSourceTransactionManagerAutoConfiguration.class,
		JdbcTemplateAutoConfiguration.class
})
@Slf4j
public class HelloSpringApplication{

	// 分别将上述两个数据源加载声明称Bean
	@Bean
	@ConfigurationProperties("foo.datasource")
	public DataSourceProperties fooDataSourceProperties(){
		return new DataSourceProperties();
	}

	@Bean
	public DataSource fooDataSource(){
		DataSourceProperties dataSourceProperties = fooDataSourceProperties();
		log.info("foo datasource : {}", dataSourceProperties.getUrl());
		return dataSourceProperties.initializeDataSourceBuilder().build();
	}

	@Bean
	@ConfigurationProperties("bar.datasource")
	public DataSourceProperties barDataSourceProperties(){
		return new DataSourceProperties();
	}

	public static void main(String[] args) {
		SpringApplication.run(HelloSpringApplication.class, args);
	}
}
```



## docker 相关

首先在windows上下载docker for windows， 修改为国内镜像源

以使用mongo为例：

```shell
docker pull mongo
docker run --name mongo -p 27017:27017 -v ~/docker-data/mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin -d mongo
# 登录到镜像当中
docker exec -it mongo bash
# 通过shell连接db
mongo -u admin -p admin
```

