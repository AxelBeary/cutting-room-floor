# 05 与公开先例的逐条避让表（本篇可信度的命门）

> 本篇属于《敏感字段的暴露面最小化》系列第 05 段。这一节是全文最需要诚实的地方：把本案例的每个零件对回公开先例，逐条声明"这是照做，不是发明"。漏了这一节，前面写得越漂亮越危险。
> 说明：以下先例来自联网查重结论，条目本身是公开事实；正式对外发表前引用须回原文逐条核对版本与措辞。
> 时点护栏：取自 Inkglean 实现快照（2026-09-前），不表示当前仍如此

## 总判定

本案例属于**"常见最佳实践的严格、完整落地"，不是新范式**。它没有提出任何新的防泄露机制；它做的是把一堆早已写进官方文档与顶会论文的标准件，在一个举报/审核模块里凑齐、互锁、并用测试钉住。

## 逐条避让表

| 本案例的零件 | 公开先例（谁早就有、在哪） | 本篇的避让声明 |
|---|---|---|
| 禁整块 `to_json`、出口要挑字段 | OWASP API3:2019 *Excessive Data Exposure* (https://owasp.org/www-project-api-security/projects/kits/)，原文即"别把 DB 对象全序列化、要 cherry-pick 属性、做 schema-based response validation" | 明说这是 API3:2019 的直接落地，不主张"发现"了它 |
| 举报接口泄露被举报人属性 = 一类典型缺陷 | OWASP API3:2023 *BOPLA* (https://owasp.org/www-project-api-security/projects/kits/)，其官方风险类别与本案例几乎撞车 | **必须引用并承认撞车**：本案例是该官方风险类别的一次真实工程处置，非独立提出该风险类别 |
| 输入侧 allowlist / blocklist / DTO 三解法 | OWASP *Mass Assignment Prevention Cheat Sheet* (https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Prevention_Cheat_Sheet.html)（输入侧同构：只接受白名单字段） | 输出侧裁剪是输入侧 allowlist 的镜像，思路来源公开 |
| "禁 `exclude` 黑名单、用 `fields` 白名单" | Django REST Framework 官方文档 + Cheat Sheet 明令 (https://www.django-rest-framework.org/api-guide/fields/) | 第 02 段"默认裁剪 + 显式重附"就是这条主张的输出翻版，不据为独创 |
| 默认不暴露、要暴露得显式声明 | NestJS `@Exclude()` (https://docs.nestjs.com/openapi/types-and-classes/#excluding-properties)、TypeORM `select:false` + 需要时 `addSelect` 重附、Spring Data REST *Projections* (https://docs.spring.io/spring-data/rest/current/reference/html/#projections) 双视图 | 三件套件二/件三是这些框架既有能力的自实现等价物，本案例没用这些框架也应说明"框架里本就是标准件" |
| 把列白名单编译进 SQL / 数据库层就挡住 | Hasura / PostGraphile column permissions、PostgreSQL 列级 `GRANT`、Prisma 类型化 `select` (https://www.prisma.io/docs/concepts/components/prisma-client/selecting-only-certain-fields) | "应用层安全列清单"只是把数据库列权限的思想前移到查询字符串层，非新层次 |
| 禁 `SELECT *` | SQLFluff (https://docs.sqlfluff.com/en/stable/rules.html#rule-al04) 把"禁 SELECT *"当 lint 规则 | 第 01 段"不用 SELECT * 再出口剔除"是这一既有 lint 规则的口头版落地 |
| "断言某文本不出现在响应里" | Django `SimpleTestCase.assertNotContains` (https://docs.djangoproject.com/en/stable/topics/testing/
overviews/#django.test.SimpleTestCase.assertNotContains) | 逆泄露断言的"值不出现在任意位置"仍落在 `assertNotContains` 语义内，不夸口为新断言原语 |
| 响应结构严格匹配 / 未知键即失败 | Tavern 默认严格 JSON 键匹配、Schemathesis `response_schema_conformance` (https://schemathesis.readthedocs.io/en/stable/codegen.html#response-schema-conformance)、JSON Schema `additionalProperties:false` | 类一"键校验"这一半有直接对应物；本案例只多做了"值扫描"那一小步 |
| 举报/审核序列化器：白名单属性 + 摘除敏感原文 + 自由文本切片不平铺 | 某开源系统 (Discourse) 的审核序列化器已是此形态（白名单属性、按 scope 摘除邮箱原文、自由文本 payload 不平铺） | 本案例的"下架原因不平铺、按对象嵌套给本人看自己那条"与该系统同构，须并列引用，不得暗示领先 |
| "把信任从应用上移到框架/编译器""把 runtime check 升为静态证明" | 信息流控制（IFC）谱系 20+ 年：SIF (USENIX Security 2009, https://www.usenix.org/conference/usenixsecurity09/presentation/li)、LIO* 系列 | 第 01 段"按构造"的精神来源即 IFC，本篇只借用其工程直觉，不碰其形式化 |

## 那么，本案例还剩什么"没被先例覆盖"的部分？

诚实盘点，只剩三点，且每一点都要立刻加限定：

1. **命名**："逆泄露断言（reverse-leak assertion）"这个名字未见公开对应物——但**命名不是发明**，它指的是 `assertNotContains` 早做过的事，只是给了个便于传播的叫法。
2. **组合的完整度**：把"按构造防泄露 + 类型二分 + 管理端显式重附 + 逆泄露断言 + 同一标识按用途分级"这四五件事，在**同一个不大的系统里闭环互锁**，据查重未见同等完整度的公开叙事——但每一件单拎出来都如上表是标准件，值钱的是"凑齐并咬合"，不是任何一件本身。
3. **"敏感值不出现在响应体任意位置"的断言覆盖面**：比只看键的 schema 校验宽一层（能抓被改名/被拼接/被嵌套的值）——但**这一条仍在 `assertNotContains` 语义之内**，是既有力量的用法扩展，不是新能力。

除这三点（且第 1、3 点几乎不构成差异），本篇不主张任何新颖性。

## 为什么"降级定位"反而让这篇更有用

一篇把自己说成"新范式"的方法论，读者一旦拿它去和 OWASP 文档、Discourse 实现、框架能力对比，就会发现处处"已存在"，转而怀疑整篇。把它如实说成"把 OWASP API3 做全的一次教学落地"，每一条都能回指公开出处，读者反而能确信：作者真的把标准做法逐条做到位了。**诚实的避让表不是自我贬低，是这篇可被照信、可被照做的地基。**

*本文件未过脱敏闸。*
