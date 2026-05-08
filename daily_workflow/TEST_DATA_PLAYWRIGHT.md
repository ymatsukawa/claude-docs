# Data registration by playwright mcp
## Goal
Register many `xxx` data from UI using playwright mcp.

## Background - why
I began to test in staging server.

On the server, `xxx` data is few and test is less effective.
So, many data is required.

## Workflow
**Given-1: Run workflow as playwright mcp**:
* In workflow, **run `playwright mcp`**
* Create playwright script as `__test/{title}.js`

**Given-2: Statement**:
* Deployed code is in current repository's branch.
* Outputting script should be created under `__test/`

**If any questions because of vague, call `AskUserQuestion`.**.

### 1st step - Login as test user
* [ ] Login staging server
  * url: https://stg.example.com/login
  * id: `test_user`
  * password: `password`
* [ ] After logged in, you can see "Hello admin" in selector '#greeting'

### 2nd step - Register `xxx`
This step is mapped logic to @`/path/to/file.ext`

**Loop followed operation for 2 times**
* [ ] Login to staging server
* [ ] Click button to "Create xxx"
  * Wait indicator dismiss
* [ ] Click selector "#yagni-edit-mode"
* [ ] Input edit textarea
  * Limit Content length within 500
* [ ] Click "save"

## 3rd step - Iterate creation
* [ ] With script of "2nd step", create `xxx` for 3 times as trial.
* [ ] After checked 3 times as success, create 97 times by the script.

# Prohibition
* Edit source code in this repository.
* Check out branch without permission.
* Run playwright's screen shot, I don't need the shot.