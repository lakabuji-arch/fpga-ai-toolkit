# AGENTS.md — JESD204B PHY Example Design

> 本文件告诉 AI 如何在本工程中正确工作。
> 来源: impakt73/ai-rust-hw-dev (AGENTS.md 概念), OpenHW CVA6 (AI政策)

## 项目元信息

| 项 | 值 |
|----|-----|
| 器件 | XCKU5P-FFVB676-2-I |
| 工具 | Vivado 2025.2 |
| 语言 | Verilog |
| 仿真 | cocotb (默认) |
| IP 核 | jesd204_phy_v4_1_3 |

## 时钟域

| 时钟名 | 频率 | 来源 | 用途 |
|--------|------|------|------|
| tx_coreclk | 156.25 MHz | 外部晶振 | TX 数据路径 |
| rx_coreclk | 156.25 MHz | 外部晶振 | RX 数据路径 |
| drpclk | 100 MHz | 外部晶振 | DRP 配置 |
| refclk_common | 156.25 MHz | 外部晶振 | GT CPLL 参考时钟 |

**异步时钟组**: `{tx_coreclk, rx_coreclk, drpclk, txoutclk, rxoutclk}` — 已在 `.xdc` 中声明

## RTL 编码规范

> 编码风格参照: lowRISC/style-guides (被 OpenHW CVA6 采用)

### 复位风格
- 使用同步复位: `always @(posedge clk) if (reset) ...`
- PHY 的 `tx_reset_done` / `rx_reset_done` 来自 PHY 内部时钟域，需同步到用户时钟域
- 所有控制寄存器必须有复位初值

### 跨时钟域 (CDC)
- 单比特: 使用工程中的 `jesd204_phy_0_example_design_sync_block` 模块
- 该模块内部使用 `xpm_cdc_single` + 额外一级 FD，共 5 级同步
- `tx_reset_done` 和 `rx_reset_done` 通过 sync_block 在 TX 和 RX 域间互相传递
- **禁止**: 跨域信号无 CDC 直连
- **约束**: 所有异步时钟组已在 `.xdc` 中声明

### 命名约定
- 低有效: `_n`
- 时钟: `core_clk`, `drpclk`
- 复位: `reset`, `txreset`, `rxreset`
- GT 信号: `gt<lane>_<signal>` (如 `gt0_txdata`)

### Vivado 加速设置
- 综合/实现前始终添加: `set_param general.maxThreads 8`
- 来源: AMD 官方推荐 (UG835)

## 工程结构

```
├── imports/                    ← RTL 源码
│   ├── jesd204_phy_0_example_design.v        ← 顶层
│   ├── jesd204_phy_0_example_design_clks_in.v ← 时钟管理
│   ├── jesd204_phy_0_example_design_sequencer.v ← 复位状态机
│   ├── jesd204_phy_0_example_design_sync_block.v ← CDC同步
│   ├── jesd204_phy_0_example_design_data_generator.v ← TX数据发生器
│   ├── jesd204_phy_0_example_design_data_checker.v ← RX数据校验器
│   ├── jesd204_phy_0_support.v               ← PHY 支撑层
│   └── demo_tb.v                              ← 集成仿真
├── jesd204_phy_0_example_design.xdc           ← 时序约束
└── fpgahub/                    ← AI 辅助开发工具
```

## 关键设计决策

- `bypass_fake_ila = 1` — 跳过 ILA 序列（环回模式不需要多通道对齐）
- 环回模式 — TX → RX 内部环回，无需外部接线
- data_generate_enable = 2 (FREERUN) 时发递增数据 `{4{sequence_counter}}`
- checker 从第一个非逗号字节自动同步期望值

## AI 使用规则

> 来源: OpenHW CVA6 (CONTRIBUTING.md)

### ✅ 允许
- AI 辅助写代码、写 testbench、写约束
- AI 审查代码、分析报告
- AI 参与设计讨论

### ❌ 禁止
- AI 生成的代码不经人类审查直接提交
- 自动化 AI PR
- 让 AI 代替你和 reviewer 讨论
- 提交你自己不理解的代码

## 调试哲学

> 来源: impakt73/ai-rust-hw-dev (AGENTS.md)

- ❌ 永远不要靠抽象推理
- ✅ 永远用仿真数据说话 ($display)
- ✅ 先缩小范围，再二分法定位
