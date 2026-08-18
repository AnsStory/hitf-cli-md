# hitf

[English](./README.en-US.md) | 简体中文

CLI 国际化工具链，支持从 Vue/React/uni-app 源码中自动提取中文文本，通过百度/腾讯/OpenAI/Google 翻译 API 生成多语言文件，并提供项目脚手架、翻译缓存管理、IDE 代码片段生成等功能。

## 安装

```bash
npm install -g @ansstory/hitf
```

## 命令一览

| 命令 | 简写 | 说明 |
|------|------|------|
| `hitf create <project>` | `crt` | 从 Git 仓库下载项目模板 |
| `hitf tf <file> [name]` | | 翻译单个文件，提取中文并生成语言包 |
| `hitf tfo <folder> [name]` | | 递归翻译整个文件夹 |
| `hitf tcheck [target]` | | 检查 i18n 引用与语言文件的一致性 |
| `hitf cache-import <file>` | `tci` | 导入公司 TSV/XLSX 翻译表到本地缓存 |
| `hitf cache-apply [name]` | `tca` | 将缓存译文回写到语言包 JSON |
| `hitf rollback` | | 回滚上一次翻译操作 |
| `hitf setting` | | 生成项目级配置文件 |
| `hitf global-config` | | 生成全局配置文件 |
| `hitf lang [language]` | | 切换 CLI 界面语言（zh-CN / en） |
| `hitf codesnippets [ide]` | | 生成 IDE i18n 忽略注释片段 |
| `hitf gitignore` | | 将 `.hitf` 添加到 `.gitignore` |

## 通用参数

| 参数 | 简写 | 说明 |
|------|------|------|
| `--name <name>` | `-n` | 国际化对象 key（默认使用父级目录名） |
| `--project <project>` | `-p` | 当 outDir 为对象时指定项目 |
| `--dry-run` | `-d` | 列出所有提取的中文文本及其位置，不执行翻译 |
| `--interactive` | `-i` | 预览变更并询问是否应用 |
| `--report <file>` | `-r` | 输出 JSON 报告文件 |
| `--verbose` | `-v` | 详细输出，显示每个文件的日志 |

## 快速开始

### 翻译单个文件

```bash
hitf tf src/views/Home.vue
```

- 自动提取 `<template>`、`<script>` 中的中文文本
- 调用翻译 API 生成目标语言
- 在 `.hitf/lang/` 下生成语言包 JSON
- 将源码中的中文替换为 `$t('key')` 调用

### 翻译整个文件夹

```bash
hitf tfo src --exclude "**/*.test.js"
```

递归处理目录下所有 `.vue`、`.js`、`.ts`、`.jsx`、`.tsx`、`.json` 文件。

### 预览模式

```bash
hitf tf src/App.vue -d
hitf tfo src -d
```

`-d` / `--dry-run` 列出所有提取的中文文本及其在源码中的位置（文件:行:列），不执行翻译。

```
  合适 → src.suitable ~ src/App.vue:2:21
  测试 → src.test ~ src/App.vue:3:21
```

### 交互式确认

```bash
hitf tf src/App.vue -i
```

`-i` / `--interactive` 逐文件确认是否应用翻译。

### 检查一致性

```bash
hitf tcheck src
```

检查源码中的 i18n 引用与语言包文件的一致性，输出缺失/未使用/不一致的 key 及位置。

```
Translation check summary
  missing: 2
  missing hello
    src/App.vue:2:11
  missing welcome
    src/App.vue:3:9
```

## 配置

运行 `hitf setting` 在项目根目录生成 `.hitf/setting.json`：

```json
{
  "$schema": "https://ansstory.github.io/hitf-cli-md/schema_setting.json",
  "translationSetting": {
    "locales": ["zh-CN", "en-US"],
    "outDir": ".hitf/lang",
    "i18nCallTemplate": "$t",
    "i18nImport": "import i18n from '@/i18n'",
    "translationService": "tencent",
    "servicePriority": ["tencent", "baidu", "openai", "google"],
    "services": {
      "tencent": { "secretId": "", "secretKey": "", "region": "ap-guangzhou" },
      "baidu": { "appId": "", "secretKey": "" },
      "openai": { "apiKey": "", "baseUrl": "https://api.openai.com", "model": "gpt-3.5-turbo" },
      "google": { "apiKey": "" }
    }
  }
}
```

### 配置项说明

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `locales` | `string[]` | `["zh-CN", "en-US"]` | 语言列表，第一个为源语言 |
| `outDir` | `string \| object` | `".hitf/lang"` | 语言包输出目录，支持按项目名映射 |
| `i18nCallTemplate` | `string \| string[]` | `"$t"` | i18n 函数调用模板 |
| `i18nImport` | `string \| string[]` | `""` | 注入到文件顶部的 import 语句 |
| `translationService` | `string` | `"tencent"` | 首选翻译服务 |
| `servicePriority` | `string[]` | `["tencent","baidu","openai","google"]` | 翻译服务降级优先级 |
| `capitalizeTranslations` | `boolean` | `false` | 译文首字母大写 |
| `pruneUnusedKeys` | `boolean` | `false` | 清理语言包中未使用的 key |
| `replaceOriginalFile` | `boolean` | `false` | 覆盖原文件而非输出到 outDir |

### 翻译服务配置

#### 腾讯翻译

```json
{
  "tencent": {
    "secretId": "YOUR_SECRET_ID",
    "secretKey": "YOUR_SECRET_KEY",
    "region": "ap-guangzhou"
  }
}
```

#### 百度翻译

```json
{
  "baidu": {
    "appId": "YOUR_APP_ID",
    "secretKey": "YOUR_SECRET_KEY"
  }
}
```

#### OpenAI

```json
{
  "openai": {
    "apiKey": "YOUR_API_KEY",
    "baseUrl": "https://api.openai.com",
    "model": "gpt-3.5-turbo"
  }
}
```

#### Google Translate

```json
{
  "google": {
    "apiKey": "YOUR_API_KEY"
  }
}
```

## 翻译缓存

### 导入公司翻译表

```bash
hitf cache-import translations.xlsx -s zh-CN
```

支持 `.tsv` 和 `.xlsx` 格式，导入后本地缓存可直接复用，无需重复调用 API。

### 应用缓存到语言包

```bash
hitf cache-apply
```

将缓存中的最新译文写回 `.hitf/lang/` 下的语言包 JSON 文件。

### 回滚操作

```bash
hitf rollback -l          # 查看可用备份
hitf rollback              # 回滚最近一次
hitf rollback -n xxx       # 回滚指定备份
hitf rollback -k 5         # 仅保留最近 5 个备份
```

## 忽略注释

在源码中添加注释可跳过特定文本的提取：

```vue
<!-- @hitf-i18n-ignore-next -->
<div>{{ brandName }}</div>

<!-- @hitf-i18n-ignore-start -->
<div>
  <span>不翻译</span>
  <span>也不翻译</span>
</div>
<!-- @hitf-i18n-ignore-end -->
```

```js
// @hitf-i18n-ignore-file
const msg = '整个文件跳过翻译'
```

生成 IDE 代码片段：

```bash
hitf codesnippets        # 自动检测 IDE
hitf codesnippets vscode # 手动指定
```

支持 VS Code、WebStorm、Cursor、Sublime Text。

## CLI 语言切换

```bash
hitf lang -l         # 查看当前语言
hitf lang zh-CN      # 切换为中文
hitf lang en         # 切换为英文
```

## License

[MIT](./LICENSE)
