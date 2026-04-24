# 股权税务架构分析 Skill

一个经过[Skill Forge](链接)训练的股权税务分析AI系统。覆盖境内非上市公司股权交易的主要场景，能够针对具体案例出具结构化的税务分析备忘录。

## 能力范围

- **13种交易类型**：自然人/法人直接转让、合伙企业穿透、创投基金、减资退出、对赌回购、离婚股权分割、赠与、继承、股权代持、非货币资产出资、资本公积转增、股权激励行权
- **35条法条**：逐条从国家税务总局法规库核验原文
- **9个争议口径**：双口径定量分析，两种算法各算到具体金额
- **11个测试案例**：含多事件并行的复合压力测试

## 工作流程

```
Phase 1 · 信息收集（多轮对话，追问关键变量）
    ↓ 用户确认
Phase 2 · 结构化确认（Case State锁定）
    ↓ 确认通过
Phase 3 · 分析与交付（全自动，输出备忘录）
```

## 文件结构

### 部署文件（上传到Claude Project）

| 文件 | 角色 |
|------|------|
| `instructions.txt` | Custom Instructions（主控路由+铁律+门禁） |
| `phase1-collect.md` | Phase 1 信息收集逻辑 |
| `phase2-confirm.md` | Phase 2 确认表格式与规则 |
| `phase3-analyze.md` | Phase 3 分析逻辑（10个并行思考维度） |
| `memo-template.md` | 备忘录格式规范（§一至§九） |
| `transaction-types.md` | 13种交易类型预制件库 |
| `tax-rules.md` | 12节税种规则 |
| `dispute-cases.md` | 9个争议口径 |
| `anti-patterns.md` | 10个反模式检查 |
| `statutes.json` | 35条法条（含关键条款原文+时效标注） |
| `source-registry.md` | 信息源分层清单 |
| `data-architecture.md` | Case State结构定义 |

## 怎么用

1. 在Claude中创建一个新的Project
2. 将`instructions.txt`的内容粘贴到Project的Custom Instructions
3. 将其余文件作为Knowledge Files上传
4. 开始对话，描述你的股权交易场景

## 局限性

- 仅覆盖境内非上市公司股权交易，不含跨境和上市公司
- 缺少非公开的地方执行口径和实务案例积累
- 法条时效性依赖定期更新（12号公告/创投8号文/49号公告均至2027年底）
- 辅助分析工具，不构成法律意见，不替代专业判断

## 开发过程

本Skill使用[Skill Forge](链接)框架，从零开发耗时6小时。

## 许可

本项目采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) 许可协议。允许学习、修改、分享，禁止商业使用。

## 作者

唐梦 (TANG Meng)
tomnovember@gmail.com
