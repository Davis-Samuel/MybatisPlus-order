# 1.快速开始

![image-20200320134759713](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200320134759713.png)

- pom依赖：

  ```xml
   <!-- 数据库驱动 -->
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
  </dependency>
  <!-- lombok -->
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
  </dependency>
  <!-- mybatis-plus -->
  <!-- mybatis-plus 是自己开发，并非官方的！ -->
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.0.5</version>
  </dependency>
  ```

- application.properties：

  ```properties
  # 设置开发环境
  spring.profiles.active=dev
  
  # 数据库连接配置
  spring.datasource.username=root
  spring.datasource.password=3105311
  spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
  spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
  
  # 配置日志
  mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
  # 配置逻辑删除
  mybatis-plus.global-config.db-config.logic-delete-value=1
  mybatis-plus.global-config.db-config.logic-not-delete-value=0
  ```

- pojo  User.java：

  ```java
  package com.kuang.pojo;
  import com.baomidou.mybatisplus.annotation.*;
  import lombok.AllArgsConstructor;
  import lombok.Data;
  import lombok.NoArgsConstructor;
  import java.util.Date;
  
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class User {
  
      // 对应数据库中的主键 (uuid、自增id、雪花算法、redis、zookeeper！)
      @TableId(type = IdType.AUTO)
      private Long id;
  
      private String name;
      private Integer age;
      private String email;
  
      @Version //乐观锁Version注解
      private Integer version;
  
      @TableLogic //逻辑删除
      private Integer deleted;
  
      // 字段添加填充内容
      @TableField(fill = FieldFill.INSERT)
      private Date createTime;
  
      @TableField(fill = FieldFill.INSERT_UPDATE)
      private Date updateTime;
  
  }
  ```

- mapper UserMapper.java：

  ```Java
  package com.kuang.mapper;
  import com.baomidou.mybatisplus.core.mapper.BaseMapper;
  import com.kuang.pojo.User;
  import org.springframework.stereotype.Repository;
  
  // 在对应的Mapper上面继承基本的类 BaseMapper
  @Repository // 代表持久层
  public interface UserMapper extends BaseMapper<User> {
      // 所有的CRUD操作都已经编写完成了
      // 你不需要像以前的配置一大堆文件了！
  }
  ```

- config MyBatisPlusConfig.java：

  ```java
  package com.kuang.config;
  import com.baomidou.mybatisplus.core.injector.ISqlInjector;
  import com.baomidou.mybatisplus.extension.injector.LogicSqlInjector;
  import com.baomidou.mybatisplus.extension.plugins.OptimisticLockerInterceptor;
  import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
  import com.baomidou.mybatisplus.extension.plugins.PerformanceInterceptor;
  import com.baomidou.mybatisplus.extension.plugins.pagination.optimize.JsqlParserCountOptimize;
  import org.mybatis.spring.annotation.MapperScan;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Profile;
  import org.springframework.transaction.annotation.EnableTransactionManagement;
  
  // 扫描我们的 mapper 文件夹
  @MapperScan("com.kuang.mapper")
  @EnableTransactionManagement
  @Configuration // 配置类
  public class MyBatisPlusConfig {
  
      // 注册乐观锁插件
      @Bean
      public OptimisticLockerInterceptor optimisticLockerInterceptor() {
          return new OptimisticLockerInterceptor();
      }
  
      // 分页插件
      @Bean
      public PaginationInterceptor paginationInterceptor() {
          return  new PaginationInterceptor();
      }
  
      // 逻辑删除组件！
      @Bean
      public ISqlInjector sqlInjector() {
          return new LogicSqlInjector();
      }
  
  
      /**
       * SQL执行效率插件
       */
      @Bean
      @Profile({"dev","test"})// 设置 dev test 环境开启，保证我们的效率
      public PerformanceInterceptor performanceInterceptor() {
          PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
          performanceInterceptor.setMaxTime(100); //ms 设置sql执行的最大时间，如果超过了则不执行
          performanceInterceptor.setFormat(true);
          return performanceInterceptor;
      }
  
  }
  ```

- 主启动类 MybatisPlusApplication.java：

  ```java
  package com.kuang;
  import org.mybatis.spring.annotation.MapperScan;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  //扫描我们的mapper文件夹
  @MapperScan("com.kuang.mapper")
  @SpringBootApplication
  public class MybatisPlusApplication {
      public static void main(String[] args) {
          SpringApplication.run(MybatisPlusApplication.class, args);
      }
  }
  ```

# 2.日志 

-  handler MyMetaObjectHandler.java：

  ```java
    package com.kuang.handler;
    import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
    import lombok.extern.slf4j.Slf4j;
    import org.apache.ibatis.reflection.MetaObject;
    import org.springframework.stereotype.Component;
    import java.util.Date;
    
    @Slf4j
    @Component // 一定不要忘记把处理器加到IOC容器中！
    public class MyMetaObjectHandler implements MetaObjectHandler {
        // 插入时的填充策略
        @Override
        public void insertFill(MetaObject metaObject) {
            log.info("start insert fill.....");
            // setFieldValByName(String fieldName, Object fieldVal, MetaObject metaObject
            this.setFieldValByName("createTime",new Date(),metaObject);
            this.setFieldValByName("updateTime",new Date(),metaObject);
        }
    
        // 更新时的填充策略
        @Override
        public void updateFill(MetaObject metaObject) {
            log.info("start update fill.....");
            this.setFieldValByName("updateTime",new Date(),metaObject);
        }
    }
  ```

# 3.简单测试

- MybatisPlusApplicationTests.java ：

  ```java
  package com.kuang;
  import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
  import com.kuang.mapper.UserMapper;
  import com.kuang.pojo.User;
  import org.junit.jupiter.api.Test;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import java.util.Arrays;
  import java.util.HashMap;
  import java.util.List;
  
  @SpringBootTest
  class MybatisPlusApplicationTests {
  
      // 继承了BaseMapper，所有的方法都来自己父类
      // 我们也可以编写自己的扩展方法！
      @Autowired
      private UserMapper userMapper;
  
      @Test
      void contextLoads() {
          // 参数是一个 Wrapper ，条件构造器，这里我们先不用 null
          // 查询全部用户
          List<User> users = userMapper.selectList(null);
          users.forEach(System.out::println);
      }
  
      // 测试插入
      @Test
      public void testInsert(){
          User user = new User();
          user.setName("狂神说Java");
          user.setAge(3);
          user.setEmail("24736743@qq.com");
  
          int result = userMapper.insert(user); // 帮我们自动生成id
          System.out.println(result); // 受影响的行数
          System.out.println(user); // 发现，id会自动回填
      }
  
      // 测试更新
      @Test
      public void testUpdate(){
          User user = new User();
          // 通过条件自动拼接动态sql
          user.setId(1240620674645544965L);
          user.setName("关注公众号：狂神说");
          user.setAge(20);
          // 注意：updateById 但是参数是一个 对象！
          int i = userMapper.updateById(user);
          System.out.println(i);
      }
  
      // 测试乐观锁成功！
      @Test
      public void testOptimisticLocker(){
          // 1、查询用户信息
          User user = userMapper.selectById(1L);
          // 2、修改用户信息
          user.setName("kuangshen");
          user.setEmail("24736743@qq.com");
          // 3、执行更新操作
          userMapper.updateById(user);
      }
  
  
      // 测试乐观锁失败！多线程下
      @Test
      public void testOptimisticLocker2(){
  
          // 线程 1
          User user = userMapper.selectById(1L);
          user.setName("kuangshen111");
          user.setEmail("24736743@qq.com");
  
          // 模拟另外一个线程执行了插队操作
          User user2 = userMapper.selectById(1L);
          user2.setName("kuangshen222");
          user2.setEmail("24736743@qq.com");
          userMapper.updateById(user2);
  
          // 自旋锁来多次尝试提交！
          userMapper.updateById(user); // 如果没有乐观锁就会覆盖插队线程的值！
      }
  
      // 测试查询
      @Test
      public void testSelectById(){
          User user = userMapper.selectById(1L);
          System.out.println(user);
      }
  
      // 测试批量查询！
      @Test
      public void testSelectByBatchId(){
          List<User> users = userMapper.selectBatchIds(Arrays.asList(1, 2, 3));
          users.forEach(System.out::println);
      }
      
      // 按条件查询之一使用map操作
      @Test
      public void testSelectByBatchIds(){
          HashMap<String, Object> map = new HashMap<>();
          // 自定义要查询
          map.put("name","狂神说Java");
          map.put("age",3);
  
          List<User> users = userMapper.selectByMap(map);
          users.forEach(System.out::println);
      }
  
      // 测试分页查询
      @Test
      public void testPage(){
          //  参数一：当前页
          //  参数二：页面大小
          //  使用了分页插件之后，所有的分页操作也变得简单的！
          Page<User> page = new Page<>(2,5);
          userMapper.selectPage(page,null);
  
          page.getRecords().forEach(System.out::println);
          System.out.println(page.getTotal());
  
      }
  
  
      // 测试删除
      @Test
      public void testDeleteById(){
          userMapper.deleteById(1L);
      }
  
      // 通过id批量删除
      @Test
      public void testDeleteBatchId(){
          userMapper.deleteBatchIds(Arrays.asList(1240620674645544961L,1240620674645544962L));
      }
  
      // 通过map删除
      @Test
      public void testDeleteMap(){
          HashMap<String, Object> map = new HashMap<>();
          map.put("name","狂神说Java");
          userMapper.deleteByMap(map);
      }
  }
  ```

  

# 4.复杂测试

- WrapperTest.java：

  ```java
  package com.kuang;
  import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
  import com.kuang.mapper.UserMapper;
  import com.kuang.pojo.Role;
  import com.kuang.pojo.User;
  import org.junit.jupiter.api.Test;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  import java.util.List;
  import java.util.Map;
  
  @SpringBootTest
  public class WrapperTest {
  
      @Autowired
      private UserMapper userMapper;
  
      @Test
      void contextLoads() {
          // 查询name不为空的用户，并且邮箱不为空的用户，年龄大于等于12
          QueryWrapper<User> wrapper = new QueryWrapper<>();
          wrapper
                  .isNotNull("name")
                  .isNotNull("email")
                  .ge("age",12);
          userMapper.selectList(wrapper).forEach(System.out::println); // 和我们刚才学习的map对比一下
      }
  
      @Test
      void test2(){
          // 查询名字狂神说
          QueryWrapper<User> wrapper = new QueryWrapper<>();
          wrapper.eq("name","狂神说");
          User user = userMapper.selectOne(wrapper); // 查询一个数据，出现多个结果使用List 或者 Map
          System.out.println(user);
      }
  
      @Test
      void test3(){
          // 查询年龄在 20 ~ 30 岁之间的用户
          QueryWrapper<User> wrapper = new QueryWrapper<>();
          wrapper.between("age",20,30); // 区间
          Integer count = userMapper.selectCount(wrapper);// 查询结果数
          System.out.println(count);
      }
  
      // 模糊查询
      @Test
      void test4(){
          // 查询年龄在 20 ~ 30 岁之间的用户
          QueryWrapper<User> wrapper = new QueryWrapper<>();
          // 左和右  t%
          wrapper
                  .notLike("name","e")
                  .likeRight("email","t");
  
          List<Map<String, Object>> maps = userMapper.selectMaps(wrapper);
          maps.forEach(System.out::println);
      }
  
      // 模糊查询
      @Test
      void test5(){
  
          QueryWrapper<User> wrapper = new QueryWrapper<>();
          // id 在子查询中查出来
          wrapper.inSql("id","select id from user where id<3");
  
          List<Object> objects = userMapper.selectObjs(wrapper);
          objects.forEach(System.out::println);
      }
  
      //测试六
      @Test
      void test6(){
          QueryWrapper<User> wrapper = new QueryWrapper<>();
          // 通过id进行排序
          wrapper.orderByAsc("id");
  
          List<User> users = userMapper.selectList(wrapper);
          users.forEach(System.out::println);
      }
  }
  ```

# 5.注解解释

- ```java
  //指向表table_biao
  @TableName("table_biao)
             
  // 注解加载bean属性上，表示当前属性不是数据库的字段，但在项目中必须使用，这样在新增等使用bean的时候，mybatis-plus就会忽略这个，不会报错
  @TableField(exist = false)
             
  // 对应数据库中的主键 (uuid、自增id、雪花算法、redis、zookeeper！)
             @TableId(type = IdType.AUTO)
             private Long id;
  
             private String name;
             private Integer age;
             private String email;
  
             @Version //乐观锁Version注解
             private Integer version;
  
             @TableLogic //逻辑删除
             private Integer deleted;
  
             // 字段添加填充内容
             @TableField(fill = FieldFill.INSERT)
             private Date createTime;
  
             @TableField(fill = FieldFill.INSERT_UPDATE)
             private Date updateTime;
  ```

