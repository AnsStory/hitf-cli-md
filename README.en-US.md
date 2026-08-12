# hias-cli Operation Manual

[![npm version](https://img.shields.io/npm/v/@ansstory/hias.svg)](https://www.npmjs.com/package/@ansstory/hias)
[![license](https://img.shields.io/npm/l/@ansstory/hias.svg)](https://github.com/ansstory/hias-cli/blob/main/LICENSE)

`hias-cli` is a frontend command-line toolkit for project scaffolding, Vue/React/Redux file generation, killing occupied ports, switching CLI language, and migrating Vue, React, JS, TS, JSX, TSX, and JSON source files to i18n.

This document is an operation manual. It is organized by real usage flow: install, configure, translate, verify, roll back, and maintain.

## Requirements

- Node.js `>= 18`
- Windows, macOS, and Linux are supported
- Translation supports Baidu Translate and Tencent Cloud TMT
- API credentials are optional when fallback mode is acceptable

## Install

```sh
npm i @ansstory/hias -g
```

Verify installation:

```sh
hias -v
hias --help
```

Clean global config only:

```sh
hias clear-config
```

## Command Overview

| Command                 | Alias | Purpose                                      |
| ----------------------- | ----- | -------------------------------------------- |
| `create <project>`      | `crt` | Create a project from templates              |
| `adv <name>`            | -     | Generate a Vue component                     |
| `adr <name>`            | -     | Generate a React JSX component               |
| `adrt <name>`           | -     | Generate a React TSX component               |
| `adrd <name>`           | -     | Generate a Redux JSX store                   |
| `adrdt <name>`          | -     | Generate a Redux TSX store                   |
| `tf [file] [name]`      | -     | Translate one file                           |
| `tfo [folder] [name]`   | -     | Translate files in a folder                  |
| `tcheck [target]`       | -     | Check source references and locale files     |
| `cache-import [file]`   | `tci` | Import XLSX/TSV/TXT translations into cache  |
| `cache-apply [name]`    | `tca` | Apply cached translations to locale JSON     |
| `codesnippets [ide]`    | -     | Generate IDE code snippets                   |
| `setting`               | -     | Create project translation config            |
| `global-config`         | -     | Create global translation config             |
| `rollback`              | -     | Roll back translation output or source files |
| `gitignore`             | -     | Add `.hias` to `.gitignore`                  |
| `close-port <ports...>` | -     | Kill processes occupying ports               |
| `lang [language]`       | -     | Set or view CLI display language             |

## Recommended Workflow

For the first migration in a project:

```sh
cd your-project
hias setting
hias gitignore
hias tfo src/views views --show-extractions
hias tfo src/views views --dry-run --report .hias/reports/views.preview.json
hias tfo src/views views --report .hias/reports/views.write.json
hias tcheck src --report .hias/reports/tcheck.json
```

If the result is not what you expected:

```sh
hias rollback
```

## Create A Project

```sh
hias create my-app
```

The command opens an interactive prompt and downloads the selected template into `my-app`.

Available templates:

| Template     | Description                 |
| ------------ | --------------------------- |
| `vue2`       | Vue 2 template              |
| `vue3`       | Vue 3 template              |
| `vue-ts`     | Vue 3 + TypeScript template |
| `uni-vite`   | uni-app + Vite template     |
| `nuxt-web`   | Nuxt.js web template        |
| `vue-screen` | Vue data screen template    |
| `react`      | React JSX template          |
| `react-ts`   | React TypeScript template   |

## Generate Components And Stores

Generate into `src/components`:

```sh
hias adv Demo
hias adr Demo
hias adrt Demo
```

Specify a target folder:

```sh
hias adv Demo -d src/views/user
hias adr Demo -d src/components
hias adrt Demo -d src/components
```

Generate Redux stores:

```sh
hias adrd useUserStore -d src/store/modules
hias adrdt useUserStore -d src/store/modules
```

When the name is `index`, the generated file is named after the parent directory:

```sh
hias adv index -d src/views/user
```

This generates a file like `src/views/user/user.vue`.

## Translation Config

Run this in the business project root:

```sh
hias setting
```

It creates:

```text
.hias/setting.json
```

Full example:

```json
{
  "translationSetting": {
    "locales": ["zh-CN", "en-US"],
    "outDir": ".hias/lang",
    "fallbackToKey": true,
    "replaceOriginalFile": false,
    "translationService": "tencent",
    "servicePriority": ["tencent", "baidu", "openai", "google"],
    "services": {
      "openai": {
        "apiKey": "",
        "baseUrl": "https://api.openai.com",
        "model": "gpt-3.5-turbo"
      },
      "google": {
        "apiKey": ""
      },
      "baidu": {
        "appId": "",
        "secretKey": ""
      },
      "tencent": {
        "secretId": "",
        "secretKey": "",
        "region": "ap-guangzhou"
      }
    },
    "i18nCallTemplate": "$t",
    "i18nImport": "",
    "extensions": [".vue", ".js", ".ts", ".jsx", ".tsx", ".json"],
    "capitalizeTranslations": false,
    "capitalizeMaxWords": 0,
    "pruneUnusedKeys": false,
    "keyStrategy": {
      "maxLength": 40,
      "collision": "number",
      "hashLength": 6
    }
  },
  "gitRepoSetting": {
    "gitRepoAutoUpdate": false,
    "gitRepoMode": "merge",
    "gitRepo": {}
  }
}
```

Field reference:

| Field                              | Default              | Description                                                                          |
| ---------------------------------- | -------------------- | ------------------------------------------------------------------------------------ |
| `locales`                          | `["zh-CN", "en-US"]` | `locales[0]` is the source locale; the rest are target locales                       |
| `outDir`                           | `.hias/lang`         | Output directory for locale files and translated copies (string or object format)    |
| `fallbackToKey`                    | `true`               | Fall back to original text when translation fails                                    |
| `replaceOriginalFile`              | `false`              | Replace source files in place                                                        |
| `translationSetting.translationService` | `tencent`       | Current translation service: `baidu`, `tencent`, `openai`, or `google`               |
| `translationSetting.servicePriority` | `["tencent", "baidu", "openai", "google"]` | Fallback priority |
| `translationSetting.services.openai.apiKey` | -             | OpenAI API key                                                                       |
| `translationSetting.services.openai.baseUrl` | `https://api.openai.com` | Custom API endpoint                                                         |
| `translationSetting.services.openai.model` | `gpt-3.5-turbo`   | Model name                                                                           |
| `translationSetting.services.google.apiKey` | -             | Google Cloud Translation API key                                                     |
| `translationSetting.services.baidu.appId` | -                | Baidu translation app ID                                                             |
| `translationSetting.services.baidu.secretKey` | -            | Baidu translation secret key                                                         |
| `translationSetting.services.tencent.secretId` | -            | Tencent Cloud secret ID                                                              |
| `translationSetting.services.tencent.secretKey` | -           | Tencent Cloud secret key                                                             |
| `translationSetting.services.tencent.region` | `ap-guangzhou` | Region                                                                              |
| `i18nCallTemplate`                 | `$t`                 | i18n call used during replacement, supports string or array format                   |
| `i18nImport`                       | `""`                 | Optional import injection; supports string or array format; works with Vue/JS/TS/JSX |
| `extensions`                       | `[]`                 | Custom extensions; empty means all supported types                                   |
| `capitalizeTranslations`           | `false`              | Capitalize Latin translation results                                                 |
| `capitalizeMaxWords`               | `0`                  | Title Case threshold; only applies when `capitalizeTranslations` is `true`. `0` disables it (first word only); when `>0` and an English translation has ≤ that many words, every word is capitalized (English target locales only) |
| `pruneUnusedKeys`                  | `false`              | Remove old keys not referenced by source in the current namespace                    |
| `keyStrategy.maxLength`            | `40`                 | Maximum generated key length                                                         |
| `keyStrategy.collision`            | `number`             | Key collision strategy: `number` or `hash`                                           |
| `keyStrategy.hashLength`           | `6`                  | Short hash length for the hash collision strategy                                    |
| `gitRepoSetting.gitRepoAutoUpdate` | `false`              | Auto-fetch remote template list on `create` (project default off, global default on) |
| `gitRepoSetting.gitRepoMode`       | `"merge"`            | Conflict strategy: `merge` (-new suffix) or `cover` (overwrite)                      |
| `gitRepoSetting.gitRepo`           | `{}`                 | Custom template repo map: template name → git URL                                    |

Config resolution order:

1. Project `.hias/setting.json`
2. Global `~/.hias-cli/config.json`
3. Built-in defaults

Create global config:

```sh
hias global-config
```

## Template Repository Management

Template lists used by `hias create` come from the `gitRepo` config. Resolution order:

1. Project `.hias/setting.json` → `gitRepoSetting.gitRepo` field
2. Global `~/.hias-cli/config.json` → `gitRepoSetting.gitRepo` field
3. Built-in hardcoded defaults (vue2, vue3, vue-ts, react, etc.)

All three levels are merged **field-by-field** (unlike translation config which uses full-object override), with the project level taking highest priority.

### Auto Update

```json
{
  "translationSetting": {},
  "gitRepoSetting": {
    "gitRepoAutoUpdate": true,
    "gitRepoMode": "merge"
  }
}
```

- `gitRepoSetting.gitRepoAutoUpdate`: Whether to fetch the latest remote template list on `create`
  - Global default: `true` — all projects benefit from updates
  - Project default: `false` — avoids unexpected network requests
- Remote template list URL: `https://ansstory.github.io/hias-cli-md/git-storage.json`
- Network failures degrade silently to local config without blocking creation

### Conflict Strategy

`gitRepoSetting.gitRepoMode` controls how local and remote templates with the same key are resolved:

| Strategy | Behavior                                                       |
| -------- | -------------------------------------------------------------- |
| `merge`  | Local key is kept; remote conflicting key gets a `-new` suffix |
| `cover`  | Remote value overwrites the local value entirely               |

Example: local has `vue3` → `urlA`, remote has `vue3` → `urlB`

- merge result: `{ vue3: urlA, "vue3-new": urlB }`
- cover result: `{ vue3: urlB }`

### Custom Template Repositories

Set `gitRepoSetting.gitRepo` in project or global config to use your own templates:

````json
{
  "gitRepoSetting": {
    "gitRepo": {
      "my-vue-template": "https://gitee.com/my-org/my-vue-template.git",
      "my-react-template": "https://github.com/my-org/my-react-template.git"
    }
  }
}

Any git URL is supported. Use it via `hias create my-vue-template`.

## Locale Direction

The source locale is the first item in the `locales` array; the rest are target locales.

Chinese source, generate English and Japanese:

```json
{
  "translationSetting": {
    "locales": ["zh-CN", "en-US", "ja-JP"]
  }
}
````

English source, generate Chinese and Japanese:

```json
{
  "translationSetting": {
    "locales": ["en-US", "zh-CN", "ja-JP"]
  }
}
```

## Translation Providers

hias-cli supports multiple translation services, including Tencent Cloud, Baidu, OpenAI (GPT), and Google Translate.

### Configuration Structure

```json
{
  "translationSetting": {
    "translationService": "tencent",
    "servicePriority": ["tencent", "baidu", "openai", "google"],
    "services": {
      "openai": {
        "apiKey": "sk-xxxxxxxx",
        "baseUrl": "https://api.openai.com",
        "model": "gpt-3.5-turbo"
      },
      "google": {
        "apiKey": "your-google-api-key"
      },
      "baidu": {
        "appId": "your-app-id",
        "secretKey": "your-secret-key"
      },
      "tencent": {
        "secretId": "your-secret-id",
        "secretKey": "your-secret-key",
        "region": "ap-guangzhou"
      }
    }
  }
}
```

### Configuration Fields

| Field | Description | Default |
|-------|-------------|---------|
| `translationService` | Current translation service | `tencent` |
| `servicePriority` | Fallback priority | `["tencent", "baidu", "openai", "google"]` |
| `services.openai.apiKey` | OpenAI API key | - |
| `services.openai.baseUrl` | Custom API endpoint | `https://api.openai.com` |
| `services.openai.model` | Model name | `gpt-3.5-turbo` |
| `services.google.apiKey` | Google Cloud Translation API key | - |
| `services.baidu.appId` | Baidu translation app ID | - |
| `services.baidu.secretKey` | Baidu translation secret key | - |
| `services.tencent.secretId` | Tencent Cloud secret ID | - |
| `services.tencent.secretKey` | Tencent Cloud secret key | - |
| `services.tencent.region` | Region | `ap-guangzhou` |

You can leave credentials empty. The CLI prints a warning and, when `fallbackToKey` is `true`, falls back to original text so the migration can still proceed.

## Multi-Project outDir Configuration

`outDir` supports string or object format. Object format is useful for monorepo setups:

String format (default):

```json
{
  "translationSetting": {
    "outDir": ".hias/lang"
  }
}
```

Object format (monorepo):

```json
{
  "translationSetting": {
    "outDir": {
      "main": "apps/main/src/locale/module",
      "web": "apps/web/src/locale/module"
    }
  }
}
```

Use `-p, --project` to specify the project. Without it, the first key is used by default:

```sh
# Default uses "main" (first key in the object)
hias tf apps/main/src/views/index.vue home

# Specify project
hias tf apps/main/src/views/index.vue -p main
hias tf apps/web/src/views/index.vue home -p web

# tfo also supports -p
hias tfo apps/main/src/views views -p main
```

## Translate One File

```sh
hias tf src/views/user/index.vue user
```

Common options:

| Option                      | Description                              |
| --------------------------- | ---------------------------------------- |
| `-n, --name <name>`         | Set namespace                            |
| `-p, --project <project>`   | Specify project when outDir is an object |
| `--show-extractions`        | Print extracted source-locale texts only |
| `--dry-run`                 | Preview only; do not write files         |
| `--interactive`             | Preview and ask whether to apply         |
| `--report <file>`           | Write a JSON report                      |
| `-v, --verbose`             | Print detailed logs                      |

Examples:

```sh
hias tf src/views/user/index.vue user --show-extractions
hias tf src/views/user/index.vue user --dry-run
hias tf src/views/user/index.vue user --interactive
hias tf src/views/user/index.vue user --report .hias/reports/user.json
```

## Translate A Folder

```sh
hias tfo src/views views
```

Exclude tests and temporary files:

```sh
hias tfo src/views views --exclude "**/*.test.js" --exclude "**/*.spec.ts"
```

Preview with report:

```sh
hias tfo src/views views --dry-run --report .hias/reports/views.preview.json
```

Write with report:

```sh
hias tfo src/views views --report .hias/reports/views.write.json
```

Folder translation prints a summary:

```text
Translation summary
  scanned: 12
  replacements: 35
  written: 10
  skipped: 1
  failed: 1
```

If a file fails post-replacement syntax validation, it is skipped and recorded instead of being written with invalid syntax.

## Supported Files

| Type    | Supported content                                                    |
| ------- | -------------------------------------------------------------------- |
| `.vue`  | template, script, script setup, `lang="jsx"`, `lang="tsx"`           |
| `.js`   | JavaScript runtime strings, template literals, display object fields |
| `.ts`   | TypeScript runtime strings, type-only syntax skipped                 |
| `.jsx`  | JSX text, attributes, expression strings                             |
| `.tsx`  | TSX text, attributes, expression strings, TS types skipped           |
| `.json` | JSON value text                                                      |

Vue `<style>`, CSS/SCSS, class, className, style, asset paths, regular expressions, type declarations, and import/export paths are skipped by default.

## Extraction Rules

The extractor targets user-visible or runtime-display text:

```js
message.success('Save successfully')
message.error('Delete failed')
toast('Copied')
alert('Are you sure?')
confirm('Delete this user?')

const item = {
  label: 'User Name',
  title: 'User Detail',
  placeholder: 'Enter username',
  message: 'Save successfully',
  description: 'Please select a user',
}
```

Fallback UI text in expressions is supported:

```js
const text = data?.text || 'Default text'
const desc = data?.description ?? 'No description'
```

Vue templates:

```vue
{{ data?.text || 'Default text' }} {{ data?.description ?? 'No description' }}
```

JSX/TSX:

```tsx
<Comp title="User Detail" label="User Name" />
<div>{data?.text || 'Default text'}</div>
```

Skipped by default:

- Existing `$t(...)`, `this.$t(...)`, and `t(...)`
- Comments, regular expressions, paths, URLs
- `class`, `className`, `style`, Vue `<style>` blocks
- `type`, `role`, event names, `data-*`, `aria-*`
- TypeScript literal types, interface, type declarations, generics
- Destructuring defaults and function parameter defaults

## Ignore Comments

Ignore the next extractable segment:

```js
// @hias-i18n-ignore-next
message.success('Keep original')

/* @hias-i18n-ignore-next */
toast('Keep original')
```

Vue template:

```vue
<!-- @hias-i18n-ignore-next -->
<div>Keep original</div>
```

Ignore a range:

```js
/* @hias-i18n-ignore-start */
const title = 'Keep original'
const desc = 'Keep original too'
/* @hias-i18n-ignore-end */
```

Ignore the whole file:

```js
// @hias-i18n-ignore-file
```

## IDE Code Snippets

Generate IDE code snippets for quick insertion of ignore comments:

```sh
hias codesnippets           # Auto-detect IDE (.vscode / .idea / .cursor)
hias codesnippets vscode    # Generate VS Code snippets
hias codesnippets webstorm  # Generate WebStorm templates
hias codesnippets cursor    # Generate Cursor snippets
hias codesnippets sublime   # Generate Sublime Text snippets
```

Generated snippet prefixes:

| Prefix                    | Purpose                         |
| ------------------------- | ------------------------------- |
| `@hias-i18n-ignore-next`  | Ignore next extractable segment |
| `@hias-i18n-ignore-start` | Ignore range start              |
| `@hias-i18n-ignore-end`   | Ignore range end                |
| `@hias-i18n-ignore-file`  | Ignore entire file              |

## Replacement Examples

Vue text:

```vue
<button>登录</button>
```

```vue
<button>{{ $t('user.login') }}</button>
```

Vue attribute:

```vue
<input placeholder="请输入用户名" />
```

```vue
<input :placeholder="$t('user.please_enter_username')" />
```

JS:

```js
message.success('Save successfully')
```

```js
message.success($t('user.save_successfully'))
```

JSX:

```tsx
<Button title='User Detail'>Save</Button>
```

```tsx
<Button title={$t('user.user_detail')}>{$t('user.save')}</Button>
```

Template literals keep expressions and replace static text segments:

```js
const text = `Hello ${name}, save successfully`
```

## Key Generation

By default, keys are generated from translated English in snake_case. When translation fails or credentials are missing, keys are generated from original text.

```json
{
  "translationSetting": {
    "keyStrategy": {
      "maxLength": 40,
      "collision": "number",
      "hashLength": 6
    }
  }
}
```

Collision strategies:

| Strategy | Behavior                         |
| -------- | -------------------------------- |
| `number` | Append `_1`, `_2` on collision   |
| `hash`   | Append a short hash on collision |

Extracted source texts are deduplicated before translation to reduce repeated API calls.

## i18n Call Template

Default:

```json
{
  "translationSetting": {
    "i18nCallTemplate": "$t"
  }
}
```

Output:

```js
$t('user.login')
```

Examples:

| Config                       | Output                        |
| ---------------------------- | ----------------------------- |
| `"$t"`                       | `$t('user.login')`            |
| `"t"`                        | `t('user.login')`             |
| `"{{key}}"`                  | `user.login`                  |
| `"i18n.global.t('{{key}}')"` | `i18n.global.t('user.login')` |

Existing Vue 2 `this.$t(...)` calls are recognized and skipped. The CLI does not inject `useI18n` by default, which fits projects using auto-import plugins.

## Locale Output

With `replaceOriginalFile: false`, output goes to:

```text
.hias/
├── setting.json
├── lang/
│   └── <namespace>/
│       ├── zh-CN.json
│       ├── en-US.json
│       ├── ja-JP.json
│       └── <translated-file>
├── .translation-cache.json
└── .langbackup/
```

To replace source files directly:

```json
{
  "translationSetting": {
    "replaceOriginalFile": true
  }
}
```

Backups are stored under `.hias/.langbackup/`.

Locale file format:

```json
{
  "user": {
    "login": "登录",
    "save_successfully": "保存成功"
  }
}
```

## Import Translation Cache

If you already have a company translation table, import it before translation:

```sh
hias cache-import company.xlsx
hias tci company.xlsx
```

Set source locale column:

```sh
hias tci company.xlsx --source zh-CN
```

Set Excel sheet:

```sh
hias tci company.xlsx --sheet Sheet1
```

Overwrite existing cache entries:

```sh
hias tci company.xlsx --overwrite
```

Recommended table:

| zh-CN    | en-US              | ja-JP              |
| -------- | ------------------ | ------------------ |
| 登录     | Login              | ログイン           |
| 保存成功 | Saved successfully | 保存に成功しました |

> Note: Only `.xlsx` / `.xlsm` are supported. The legacy binary `.xls` format is not supported — please re-save it as `.xlsx` in Excel first.

TSV/TXT is also supported:

```text
zh-CN	en-US	ja-JP
登录	Login	ログイン
保存成功	Saved successfully	保存に成功しました
```

Cache file:

```text
.hias/.translation-cache.json
```

## Apply Cache to Locale Files

`cache-apply` (short command `tca`) writes translation results to locale JSON files under `outDir`. It checks the cache first; for cache misses, it automatically calls the translation API and caches the results.

Use cases:
- `replaceOriginalFile: true` mode: source files have been rewritten to `$t()` calls, re-running `tf` / `tfo` can no longer extract Chinese text
- Translation service switch: need to re-translate uncached entries after changing API provider
- Post-network recovery: re-translate entries that previously failed due to network issues

```sh
hias cache-apply
hias tca
```

Process a single namespace only (a subdirectory under `outDir`):

```sh
hias tca views
```

Specify the project when `outDir` is an object:

```sh
hias tca -p admin
```

## Reports

`tf` / `tfo` support:

```sh
hias tfo src/views views --report .hias/reports/views.json
```

Report shape:

```json
{
  "mode": "write",
  "summary": {
    "filesScanned": 1,
    "filesWithExtractions": 1,
    "replacements": 3,
    "written": 1,
    "skipped": 0,
    "failed": 0
  },
  "files": [
    {
      "file": "src/views/user/index.vue",
      "outputFile": ".hias/lang/user/index.vue",
      "replacements": [
        {
          "text": "User Detail",
          "key": "user.user_detail",
          "context": "attribute",
          "start": 42,
          "end": 53
        }
      ]
    }
  ],
  "failedFiles": []
}
```

## Check Locale Files

```sh
hias tcheck src
hias tcheck src --report .hias/reports/tcheck.json
```

It reports:

- Keys referenced by source but missing from locale files
- Locale keys not referenced by source
- Keys inconsistent between locale JSON files
- Invalid locale JSON files

This is useful after bulk migration and in CI.

## Rollback

List backups:

```sh
hias rollback --list
```

Rollback latest:

```sh
hias rollback
```

Rollback a named backup:

```sh
hias rollback --name 20250101_120000_user
```

Keep only the latest 3 backups:

```sh
hias rollback --keep 3
```

When `replaceOriginalFile` is `true`, source files are restored. When it is `false`, the related output directory is removed.

## Git Ignore

Add `.hias` to the current project's `.gitignore`:

```sh
hias gitignore
```

Do not commit API credentials, translation caches, backups, or report files unless your team explicitly wants them versioned.

## Kill Ports

```sh
hias close-port 3000
hias close-port 3000 5173
hias close-port 3000,5173
```

Windows uses `netstat` + `taskkill`; macOS/Linux use `lsof` + `kill`.

## CLI Language

```sh
hias lang zh-CN
hias lang en
hias lang -l
```

Supported:

- `zh-CN`
- `en`

Stored in:

```text
~/.hias-cli/config.json
```

## Development Commands

Inside the `hias-cli` repository:

```sh
npm install
npm test
npm run build
npm run doc:dev
npm run doc:build
```

Scripts:

| Script                     | Purpose                             |
| -------------------------- | ----------------------------------- |
| `npm test`                 | Run Node.js tests                   |
| `npm run build`            | Build `dist/index.js` and templates |
| `npm run build:obfuscated` | Obfuscated build for publish        |
| `npm run doc:dev`          | Start VitePress docs                |
| `npm run doc:build`        | Build docs site                     |
| `npm run doc:preview`      | Preview built docs                  |

## FAQ

### Why are some English strings not extracted?

English source contains many technical strings such as class names, types, paths, event names, and protocol values. The extractor prioritizes UI-copy contexts and skips technical contexts to avoid false positives.

### Why are destructuring defaults and parameter defaults skipped?

They often describe data fallback rather than UI text:

```js
const { name = 'User name' } = data
function submit(label = 'Save') {}
```

If the value is truly UI copy, rewrite it as runtime display logic or handle it manually.

### Are regular expressions translated?

No. Regular expressions are matching rules. Translating them can change business logic.

### Are Vue styles translated?

No. `<style>` blocks, `style` attributes, `class`, and dynamic class bindings are skipped.

### Is JSX className translated?

No. `className`, `style`, event props, `type`, `role`, `data-*`, and `aria-*` are skipped.

### Are existing `$t` calls processed again?

No. `$t(...)`, `this.$t(...)`, and `t(...)` are recognized and skipped.

### Can I use it without translation API credentials?

Yes. With `fallbackToKey: true`, the CLI falls back to original text and still generates i18n structure.
