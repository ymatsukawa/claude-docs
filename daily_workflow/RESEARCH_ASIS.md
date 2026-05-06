# Research AsIs Task
## Goal
Researching features and functions of

* `xxx`
* `xxx`

## Background - Why
I joined this repository yesterday and begun modifications for business,
but have no shallow/deep knowledge around this.

Another dev workers had been left, so I should do alone.
So, I want you as supporter to research features and functions.

## Definitions
**research**:
From current branch, to watch source code and gather information.
And to organize "features" and "functions".

**features**:
What it is.
This is for user's point of view.
In other words, it's required "acceptance test case".

**functions**:
How it works.
This is for dev worker's point of view.

* Why happens so?
* What is route?
* What is class/method and logics?
* What is DB table(s)?

## Prohibition
* Launching server including DB
* Editing exist files
* Accessing external API
* Read under `tmp/`

## Workflow
There are mainly two steps.

### Given workflow statement
* Output should be Japanese/UTF-8

### 1st step - for features
Goal is covering feature about target route(s)

Summary:

* Input:
  * route(s) of user's request
* Output:
  * markdown that acceptance test gherkin style

Detail:

* [ ] Receive route(s) from user
  * If route(s) is not pointed, stop workflow
* [ ] **Before start follows, AskUserQuestion 5+ for clear researching contents.**
* [ ] Research under route(s) controller / model / view
* [ ] Gather info as "acceptance test cases"
* [ ] Output info as "Gherkin Style test"
  * Follow "Output format → Gherkin Style Test Cases"
  * File should be `report/{yyyymmdd-HHMM}-{title}-feature.md`

### 2nd step - for functions
Goal is covering functions about features

Summary:

* Input:
  * Feature file path of 1st step
* Output:
  * Plant uml sequence diagram; related to targets
  * Plant uml ER diagram; related to targets

Detail:

* [ ] Receive feature file path from user
  * If feature file path is not pointed, stop workflow
* [ ] **Before start follows, AskUserQuestion 5+ for clear researching contents.**
* [ ] Research under route(s) / controller / model / view
* [ ] Gather info for sequence diagram
* [ ] Output sequence diagram
  * File should be `report/{yyyymmdd-HHMM}-{title}-function-sequence.puml`
* [ ] Gather info for ER diagram
* [ ] Output ER diagram
  * File should be `report/{yyyymmdd-HHMM}-{title}-function-er.puml`

### 3rd step - summarization
Goal is making summary report from "features" for PdM and PM.

Think situation: new PdM and PM joined and you explain what is this system without technical words.

Summary:

* Input:
  * Feature file path of 1st step
* Output:
  * Basic specification doc as markdown

Details:

* [ ] Receive feature file path from user
  * If feature file path is not pointed, stop workflow
* [ ] **Before start follows, AskUserQuestion 5+ for clear researching contents.**
* [ ] Read feature file(s) to understand specifications.
* [ ] Organize specifications
* [ ] Output basic specifications as markdown
  * Follow "Output format → Basic spec format"
  * File should be `report/{yyyymmdd-HHMM}-{title}-basic-step.md`

**Remarks**:
* No technical words for developers like `variable`, `class`, `DB`, `file`
* Think markdown reader as "Most near by end user"
* Format should follow

## Output format
### Gherkin Style Test Cases

<example>
## Feature: Listing blog contents

* Given: User is logged in as editor.
* When: User go to `/blogs`.
* And: Clicked button "list".
* Then: 10 blog items are shown in `/blogs`.
* And: At the bottom, user can see pagination.

## Feature: Editing blog contents
</example>
...
```

**Remarks**:
When output as Japanese, style should be
* Given: `だ/である`
* When: `だ/である`
* Then: `だ/である` + `こと`

Examples:
<example>
## Feature: ブログコンテンツの表示

* Given: ユーザーが編集者としてログインしている
* When: ユーザーが `/blogs` へ遷移する
* And: "list" ボタンをクリックする
* Then: `/blogs` ページに10個の項目が表示されること
* And: 底部にページネーション項目が表示されること　
</example>

### Basic spec format

```
## Feature list
* (1) Showing blog
  * Anonymous user can read blog content without authentication.
* (2) Editing blog
  * Authenticated writer can edit blog content from edit page.
* ...
* (N) {Feature Name}
  * {Explanation in 20 words}

## Feature specs
### (1) Showing blog
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

## Matters require ATTENTION
* Specification is status on 20260-04-01
* "Writer" means "Authenticated user"
```

**Remarks**:
When output as Japanese, style should be `だ/である`

Example:
```
## フイーチャー一覧
* (1) ブログ表示
  * 認証なしでブログ表示が可能
* (2) ブログ編集
  * 認証したライターはブログ編集可能

## フィーチャー仕様
### (1) ブログ表示
**仕様**
認証なしでブログ表示が可能

**利用者と期待値**
* ペルソナ: 不特定多数のユーザー
* トリガー: ブログへのアクセス
* 期待値: ブログコンテンツの表示

**挙動 - 正常/異常**
* 正常
  * ブログコンテンツが表示される
  * ライターが隠しモードにしている今鉄は表示されない
* invalid
  * 存在しないブログコンテンツにアクセスすると404が表示される

## 留意事項
* 本仕様は 2026-04-01 時点のものである
* "ライター" とは "ログイン済みのユーザー" を示している
```
