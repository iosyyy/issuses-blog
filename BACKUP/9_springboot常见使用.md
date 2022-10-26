# [springboot常见使用](https://github.com/iosyyy/issuses-blog/issues/9)

# **在springboot中使用h2数据库**

## **一、h2数据库介绍**

h2database为我们提供了十分轻量，十分快捷方便的内嵌式数据库

- H2是一个用Java开发的嵌入式数据库，它本身只是一个类库，可以直接嵌入到应用项目中。
- 可以同应用程序打包在一起发布
- 它的另一个用途是用于单元测试。启动速度快，而且可以关闭持久化功能，每一个用例执行完随即还原到初始状态
- 提供JDBC访问接口，提供基于浏览器的控制台，可以执行sql
- 免费，开源，够快
- 还方便了程序刚开始dao层单元测试测试，不需要搭建oracle，不需要加载mysql，快速测试写的dao

## **二、导入过程**

### **1. 在pom.xml中导入相关依赖**

```xml
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
```

### **2. 修改application.yml文件，加入H2相关配置**

```yaml
server:
  port: 8089
spring:
  datasource:
    url: jdbc:h2:~/test
    driver-class-name: org.h2.Driver
    username: sa
    password: 123456

  #    schema: classpath:db/schema.sql
#    data: classpath:db/data.sql
  jpa:
    database: h2
    hibernate:
      ddl-auto: update
    show-sql: true
  h2:
    console:
      path: /h2-console
      enabled: true

```

### **3. domain层，即Location类(entity)：**

```java
package com.springboot.demo.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Entity
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Location {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String type;
    private double latitude;
    private double longtitude;
}

```

### **4. dao层，即LocationRepository接口：**

```java
package com.springboot.demo.repository;

import com.springboot.demo.entity.Location;
import org.springframework.context.annotation.Bean;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface LocationRepository extends JpaRepository<Location,Long> {
    List<Location> getLocationsByType(String type);
}

```

### **5. controller层，即LocationController：**

```java
package com.springboot.demo.controller;

import com.springboot.demo.entity.Location;
import com.springboot.demo.repository.LocationRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
public class HelloContraller {

    @Autowired
    private LocationRepository locationRepository;
    @ResponseBody
    @RequestMapping("/hello")
    public List<Location> hello(){
        return locationRepository.findAll();
    }
}

```

### **6. 编写DemoApplication**

```java
package com.springboot.demo;

import com.springboot.demo.entity.Location;
import com.springboot.demo.repository.LocationRepository;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class DemoApplication {

    @Bean
    InitializingBean saveData(LocationRepository repo){
        return ()->{
            repo.save(new Location((long) 1,"1",38.998064, 117.317267));
            repo.save(new Location((long)2,"2",38.997793, 117.317069));
            repo.save(new Location((long)3,"3",38.998006, 117.317101));
            repo.save(new Location((long)4,"4",38.997814, 117.317332));
        };
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}

```

### **7. 启动项目，打开浏览器访问http://localhost:8089/hello**

![https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203135084-64423169.png](https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203135084-64423169.png)

### **8. 下面使用H2控制台查看：**

[http://localhost:8089/h2-console](http://localhost:8089/h2-console)

输入用户名sa，密码123456

![https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203227439-203171879.png](https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203227439-203171879.png)

### **9. 在打开的页面中点击左侧的Location。**

![https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203317264-2000646733.png](https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203317264-2000646733.png)

### **10. 可以看到右侧显示了SQL：**

SELECT * FROM USER

点击上面的Run执行。

![https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203401714-911208884.png](https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203401714-911208884.png)

### **11. 执行完毕后，可以看到下面显示了我们加入的数据。**

![https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203435539-599059483.png](https://img2018.cnblogs.com/blog/1471050/201904/1471050-20190412203435539-599059483.png)

# springBoot 使用okhttp3

## 1.添加pom.xml依赖

```xml
    <dependency>
        <groupId>com.squareup.okhttp3</groupId>
        <artifactId>okhttp</artifactId>
        <version>3.6.0</version>
    </dependency>

```

## 2.配置类

```java
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.concurrent.TimeUnit;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;
import okhttp3.ConnectionPool;
import okhttp3.OkHttpClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**

- Created by qhong on 2018/7/3 16:52
**/
@Configuration
public class OkHttpConfig {

    @Bean
    public X509TrustManager x509TrustManager() {
    return new X509TrustManager() {
    @Override
    public void checkClientTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {
    }
    @Override
    public void checkServerTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {
    }
    @Override
    public X509Certificate[] getAcceptedIssuers() {
    return new X509Certificate[0];
    }
    };
    }
    @Bean
    public SSLSocketFactory sslSocketFactory() {
    try {
    //信任任何链接
    SSLContext sslContext = SSLContext.getInstance("TLS");
    sslContext.init(null, new TrustManager[]{x509TrustManager()}, new SecureRandom());
    return sslContext.getSocketFactory();
    } catch (NoSuchAlgorithmException e) {
    e.printStackTrace();
    } catch (KeyManagementException e) {
    e.printStackTrace();
    }
    return null;
    }
    /**

    - Create a new connection pool with tuning parameters appropriate for a single-user application.
    - The tuning parameters in this pool are subject to change in future OkHttp releases. Currently
    */
    @Bean
    public ConnectionPool pool() {
    return new ConnectionPool(200, 5, TimeUnit.MINUTES);
    }
    @Bean
    public OkHttpClient okHttpClient() {
    return new OkHttpClient.Builder()
    .sslSocketFactory(sslSocketFactory(), x509TrustManager())
    .retryOnConnectionFailure(false)//是否开启缓存
    .connectionPool(pool())//连接池
    .connectTimeout(10L, TimeUnit.SECONDS)
    .readTimeout(10L, TimeUnit.SECONDS)
    .build();
    }
    }
```

## 3.工具类：

```java
import java.util.Iterator;
import java.util.Map;
import okhttp3.FormBody;
import okhttp3.MediaType;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;
import org.apache.commons.lang3.exception.ExceptionUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**

- Created by qhong on 2018/7/3 16:55
**/
public class OkHttpUtil{

    private static final Logger logger = LoggerFactory.getLogger(OkHttpUtil.class);

    /**

    - 根据map获取get请求参数
    - @param queries
    - @return
    */
    public static StringBuffer getQueryString(String url,Map<String,String> queries){
    StringBuffer sb = new StringBuffer(url);
    if (queries != null && queries.keySet().size() > 0) {
    boolean firstFlag = true;
    Iterator iterator = queries.entrySet().iterator();
    while (iterator.hasNext()) {
    Map.Entry entry = (Map.Entry<String, String>) iterator.next();
    if (firstFlag) {
    sb.append("?" + entry.getKey() + "=" + entry.getValue());
    firstFlag = false;
    } else {
    sb.append("&" + entry.getKey() + "=" + entry.getValue());
    }
    }
    }
    return sb;
    }

    /**

    - 调用okhttp的newCall方法
    - @param request
    - @return
    */
    private static String execNewCall(Request request){
    Response response = null;
    try {
    OkHttpClient okHttpClient = SpringUtils.getBean(OkHttpClient.class);
    response = okHttpClient.newCall(request).execute();
    int status = response.code();
    if (response.isSuccessful()) {
    return response.body().string();
    }
    } catch (Exception e) {
    logger.error("okhttp3 put error >> ex = {}", ExceptionUtils.getStackTrace(e));
    } finally {
    if (response != null) {
    response.close();
    }
    }
    return "";
    }

    /**

    - get
    - @param url 请求的url
    - @param queries 请求的参数，在浏览器？后面的数据，没有可以传null
    - @return
    */
    public static String get(String url, Map<String, String> queries) {
    StringBuffer sb = getQueryString(url,queries);
    Request request = new Request.Builder()
    .url(sb.toString())
    .build();
    return execNewCall(request);
    }

    /**

    - post
    - 
    - @param url 请求的url
    - @param params post form 提交的参数
    - @return
    */
    public static String postFormParams(String url, Map<String, String> params) {
    FormBody.Builder builder = new FormBody.Builder();
    //添加参数
    if (params != null && params.keySet().size() > 0) {
    for (String key : params.keySet()) {
    builder.add(key, params.get(key));
    }
    }
    Request request = new Request.Builder()
    .url(url)
    .post(builder.build())
    .build();
    return execNewCall(request);
    }

    /**

    - Post请求发送JSON数据....{"name":"zhangsan","pwd":"123456"}
    - 参数一：请求Url
    - 参数二：请求的JSON
    - 参数三：请求回调
    */
    public static String postJsonParams(String url, String jsonParams) {
    RequestBody requestBody = RequestBody.create(MediaType.parse("application/json; charset=utf-8"), jsonParams);
    Request request = new Request.Builder()
    .url(url)
    .post(requestBody)
    .build();
    return execNewCall(request);
    }

    /**

    - Post请求发送xml数据....
    - 参数一：请求Url
    - 参数二：请求的xmlString
    - 参数三：请求回调
    */
    public static String postXmlParams(String url, String xml) {
    RequestBody requestBody = RequestBody.create(MediaType.parse("application/xml; charset=utf-8"), xml);
    Request request = new Request.Builder()
    .url(url)
    .post(requestBody)
    .build();
    return execNewCall(request);
    }
    }
```

[[https://miaoxinwei.github.io/2017/04/21/spring-集成-okhttp3/](https://miaoxinwei.github.io/2017/04/21/spring-%E9%9B%86%E6%88%90-okhttp3/)](https://miaoxinwei.github.io/2017/04/21/spring-%E9%9B%86%E6%88%90-okhttp3/)

[https://blog.csdn.net/wangh92/article/details/79714375](https://blog.csdn.net/wangh92/article/details/79714375)

[https://www.programcreek.com/java-api-examples/?code=hjw541988478/OkHttpTutorial/OkHttpTutorial-master/app/src/main/java/in/harvestday/okhttpdemo/OkHttpUtil.java](https://www.programcreek.com/java-api-examples/?code=hjw541988478/OkHttpTutorial/OkHttpTutorial-master/app/src/main/java/in/harvestday/okhttpdemo/OkHttpUtil.java)

[https://github.com/guozhengXia/OkHttpUtils](https://github.com/guozhengXia/OkHttpUtils)

[https://github.com/hongyangAndroid/okhttputils](https://github.com/hongyangAndroid/okhttputils)