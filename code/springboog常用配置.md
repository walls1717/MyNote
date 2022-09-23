# 基础配置

## 端口

```yaml
server:
  port: 80
```

## 数据源

```yaml
spring:
  datasource:
  	# 使用了druid数据源
    druid:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/springboot_ssmp?serverTimezone=UTC
      username: root
      password: chengjie
```



# MybatisPlus

```yaml
mybatis-plus:
  global-config:
    db-config:
      # 数据库表前缀
      table-prefix: tbl_
      # 设置id为数据库字段的自增长
      # 如果不设置默认为mybatisPlus的雪花算法计算id assig
      id-type: auto
      # 逻辑删除配置
      logic-delete-field: 代表逻辑删除的字段名
      # 设置已删除数据用0表示（默认已删除数据用1表示）
      logic-delete-value: 0
      # 设置有效数据用1表示
      logic-not-delete-value: 1
    configuration:
      # 设置控制台输出日志
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
      # 启动结果集自动映射  NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。       FULL 会自动映射任意复杂的结果集（无论是否嵌套）。默认是partial，这是一种全局设置
      auto-mapping-behavior: partial
      # 开启驼峰映射
      map-underscore-to-camel-case: true
    # xml文件扫描路径，默认就是这个路径
    mapper-locations: classpath:/mapper/**/*.xml
```



# Redis

```yaml
spring:
  datasource:
    redis:
      # 主机地址ip
      host: redis
      # 端口
      port: 6379
      # 密码(默认为空)
      password:
      # Redis连接Java的客户端
      lettuce:
        pool:
          # 连接池最大连接数（使用负值表示没有限制）
          max-active: 10
          # 连接池中的最大空闲连接
          max-idle: 10
          # 连接池最大阻塞等待时间（使用负值表示没有限制）
          max-wait: -1ms
          # 连接池中的最小空闲连接
          min-idle: 1
          time-between-eviction-runs: 10s
```

