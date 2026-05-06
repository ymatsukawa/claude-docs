# Research AsIs Task
## Goal
Researching the features and functions of:

* `xxx`
* `xxx`

## Background - Why
I joined this repository yesterday and began modifications for the business,
but have neither shallow nor deep knowledge of it.

The other dev worker has left, so I have to do this alone.
So, I want you as a supporter to research features and functions.

## Definitions
**research**:
From the current branch, read the source code and gather information.
Then organize the "features" and "functions".

**features**:
What it is.
This is from the user's point of view.
In other words, this is the required "acceptance test case".

**functions**:
How it works.
This is from the developer's point of view.

* Why does it happen?
* What is the route?
* What are the classes/methods and their logic?
* What DB table(s) are involved?

## Prohibition
* Launching server including DB
* Editing exist files
* Accessing external API
* Reading anything under `tmp/`

## Workflow
There are mainly two steps.

### Workflow ground rules
* Output should be Japanese/UTF-8

### 1st step - for features
Goal is to cover the features of the target route(s).

Summary:

* Input:
  * route(s) of user's request
* Output:
  * markdown that acceptance test gherkin style

Detail:

* [ ] Receive route(s) from the user
  * If no route is specified, stop the workflow.
* [ ] **Before start follows, AskUserQuestion 5+ times to clarify the research scope.**
* [ ] Research the controller / model / view for the route(s)
* [ ] Gather information to form "acceptance test cases"
* [ ] Output the information as "Gherkin-style tests"
  * Follow "Output format → Gherkin Style Test Cases"
  * File should be `report/{yyyymmdd-HHMM}-{title}-feature.md`

### 2nd step - for functions
Goal is to cover the functions behind the features.

Summary:

* Input:
  * Feature file path of 1st step
* Output:
  * PlantUML sequence diagram; related to targets
  * PlantUML ER diagram; related to targets

Detail:

* [ ] Receive the feature file path from the user.
  * If no feature file path is specified, stop the workflow.
* [ ] **Before start follows, AskUserQuestion 5+ times to clarify the research scope.**
* [ ] Research the route(s) / controller / model / view
* [ ] Gather info for sequence diagram
* [ ] Output the sequence diagram
  * File should be `report/{yyyymmdd-HHMM}-{title}-function-sequence.puml`
* [ ] Gather info for ER diagram
* [ ] Output the ER diagram
  * File should be `report/{yyyymmdd-HHMM}-{title}-function-er.puml`

### 3rd step - summarization
Goal is to make a summary report from the "features" for PdM and PM.

Imagine this situation: a new PdM and PM have joined, and you explain what this system is without using technical terms.

Summary:

* Input:
  * Feature file path of 1st step
* Output:
  * Basic specification doc as markdown

Details:

* [ ] Receive the feature file path from the user.
  * If no feature file path is specified, stop the workflow.
* [ ] **Before start follows, AskUserQuestion 5+ times to clarify the research scope.**
* [ ] Read feature file(s) to understand specifications.
* [ ] Organize the specifications
* [ ] Output the basic specifications as markdown
  * Follow "Output format → Basic spec format"
  * File should be `report/{yyyymmdd-HHMM}-{title}-basic-spec.md`

**Remarks**:
* Avoid developer-only technical terms like `variable`, `class`, `DB`, `file`.
* Think of the markdown reader as "closest to the end user".
* The output format should follow the spec below.

## Output format
### Gherkin Style Test Cases

<example>
## Feature: Listing blog contents

* Given: User is logged in as editor.
* When: The user goes to `/blogs`.
* And: The user clicks the "list" button.
* Then: 10 blog items are shown in `/blogs`.
* And: At the bottom, the user can see pagination.

## Feature: Editing blog contents
</example>

**Remarks**:
When outputting in Japanese, the style should be:
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
  * An authenticated writer can edit blog content from the edit page.
* ...
* (N) {Feature Name}
  * {Explanation in 20 words}

## Feature specs
### (1) Showing blog
**What it does**
An anonymous user can read the blog without authentication.

**Who uses it & when**
* Persona: Anonymous user
* Trigger: Accessing blog
* Expects: Showing blog contents

**Current Behavior - valid/invalid**
* valid
  * Blog content is shown.
  * Blog content is not shown if the content is in hidden mode.
* invalid
  * 404 error when accessing non-existent blog content.

## Matters require ATTENTION
* This specification is current as of 2026-04-01.
* "Writer" means "Authenticated user"
```

**Remarks**:
When outputting in Japanese, the style should be: `だ/である`

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
  * ライターが隠しモードにしている場合は表示されない
* invalid
  * 存在しないブログコンテンツにアクセスすると404が表示される

## 留意事項
* 本仕様は 2026-04-01 時点のものである
* "ライター" とは "ログイン済みのユーザー" を示している
```
