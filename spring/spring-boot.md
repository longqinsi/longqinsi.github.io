# [技术备忘录](../README.md) | [Spring](README.md) | Spring Boot
## 目录
  1. [在哪里下载windows版的curl？](#where-to-download-curl-for-windows)
  
## 问题
### 1. 在哪里下载windows版的curl？<a name="where-to-download-curl-for-windows"></a>[↑](#top)

https://curl.haxx.se/download.html

### 2. spring boot基于哪些条件进行自动配置？
1. Presence/Absence of Jar
2. Presence/Absence of Bean
3. Presence/Absence of Property
4. Many more!

We can let spring-boot generate an auto-configuration report.
![How to Enable the Auto Configuration Report](/images/how-to-enable-spring-boot-auto-configuration-report.png)
* 在Spring Boot 2.1中，Auto Configuration Report已经更名为CONDITIONS EVALUATION REPORT