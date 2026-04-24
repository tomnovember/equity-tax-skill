# 数据架构设计稿

## 核心设计：Case State（案件状态）

每个Phase读取Case State → 处理 → 更新Case State → 交给下一个Phase。
用户确认/修正时，直接更新Case State中对应字段。

```
Case State = {
  
  // === A. 事实层（Phase 1填充，Phase 2锁定）===
  
  parties: [                          // 交易主体
    {
      name: "王总",
      type: "自然人|法人|合伙企业",
      holding_pct: 40%,               // 当前实际持股（增资稀释后）
      cost_basis: 500万,              // 原始成本（已扣除历史转出）
      cost_basis_trace: "2018年现金出资500万，全部实缴", // 成本来源说明
      acquisition_date: "2018",
      acquisition_method: "出资|受让|继承|赠与",
      related_to: ["B公司：无关联"]    // 关联关系
    },
    // ...更多主体
  ],
  
  target_company: {
    name: "A公司",
    type: "有限|股份",
    location: "上海",
    industry: "软件",
    net_assets: 3000万,
    net_assets_date: "2026-03",
    undistributed_profit: 2000万,
    land_property_ratio: "<5%",
    paid_vs_subscribed: "全部实缴"
  },
  
  deal: {
    method: "直接转让|减资|赠与|继承|代持还原|非货币资产出资|转增股本|股权激励行权|...",
    price: 4800万,
    price_basis: "双方协商，参照融资估值",
    special_terms: [],                 // 对赌/回购/分期
    timeline: "无紧迫要求"
  },
  
  // T10代持场景专用字段
  proxy_holding: {                     // 代持信息（仅T10场景填充）
    exists: true|false,                // 是否涉及代持
    nominal_holder: "名义股东姓名/公司名",
    actual_holder: "实际出资人姓名/公司名",
    proxy_agreement: true|false,       // 是否有书面代持协议
    proxy_reason: "资格限制|隐私|...",
    current_goal: "代持还原|代持状态对外转让|代持期间分红处理",
    dividends_during_proxy: true|false // 代持期间是否发生过分红
  },
  
  // T11非货币资产出资场景专用字段
  non_cash_investment: {               // 非货币资产出资信息（仅T11场景填充）
    asset_type: "股权|不动产|技术成果|其他",
    asset_original_value: null,        // 资产原值/计税基础
    asset_fair_value: null,            // 评估后公允价值
    cash_supplement: null,             // 现金补价
    investor_type: "自然人|法人",
    post_investment_pct: null          // 出资后持股比例
  },
  
  // T12资本公积转增场景专用字段
  capital_reserve_conversion: {        // 转增信息（仅T12场景填充）
    source: "资本溢价|股票溢价|资产评估增值|盈余公积|未分配利润",
    company_form: "有限责任公司|股份有限公司",
    conversion_amount: null,
    is_high_tech_sme: true|false,      // 是否为中小高新技术企业（影响分期资格）
    shareholder_types: []              // 各股东身份类型
  },
  
  // T13股权激励场景专用字段
  equity_incentive: {                  // 激励信息（仅T13场景填充）
    incentive_type: "期权|限制性股票|股权奖励",
    grant_price: null,                 // 授予价/行权价
    fair_value_at_exercise: null,      // 行权/解锁时公允价值
    deferred_tax_filed: true|false,    // 是否已向税务机关备案递延纳税
    holding_platform: true|false,      // 是否通过持股平台（→不可递延）
    grant_date: null,                  // 授予日
    exercise_date: null                // 行权日/解锁日
  },
  
  // 跨境/边界检测结果
  boundary_flags: {
    cross_border: false,               // 是否涉及跨境因素
    listed_company: false,             // 是否涉及上市公司
    real_estate_involved: false,       // 是否涉及不动产
    boundary_notes: []                 // 边界提示信息
  },
  
  history: [                           // 历史交易追溯
    {
      date: "2019",
      description: "张女士将20%平价转让给张老先生",
      impact: "张老先生成本=160万（原持有人原值）; 张女士成本从480万降至320万"
    }
  ],
  
  // === B. 判断层（Phase 1形成，Phase 2锁定）===
  
  events: [                            // 识别的税务事件
    {
      id: "E1",
      description: "王总将A公司40%股权转让给B公司",
      route_type: "自然人直接转让",     // 交易矩阵中的类型
      applicable_taxes: ["个税", "印花税"],
      disputes: [],                     // 涉及的争议编号
      cross_impact: []                  // 与其他事件的交叉影响
    }
  ],
  
  info_gaps: [                         // 信息缺口
    { field: "...", level: "门禁|计算|方案", impact: "..." }
  ],
  
  initial_judgment: "...",             // Phase 1的初步判断
  scheme_directions: ["先分红再转让", "通过持股平台"],  // 方案方向预判
  complexity: "简单|中等|复杂",
  
  // === C. 计算层（Phase 3填充）===
  
  baseline: {                          // 基线税负
    per_event: [
      {
        event_id: "E1",
        taxes: [
          {
            tax_type: "个人所得税",
            taxpayer: "王总",
            rate: "20%",
            taxable_income: 4298.8万,
            tax_amount: 859.76万,
            evidence_chain: "67号公告第4条→第15条→...",
            statute_ref: "国家税务总局公告2014年第67号 第4条"
          }
        ]
      }
    ],
    total: 860.96万,
    net_proceeds: 3939.04万              // 转让方实际到手
  },
  
  schemes: [                           // 替代方案
    {
      id: "方案A",
      name: "先分红再转让",
      assumptions: "分红后买方同意降价...",
      steps: [...],
      total_tax: 860.8万,
      saving_vs_baseline: 0.16万,
      saving_pct: "0%",
      risk_level: "🟡",
      risk_note: "分红决议时间间隔",
      saving_reason: "税率相同，分红税与转让税降低量精确抵消"
    }
  ],
  
  disputes_analysis: [                 // 争议分析
    {
      dispute_id: "争议1",
      event_id: "E1",
      option_a: { method: "...", tax: X },
      option_b: { method: "...", tax: Y },
      difference: Z
    }
  ],
  
  // === D. 结论层（Phase 3填充）===
  
  risk_matrix: [...],
  recommendation: {
    primary: { scheme: "基线方案", reason: "..." },
    secondary: { scheme: "...", reason: "..." },
    not_recommended: [{ scheme: "...", reason: "..." }]
  },
  execution_timeline: [...],
  uncertainties: [...],
  
  // === E. 元数据 ===
  status: "phase1_collecting | phase2_pending_confirm | phase3_delivered",
  mode: "A|B|C",
  user_expertise: "专业|半专业|非专业",  // Phase 1第零步识别，影响Phase 3备忘录详略
  created: "2026-04-24",
  last_updated: "2026-04-24"
}
```

## Phase间流转

```
用户输入
  ↓
Phase 1: 填充 A(事实层) + B(判断层) → 输出确认请求
  ↓ 用户确认 → A+B锁定
Phase 2: 整理 A+B → 输出结构化确认表 → 用户确认
  ↓ 用户确认 → 进入全自动分析
Phase 3: 读取A+B → 填充C(计算层) + D(结论层) → 输出最终备忘录
```

## 最终输出

备忘录格式见 `memo-template.md`（§一~§九）。备忘录从Case State四层提取数据，是self-contained的独立文档。

## 设计说明

1. **Case State是内部数据结构，不给用户看**。用户看到的是每个Phase的结构化输出（确认请求/确认表/最终备忘录）。

2. **最终备忘录是完整的独立文档**——不需要回溯对话历史就能理解全部内容。这是四大/顶级律所的交付标准：备忘录本身要self-contained。

3. **Phase间交互产出≠最终备忘录**。Phase 1的输出是"要素确认表+追问"，Phase 2的输出是"结构化确认表"（请用户确认），Phase 3才生成最终备忘录。前两个Phase是工作过程，第三个Phase是交付物。

4. **模式B（方案验证）的最终输出是同一个备忘录模板**，只是§三~§五简化（用户已有方案，不需要方案生成），§六风险评估是重点。

5. **模式C（政策查询）不生成备忘录**，直接回答+附法条依据。
