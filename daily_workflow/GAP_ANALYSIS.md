# Business Gap Analysis
## Goal
Find business gaps by comparing the AsIs sheet and the ToBe sheet.

## Background - Why
I joined this repository yesterday and began making business modifications,
but I have no background knowledge about it.

I have the target feature's AsIs, and want to begin a gap analysis.
So, I want you to support me by researching "the gap".

## Definitions
**gap**:
The space between AsIs and ToBe.
In other words, the "problem".

**feature(s)**:
What it is.
This is from the user's point of view.
In other words, the basic specification without technical terms.

## Prohibition
* Launching servers, including the DB
* Editing existing files
* Accessing external APIs
* Reading files under `tmp/`

## Workflow
There are mainly two steps.
* 1st step - understand feature(s) AsIs
* 2nd step - analyze gap

### Given workflow statement
* Output should be in Japanese/UTF-8

### Before step
Ask the user where the "ToBe" sheet is located.
**If no such sheet exists, stop the workflow.**

### 1st step - understand feature(s) AsIs
The goal is to understand the target feature(s).

Summary:

* Input:
  * user's pointing file(s)
* Output:
  * internal memory

Detail:

* [ ] Receive the feature path from the user
* [ ] Understand the target feature(s)
  * [ ] You can output them as memory under `/tmp`

### 2nd step - analyze gap
The goal is to find the gap between AsIs and ToBe.

Summary:

* Input:
  * ToBe sheet and AsIs memory
* Output:
  * Gap analysis report as markdown

Detail:

* [ ] Receive the ToBe sheet path
  * If no path is provided, stop the workflow
* [ ] **Before starting the steps below, use AskUserQuestion 5+ times to clarify the flow.**
* [ ] Research the gap between AsIs and ToBe
* [ ] Organize the gap info
* [ ] Output the report
  * Format follows "Output format → Gap analysis"
  * File should be `report/{yyyymmdd-HHMM}-{title}-gap-analysis.md`

**Remarks**:
* Avoid developer-oriented technical terms like `variable`, `class`, `DB`, `file`
* Treat the markdown reader as the audience "closest to the end user"

## Output format
### Gap analysis

```
# Business Gap Analysis
## (1) Showing blog
### AsIs
**What it does**
Anonymous user can read blog without authentication.

**Who uses it & when**
* Persona: Anonymous user
* Trigger: Accessing blog
* Expects: Showing blog contents

**Current Behavior - valid/invalid**
* valid
  * Can be shown blog content
  * Cannot be shown blog content if content is hidden mode
* invalid
  * 404 Error when access to non exist blog content

### ToBe
**Changed Behavior - valid/invalid**
* valid
  * (keep) Can be shown blog content
  * (change) **Hidden mode is deleted.**
    * All hidden blog item should be soft deleted.
* invalid
  * (keep) 404 Error when access to non exist blog content

### Problem
There are no specification
* whether "soft deleted item(s) should be shown to editor"
* who run "first soft deletion". Is that batch? when to do?
```

**Remarks**:
When outputting in Japanese, the style should be `だ/である`.

Example:
```
# ビジネスギャップ分析
## (1) ブログ表示
### AsIs
**仕様**
認証なしでブログ表示が可能

**利用者と期待値**
* ペルソナ: 不特定多数のユーザー
* トリガー: ブログへのアクセス
* 期待値: ブログコンテンツの表示

**挙動 - 正常/異常**
* 正常
  * ブログコンテンツが表示される
  * ライターが隠しモードにしている記事は表示されない
* 異常
  * 存在しないブログコンテンツにアクセスすると404が表示される

### ToBe
**挙動 - 正常/異常**
* 正常
  * (keep) ブログコンテンツが表示される
  * (change) **隠しモードは廃止される**
    * 隠しモードの記事はすべて論理削除する
* 異常
  * (keep) 存在しないブログコンテンツにアクセスすると404が表示される

### 問題
次の仕様が決まっていない
* 論理削除した記事は、編集者に見えるようにするのか?
* 誰が、最初の論理削除を行うのか? バッチか? いつそれを実行するのか?
```