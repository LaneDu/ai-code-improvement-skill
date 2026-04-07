# 常见错误参考（金蝶云苍穹 Java）

> 各检查分类的详细参考。SKILL.md 引用本文件以获取展开的规则和示例。

---

## A 类：逻辑与边界错误

### A1 空值安全

AI 生成的代码经常在以下操作后遗漏空值检查：

| 操作 | 风险 | 所需检查 |
|------|------|----------|
| `ORM.create().query(...)` | 查无记录时返回 null | `if (result == null)` |
| `BusinessDataServiceHelper.loadSingle(...)` | 无匹配时返回 null | `if (obj == null)` |
| `DynamicObject.get("field")` | 字段未设置时返回 null | `if (value != null)` |
| `Map.get(key)` | 键不存在时返回 null | `if (map.containsKey(key))` |
| 方法参数 | 调用方可能传入 null | 在方法入口校验 |

**应遵循的模式：**
```java
// 安全查询模式
DynamicObject result = BusinessDataServiceHelper.loadSingle(entityName, selectFields, filters);
if (result == null) {
    // 处理方式：抛异常、返回默认值或记录警告
    return;
}
// 此处可安全使用 result
```

### A2 集合安全

```java
// 错误：未检查大小直接访问
DynamicObject first = list.get(0);

// 正确：访问前先检查
if (list != null && !list.isEmpty()) {
    DynamicObject first = list.get(0);
}
```

常见陷阱：对空分录调用 `DynamicObjectCollection.get(0)`，遍历 `getEntryEntity()` 返回的空集合。

### A3 类型转换

```java
// 错误：未检查直接强制转换
Long id = (Long) obj.get("id");

// 正确：使用平台工具类
Long id = DataType.toLong(obj.get("id"), 0L);

// 正确：BigDecimal 运算
BigDecimal qty = DataType.toBigDecimal(obj.get("qty"), BigDecimal.ZERO);
```

### A4 边界条件

需要注意：
- 循环索引起始值错误（0 和 1 搞混）
- 子串截取时未检查字符串长度
- 分页偏移量超过总条数
- 日期范围比较遗漏等于的情况

### A5 条件逻辑

```java
// 错误：缺少 else 分支
if (status.equals("A")) {
    doA();
} else if (status.equals("B")) {
    doB();
}
// 如果 status 是 "C" 呢？

// 正确：处理所有情况
if ("A".equals(status)) {
    doA();
} else if ("B".equals(status)) {
    doB();
} else {
    throw new KDBizException("未知状态: " + status);
}
```

另外注意：使用 `"常量".equals(变量)` 的写法来避免空指针。

---

## B 类：安全与性能

### B1 禁止硬编码密钥

```java
// 错误
String apiKey = "sk-abc123456";
String dbPassword = "admin123";

// 正确：使用平台配置
String apiKey = SystemPropertyUtils.getProperty("app.api.key");
```

### B2 防止 SQL 注入

```java
// 错误：字符串拼接
String sql = "SELECT * FROM t_order WHERE billno = '" + userInput + "'";

// 正确：使用 QFilter 参数化方式
QFilter filter = new QFilter("billno", QCP.equals, userInput);
DynamicObject[] results = BusinessDataServiceHelper.load(entityName, selectFields, new QFilter[]{filter});
```

### B3 资源管理

```java
// 错误：资源未关闭
InputStream is = new FileInputStream(path);
// ... 使用流，可能抛异常，流永远不会关闭

// 正确：使用 try-with-resources
try (InputStream is = new FileInputStream(path)) {
    // 使用流
}
```

### B4 防止 N+1 查询

**危险信号模式**：任何在 `for`、`while` 或 `forEach` 循环内的数据库查询调用。

**解决方案**：先收集标识符，批量查询，构建 Map 进行查找。详细示例见 errors-log.md ERR-002。

### B5 大数据集处理

```java
// 错误：加载所有记录，无任何限制
DynamicObject[] allOrders = BusinessDataServiceHelper.load("sal_order", "id,billno", null);

// 正确：添加分页或合理的过滤条件
QFilter dateFilter = new QFilter("auditdate", QCP.large_equals, startDate);
DynamicObject[] orders = BusinessDataServiceHelper.load("sal_order", "id,billno", 
    new QFilter[]{dateFilter}, "auditdate desc", 1000);
```

---

## C 类：代码风格与规范

### C1 命名一致性

| 元素 | 规范 | 示例 |
|------|------|------|
| 插件类 | `XxxOpPlugin` / `XxxValPlugin` | `SalOrderAuditOpPlugin` |
| 实体标识 | 小写加下划线 | `sal_order`、`bd_material` |
| 字段标识 | 小写驼峰 | `billno`、`auditdate`、`materialid` |
| 服务类 | `XxxService` / `XxxHelper` | `OrderCalcService` |
| 常量 | 大写蛇形命名 | `MAX_RETRY_COUNT` |

### C2 未使用变量

删除已声明但从未读取的变量，重点关注：
- 从查询赋值但只用于判空的变量
- 循环中收集但从未返回的变量
- 未使用类的 import 语句

### C3 重复代码

如果相同的查询/过滤/逻辑模式出现 2 次以上，应提取为私有方法。

### C4 异常处理

```java
// 错误：静默吞掉异常
try {
    doSomething();
} catch (Exception e) {
    // 被静默忽略
}

// 正确：记录日志并处理
try {
    doSomething();
} catch (Exception e) {
    log.error("操作失败: " + e.getMessage(), e);
    throw new KDBizException("处理失败，请联系管理员");
}
```

### C5 注释质量

```java
// 错误：无意义注释
int count = 0; // 将 count 设置为 0

// 正确：解释业务逻辑
// 审核通过后需要回写源单的关联数量，避免重复下推
updateSourceBillLinkQty(order);
```

---

## D 类：金蝶云苍穹平台特有

### D1 插件 API 正确性

标准插件模式：
```java
public class SalOrderAuditOpPlugin extends AbstractOperationServicePlugIn {
    
    @Override
    public void onPreparePropertys(PreparePropertysEventArgs e) {
        // 注册后续会使用的字段
        e.getFieldKeys().add("billno");
        e.getFieldKeys().add("billstatus");
    }
    
    @Override
    public void beforeExecuteOperationTransaction(BeforeOperationArgs e) {
        // 事务提交前的业务逻辑
    }
    
    @Override
    public void afterExecuteOperationTransaction(AfterOperationArgs e) {
        // 事务提交后的业务逻辑
    }
}
```

常见错误：重写了错误的事件方法、遗漏 `onPreparePropertys` 导致后续用到的字段未注册、把应该放在 `beforeExecuteOperationTransaction` 的逻辑错误地放到了 `afterExecuteOperationTransaction`。

### D2 ORM 用法

```java
// 查询单条记录
DynamicObject obj = BusinessDataServiceHelper.loadSingle(
    "sal_order", "id,billno,billstatus", 
    new QFilter[]{new QFilter("billno", QCP.equals, billNo)});

// 查询多条记录
DynamicObject[] objs = BusinessDataServiceHelper.load(
    "sal_order", "id,billno", 
    new QFilter[]{filter}, "billno asc");

// 保存记录 - 接收 DynamicObject 数组
SaveServiceHelper.save(new DynamicObject[]{order});

// 删除记录
DeleteServiceHelper.delete(entityType, new Object[]{pkId});
```

### D3 表单操作

- 始终确认实体标识与开发环境的数据模型一致
- 分录实体使用 `DynamicObjectCollection`，不要用原始 List
- 访问分录实体：`order.getDynamicObjectCollection("entryentity")`

### D4 业务规则模式

标准校验器模式：
```java
public class SalOrderSubmitValidator extends AbstractValidator {
    @Override
    public void validate() {
        for (ExtendedDataEntity entity : this.getDataEntities()) {
            DynamicObject order = entity.getDataEntity();
            BigDecimal amount = order.getBigDecimal("totalamount");
            if (amount.compareTo(BigDecimal.ZERO) <= 0) {
                this.addErrorMessage(entity, "订单金额必须大于0");
            }
        }
    }
}
```

### D5 事务感知

- `beforeExecuteOperationTransaction` 内的操作在事务中执行
- `afterExecuteOperationTransaction` 内的操作在事务提交后执行 —— 此处失败不会回滚
- 外部 API 调用（HTTP、MQ）应放在 `afterExecuteOperationTransaction` 中，避免长时间占用事务
- 如果外部调用需要回滚，应实现补偿逻辑
