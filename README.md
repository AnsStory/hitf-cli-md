# hias-cli 操作手册

[![npm version](https://img.shields.io/npm/v/@ansstory/hias.svg)](https://www.npmjs.com/package/@ansstory/hias)
[![license](https://img.shields.io/npm/l/@ansstory/hias.svg)](https://github.com/ansstory/hias-cli/blob/main/LICENSE)

`hias-cli` 是一个面向前端项目的命令行工具，包含项目模板创建、Vue/React/Redux 文件生成、端口关闭、CLI 语言切换，以及用于 Vue、React、JS、TS、JSX、TSX、JSON 的国际化翻译迁移工具链。

本文是操作手册，按真实使用顺序组织：先安装，再配置，再执行翻译、检查、回滚和维护。

## 环境要求

- Node.js `>= 18`
- Windows、macOS、Linux 均可使用
- 翻译能力支持百度翻译和腾讯云 TMT；没有密钥时也可以使用回退模式完成迁移结构

## 安装与检查

全局安装：

```sh
npm i @ansstory/hias -g
```

检查安装结果：

```sh
hias -v
hias --help
```

只清理全局配置，不卸载包：

```sh
hias clear-config
```

## 命令总览

| 命令                    | 别名  | 用途                           |
| ----------------------- | ----- | ------------------------------ |
| `create <project>`      | `crt` | 交互式创建项目模板             |
| `adv <name>`            | -     | 生成 Vue 组件                  |
| `adr <name>`            | -     | 生成 React JSX 组件            |
| `adrt <name>`           | -     | 生成 React TSX 组件            |
| `adrd <name>`           | -     | 生成 Redux JSX store           |
| `adrdt <name>`          | -     | 生成 Redux TSX store           |
| `tf [file] [name]`      | -     | 翻译单个文件                   |
| `tfo [folder] [name]`   | -     | 翻译目录内文件                 |
| `tcheck [target]`       | -     | 检查源码引用和语言包一致性     |
| `cache-import [file]`   | `tci` | 导入 XLSX/TSV/TXT 翻译表到缓存 |
| `cache-apply [name]`    | `tca` | 将缓存译文回写到语言包 JSON   |
| `codesnippets [ide]`    | -     | 生成 IDE 代码片段              |
| `setting`               | -     | 创建项目级翻译配置             |
| `global-config`         | -     | 创建全局翻译配置               |
| `rollback`              | -     | 回滚翻译输出或原文件           |
| `gitignore`             | -     | 将 `.hias` 写入 `.gitignore`   |
| `close-port <ports...>` | -     | 关闭占用端口的进程             |
| `lang [language]`       | -     | 切换或查看 CLI 显示语言        |

## 推荐工作流

首次在项目中使用翻译功能时，建议按下面顺序执行：

```sh
cd your-project
hias setting
hias gitignore
hias tfo src\views views --show-extractions
hias tfo src\views views --dry-run --report .hias\reports\views.preview.json
hias tfo src\views views --report .hias\reports\views.write.json
hias tcheck src --report .hias\reports\tcheck.json
```

如果结果不符合预期：

```sh
hias rollback
```

## 创建项目

```sh
hias create my-app
```

命令会进入交互式选择流程，从模板仓库下载项目到 `my-app`。

可用模板：

| 模板         | 说明                    |
| ------------ | ----------------------- |
| `vue2`       | Vue 2 模板              |
| `vue3`       | Vue 3 模板              |
| `vue-ts`     | Vue 3 + TypeScript 模板 |
| `uni-vite`   | uni-app + Vite 模板     |
| `nuxt-web`   | Nuxt.js 网页模板        |
| `vue-screen` | Vue 数据大屏模板        |
| `react`      | React JSX 模板          |
| `react-ts`   | React TypeScript 模板   |

## 生成组件和 store

默认生成到 `src/components`：

```sh
hias adv Demo
hias adr Demo
hias adrt Demo
```

指定目录：

```sh
hias adv Demo -d src/views/user
hias adr Demo -d src/components
hias adrt Demo -d src/components
```

生成 Redux store：

```sh
hias adrd useUserStore -d src/store/modules
hias adrdt useUserStore -d src/store/modules
```

当名称为 `index` 时，生成文件会使用父目录名称。例如：

```sh
hias adv index -d src/views/user
```

会生成类似 `src/views/user/user.vue` 的文件。

## 翻译配置

进入业务项目根目录后执行：

```sh
hias setting
```

会创建项目级配置：

```text
.hias/setting.json
```

完整配置示例：

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

字段说明：

| 字段                               | 默认值               | 说明                                                           |
| ---------------------------------- | -------------------- | -------------------------------------------------------------- |
| `locales`                          | `["zh-CN", "en-US"]` | `locales[0]` 为源语言，其余为目标语言                          |
| `outDir`                           | `.hias/lang`         | 语言包和翻译副本输出目录，支持字符串或对象格式                 |
| `fallbackToKey`                    | `true`               | 翻译失败时是否回退到原文 key                                   |
| `replaceOriginalFile`              | `false`              | 是否直接覆盖源文件                                             |
| `translationSetting.translationService` | `tencent`       | 当前使用的翻译服务：`baidu`、`tencent`、`openai` 或 `google`   |
| `translationSetting.servicePriority` | `["tencent", "baidu", "openai", "google"]` | 故障转移优先级 |
| `translationSetting.services.openai.apiKey` | -             | OpenAI API 密钥                                                |
| `translationSetting.services.openai.baseUrl` | `https://api.openai.com` | 自定义 API 端点                                        |
| `translationSetting.services.openai.model` | `gpt-3.5-turbo`   | 模型名称                                                       |
| `translationSetting.services.google.apiKey` | -             | Google Cloud Translation API 密钥                              |
| `translationSetting.services.baidu.appId` | -                | 百度翻译应用 ID                                                |
| `translationSetting.services.baidu.secretKey` | -            | 百度翻译密钥                                                   |
| `translationSetting.services.tencent.secretId` | -            | 腾讯云密钥 ID                                                  |
| `translationSetting.services.tencent.secretKey` | -           | 腾讯云密钥                                                     |
| `translationSetting.services.tencent.region` | `ap-guangzhou` | 区域                                                          |
| `i18nCallTemplate`                 | `$t`                 | 替换源码时使用的 i18n 调用，支持字符串或数组格式               |
| `i18nImport`                       | `""`                 | 可选导入语句；支持字符串或数组格式；适用于 Vue/JS/TS/JSX/TSX   |
| `extensions`                       | `[]`                 | 自定义扫描扩展名；空数组表示全部支持类型                       |
| `capitalizeTranslations`           | `false`              | 是否把含拉丁字符的翻译结果首字母大写                           |
| `capitalizeMaxWords`               | `0`                  | 全词大写（Title Case）阈值：仅当 `capitalizeTranslations` 为 `true` 时生效。`0` 表示关闭，仅首词大写；`>0` 且英语译文单词数 ≤ 该值时，每个单词首字母大写（仅限英语目标语言） |
| `pruneUnusedKeys`                  | `false`              | 是否移除当前命名空间未被源码引用的旧 key                       |
| `keyStrategy.maxLength`            | `40`                 | 自动生成 key 的最大长度                                        |
| `keyStrategy.collision`            | `number`             | key 冲突策略：`number` 或 `hash`                               |
| `keyStrategy.hashLength`           | `6`                  | hash 冲突策略的短 hash 长度                                    |
| `gitRepoSetting.gitRepoAutoUpdate` | `false`              | 创建项目时是否自动拉取远程模板列表（项目级默认关，全局默认开） |
| `gitRepoSetting.gitRepoMode`       | `"merge"`            | 远程与本地模板冲突策略：`merge`（-new 后缀）或 `cover`（覆盖） |
| `gitRepoSetting.gitRepo`           | `{}`                 | 自定义模板仓库映射，键为模板名，值为 git 地址                  |

: 此字段位于配置文件根层级的 `gitRepoSetting` 对象内，与 `translationSetting` 同级。

配置读取优先级：

1. 当前项目 `.hias/setting.json`
2. 全局 `~/.hias-cli/config.json`
3. 内置默认值

创建全局配置：

```sh
hias global-config
```

## 模板仓库管理

`hias create` 使用的模板列表来自 `gitRepoSetting.gitRepo` 配置。解析优先级：

1. 项目级 `.hias/setting.json` 中的 `gitRepoSetting.gitRepo` 字段
2. 全局 `~/.hias-cli/config.json` 中的 `gitRepoSetting.gitRepo` 字段
3. 内置硬编码默认仓库列表（vue2 / vue3 / vue-ts / react 等）

三个层次的 gitRepo 配置按**逐字段级联**方式合并（与翻译配置的整体覆盖不同），项目级最优先。

### 自动更新

```json
{
  "translationSetting": {},
  "gitRepoSetting": {
    "gitRepoAutoUpdate": true,
    "gitRepoMode": "merge"
  }
}
```

- `gitRepoSetting.gitRepoAutoUpdate`：创建项目时是否从远程拉取最新模板列表
  - 全局配置默认 `true`，使所有项目能获取更新
  - 项目级配置默认 `false`，避免意外网络请求
- 远程模板列表地址：`https://ansstory.github.io/hias-cli-md/git-storage.json`
- 网络异常时静默降级到本地配置，不阻塞创建流程

### 冲突策略

`gitRepoSetting.gitRepoMode` 控制本地与远程模板同名冲突时的行为：

| 策略    | 行为                                     |
| ------- | ---------------------------------------- |
| `merge` | 本地保留原键，远程冲突键追加 `-new` 后缀 |
| `cover` | 远程值直接覆盖本地值                     |

示例：本地有 `vue3` → `urlA`，远程有 `vue3` → `urlB`

- merge 模式结果：`{ vue3: urlA, "vue3-new": urlB }`
- cover 模式结果：`{ vue3: urlB }`

### 自定义模板仓库

在项目或全局配置中设置 `gitRepoSetting.gitRepo`，即可使用自己的模板：

```json
{
  "gitRepoSetting": {
    "gitRepo": {
      "my-vue-template": "https://gitee.com/my-org/my-vue-template.git",
      "my-react-template": "https://github.com/my-org/my-react-template.git"
    }
  }
}
```

支持任意 git 仓库地址，创建时通过 `hias create my-vue-template` 使用。

## 语言方向

源语言由 `locales` 数组的第一项决定，其余为目标语言。

源码是中文，生成英文和日文：

```json
{
  "translationSetting": {
    "locales": ["zh-CN", "en-US", "ja-JP"]
  }
}
```

源码是英文，生成中文和日文：

```json
{
  "translationSetting": {
    "locales": ["en-US", "zh-CN", "ja-JP"]
  }
}
```

## 翻译服务商

hias-cli 支持多种翻译服务，包括腾讯云、百度、OpenAI (GPT) 和谷歌翻译。

### 配置结构

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

### 配置字段说明

| 字段 | 说明 | 默认值 |
|------|------|--------|
| `translationService` | 当前使用的翻译服务 | `tencent` |
| `servicePriority` | 故障转移优先级 | `["tencent", "baidu", "openai", "google"]` |
| `services.openai.apiKey` | OpenAI API 密钥 | - |
| `services.openai.baseUrl` | 自定义 API 端点 | `https://api.openai.com` |
| `services.openai.model` | 模型名称 | `gpt-3.5-turbo` |
| `services.google.apiKey` | Google Cloud Translation API 密钥 | - |
| `services.baidu.appId` | 百度翻译应用 ID | - |
| `services.baidu.secretKey` | 百度翻译密钥 | - |
| `services.tencent.secretId` | 腾讯云密钥 ID | - |
| `services.tencent.secretKey` | 腾讯云密钥 | - |
| `services.tencent.region` | 区域 | `ap-guangzhou` |

没有密钥时可以留空。工具会输出警告，然后在 `fallbackToKey: true` 时使用原文回退，迁移结构仍可继续。

## 多项目 outDir 配置

`outDir` 支持字符串或对象格式。对象格式适用于 monorepo 多项目场景：

字符串格式（默认）：

```json
{
  "translationSetting": {
    "outDir": ".hias/lang"
  }
}
```

对象格式（monorepo）：

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

使用 `-p, --project` 参数指定项目。不指定时默认使用对象中第一个 key：

```sh
# 默认使用 main（对象中第一个 key）
hias tf apps/main/src/views/index.vue home

# 指定项目
hias tf apps/main/src/views/index.vue -p main
hias tf apps/web/src/views/index.vue home -p web

# tfo 同样支持
hias tfo apps/main/src/views views -p main
```

## 翻译单文件

```sh
hias tf src/views/user/index.vue user
```

常用选项：

| 选项                      | 说明                                   |
| ------------------------- | -------------------------------------- |
| `-n, --name <name>`       | 指定命名空间                           |
| `-p, --project <project>` | outDir 为对象时指定项目                |
| `--show-extractions`      | 只显示取到的源语言文案                 |
| `--dry-run`               | 只预览，不写文件                       |
| `--interactive`           | 预览后询问是否应用                     |
| `--report <file>`         | 输出 JSON 报告                         |
| `-v, --verbose`           | 输出详细日志                           |

示例：

```sh
hias tf src/views/user/index.vue user --show-extractions
hias tf src/views/user/index.vue user --dry-run
hias tf src/views/user/index.vue user --interactive
hias tf src/views/user/index.vue user --report .hias/reports/user.json
```

## 翻译目录

```sh
hias tfo src/views views
```

排除测试文件和临时文件：

```sh
hias tfo src/views views --exclude "**/*.test.js" --exclude "**/*.spec.ts"
```

预览并输出报告：

```sh
hias tfo src/views views --dry-run --report .hias/reports/views.preview.json
```

正式写入并输出报告：

```sh
hias tfo src/views views --report .hias/reports/views.write.json
```

目录翻译完成后会输出汇总：

```text
Translation summary
  scanned: 12
  replacements: 35
  written: 10
  skipped: 1
  failed: 1
```

如果某个文件替换后语法校验失败，工具会跳过该文件并写入失败原因，不会把坏语法写进输出。

## 支持的文件

| 类型    | 支持内容                                                   |
| ------- | ---------------------------------------------------------- |
| `.vue`  | template、script、script setup、`lang="jsx"`、`lang="tsx"` |
| `.js`   | JavaScript 运行时字符串、模板字符串、对象展示字段          |
| `.ts`   | TypeScript 运行时字符串，跳过类型层语法                    |
| `.jsx`  | JSX 文本、属性、表达式字符串                               |
| `.tsx`  | TSX 文本、属性、表达式字符串，跳过 TS 类型                 |
| `.json` | JSON value 文案                                            |

Vue `<style>`、CSS/SCSS、class、className、style、资源路径、正则、类型声明、import/export 路径默认跳过。

## 取词规则

会提取用户可见或运行时展示的文案：

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

支持逻辑兜底文案：

```js
const text = data?.text || 'Default text'
const desc = data?.description ?? 'No description'
```

Vue 模板中也支持：

```vue
{{ data?.text || 'Default text' }} {{ data?.description ?? 'No description' }}
```

JSX/TSX：

```tsx
<Comp title="User Detail" label="User Name" />
<div>{data?.text || 'Default text'}</div>
```

默认跳过这些内容：

- 已处理的 `$t(...)`、`this.$t(...)`、`t(...)`
- 注释、正则表达式、路径、URL
- `class`、`className`、`style`、Vue `<style>` 块
- `type`、`role`、事件名、`data-*`、`aria-*`
- TypeScript 类型字面量、interface、type、泛型
- 解构默认值和函数参数默认值

## 忽略注释

忽略下一段：

```js
// @hias-i18n-ignore-next
message.success('Keep original')

/* @hias-i18n-ignore-next */
toast('Keep original')
```

Vue 模板：

```vue
<!-- @hias-i18n-ignore-next -->
<div>保持原文</div>
```

忽略范围：

```js
/* @hias-i18n-ignore-start */
const title = 'Keep original'
const desc = 'Keep original too'
/* @hias-i18n-ignore-end */
```

忽略整个文件：

```js
// @hias-i18n-ignore-file
```

## IDE 代码片段

生成 IDE 代码片段，用于快速插入忽略注释：

```sh
hias codesnippets           # 自动检测 IDE（.vscode / .idea / .cursor）
hias codesnippets vscode    # 生成 VS Code 代码片段
hias codesnippets webstorm  # 生成 WebStorm 模板
hias codesnippets cursor    # 生成 Cursor 代码片段
hias codesnippets sublime   # 生成 Sublime Text 片段
```

生成的代码片段前缀：

| 前缀                      | 功能           |
| ------------------------- | -------------- |
| `@hias-i18n-ignore-next`  | 忽略下一段翻译 |
| `@hias-i18n-ignore-start` | 忽略范围开始   |
| `@hias-i18n-ignore-end`   | 忽略范围结束   |
| `@hias-i18n-ignore-file`  | 忽略文件翻译   |

## 替换结果示例

Vue 文本：

```vue
<button>登录</button>
```

```vue
<button>{{ $t('user.login') }}</button>
```

Vue 属性：

```vue
<input placeholder="请输入用户名" />
```

```vue
<input :placeholder="$t('user.please_enter_username')" />
```

JS：

```js
message.success('Save successfully')
```

```js
message.success($t('user.save_successfully'))
```

JSX：

```tsx
<Button title='User Detail'>Save</Button>
```

```tsx
<Button title={$t('user.user_detail')}>{$t('user.save')}</Button>
```

模板字符串会保留表达式，只替换静态文案片段：

```js
const text = `Hello ${name}, save successfully`
```

## Key 生成

默认使用翻译后的英文生成 snake_case key。翻译失败或没有密钥时，会从原文生成 key。

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

冲突策略：

| 策略     | 行为                  |
| -------- | --------------------- |
| `number` | 冲突时追加 `_1`、`_2` |
| `hash`   | 冲突时追加短 hash     |

取词会先按原文去重，避免重复请求翻译 API。

## i18n 调用模板

默认：

```json
{
  "translationSetting": {
    "i18nCallTemplate": "$t"
  }
}
```

生成：

```js
$t('user.login')
```

自定义示例：

| 配置                         | 结果                          |
| ---------------------------- | ----------------------------- |
| `"$t"`                       | `$t('user.login')`            |
| `"t"`                        | `t('user.login')`             |
| `"{{key}}"`                  | `user.login`                  |
| `"i18n.global.t('{{key}}')"` | `i18n.global.t('user.login')` |

Vue2 的 `this.$t(...)` 已处理内容会被识别并跳过。默认不注入 `useI18n`，适合配合项目自己的自动导入方案。

## 语言包输出

默认 `replaceOriginalFile: false` 时，输出到：

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

如果设置：

```json
{
  "translationSetting": {
    "replaceOriginalFile": true
  }
}
```

则会直接替换原文件，并在 `.hias/.langbackup/` 中保存备份。

语言包格式：

```json
{
  "user": {
    "login": "登录",
    "save_successfully": "保存成功"
  }
}
```

## 导入翻译缓存

如果公司已有翻译表，建议先导入缓存，再执行翻译：

```sh
hias cache-import company.xlsx
hias tci company.xlsx
```

指定源语言列：

```sh
hias tci company.xlsx --source zh-CN
```

指定 Excel 工作表：

```sh
hias tci company.xlsx --sheet Sheet1
```

覆盖已有缓存：

```sh
hias tci company.xlsx --overwrite
```

推荐表格格式：

| zh-CN    | en-US              | ja-JP              |
| -------- | ------------------ | ------------------ |
| 登录     | Login              | ログイン           |
| 保存成功 | Saved successfully | 保存に成功しました |

> 注意：Excel 仅支持 `.xlsx` / `.xlsm` 格式，不支持旧版二进制 `.xls`，请先在 Excel 中另存为 `.xlsx`。

也支持 TSV/TXT：

```text
zh-CN	en-US	ja-JP
登录	Login	ログイン
保存成功	Saved successfully	保存に成功しました
```

缓存文件：

```text
.hias/.translation-cache.json
```

## 回写语言包

`cache-apply`（短命令 `tca`）用于将翻译结果写入 `outDir` 下的语言包 JSON。工作逻辑：缓存命中直接使用，未命中自动调用翻译 API 翻译。

适用场景：
- `replaceOriginalFile: true` 模式下源文件已被 `$t()` 替换，无法重新提取原文
- 翻译服务商切换后需要重新翻译未缓存的条目
- 网络恢复后补翻之前失败的条目

```sh
hias cache-apply
hias tca
```

只处理指定命名空间（`outDir` 下的子目录名）：

```sh
hias tca views
```

`outDir` 为对象格式时指定项目：

```sh
hias tca -p admin
```

## 报告文件

`tf` / `tfo` 支持：

```sh
hias tfo src/views views --report .hias/reports/views.json
```

报告结构：

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

## 检查语言包

```sh
hias tcheck src
hias tcheck src --report .hias/reports/tcheck.json
```

检查内容：

- 源码引用了但语言包不存在的 key
- 语言包存在但源码未引用的 key
- 不同 locale JSON 之间 key 不一致
- 无法解析的 locale JSON 文件

适合在批量迁移后执行，也适合放进 CI。

## 回滚

查看备份：

```sh
hias rollback --list
```

回滚最近一次：

```sh
hias rollback
```

回滚指定备份：

```sh
hias rollback --name 20250101_120000_user
```

只保留最近 3 个备份：

```sh
hias rollback --keep 3
```

`replaceOriginalFile: true` 时会恢复原文件；`false` 时会清理对应输出目录。

## Git 忽略

将 `.hias` 添加到当前项目 `.gitignore`：

```sh
hias gitignore
```

建议不要提交 `.hias/setting.json` 中的密钥、翻译缓存、备份和报告文件。

## 关闭端口

```sh
hias close-port 3000
hias close-port 3000 5173
hias close-port 3000,5173
```

Windows 使用 `netstat` + `taskkill`，macOS/Linux 使用 `lsof` + `kill`。

## 切换 CLI 语言

```sh
hias lang zh-CN
hias lang en
hias lang -l
```

支持：

- `zh-CN`
- `en`

配置保存在：

```text
~/.hias-cli/config.json
```

## 本项目开发命令

在 `hias-cli` 仓库中：

```sh
npm install
npm test
npm run build
npm run doc:dev
npm run doc:build
```

脚本说明：

| 脚本                       | 用途                        |
| -------------------------- | --------------------------- |
| `npm test`                 | 运行 Node.js 测试           |
| `npm run build`            | 构建 `dist/index.js` 和模板 |
| `npm run build:obfuscated` | 混淆构建，用于发布          |
| `npm run doc:dev`          | 启动 VitePress 文档站       |
| `npm run doc:build`        | 构建文档站                  |
| `npm run doc:preview`      | 预览文档站构建产物          |

## 常见问题

### 为什么有些英文没有被提取

英文源码里有大量技术字符串，例如类名、类型、路径、事件名、协议值。工具会优先提取 UI 文案和展示上下文，跳过技术上下文，避免误翻译。

### 为什么解构默认值和函数参数默认值不翻译

这类值通常是数据结构或函数兜底，不一定会展示给用户：

```js
const { name = 'User name' } = data
function submit(label = 'Save') {}
```

如果它确实是 UI 文案，可以改成运行时展示表达式或手动处理。

### 正则里的文案会翻译吗

不会。正则是业务匹配规则，翻译后可能改变程序逻辑。

### Vue 的样式会翻译吗

不会。`<style>` 块、`style` 属性、`class`、动态 class 都会跳过。

### JSX 的 className 会翻译吗

不会。`className`、`style`、事件属性、`type`、`role`、`data-*`、`aria-*` 会跳过。

### 已经处理过的 `$t` 会重复处理吗

不会。`$t(...)`、`this.$t(...)`、`t(...)` 会被识别并跳过。

### 没有翻译 API 密钥还能用吗

可以。开启 `fallbackToKey: true` 时，工具会用原文回退，先完成代码迁移和语言包结构生成。
