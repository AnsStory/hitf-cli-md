# hitf

English | [简体中文](./README.md)

CLI i18n toolkit that extracts Chinese text from Vue/React/uni-app source files, translates via Baidu/Tencent/OpenAI/Google APIs, and generates locale files. Also includes project scaffolding, translation cache management, and IDE code snippet generation.

## Install

```bash
npm install -g @ansstory/hitf
```

## Commands

| Command | Alias | Description |
|---------|-------|-------------|
| `hitf create <project>` | `crt` | Scaffold a project from a Git template |
| `hitf tf <file> [name]` | | Translate a single file and generate locale JSON |
| `hitf tfo <folder> [name]` | | Recursively translate an entire folder |
| `hitf tcheck [target]` | | Check i18n references against locale files |
| `hitf export-source <file>` | `tes` | Extract unique source texts from code and export to XLSX |
| `hitf export-translated [file]` | `etr` | Read texts from translated locale JSON and export to XLSX |
| `hitf cache-import <file>` | `tci` | Import a TSV/XLSX translation table into local cache |
| `hitf cache-apply [name]` | `tca` | Write cached translations back to locale JSON |
| `hitf rollback` | | Undo the last translation operation |
| `hitf setting` | | Generate a project-level config file |
| `hitf global-config` | | Generate a global config file |
| `hitf lang [language]` | | Switch CLI interface language (zh-CN / en) |
| `hitf codesnippets [ide]` | | Generate IDE i18n-ignore comment snippets |
| `hitf gitignore` | | Add `.hitf/services.json` to `.gitignore` |

## Common Options

| Option | Short | Description |
|--------|-------|-------------|
| `--name <name>` | `-n` | i18n namespace key (defaults to parent directory name) |
| `--project <project>` | `-p` | Specify project when outDir is an object |
| `--dry-run` | `-d` | List extracted Chinese texts with locations without translating |
| `--interactive` | `-i` | Preview changes and ask whether to apply them |
| `--report <file>` | `-r` | Output JSON report file |
| `--verbose` | `-v` | Verbose output with individual file logs |

## Translation Workflow

### Translate a single file

```bash
hitf tf src/views/Home.vue
```

- Extracts Chinese text from `<template>` and `<script>` blocks
- Calls the translation API for target languages
- Generates locale JSON under `.hitf/lang/`
- Replaces Chinese in source with `$t('key')` calls

### Translate a folder

```bash
hitf tfo src --exclude "**/*.test.js"
```

Recursively processes all `.vue`, `.js`, `.ts`, `.jsx`, `.tsx`, `.json` files.

### Dry-run mode

```bash
hitf tf src/App.vue -d
hitf tfo src -d
```

`-d` / `--dry-run` lists all extracted Chinese texts with their source locations (file:line:column) without performing translation.

```
  suitable → src.suitable ~ src/App.vue:2:21
  test → src.test ~ src/App.vue:3:21
```

### Interactive confirmation

```bash
hitf tf src/App.vue -i
```

`-i` / `--interactive` prompts per file to confirm whether to apply translations.

### Check consistency

```bash
hitf tcheck src
```

Checks i18n references in source code against locale files, showing missing/unused/inconsistent keys with locations.

```
Translation check summary
  missing: 2
  missing hello
    src/App.vue:2:11
  missing welcome
    src/App.vue:3:9
```

### Rollback

```bash
hitf rollback -l          # List available backups
hitf rollback              # Rollback the latest
hitf rollback -n xxx       # Rollback a specific backup
hitf rollback -k 5         # Keep only 5 most recent backups
```

## Translation Cache Management

### Export source texts

```bash
hitf export-source src/views      # Extract from source code
hitf export-translated            # Read from translated locale JSONs
```

Both commands share the same output file `source-texts-<locale>.xlsx` with incremental merge and deduplication. They automatically fill all locale columns based on `locales` configuration. Missing translations are attempted via translation API (left empty if API is not configured). The output format is compatible with `cache-import`.

### Import company translation table

```bash
hitf cache-import translations.xlsx -s zh-CN
```

Supports `.xlsx` format. Imported translations are cached locally and reused without API calls.

### Apply cache to locale files

```bash
hitf cache-apply
```

Writes the latest cached translations back to locale JSON files under `.hitf/lang/`.

## Configuration & Setup

### Global configuration

```bash
hitf global-config
```

Generates a global config file for setting default translation services, API keys, etc.

### Project configuration

```bash
hitf setting
```

Generates `.hitf/setting.json` at project root:

```json
{
  "$schema": "https://www.schemastore.org/hias-hitf.json",
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

### Options

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `locales` | `string[]` | `["zh-CN", "en-US"]` | Language list; first entry is the source language |
| `outDir` | `string \| object` | `".hitf/lang"` | Locale file output directory; supports per-project mapping |
| `i18nCallTemplate` | `string \| string[]` | `"$t"` | i18n function call template |
| `i18nImport` | `string \| string[]` | `""` | Import statement injected at the top of files |
| `translationService` | `string` | `"tencent"` | Primary translation provider |
| `servicePriority` | `string[]` | `["tencent","baidu","openai","google"]` | Fallback order for translation providers |
| `capitalizeTranslations` | `boolean` | `false` | Capitalize first letter of translations |
| `pruneUnusedKeys` | `boolean` | `false` | Remove unused keys from locale files |
| `replaceOriginalFile` | `boolean` | `false` | Overwrite original files instead of outputting to outDir |

### Translation Providers

#### Tencent Translate

**Setup Steps:**

1. Open [Tencent Cloud Console](https://console.cloud.tencent.com/cam/capi) and log in
2. Click【Create Key】to generate a SecretId and SecretKey pair
3. Copy and paste them into the configuration

```json
{
  "tencent": {
    "secretId": "YOUR_SECRET_ID",
    "secretKey": "YOUR_SECRET_KEY",
    "region": "ap-guangzhou"
  }
}
```

**Pricing:** 5 million characters free per month, then ¥58/million characters.

#### Baidu Translate

**Setup Steps:**

1. Open [Baidu Translate Open Platform](https://fanyi-api.baidu.com/) and log in with your Baidu account
2. Click【Console】and register as a developer (select "Individual Developer")
3. Enable "General Translation Service" and choose the Standard plan
4. Get your APP ID and Secret Key from the bottom of the console

```json
{
  "baidu": {
    "appId": "YOUR_APP_ID",
    "secretKey": "YOUR_SECRET_KEY"
  }
}
```

**Pricing:** Standard plan: 50,000 free characters/month. After identity verification, upgrade to Advanced: 1 million free characters/month. Overage: ¥49/million characters.

#### OpenAI

**Setup Steps:**

1. Visit [OpenAI Platform](https://platform.openai.com/) and create an account
2. Go to [API Keys page](https://platform.openai.com/account/api-keys)
3. Click【Create new secret key】
4. Copy the key into the configuration

> Note: OpenAI requires a network proxy from mainland China. No free tier available; a foreign credit card is required.

```json
{
  "openai": {
    "apiKey": "YOUR_API_KEY",
    "baseUrl": "https://api.openai.com",
    "model": "gpt-3.5-turbo"
  }
}
```

**Recommended Models & Pricing:**

| Model | Input Price | Output Price |
|-------|-------------|--------------|
| gpt-4o-mini | $0.15/M tokens | $0.60/M tokens |
| gpt-4o | $2.50/M tokens | $10.00/M tokens |

#### Google Translate

**Setup Steps:**

1. Open [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable Cloud Translation API
4. Create an API Key and copy it

```json
{
  "google": {
    "apiKey": "YOUR_API_KEY"
  }
}
```

**Pricing:** 500,000 free characters per month, then $20/million characters.

### IDE Code Snippets

```bash
hitf codesnippets        # auto-detect IDE
hitf codesnippets vscode # manual selection
```

Supports VS Code, WebStorm, Cursor, and Sublime Text.

## Ignore Comments

Add comments in source to skip specific text extraction:

```vue
<!-- @hitf-i18n-ignore-next -->
<div>{{ brandName }}</div>

<!-- @hitf-i18n-ignore-start -->
<div>
  <span>skipped</span>
  <span>also skipped</span>
</div>
<!-- @hitf-i18n-ignore-end -->
```

```js
// @hitf-i18n-ignore-file
const msg = 'entire file is skipped'
```

## CLI Language

```bash
hitf lang -l         # show current language
hitf lang zh-CN      # switch to Chinese
hitf lang en         # switch to English
```

## License

[MIT](./LICENSE)
