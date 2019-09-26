---
title: Jekyll - Code Blocks
key: 20160616
tags: Jekyll
---

## Code Spans

Use `<html>` tags for this.

Here is a literal `` ` `` backtick.
And here is ``  `some`  `` text (note the two spaces so that one is left
in the output!).

```javascript
(() => console.log('hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world!'))();
```

<!--more-->

**markdown:**

    Here is a literal `` ` `` backtick.
    And here is ``  `some`  `` text (note the two spaces so that one is left
    in the output!).

## Standard Code Blocks

    Here comes some code

    This text belongs to the same code block.

^
    This one is separate.

**markdown:**

```
    Here comes some code

    This text belongs to the same code block.

^
    This one is separate.
```

---

```
(() => console.log('hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world!'))();
```

**markdown:**

    ```
    (() => console.log('hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world!'))();
    ```

---

```javascript
(() => console.log('hello, world!'))();
```


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


**markdown:**

    ```javascript
    (() => console.log('hello, world!'))();
    ```

---

```none
(() => console.log('hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world!'))();
```

## Highlighting Code Snippets

{% highlight javascript %}
(() => console.log('hello, world!'))();
{% endhighlight %}

**markdown:**

```
{%- raw -%}
{% highlight javascript %}
(() => console.log('hello, world!'))();
{% endhighlight %}
{% endraw %}
```

### Line Numbers

{% highlight javascript linenos %}
var hello = 'hello';
var world = 'world';
var space = ' ';
(() => console.log(hello + space + world + space + hello + space + world + space + hello + space + world + space + hello + space + world))();
{% endhighlight %}

**markdown:**

```
{%- raw -%}
{% highlight javascript linenos %}
var hello = 'hello';
var world = 'world';
var space = ' ';
(() => console.log(hello + space + world + space + hello + space + world + space + hello + space + world + space + hello + space + world))();
{% endhighlight %}
{% endraw %}
```

---

{% highlight javascript %}
(() => console.log('hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world!'))();
{% endhighlight %}

{% highlight none %}
(() => console.log('hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world! hello, world!'))();
{% endhighlight %}

## Fenced Code Blocks

~~~
Here comes some code.
~~~

**markdown:**

    ~~~
    Here comes some code.
    ~~~

---

~~~~~~~~~~~~
~~~~~~~
code with tildes
~~~~~~~~
~~~~~~~~~~~~~~~~~~

**markdown:**

    ~~~~~~~~~~~~
    ~~~~~~~
    code with tildes
    ~~~~~~~~
    ~~~~~~~~~~~~~~~~~~

## Language of Code Blocks

~~~
def what?
  42
end
~~~
{: .language-ruby}

**markdown:**

    ~~~
    def what?
      42
    end
    ~~~
    {: .language-ruby}

---

~~~ ruby
def what?
  42
end
~~~





**markdown:**

    ~~~ ruby
    def what?
      42
    end
    ~~~