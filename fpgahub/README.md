# FPGA AI 辅助开发工具集 (FPGAHub)

> 提炼自顶级 FPGA 开源项目：impakt73/ai-rust-hw-dev、OpenHW CVA6、darklife/darkriscv、analogdevicesinc/hdl、cocotb/cocotb

---

## 目录结构

```
fpgahub/
├── README.md                        ← 本文件
├── templates/
│   └── AGENTS.md                    ← 新工程模板 (AI行为宪法)
├── skills/                          ← FPGA 专用 Skills
│   ├── xpm-selector/SKILL.md        ← CDC/FIFO 原语选择器
│   ├── vivado-tcl-scripting/SKILL.md ← Vivado Tcl 自动化
│   ├── timing-closure/SKILL.md      ← 时序收敛诊断+修复
│   ├── fpga-debug/SKILL.md          ← 结构化硬件调试
│   └── axi-bus-checklist/SKILL.md   ← AXI 总线协议检查
└── tools/
    ├── vcd-mcp/README.md            ← AI 波形分析 (MCP)
    └── scripts/
        ├── Makefile                 ← 一键命令
        ├── run_synth.tcl            ← 综合脚本
        └── run_build.tcl            ← 全流程脚本
```

## 快速开始

### 1. 新工程初始化

```bash
# 复制 AGENTS.md 模板到新工程
cp fpgahub/templates/AGENTS.md <new_project>/AGENTS.md

# 改 3 行: 器件、时钟、仿真工具
# 然后 AI 自动遵守这些规范
```

### 2. 安装 Skills

```bash
# 将 skills 目录复制到 ~/.claude/skills/
# 或直接使用 VS Code Copilot 内置 Skills 目录
```

### 3. 工具链配置 (可选)

```bash
# Verible (SystemVerilog 格式化)
# https://github.com/chipsalliance/verible/releases

# vcd-mcp (AI 分析波形)
# 见 tools/vcd-mcp/README.md
```

### 4. 日常命令

```bash
make format     # 格式化所有 RTL
make lint       # 代码检查
make sim        # 运行 cocotb 仿真
make synth      # 综合
make build      # 综合 + 实现 + bitstream
make timing     # 时序检查
```

## 来源标注

| 组件 | 灵感来源 |
|------|----------|
| AGENTS.md 模板 | impakt73/ai-rust-hw-dev + OpenHW CVA6 |
| 编码规范 | lowRISC/style-guides (CVA6 采用) |
| AI 使用政策 | OpenHW CVA6 (CONTRIBUTING.md) |
| 仿真验证 (cocotb) | cocotb/cocotb |
| 项目结构 | darklife/darkriscv |
| Vivado 自动化 | analogdevicesinc/hdl |
| Skills 设计方法 | Superpowers (obra/superpowers) |
| vcd-mcp | impakt73/ai-rust-hw-dev |
| Verible | chipsalliance/verible (lowRISC) |

## 核心原则

1. **AI 辅助，人类决策** — AI 写代码，人类审查和负责
2. **仿真优先，数据说话** — 不靠抽象推理，用 $display 和波形
3. **cocotb 默认** — Python testbench，除非明确要求才用 Verilog TB
4. **maxThreads 8** — 所有 Vivado 脚本默认多线程加速
5. **先设计，后实现** — brainstorming → writing-plans → TDD
6. **不验证，不完成** — 编译+仿真+时序 全过才算"完成"
