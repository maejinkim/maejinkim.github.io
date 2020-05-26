---
title: "[Spring] Mybatis를 이용하여 DB 연동하기"

categories:
- Spring

tags: 
 - Spring
 - WEB
 - DB
 - JDBC
 - mybatis


---

[MyBatis 홈페이지 참고](https://mybatis.org/mybatis-3/ko/index.html)

### pom.xml

1. MySQL 드라이버

   ```xml
   <!-- MySQL 드라이버추가 -->
   		<dependency>
   		    <groupId>mysql</groupId>
   		    <artifactId>mysql-connector-java</artifactId>
   		    <version>8.0.19</version>
   	    </dependency>
   ```

2. MyBatis 라이브러리

   ```xml
   <!-- MyBatis라이브러리 -->
           <dependency>
             <groupId>org.mybatis</groupId>
   		  <artifactId>mybatis</artifactId>
   		  <version>3.5.4</version> 
           </dependency>
   ```

3. 스프링 연동 라이브러리

   ```xml
   <!-- 스프링 MyBatis연동 -->
           <dependency>
             <groupId>org.mybatis</groupId>
   		  <artifactId>mybatis-spring</artifactId>
   		  <version>1.3.3</version>
           </dependency>
   
   <!-- 스프링-JDBC 연동 -->
   		<dependency>
   			<groupId>org.springframework</groupId>
   			<artifactId>spring-jdbc</artifactId>
   			<version>${org.springframework-version}</version>
   		</dependency>
   ```

4. Connection Pool 사용시

   ```xml
   <!-- commons-dbcp -->
   		<dependency>
   			<groupId>commons-dbcp</groupId>
   			<artifactId>commons-dbcp</artifactId>
   			<version>1.4</version>
   		</dependency>
   ```

5. Transactions

   ```html
   <!-- Spring and Transactions -->
   		<dependency>
   			<groupId>org.springframework</groupId>
   			<artifactId>spring-tx</artifactId>
   			<version>${org.springframework-version}</version>
   		</dependency>
   ```

### mybatiis-config.xml (생략가능)

1. Mybatis-config.xml 설정 mybatis 홈페이지에 있으니 갖다 쓸 것. 단독으로 쓸때는 꼭 필요한데 스프링 연동에서는 딱히 필요없음 단, mapper.xml에서 resultType의 패키지명을 아래와 같이 풀로 써준다.

   ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c01df7c-81c0-4dcc-8d8c-e4d48bbf8113/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c01df7c-81c0-4dcc-8d8c-e4d48bbf8113/Untitled.png)

### mapper.xml

1. DOCTYPE - 홈페이지 참고

   ```xml
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
   	"<http://mybatis.org/dtd/mybatis-3-mapper.dtd>">
   ```

2. Namespace 설정

   ```xml
   <mapper namespace="guest">
   ```

3. CRUD 구현

   - 예시

     \#()안쪽의 변수는 호출되는 객체의 변수와 동일한 이름 사용

     ```xml
     <!-- 글 입력 -->
        <insert id="insert" >
            insert into guest (writer,email,tel,pass,contents,wdate) values 
             (#{writer},#{email},#{tel},#{pass},#{contents},now()) 
        </insert>
        
        <!-- 전체 글 조회 -->
        <select id="selectAll" resultType="Guest"> <!-- resultType속성 필수!! -->
            select no,writer,tel,wdate,contents
            from guest
            order by no desc
        </select>
        
        <!-- (수정폼에 출력할) 글 조회 -->
        <select id="select" resultType="Guest">
            select no, writer, email, tel, pass, contents
            from guest
            where no=#{no}
        </select>
        
        <!-- (수정폼에 입력된)글 수정 -->
        <update id="update">
            update guest
            set email=#{email}, tel=#{tel}, pass=#{pass}, contents=#{contents}
            where no=#{no}
        </update>
        
        <!-- (글번호로 구분하는 )특정글 삭제 -->
        <delete id="delete">
            delete from guest
            where no=#{no}
        </delete>
     ```

   - 전체 코드

### root-context.xml

1. db 설정 - 선택 1

   ```xml
   <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
   	    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
           <property name="url" value="jdbc:mysql://127.0.0.1:3306/ssafydb?serverTimezone=UTC&amp;useUniCode=yes&amp;characterEncoding=UTF-8"></property>
           <property name="username" value="ssafy"></property>
           <property name="password" value="ssafy"></property>                  	               
   	</bean>
   ```

   connection pool 사용시 jdbc.properties 추가 후 사용 - 선택 2

   ```
   jdbc.driver=com.mysql.cj.jdbc.Driver
   jdbc.url=jdbc:mysql://localhost:3306/ssafydb?serverTimezone=UTC&useSSL=false&allowPublicKeyRetrieval=true
   jdbc.user=ssafy
   jdbc.password=ssafy
   jdbc.maxPoolSize=20
   ```

   ```xml
   <context:property-placeholder location="classpath:jdbc.properties"/>
   
   <!-- Connection Pool을 위한 DataSource 설정 -->
   	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
   		<property name="driverClassName"  	value="${jdbc.driver}" />
   		<property name="url"  				value="${jdbc.url}" />
   		<property name="username"  			value="${jdbc.user}" />
   		<property name="password"  			value="${jdbc.password}" />
   		<property name="maxActive"  		value="${jdbc.maxPoolSize}" />
   	</bean>
   ```

2. SQL문 호출 객체

   ```xml
   <!-- XML내에 작성된 sql문을 호출하는 객체: SqlMapClient(iBatis), SqlSession(MyBatis) -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource"></property>
          
   			<!-- ResultType에 full package 작성시 생략 가능 -->
          <property name="configLocation" value="classpath:/mybatis-config.xml"></property>
   
          <property name="mapperLocations" value="classpath:mappers/*.xml"></property>                       
       </bean>
             
   	<bean class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
   	      <constructor-arg ref="sqlSessionFactory"></constructor-arg>
   	</bean>
   ```

3. dao, daoImpl 클래스

   ```xml
   <!-- 모든 DAO, DAOImpl클래스는  persistence 패키지 밑에 작성하겠음!! -->
   	<context:component-scan base-package="com.ssafy.day3.repository"/>
   	<context:component-scan base-package="com.ssafy.day3.service"/>
   ```

### 그리고 Dao,  Service구현~!



