##身份验证
Kylin的Web模块使用Spring框架构建，在安全实现上选择了Spring Security。

http://docs.oracle.com/javaee/7/tutorial/partwebtier.htm#BNADP
http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/
#####1.自定义验证
下面是自定义验证（也就是“testing”profile）在kylinSecurity.xml中相关的配
置，仅保留了ADMIN用户的配置以作说明：
```
<beans profile="testing">
 <util:list id="adminAuthorities"
value-type="org.springframework.security.core.authority.SimpleGrantedAuthority">
 <value>ROLE_ADMIN</value>
 <value>ROLE_MODELER</value>
 <value>ROLE_ANALYST</value>
 </util:list>
 <bean class="org.springframework.security.core.userdetails.User" id="adminUser">
 <constructor-arg value="ADMIN"/>
 <constructor-arg
 value="$2a$10$o3ktIWsGYxXNuUWQiYlZXOW5hWcqyNAFQsSSCSEWoC/BRVMAUjL32"/>
 <constructor-arg ref="adminAuthorities"/>
 </bean>
 <bean id="kylinUserAuthProvider"
 class="org.apache.kylin.rest.security.KylinAuthenticationProvider">
 <constructor-arg>
 <bean class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
 <property name="userDetailsService">
 <bean class="org.springframework.security.provisioning.InMemoryUserDetailsManager">
 <constructor-arg>
 <util:list
value-type="org.springframework.security.core.userdetails.User">
 <ref bean="adminUser"></ref>
 <ref bean="modelerUser"></ref>
 <ref bean="analystUser"></ref>
 </util:list>
 </constructor-arg>
 </bean>
 </property>
 <property name="passwordEncoder"
ref="passwordEncoder"></property>
 </bean>
 </constructor-arg>
 </bean>
 <bean id="passwordEncoder"
class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/
 <scr:authentication-manager alias="testingAuthenticationManager">
 <!-- do user ldap auth -->
 <scr:authentication-provider ref="kylinUserAuthProvider"></scr:authentication-provider>
 </scr:authentication-manager>
</beans>
```
#####2.LDAP验证
LDAP（Lightweight Directory Access Protocol，轻量级目录访问协议）用于提供被称为目录服务的信息服务。目录以树状的层次结构来存储数据，可以存储包括组织信息、个人信息、Web链接、JPEG图像等各种信息。

#####3.单点登录
单点登录（Single Sign On，SSO）是一种高级的企业级认证服务。用户只需要登录一次，就可以访问所有相互信任的应用系统；它具有一个账户多处使用、避免了频繁登录、降低了信息的泄漏风险等优点。
（网址：http://docs.spring.io/autorepo/docs/spring-security-saml/1.0.xSNAPSHOT/reference/htmlsingle/)
##授权
#####1.新的访问权限控制
同时用户是否可以访问项目并使用项目中的某些功能由项目级访问控制决定，Apache Kylin中的项目级别设置了四种类型的访问权限角色。它们分别是ADMIN、MANAGEMENT、OPERATION和QUERY。并为每个角色都定义了用户可以在Apache Kylin中执行的功能列表
1. QUERY：查询访问权限，旨在供仅需要访问权限的分析师用于查询项目中的表或Cube
2. OPERATION：操作访问权限，旨在供需要权限以维护Cube的公司或组织的运营团队使用。OPERATION访问权限包括了QUERY
3. MANAGEMENT：管理访问权限，是用于完全了解数据、模型和Cube的商业含义的模型管理和Cube设计。管理访问权限包括OPERATION和QUERY
4. ADMIN：项目管理访问权限，旨在完全管理项目。ADMIN访问权限包括MANAGEMENT、OPERATION和QUERY
#####2.统一的项目级别访问控制
![有关每个访问权限角色可以访问的详细功能](https://upload-images.jianshu.io/upload_images/9049859-15562273165ae1ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####3.管理数据访问权限
在数据源页面能够对于用户和群组在不同表下的访问权限进行管理。选择要进行数据访问权限控制的表，再点击“Access”标签页，会显示已有的数据访问权限。如果要添加，点击“Grant”按钮进行添加；如果要删除，点击“Delete”按钮进行删除。


