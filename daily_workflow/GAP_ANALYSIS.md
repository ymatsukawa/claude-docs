# Business Gap ANALYSIS
## Goal
Finding business gap analysis from AsIs sheet and ToBe sheet.

## Background - Why
I joined this repository yesterday and begun modifications for business,
but have no knowledge around this.

I got target feature AsIs, and begin to gap analysis.
So, I want you as supporter to research "the gap".

## Definitions
**gap**:
Space between AsIs and ToBe.
In other word, "problem".

**feature(s)**:
What it is.
This is for user's point of view.
In other words, basic specification without technical word.

## Prohibition
* Launching server including DB
* Editing exist files
* Accessing external API
* Read under `tmp/`

## Workflow
There are mainly two steps.
* 1st step - understand feature(s) AsIs
* 2nd step - analyze gap

### Given workflow statement
* Output should be Japanese/UTF-8

### before step
Ask user where is "ToBe" sheet.
**If there is nothing the sheet, stop workflow.**

### 1st step - understand feature(s) AsIs
Goal is to understand target feature(s)

Summary:

* Input:
  * user's pointing file(s)
* Output:
  * internal memory

Detail:

* [ ] Receive feature path from user
* [ ] Understand target feature(s)
  * [ ] You can output them as memory under `/tmp`

### 2nd step - analyze gap
Goal is find gap between AsIs and ToBe

Summary:

* Input:
  * ToBe sheet and AsIs memory
* Output:
  * Gap analysis report as markdown

Detail:

* [ ] Receive ToBe sheet path
  * If path is not pointed, stop workflow
* [ ] **Before start follows, AskUserQuestion 5+ for clearing flow.**
* [ ] Research the gap between AsIs and ToBe
* [ ] Organize gap info
* [ ] Output report
  * Format follows "Output format → Gap analysis"
  * File should be `report/{yyyymmdd-HHMM}-{title}-gap-analysis.md`

**Remarks**:
* No technical words for developers like `variable`, `class`, `DB`, `file`
* Think markdown reader as "Most near by end user"

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
When output as Japanese, style should be `だ/である`

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
  * (keep) ブログコンテンツが表示される
  * (change) ライターが隠しモードにしている今鉄は表示されない
* invalid
  * (keep) 存在しないブログコンテンツにアクセスすると404が表示される

### ToBe
**挙動 - 正常/異常**
* 正常
  * (keep) ブログコンテンツが表示される
  * (change) ライターが隠しモードにしている今鉄は表示されない
* invalid
  * (keep) 存在しないブログコンテンツにアクセスすると404が表示される

### 問題
次の仕様が決まっていない
* 論理削除した記事は、編集者に見えるようにするのか?
* 誰が、最初の論理削除を行うのか? バッチか? いつそれを実行するのか?
```