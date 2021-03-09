# 亲测可用！！！Spring Cloud Gateway 与 Spring Boot Starter Web 冲突解决方案

1. 找到冲突的Spring Boot Starter Web 的来源

    ![%E4%BA%B2%E6%B5%8B%E5%8F%AF%E7%94%A8%EF%BC%81%EF%BC%81%EF%BC%81Spring%20Cloud%20Gateway%20%E4%B8%8E%20Spring%20Boot%20Starter%20%20cd0bd68275ae4724a61b4b5f03510b1c/Untitled.png](%E4%BA%B2%E6%B5%8B%E5%8F%AF%E7%94%A8%EF%BC%81%EF%BC%81%EF%BC%81Spring%20Cloud%20Gateway%20%E4%B8%8E%20Spring%20Boot%20Starter%20%20cd0bd68275ae4724a61b4b5f03510b1c/Untitled.png)

2. 我的是因为eureka里面的引用导致的冲突

    ![%E4%BA%B2%E6%B5%8B%E5%8F%AF%E7%94%A8%EF%BC%81%EF%BC%81%EF%BC%81Spring%20Cloud%20Gateway%20%E4%B8%8E%20Spring%20Boot%20Starter%20%20cd0bd68275ae4724a61b4b5f03510b1c/Untitled%201.png](%E4%BA%B2%E6%B5%8B%E5%8F%AF%E7%94%A8%EF%BC%81%EF%BC%81%EF%BC%81Spring%20Cloud%20Gateway%20%E4%B8%8E%20Spring%20Boot%20Starter%20%20cd0bd68275ae4724a61b4b5f03510b1c/Untitled%201.png)

3. 所以排除掉Spring Boot Starter Web

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    ```

4. 但是排除掉后启动会报错，经过排查缺少spring-webmvc，所以添加这个的依赖即可

    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
    </dependency>
    ```