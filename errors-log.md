# AI 代码自检错误经验库

> 本文件由 code-self-check 技能自动维护。
> 自检过程中发现的新错误模式会追加到此处。
> AI 在每次审查前读取此文件，避免重复犯已知的错误。

---

## 错误索引

| 编号 | 分类 | 描述 | 日期 |
|------|------|------|------|
| ERR-001 | A1 | ORM 查询结果未判空直接使用 | 2026-04-03 |
| ERR-002 | B4 | 循环内执行数据库查询导致 N+1 问题 | 2026-04-03 |
| ERR-003 | D2 | SaveServiceHelper 用法错误 | 2026-04-03 |

---

## 详细记录

### ERR-001 ORM 查询结果未判空直接使用
- **日期**：2026-04-03
- **分类**：A1 - 空值安全
- **语言/框架**：Java / 金蝶云苍穹
- **问题描述**：直接对 `ORM.create().query()` 的返回结果调用方法，未检查是否为 null，当查无记录时导致 NullPointerException
- **错误代码示例**：
  ```java
  DynamicObject order = ORM.create().query("sal_order", "id,billno", 
      new QFilter("billno", QCP.equals, billNo));
  String id = order.getString("id"); // 查无记录时抛 NPE
  ```
- **正确代码示例**：
  ```java
  DynamicObject order = ORM.create().query("sal_order", "id,billno", 
      new QFilter("billno", QCP.equals, billNo));
  if (order == null) {
      throw new KDBizException("未找到单据: " + billNo);
  }
  String id = order.getString("id");
  ```
- **根本原因**：AI 假设查询总会返回结果
- **预防规则**：任何 ORM 查询后，访问结果前必须先判空

---

### ERR-002 循环内执行数据库查询导致 N+1 问题
- **日期**：2026-04-03
- **分类**：B4 - 防止 N+1 查询
- **语言/框架**：Java / 金蝶云苍穹
- **问题描述**：在 for 循环中为集合的每一项单独查询数据库，没有使用批量查询
- **错误代码示例**：
  ```java
  for (DynamicObject entry : entries) {
      String materialId = entry.getString("materialid");
      DynamicObject material = BusinessDataServiceHelper.loadSingle(
          "bd_material", "id,number,name", 
          new QFilter("id", QCP.equals, materialId));
      entry.set("materialname", material.getString("name"));
  }
  ```
- **正确代码示例**：
  ```java
  // 先批量收集 ID
  Set<Object> materialIds = entries.stream()
      .map(e -> e.get("materialid"))
      .collect(Collectors.toSet());
  // 一次批量查询
  QFilter filter = new QFilter("id", QCP.in, materialIds);
  DynamicObject[] materials = BusinessDataServiceHelper.load(
      "bd_material", "id,number,name", new QFilter[]{filter});
  // 构建查找 Map
  Map<String, DynamicObject> materialMap = Arrays.stream(materials)
      .collect(Collectors.toMap(m -> m.getString("id"), m -> m));
  // 使用 Map 填充数据
  for (DynamicObject entry : entries) {
      DynamicObject material = materialMap.get(entry.getString("materialid"));
      if (material != null) {
          entry.set("materialname", material.getString("name"));
      }
  }
  ```
- **根本原因**：AI 默认使用最简单的循环模式，未考虑性能问题
- **预防规则**：禁止在循环内执行数据库查询；始终先收集 ID 再批量查询

---

### ERR-003 SaveServiceHelper 用法错误
- **日期**：2026-04-03
- **分类**：D2 - ORM 用法
- **语言/框架**：Java / 金蝶云苍穹
- **问题描述**：`SaveServiceHelper.save()` 传参错误，传入了实体名字符串而非 DynamicObject 数组
- **错误代码示例**：
  ```java
  SaveServiceHelper.save("sal_order", order);
  ```
- **正确代码示例**：
  ```java
  SaveServiceHelper.save(new DynamicObject[]{order});
  ```
- **根本原因**：混淆了 API 方法签名；save() 接收的是 DynamicObject 数组
- **预防规则**：检查 SaveServiceHelper 方法签名；save() 接收 DynamicObject[]，不是实体名 + 对象

---

<!-- 新错误将追加在此行下方 -->
