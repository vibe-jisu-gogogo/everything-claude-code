---
name: nutrient-document-processing
description: 使用 Nutrient DWS API 进行文档处理、转换、OCR、提取、编辑、签名、表单填写。支持 PDF、DOCX、XLSX、PPTX、HTML、图像。
---

# Nutrient Document Processing

使用 [Nutrient DWS Processor API](https://www.nutrient.io/api/) 处理文档。支持格式转换、文本和表格提取、扫描文档的 OCR、PII 编辑、添加水印、数字签名、PDF 表单填写。

## 安装

请在 **[nutrient.io](https://dashboard.nutrient.io/sign_up/?product=processor)** 获取免费 API Key

```bash
export NUTRIENT_API_KEY="pdf_live_..."
```

所有请求都作为 multipart POST 发送到 `https://api.nutrient.io/build`，包含 `instructions` JSON 字段。

## 操作

### 文档转换

```bash
# DOCX 到 PDF
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.docx=@document.docx" \
  -F 'instructions={"parts":[{"file":"document.docx"}]}' \
  -o output.pdf

# PDF 到 DOCX
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"docx"}}' \
  -o output.docx

# HTML 到 PDF
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "index.html=@index.html" \
  -F 'instructions={"parts":[{"html":"index.html"}]}' \
  -o output.pdf
```

支持的输入格式: PDF、DOCX、XLSX、PPTX、DOC、XLS、PPT、PPS、PPSX、ODT、RTF、HTML、JPG、PNG、TIFF、HEIC、GIF、WebP、SVG、TGA、EPS。

### 文本和数据提取

```bash
# 提取纯文本
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"text"}}' \
  -o output.txt

# 将表格提取为 Excel
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"xlsx"}}' \
  -o tables.xlsx
```

### 扫描文档的 OCR

```bash
# OCR 转为可搜索 PDF（支持 100 多种语言）
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "scanned.pdf=@scanned.pdf" \
  -F 'instructions={"parts":[{"file":"scanned.pdf"}],"actions":[{"type":"ocr","language":"english"}]}' \
  -o searchable.pdf
```

语言: 通过 ISO 639-2 代码（例如 `eng`、`deu`、`fra`、`spa`、`jpn`、`kor`、`chi_sim`、`chi_tra`、`ara`、`hin`、`rus`）支持 100 多种语言。`english` 或 `german` 等完整语言名称也适用。请参阅 [完整 OCR 语言表](https://www.nutrient.io/guides/document-engine/ocr/language-support/) 了解所有支持的代码。

### 编辑机密信息

```bash
# 基于预设模式（SSN、邮件）
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"redaction","strategy":"preset","strategyOptions":{"preset":"social-security-number"}},{"type":"redaction","strategy":"preset","strategyOptions":{"preset":"email-address"}}]}' \
  -o redacted.pdf

# 基于正则表达式
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"redaction","strategy":"regex","strategyOptions":{"regex":"\\b[A-Z]{2}\\d{6}\\b"}}]}' \
  -o redacted.pdf
```

预设: `social-security-number`、`email-address`、`credit-card-number`、`international-phone-number`、`north-american-phone-number`、`date`、`time`、`url`、`ipv4`、`ipv6`、`mac-address`、`us-zip-code`、`vin`。

### 添加水印

```bash
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"watermark","text":"CONFIDENTIAL","fontSize":72,"opacity":0.3,"rotation":-45}]}' \
  -o watermarked.pdf
```

### 数字签名

```bash
# 自签名 CMS 签名
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"sign","signatureType":"cms"}]}' \
  -o signed.pdf
```

### 填写 PDF 表单

```bash
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "form.pdf=@form.pdf" \
  -F 'instructions={"parts":[{"file":"form.pdf"}],"actions":[{"type":"fillForm","formFields":{"name":"Jane Smith","email":"jane@example.com","date":"2026-02-06"}}]}' \
  -o filled.pdf
```

## MCP Server（替代方案）

对于原生工具集成，请使用 MCP Server 代替 curl：

```json
{
  "mcpServers": {
    "nutrient-dws": {
      "command": "npx",
      "args": ["-y", "@nutrient-sdk/dws-mcp-server"],
      "env": {
        "NUTRIENT_DWS_API_KEY": "YOUR_API_KEY",
        "SANDBOX_PATH": "/path/to/working/directory"
      }
    }
  }
}
```

## 使用时机

- 格式间的文档转换（PDF、DOCX、XLSX、PPTX、HTML、图像）
- 从 PDF 提取文本、表格、键值对
- 扫描文档或图像的 OCR
- 共享文档前的 PII 编辑
- 为草稿或机密文档添加水印
- 为合同或协议书添加数字签名
- 程序化填写 PDF 表单

## 链接

- [API Playground](https://dashboard.nutrient.io/processor-api/playground/)
- [完整 API 文档](https://www.nutrient.io/guides/dws-processor/)
- [npm MCP Server](https://www.npmjs.com/package/@nutrient-sdk/dws-mcp-server)
