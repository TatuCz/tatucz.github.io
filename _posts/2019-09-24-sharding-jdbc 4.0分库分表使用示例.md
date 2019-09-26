---
layout: article
title: sharding-jdbc 4.0分库分表使用示例
mathjax: true
aside:
  toc: true
sidebar:
  nav: my-navi
---

# 前期准备
## 一、创建库和表

创建两个库，每个库中各有t_order和t_order_item的两个分表
```
ds_0 --- t_order_0
     |-- t_order_1
     |-- t_order_item_0
     |-- t_order_item_1


ds_1 --- t_order_0
     |-- t_order_1
     |-- t_order_item_0
     |-- t_order_item_1
```

SQL语句:

```sql
CREATE DATABASE `ds_0` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;
CREATE DATABASE `ds_1` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;

use `ds_0`;

CREATE TABLE `t_order_0` (
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `t_order_1` (
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `t_order_item_0` (
  `item_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `t_order_item_1` (
  `item_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO `ds_0`.`t_order_1` (`order_id`,`user_id`) VALUES (101, 2);
INSERT INTO `ds_0`.`t_order_item_1`(`item_id`,`order_id`,`user_id`)VALUES(2000,101,2);



use `ds_1`;

CREATE TABLE `t_order_0` (
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `t_order_1` (
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `t_order_item_0` (
  `item_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `t_order_item_1` (
  `item_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## 二、加入maven依赖
```xml
<properties>
        <sharding-sphere.version>4.0.0-RC1</sharding-sphere.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-core</artifactId>
            <version>${sharding-sphere.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-dbcp2 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-dbcp2</artifactId>
            <version>2.7.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
        </dependency>
    </dependencies>
```

# 代码

```java
public class TestSJdbc4 {
    public static void main(String[] args) throws SQLException {
        // 配置真实数据源
        Map<String, DataSource> dataSourceMap = new HashMap<String, DataSource>();
        dataSourceMap.put("ds_0", buildDataSource("ds_0"));
        dataSourceMap.put("ds_1", buildDataSource("ds_1"));

        // 配置Order表规则
        TableRuleConfiguration orderTableRuleConfig = new TableRuleConfiguration("t_order",
                                                                  "ds_${0..1}.t_order_${0..1}");
        // 配置分库
        orderTableRuleConfig.setDatabaseShardingStrategyConfig(
                new InlineShardingStrategyConfiguration("user_id",
                                      "ds_${user_id % 2}"));
        // 配置分表
        orderTableRuleConfig.setTableShardingStrategyConfig(
                new InlineShardingStrategyConfiguration("order_id",
                        "t_order_${order_id % 2}"));

        // 配置OrderItem表规则
        TableRuleConfiguration orderItemTableRuleConfig = new TableRuleConfiguration("t_order_item",
                                                                      "ds_${0..1}.t_order_item_${0..1}");
        orderItemTableRuleConfig.setDatabaseShardingStrategyConfig(
                new InlineShardingStrategyConfiguration("user_id",
                                      "ds_${user_id % 2}"));
        orderItemTableRuleConfig.setTableShardingStrategyConfig(
                new InlineShardingStrategyConfiguration("order_id",
                                      "t_order_item_${order_id % 2}"));

        // 配置分片规则，将表规则添加到configs里
        ShardingRuleConfiguration shardingRuleConfiguration = new ShardingRuleConfiguration();
        shardingRuleConfiguration.getTableRuleConfigs().add(orderTableRuleConfig);
        shardingRuleConfiguration.getTableRuleConfigs().add(orderItemTableRuleConfig);

        // 获取数据源对象
        DataSource dataSource = ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingRuleConfiguration, new Properties());

        Connection connection = dataSource.getConnection();
        String sql = "SELECT i.* FROM t_order o JOIN t_order_item i ON o.order_id = i.order_id WHERE o.user_id=? AND o.order_id=?";
        PreparedStatement statement = connection.prepareStatement(sql);

        statement.setInt(1, 2); //user_id
        statement.setInt(2, 101); //order_id

        ResultSet rs = statement.executeQuery();
        while (rs.next()) {
            System.out.println(rs.getInt(1));
            System.out.println(rs.getInt(2));
            System.out.println(rs.getInt(3));
        }

        
        statement.close();
        connection.close();
    }

    public static DataSource buildDataSource(String databaseName) {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/" + databaseName);
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        return dataSource;
    }
}
```

运行结果:
```
2000
101
2
```
