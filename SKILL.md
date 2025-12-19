---
name: gdpval
description: "GDPVAL专业任务执行增强skill。针对需要网络数据获取、参考文件处理、专业文档生成、定量分析的知识密集型任务，提供Prompt需求严格执行、能力检测、数据验证、幻觉预防和优雅降级的增强能力。适用于涉及外部数据获取、多源数据整合、专业报告生成的复杂任务。可与docx、xlsx、pdf等skill协同使用。"
license: MIT
---

# GDPVAL 专业任务执行增强 Skill

## 概述

此skill解决知识密集型专业任务执行中的系统性问题，增强以下核心能力：

1. **需求层增强** - Prompt需求解析与严格执行（**最高优先级**）
2. **感知层增强** - 外部数据获取与参考文件访问
3. **推理层增强** - 数据幻觉预防与约束理解
4. **行动层增强** - 优雅降级与透明报告
5. **验证层增强** - 输出可追溯性与事实核查
6. **集成层增强** - 与其他skill（docx/xlsx/pdf）协同

---

## 零、需求层增强：Prompt严格执行（最高优先级）

### 0.1 核心原则

**Prompt中的要求是任务执行的最高优先级约束。任何其他考虑（包括降级策略）都不能违背prompt的明确要求。**

### 0.2 Prompt需求解析流程

**任务开始前，必须执行以下解析步骤：**

```
┌─────────────────────────────────────────────────────────────┐
│                 Prompt需求解析流程                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Step 1: 提取输出格式要求                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 扫描prompt中的关键词：                                 │  │
│  │ - "Word document" / ".docx" → 需要docx skill          │  │
│  │ - "Excel" / ".xlsx" / "spreadsheet" → 需要xlsx skill  │  │
│  │ - "PDF" / ".pdf" / "diagram" → 需要生成PDF            │  │
│  │ - "PowerPoint" / ".pptx" → 需要pptx skill             │  │
│  └───────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│  Step 2: 提取交付物清单                                     │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 列出prompt中要求的所有交付物：                         │  │
│  │ - 每个交付物的名称                                     │  │
│  │ - 每个交付物的格式要求                                 │  │
│  │ - 每个交付物的内容要求                                 │  │
│  └───────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│  Step 3: 提取参考文件要求                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ - 必须使用的参考文件列表                               │  │
│  │ - 需要参照的格式/风格                                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│  Step 4: 提取特殊约束                                       │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ - 页数限制                                             │  │
│  │ - 格式风格（如"mirror the bulleted style"）            │  │
│  │ - 使用的资源（如"official GCP icons"）                 │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 0.3 输出格式检测与Skill调用

**根据prompt要求，自动确定需要调用的skill：**

| Prompt关键词 | 输出格式 | 需要调用的Skill | 调用方式 |
|--------------|----------|-----------------|----------|
| "Word document", ".docx" | .docx | docx skill | 使用docx-js创建 |
| "Excel", ".xlsx", "spreadsheet" | .xlsx | xlsx skill | 使用xlsx库创建 |
| "PDF diagram", ".pdf" | .pdf | 转换或专用工具 | HTML转PDF或专用库 |
| "PowerPoint", ".pptx" | .pptx | pptx skill | 使用pptx库 |

### 0.4 交付物清单模板

**任务开始时，必须生成并跟踪交付物清单：**

```markdown
## 交付物清单（从Prompt提取）

| # | 交付物名称 | 格式要求 | 内容要求 | 状态 |
|---|-----------|----------|----------|------|
| 1 | [名称] | [.docx/.xlsx/.pdf] | [具体内容] | ⏳待完成 |
| 2 | [名称] | [格式] | [内容] | ⏳待完成 |

## 需要调用的Skills
- [ ] docx skill - 用于生成Word文档
- [ ] xlsx skill - 用于生成Excel文件
- [ ] 其他
```

### 0.5 Skill集成指南

#### 调用docx skill生成Word文档

当prompt要求生成.docx文件时，**必须**使用docx skill：

```javascript
// 使用docx-js生成Word文档
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
        HeadingLevel, AlignmentType, BorderStyle, WidthType } = require('docx');

const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } },
    paragraphStyles: [
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal",
        run: { size: 32, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 240, after: 240 } } },
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal",
        run: { size: 28, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 180, after: 180 } } }
    ]
  },
  sections: [{
    children: [
      // 文档内容
    ]
  }]
});

// 保存文件
const buffer = await Packer.toBuffer(doc);
fs.writeFileSync("output.docx", buffer);
```

#### 生成PDF架构图

当prompt要求生成PDF图表时，可选方案：

**方案A：HTML转PDF（推荐）**
```bash
# 使用Chrome/Puppeteer
npx puppeteer print diagram.html diagram.pdf

# 或使用wkhtmltopdf
wkhtmltopdf diagram.html diagram.pdf
```

**方案B：使用draw.io/diagrams.net导出**
- 创建.drawio文件
- 使用drawio CLI导出PDF

**方案C：使用Mermaid生成**
```bash
npx mmdc -i diagram.mmd -o diagram.pdf
```

---

## 一、感知层增强：数据获取能力

### 1.1 网络访问能力检测与替代方案

**问题**：WebFetch/WebSearch可能因网络环境（代理、防火墙）失效

**执行流程**：
```
┌─────────────────────────────────────────────────────────────┐
│                 网络能力检测流程                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Step 1: 检测系统代理配置                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ powershell -Command "Get-ItemProperty -Path           │  │
│  │   'HKCU:\...\Internet Settings' | Select ProxyServer" │  │
│  └───────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│  Step 2: 尝试WebFetch                                       │
│          ├─ 成功 → 使用WebFetch                             │
│          └─ 失败 → Step 3                                   │
│                          ↓                                  │
│  Step 3: 尝试curl通过代理                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ curl -x http://PROXY:PORT -s "URL" -o output.html     │  │
│  └───────────────────────────────────────────────────────┘  │
│          ├─ 成功 → 解析HTML提取数据                         │
│          └─ 失败 → Step 4                                   │
│                          ↓                                  │
│  Step 4: 优雅降级                                           │
│          → 明确告知用户能力受限                             │
│          → 提供替代数据获取方案                             │
│          → 请求用户提供本地数据文件                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**代理环境下的数据获取**：
```bash
# 检测代理
powershell -Command "Get-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' | Select-Object ProxyEnable, ProxyServer"

# 通过代理获取网页
curl -x http://[PROXY_SERVER]:[PORT] -s "[TARGET_URL]" -o temp.html

# 解析HTML（使用Node.js）
node -e "const fs=require('fs');const html=fs.readFileSync('temp.html','utf8');const matches=[...html.matchAll(/PATTERN/g)];matches.forEach(m=>console.log(m[1]))"
```

### 1.2 参考文件访问验证

**问题**：参考文件可能不在预期路径或格式不可读

**必须执行的验证**：
```bash
# 1. 确认文件存在
ls -la "[REFERENCE_FILE_PATH]"

# 2. 确认文件可读
file "[REFERENCE_FILE_PATH]"  # 检查文件类型

# 3. 读取并验证内容
# .xlsx: 使用openpyxl或xlsx库
# .docx: 使用JSZip解压word/document.xml
# .pdf: 使用pdftotext或Read工具（支持PDF）
# .csv/.json: 直接读取
```

**文件读取模板**：

```python
# Python - 读取Excel
import openpyxl
wb = openpyxl.load_workbook('file.xlsx')
ws = wb.active
data = [[cell.value for cell in row] for row in ws.iter_rows()]
```

```javascript
// JavaScript - 读取DOCX
const JSZip = require('jszip');
const fs = require('fs');
const data = fs.readFileSync('file.docx');
JSZip.loadAsync(data).then(zip => {
    zip.file('word/document.xml').async('string').then(xml => {
        const text = xml.replace(/<[^>]+>/g, ' ').replace(/\s+/g, ' ');
        console.log(text);
    });
});
```

---

## 二、推理层增强：幻觉预防机制

### 2.1 数据幻觉的识别与预防

**核心原则**：**宁可承认不知道，也不要捏造数据**

**幻觉高发场景**：
| 场景 | 幻觉表现 | 正确做法 |
|------|----------|----------|
| 无法获取网络数据 | 编造具体数值（如"P/E为31.2x"） | 明确说明无法获取，提供替代方案 |
| 参考文件缺失 | 假设文件内容并生成分析 | 报告文件缺失，请求用户提供 |
| 无法验证实体 | 使用不可验证的名称/链接 | 标注"无法独立验证"或提供可验证替代 |
| 数据来源模糊 | 输出数值但不标注来源 | 每个数据点必须关联来源 |

### 2.2 数据来源强制标注

**规则**：所有定量数据必须包含来源标注

```markdown
## 正确示例
根据 Population.xlsx 第15行（Row ID: AFC-015），Q2 2024 的 High Risk Customer Count 为 145。

## 错误示例
Q2 2024 的 High Risk Customer Count 为 145。（无来源）
```

**输出格式要求**：
```markdown
| 数据项 | 数值 | 来源 | 验证状态 |
|--------|------|------|----------|
| 样本量 | 13 | 计算公式 n=(Z²×p×(1-p))/E² | ✓可验证 |
| Q2客户数 | 145 | Population.xlsx:Row15:ColH | ✓可验证 |
| P/E倍数 | N/A | 网络获取失败 | ✗待补充 |
```

### 2.3 约束条件理解验证

**在处理任务约束前，必须**：
1. 提取并列出所有约束条件
2. 用自己的话复述约束含义
3. 标注任何可能的歧义
4. 如有不确定，询问用户确认

---

## 三、行动层增强：优雅降级与透明报告

### 3.1 能力状态报告模板

**任务执行前，必须输出**：
```markdown
## 能力状态检查

| 能力项 | 状态 | 备注 |
|--------|------|------|
| 网络数据获取 | ✓可用 / ⚠部分可用 / ✗不可用 | 使用curl+代理 / WebFetch失效 |
| 参考文件访问 | ✓已读取 / ✗未找到 | 列出已读取/缺失的文件 |
| 输出文件生成 | ✓可执行 | Excel/Word/PDF |
| 外部API访问 | ✓可用 / ✗不可用 | 具体说明 |

## 影响评估
- [能力受限项]: [对任务的具体影响] + [替代方案]
```

### 3.2 降级策略执行

**当核心能力不可用时**：

```markdown
## 能力受限通知

### 受限能力
WebFetch/WebSearch 功能在当前网络环境下不可用。

### 对任务的影响
无法自动获取 [具体数据源] 的实时数据。

### 替代方案
**方案A（推荐）**：用户手动访问以下URL并下载数据文件：
- [URL1] - [数据描述]
- [URL2] - [数据描述]

**方案B**：使用curl通过代理获取（需用户确认代理配置）：
```bash
curl -x http://127.0.0.1:10808 -s "[URL]" -o data.html
```

**方案C**：用户提供本地数据文件，我将继续完成分析。

### 已完成的工作
- [x] 输出模板/结构已创建
- [x] 公式/逻辑已实现
- [ ] 待：实际数据填充
```

### 3.3 部分完成报告

**当只能部分完成任务时**：
```markdown
## 任务完成状态

### 已完成 (✓)
1. [具体完成项] - 来源: [数据来源]
2. [具体完成项] - 来源: [数据来源]

### 未完成 (✗)
1. [未完成项] - 原因: [具体原因]
   → 后续操作: [用户需要做什么]

### 可验证性声明
本输出中的所有数据可通过以下方式验证：
- [验证方法1]
- [验证方法2]
```

---

## 四、验证层增强：输出可审计性

### 4.1 输出可追溯性要求

**每个输出文件必须包含**：
1. **元数据块**：创建时间、输入文件列表、能力状态
2. **数据血缘**：每个关键数据点的来源链
3. **假设声明**：所有隐含假设必须明确列出

### 4.2 自我验证清单

**输出前必须检查**：
- [ ] 所有数据点都有明确来源
- [ ] 没有无来源的具体数值
- [ ] 能力受限已明确告知
- [ ] 输出格式符合任务要求
- [ ] 假设和约束理解已验证

---

## 五、通用代码模板

### 5.1 代理网络获取模块
```javascript
// proxy_fetch.js - 通过代理获取网页内容
const { execSync } = require('child_process');
const fs = require('fs');

function detectProxy() {
    try {
        const result = execSync(
            'powershell -Command "Get-ItemProperty -Path \'HKCU:\\Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings\' | Select-Object -ExpandProperty ProxyServer"',
            { encoding: 'utf8' }
        ).trim();
        return result || null;
    } catch {
        return null;
    }
}

function fetchViaProxy(url, outputFile, proxy = null) {
    const proxyArg = proxy ? `-x ${proxy}` : '';
    try {
        execSync(`curl ${proxyArg} -s "${url}" -o "${outputFile}"`, { timeout: 30000 });
        return fs.readFileSync(outputFile, 'utf8');
    } catch (error) {
        console.error(`获取失败: ${url} - ${error.message}`);
        return null;
    }
}

module.exports = { detectProxy, fetchViaProxy };
```

### 5.2 数据验证模块
```python
# data_validator.py - 数据完整性验证
import json
from typing import Dict, List, Any

class DataValidator:
    def __init__(self):
        self.sources = {}
        self.missing = []

    def register_source(self, data_key: str, source: str, value: Any):
        """注册数据来源"""
        self.sources[data_key] = {
            'source': source,
            'value': value,
            'verified': value is not None
        }
        if value is None:
            self.missing.append(data_key)

    def generate_report(self) -> str:
        """生成数据验证报告"""
        report = "## 数据来源报告\n\n"
        report += "| 数据项 | 来源 | 状态 |\n|--------|------|------|\n"
        for key, info in self.sources.items():
            status = "✓" if info['verified'] else "✗"
            report += f"| {key} | {info['source']} | {status} |\n"

        if self.missing:
            report += f"\n**缺失数据**: {', '.join(self.missing)}\n"

        return report
```

### 5.3 能力检测模块
```python
# capability_checker.py - 执行能力检测
import subprocess
import os

def check_capabilities():
    """检测当前执行环境的能力"""
    capabilities = {
        'network': {'status': 'unknown', 'method': None},
        'files': {'status': 'unknown', 'accessible': []},
        'tools': {'status': 'unknown', 'available': []}
    }

    # 检测代理
    try:
        result = subprocess.run(
            ['powershell', '-Command',
             "Get-ItemProperty -Path 'HKCU:\\Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings' | Select-Object -ExpandProperty ProxyServer"],
            capture_output=True, text=True, timeout=5
        )
        if result.stdout.strip():
            capabilities['network']['status'] = 'proxy_available'
            capabilities['network']['method'] = f"curl -x {result.stdout.strip()}"
    except:
        capabilities['network']['status'] = 'check_failed'

    return capabilities

def report_capabilities(caps: dict, required_files: list = None):
    """生成能力状态报告"""
    print("## 能力状态检查\n")
    print(f"网络访问: {caps['network']['status']}")
    if caps['network']['method']:
        print(f"  方法: {caps['network']['method']}")

    if required_files:
        print("\n参考文件:")
        for f in required_files:
            exists = os.path.exists(f)
            print(f"  {'✓' if exists else '✗'} {f}")
```

---

## 六、执行检查清单

### 任务开始前
- [ ] 执行能力检测（网络、文件、工具）
- [ ] 确认所有参考文件可访问
- [ ] 理解并复述任务约束
- [ ] 识别可能的数据来源

### 任务执行中
- [ ] 每个数据点标注来源
- [ ] 能力受限时立即报告
- [ ] 不捏造无来源的数据
- [ ] 记录所有假设

### 任务完成后
- [ ] 输出数据来源报告
- [ ] 标注完成/未完成项
- [ ] 提供验证方法
- [ ] 说明后续所需操作

---

## 七、输出格式规范（评分系统兼容）

### 7.1 双重输出要求（关键）

**GDPVAL任务有两类必须输出的文件：**

```
┌─────────────────────────────────────────────────────────────┐
│                    输出文件要求                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  A. Prompt要求的文件（最高优先级）                          │
│     ────────────────────────────────────                    │
│     根据prompt明确要求的格式生成：                          │
│     - Word文档 (.docx) → 使用docx skill                    │
│     - Excel文件 (.xlsx) → 使用xlsx skill                   │
│     - PDF图表 (.pdf) → HTML转换或专用工具                  │
│                                                             │
│  B. 评分系统文件（必须）                                    │
│     ────────────────────                                    │
│     - claude_output.md → 评分程序读取的主文件               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 完整输出文件结构

```
runs/task_X/
├── prompt.txt                    # 任务原始prompt（通常已存在）
├── claude_output.md              # 【必须】评分程序主输入文件
│
├── [Prompt要求的文件]            # 【必须】按prompt要求生成
│   ├── xxx.docx                  # Word文档（如prompt要求）
│   ├── xxx.xlsx                  # Excel文件（如prompt要求）
│   └── xxx.pdf                   # PDF文件（如prompt要求）
│
└── [辅助文件]                    # 可选
    ├── *.html                    # 中间文件
    └── data/                     # 获取的数据
```

### 7.3 claude_output.md 文件结构

**此文件是评分程序的输入，必须包含完整任务执行结果，并引用生成的其他文件。**

```markdown
# [任务标题]

## Task Metadata
- Task ID: [task_id]
- Sector: [sector]
- Occupation: [occupation]
- Execution Date: [date]

---

## Executive Summary
[简要概述任务完成情况，1-2段]

---

## Capability Status
| Capability | Status | Notes |
|------------|--------|-------|
| Network Access | ✓/⚠/✗ | [具体说明] |
| Reference Files | ✓/✗ | [列出文件状态] |
| Output Generation | ✓/✗ | [说明] |

---

## Deliverables Generated（关键部分）

### 已生成的文件列表
| 文件名 | 格式 | 对应Prompt要求 | 状态 |
|--------|------|----------------|------|
| Proposed_Architecture_Summary.docx | .docx | "Word document...architecture summary" | ✓已生成 |
| Proposed_Architecture_Diagram.pdf | .pdf | "PDF diagram...architecture" | ✓已生成 |
| POC_Implementation_Guide.docx | .docx | "Word document...POC" | ✓已生成 |

### 交付物内容摘要

#### 1. [交付物名称] (filename.docx)
[内容摘要或嵌入关键内容...]

#### 2. [交付物名称] (filename.pdf)
[内容描述...]

---

## Data Sources & Traceability
| Data Item | Source | Verification |
|-----------|--------|--------------|
| [数据项] | [来源] | ✓/✗ |

---

## Assumptions (if any)
[列出所有假设]

---

## Completion Status
- [x] [已完成项]
- [ ] [未完成项] - Reason: [原因]

---

*Generated with GDPVAL Skill v1.2.0*
```

### 7.4 Prompt要求 vs 评分文件的关系

```
┌─────────────────────────────────────────────────────────────┐
│                    文件关系说明                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Prompt要求:                                                │
│  "Create a Word document outlining..."                      │
│  "A PDF diagram representing..."                            │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │ Summary.docx    │  │ Diagram.pdf     │                  │
│  │ (实际Word文件)  │  │ (实际PDF文件)   │                  │
│  └────────┬────────┘  └────────┬────────┘                  │
│           │                    │                            │
│           └────────┬───────────┘                            │
│                    │                                        │
│                    ▼                                        │
│  ┌──────────────────────────────────────────────┐          │
│  │           claude_output.md                    │          │
│  │  ┌────────────────────────────────────────┐  │          │
│  │  │ ## Deliverables Generated              │  │          │
│  │  │ - Summary.docx ✓                       │  │          │
│  │  │ - Diagram.pdf ✓                        │  │          │
│  │  │                                        │  │          │
│  │  │ ## Content Summary                     │  │          │
│  │  │ [Key content from deliverables]        │  │          │
│  │  └────────────────────────────────────────┘  │          │
│  └──────────────────────────────────────────────┘          │
│                    │                                        │
│                    ▼                                        │
│           评分程序读取并评估                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 7.5 执行检查清单（完成前必须确认）

**A. Prompt要求检查：**
- [ ] 所有prompt要求的文件格式都已生成（.docx, .xlsx, .pdf等）
- [ ] 文件内容符合prompt的具体要求
- [ ] 如prompt要求"mirror the style"，则已参照样式

**B. 评分文件检查：**
- [ ] claude_output.md 文件已创建
- [ ] Deliverables Generated 部分列出所有生成的文件
- [ ] 关键内容已嵌入或摘要在claude_output.md中
- [ ] Data Sources表格已填写
- [ ] Completion Status准确反映完成情况

### 7.6 评分维度对照

评分程序使用以下四个维度评估（0-5分）：

| 评分维度 | 说明 | 关键对应项 |
|----------|------|------------|
| **Task Correctness** | 是否正确完成任务要求 | Prompt要求的文件是否按格式生成 |
| **Completeness** | 是否完成所有要求组件 | Deliverables Generated列表是否完整 |
| **Professional Quality** | 是否达到专业标准 | 文档质量、数据来源标注、格式规范 |
| **Clarity and Structure** | 是否清晰有组织 | claude_output.md结构、标题层次 |

---

## 版本信息

- Skill版本: 1.2.0
- 创建日期: 2025-12-18
- 更新日期: 2025-12-18
- 适用范围: GDPVAL基准测试任务
- 核心能力:
  - **Prompt需求严格执行** (新增)
  - **Skill协同调用** (新增: docx, xlsx, pdf)
  - 数据获取增强
  - 幻觉预防
  - 优雅降级
  - 可追溯输出
  - 评分系统兼容输出
