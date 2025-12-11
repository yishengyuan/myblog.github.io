## 1. ProxySQL 是什么

ProxySQL 是一个高性能的 **MySQL/MariaDB 代理层**，主要用于：

- **SQL 解析与路由**
    
- **读写分离**
    
- **负载均衡**
    
- **连接池管理**
    
- **SQL 缓存**
    

它位于 **应用程序和数据库之间**，应用访问的是 ProxySQL，而 ProxySQL 再转发请求到后端数据库。
## 2. 代理原理核心

### （1）网络代理

- ProxySQL 对客户端提供 **MySQL 协议接口**（默认端口 6033），客户端像连接 MySQL 一样连接 ProxySQL。
    
- 它实现了 **TCP 层的代理**，能够解析 MySQL 协议的数据包。
    
- 当请求到达 ProxySQL 时，它会解析协议，提取 SQL 语句、用户信息、连接信息等。
    

### （2）连接池管理

- ProxySQL 自带 **连接池**，客户端连接 ProxySQL 后，并不是直接为每个请求去创建后端数据库连接，而是从连接池复用后端连接，提高性能。
    
- 支持 **前端连接池**（客户端到 ProxySQL）和 **后端连接池**（ProxySQL 到 MySQL）。
    

### （3）SQL 解析与路由

- ProxySQL 会解析 SQL 语句的 **类型、表名、用户、schema** 等信息。
    
- 根据配置的规则（Query Rules）决定将请求路由到哪个后端节点。
    

典型规则：

- **读写分离**：`SELECT` 到从库，`INSERT/UPDATE/DELETE` 到主库
    
- **分表分库**：不同的表或数据库路由到不同节点
    
- **负载均衡**：同类请求分散到多台后端服务器
    

> 核心是：**ProxySQL 能够在应用层理解 SQL 语义，而不仅仅是 TCP 转发**。

### （4）负载均衡策略

- 支持多种策略：
    
    - **轮询**（round-robin）
        
    - **最少连接**（least-connections）
        
    - **权重分配**
        
- ProxySQL 监控后端 MySQL 节点的健康状态，如果节点宕机，会自动从路由表中移除。
    

### （5）SQL 缓存

- ProxySQL 可以缓存 **只读查询结果**（SELECT），避免每次都访问数据库，提高性能。
    
- 支持 TTL 配置，缓存失效策略可灵活定制。
    

### （6）监控和统计

- ProxySQL 会维护统计信息：
    
    - SQL 执行次数
        
    - 执行时间
        
    - 后端连接状态
        
    - Query Rules 命中情况
        

这些信息可以通过 `admin` 接口查询，用于性能调优。

---

## 3. 请求处理流程示意

以客户端发起查询为例：

`客户端 → ProxySQL → SQL解析 → 路由选择 → 后端数据库 → 返回结果 → ProxySQL → 客户端`

详细流程：

1. **连接建立**：客户端连接 ProxySQL（TCP/3306 或 6033）
    
2. **SQL 解析**：ProxySQL 解析 SQL 语句，判断类型（读/写/其他）
    
3. **路由选择**：匹配 Query Rules，选择目标后端 MySQL
    
4. **负载均衡**：选择具体后端实例
    
5. **转发执行**：通过后端连接池发送 SQL
    
6. **返回结果**：ProxySQL 收到结果并转发给客户端
    
7. **统计记录**：更新监控数据
    

---

## 4. 特点总结

- **高性能**：通过连接池和缓存减少数据库压力
    
- **透明代理**：客户端无感知，使用 MySQL 协议
    
- **灵活路由**：SQL 层面路由、读写分离
    
- **高可用**：节点故障自动剔除
    
- **扩展性**：可以动态修改规则，无需重启
## 1️⃣ 配置思路

ProxySQL 支持 **SQL 语句匹配规则**（Query Rules），规则可以根据：

- SQL 类型（SELECT/UPDATE/INSERT/DELETE）
    
- 数据库/表名
    
- 用户
    
- 正则匹配 SQL 内容
    

来决定 **走主库还是从库**。

所以，你可以针对敏感字段写规则，例如：

- 规则匹配 `SELECT amount FROM account ...`
    
- 路由到主库后端组
    
- 其他普通查询走从库
    

---

## 2️⃣ 配置步骤示例

假设：

- 主库后端组：`writer_hostgroup = 10`
    
- 从库后端组：`reader_hostgroup = 20`
    

### （1）添加后端主从库

```
-- 添加主库
INSERT INTO mysql_servers(hostgroup_id, hostname, port, weight)
VALUES (10, '192.168.0.1', 3306, 100);

-- 添加从库
INSERT INTO mysql_servers(hostgroup_id, hostname, port, weight)
VALUES (20, '192.168.0.2', 3306, 100),
       (20, '192.168.0.3', 3306, 100);

```

### （2）配置 Query Rule

匹配敏感字段查询走主库：

```
INSERT INTO mysql_query_rules(
    rule_id,
    active,
    match_pattern,
    destination_hostgroup,
    apply
) VALUES (
    1,
    1,
    '^SELECT .*amount.* FROM account',  -- 正则匹配 SQL
    10,                                 -- 指向主库 hostgroup
    1
);

```

- `match_pattern` 支持正则，可以精确匹配表名和字段名
    
- `destination_hostgroup` 指定走主库
    

### （3）刷新配置

`LOAD MYSQL SERVERS TO RUNTIME; LOAD MYSQL QUERY RULES TO RUNTIME; SAVE MYSQL SERVERS TO DISK; SAVE MYSQL QUERY RULES TO DISK;`

---

## 3️⃣ 补充说明

1. **规则优先级**
    
    - ProxySQL 会按 `rule_id` 顺序匹配规则
        
    - 匹配到第一条就生效
        
    - 所以敏感查询要放在前面
        
2. **读写分离**
    
    - 其他普通 SELECT 查询可以默认走从库（reader_hostgroup）
        
    - 主库用于写入 + 特殊敏感读取
        
3. **使用用户维度**（可选）
    
    - 如果只希望特定应用/用户走主库，也可以在规则里加 `username` 条件
        
4. **性能注意**
    
    - SQL 正则匹配有开销，尽量写精准规则，避免全表匹配
        

---

✅ 结论：  
通过 **Query Rules 正则匹配敏感字段**，可以让 ProxySQL **指定 SQL 必须走主库**，从而防止脏读。