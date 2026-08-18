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
| `hitf cache-import <file>` | `tci` | Import a TSV/XLSX translation table into local cache |
| `hitf cache-apply [name]` | `tca` | Write cached translations back to locale JSON |
| `hitf rollback` | | Undo the last translation operation |
| `hitf setting` | | Generate a project-level config file |
| `hitf global-config` | | Generate a global config file |
| `hitf lang [language]` | | Switch CLI interface language (zh-CN / en) |
| `hitf codesnippets [ide]` | | Generate IDE i18n-ignore comment snippets |
| `hitf gitignore` | | Add `.hitf` to `.gitignore` |

## Common Options

| Option | Short | Description |
|--------|-------|-------------|
| `--name <name>` | `-n` | i18n namespace key (defaults to parent directory name) |
| `--project <project>` | `-p` | Specify project when outDir is an object |
| `--dry-run` | `-d` | List extracted Chinese texts with locations without translating |
| `--interactive` | `-i` | Preview changes and ask whether to apply them |
| `--report <file>` | `-r` | Output JSON report file |
| `--verbose` | `-v` | Verbose output with individual file logs |

## Quick Start

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

## Configuration

Run `hitf setting` to generate `.hitf/setting.json` at project root:

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

```json
{
  "tencent": {
    "secretId": "YOUR_SECRET_ID",
    "secretKey": "YOUR_SECRET_KEY",
    "region": "ap-guangzhou"
  }
}
```

#### Baidu Translate

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

## Translation Cache

### Import company translation table

```bash
hitf cache-import translations.xlsx -s zh-CN
```

Supports `.tsv` and `.xlsx` formats. Imported translations are cached locally and reused without API calls.

### Apply cache to locale files

```bash
hitf cache-apply
```

Writes the latest cached translations back to locale JSON files under `.hitf/lang/`.

### Rollback

```bash
hitf rollback -l          # List available backups
hitf rollback              # Rollback the latest
hitf rollback -n xxx       # Rollback a specific backup
hitf rollback -k 5         # Keep only 5 most recent backups
```

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

Generate IDE code snippets for these comments:

```bash
hitf codesnippets        # auto-detect IDE
hitf codesnippets vscode # manual selection
```

Supports VS Code, WebStorm, Cursor, and Sublime Text.

## CLI Language

```bash
hitf lang -l         # show current language
hitf lang zh-CN      # switch to Chinese
hitf lang en         # switch to English
```

## License

[MIT](./LICENSE)
