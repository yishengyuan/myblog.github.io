# MySQL 关联查询（为什么大厂"禁止 join"）

## 三条铁律

1. **跨库 join 禁止** —— 一旦分库分表，join 根本无法下推
2. **大表 join 禁止** —— 即使同库，几百万行 join 会拖垮数据库
3. **必要时 join 也要小表驱动大表**

---

## 为什么大厂不让 join

```
逻辑上：
  SELECT u.name, o.amount
  FROM orders o JOIN users u ON o.user_id = u.id

实际跑起来：
  1. 扫描驱动表（假设是 orders 1000 万行）
  2. 每一行去被驱动表查一次（即使有索引也是 1000 万次 IO）
  3. 网络/锁/CPU 全被占用
  4. 如果分库分表，join 跨节点 → 把所有数据拉到一台机器再算（OOM）
```

更深层原因：
- **不可水平扩展**：数据库的连接数、CPU、内存是垂直瓶颈
- **慢 SQL 拖死整个库**：一条复杂 join 会拖慢其他业务
- **执行计划不可控**：MySQL 优化器有时选错索引、错驱动表

---

## 小表驱动大表（必要时的 join 优化）

```
错的：大表驱动小表
  ┌─────────────┐         ┌──────────────┐
  │ orders 1kw  │────每行→│ users 1w     │
  └─────────────┘         └──────────────┘
  扫 1000 万行，回表 1000 万次

对的：小表驱动大表
  ┌──────────┐            ┌─────────────┐
  │ users 1w │────每行→  │ orders 1kw  │ ← user_id 必须有索引
  └──────────┘            └─────────────┘
  扫 1 万行，每行索引查找 → 索引命中
```

MySQL 的 `STRAIGHT_JOIN` 可以强制驱动顺序：
```sql
SELECT u.name, o.amount
FROM users u STRAIGHT_JOIN orders o ON o.user_id = u.id
WHERE u.vip = 1
```

---

## 替代方案：业务层"手动 join"

```mermaid
flowchart LR
  A[Step1: 查 users<br/>SELECT id,name FROM users WHERE vip=1] --> B[拿到 userIds 集合]
  B --> C[Step2: 用 IN 查 orders<br/>SELECT * FROM orders<br/>WHERE user_id IN [...]]
  C --> D[Step3: 业务层 Map 拼接<br/>orderList.forEach(o -> o.userName = userMap.get(o.userId))]
```

代码模板：
```java
// 1. 先查小表
List<User> users = userMapper.selectVip();
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, u -> u));

// 2. 用 IN 查大表
List<Long> ids = new ArrayList<>(userMap.keySet());
List<Order> orders = orderMapper.selectByUserIds(ids);

// 3. 内存里拼装
orders.forEach(o -> o.setUserName(userMap.get(o.getUserId()).getName()));
```

**好处**：
- 跨库也能跑
- 每条 SQL 都简单，DBA 容易优化
- 业务层可以加缓存（userMap 走 Redis）

**注意**：
- `IN` 的元素数不要超过 1000（MySQL 解析慢），分批查
- 防 N+1：先收集所有 id 一次查完，**不要循环里查 DB**

---

## 何时可以用 join

- 都是小表（< 1 万行）
- 一次性报表/管理后台、非线上接口
- 强一致性场景，且业务层拼装太复杂

线上 C 端高并发接口：**一律业务层拼装 + 缓存**。
