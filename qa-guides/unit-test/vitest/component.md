# コンポーネントテスト: 前提

## モック

- [ ] top level で用意する
- [ ] 個別のテストケースは用意しない
- [ ] **コンポーネント自体をモックしない**

```javascript
import { vi } from "vitest"

// firebase のモック
vi.mock('firebase/app', () => ({
  initializeApp: vi.fn(),
  getApps: vi.fn(() => [{}]),
  getApp: vi.fn(() => ({}))
}))

// hooks のモック
vi.mock('@/features/login/useLogin', () => ({
  useLogin: vi.fn()
}))

// 以降、テストケース
```

## ライブラリーの読込ローディング

- [ ] `require` を使わない
  - 互換性問題が発生する場合があるから
- [ ] どうしても必要な場合は top level で `import` を使う。require-to-import-example を参照

```
NG: vitest では require を使えない
const { useLogin } = require('@/features/login/useLogin')

OK: top level で import 宣言する
import { useLogin } from '@/features/login/useLogin'
```

## スタイルに対するテスト

- [ ] コンポーネントに紐つく `styles.ts` 自体のテストは不要

# コンポーネントテスト: 方針

## テスト原則

### 0. コンポーネントテストでは、カスタム hooks の内部詳細をテストしない

実装詳細ではなく「動作」をテストする

- NG
  - `renderHook` を使った単独のHookテスト
  - Hook内部の状態(state)やメソッドの直接的なテスト
- OK
  - Hookを使用するコンポーネントを通じた統合テスト
  - ユーザーの操作と結果に焦点を当てたテスト

```javascript
// NG: 悪い例
const { result } = renderHook(() => useLogin())
expect(result.current.isFailed).toBe(false)

// OK: 良い例
render(<LoginComponent />)
await user.click(screen.getByRole('button', { name: 'ログイン' }))
expect(await screen.findByText('ログインに成功しました')).toBeInTheDocument()
```

> "Your hook is easy to test by just testing the components using it"
> — Kent C. Dodds [Testing Implementation Details](https://kentcdodds.com/blog/testing-implementation-details)

### 1. `wrapper`という変数名を使わない

`render`の戻り値に`wrapper`という名前を使うのは、enzymeの古い慣習である。

```javascript
// NG: 悪い例
const wrapper = render(<Example prop="1" />)
wrapper.rerender(<Example prop="2" />)

// OK: 良い例
const { rerender } = render(<Example prop="1" />)
rerender(<Example prop="2" />)
```

### 2. `screen` を使用する

`screen`を使うことで、クエリの管理を簡潔にする

```javascript
// NG: 悪い例
const { getByRole } = render(<Example />)
const errorMessageNode = getByRole('alert')

// OK: 良い例
import { render, screen } from '@testing-library/react-hooks'

render(<Example />)
const errorMessageNode = screen.getByRole('alert')
```

### 3. 適切なアサーションを使う

`@testing-library/jest-dom` を使用し、より適切なエラーメッセージを取得する

```javascript
// NG: 悪い例
const button = screen.getByRole('button', { name: /disabled button/i })
expect(button.disabled).toBe(true)

// OK: 良い例
import '@testing-library/jest-dom'

const button = screen.getByRole('button', { name: /disabled button/i })
expect(button).toBeDisabled()
```

### 4. 不必要に `act` でラップしない

`render` と `fireEvent` は既に `act` でラップされている。

```javascript
// NG: 悪い例
act(() => {
  render(<Example />)
})
const input = screen.getByRole('textbox', { name: /choose a fruit/i })
act(() => {
  fireEvent.keyDown(input, { key: 'ArrowDown' })
})

// OK: 良い例
render(<Example />)
const input = screen.getByRole('textbox', { name: /choose a fruit/i })
fireEvent.keyDown(input, { key: 'ArrowDown' })
```

### 5. ユーザー操作を想定してクエリを使う

getByTestId を使わない。クエリを使用する

```javascript
// NG: 悪い例
// DOM: <label>Username</label><input data-testid="username" />
screen.getByTestId('username')

// OK: 良い例
// DOM: <label for="username">Username</label><input id="username" type="text" />
screen.getByRole('textbox', { name: /username/i })
```

**クエリの優先順位**

1. `getByRole` - アクセシビリティを考慮した最も推奨される方法
2. `getByLabelText` - フォーム要素用
3. `getByPlaceholderText` - プレースホルダーがある場合
4. `getByText` - 非インタラクティブ要素用
5. `getByDisplayValue` - フォーム要素の現在値
6. `getByAltText` - 画像用
7. `getByTitle` - タイトル属性がある場合
8. `getByTestId` - 最後の手段

### 6. `container` を使ったクエリを避ける

```javascript
// NG: 悪い例
const { container } = render(<Example />)
const button = container.querySelector('.btn-primary')
expect(button).toHaveTextContent(/click me/i)

// OK: 良い例
render(<Example />)
screen.getByRole('button', { name: /click me/i })
```

### 7. テキストでクエリする

実際のテキストコンテンツでクエリすることで、翻訳やコンテンツ変更を検知できる。

```javascript
// NG: 悪い例
screen.getByTestId('submit-button')

// OK: 良い例
screen.getByRole('button', { name: /submit/i })
```

### 8. `*ByRole` を優先的に使う

`*ByRole`クエリは、アクセス可能な名前で要素検索が可能で、複数要素にまたがるテキストも扱える。

```javascript
// DOM: <button><span>Hello</span> <span>World</span></button>

// NG: 悪い例 - エラーになる
screen.getByText(/hello world/i)

// OK: 良い例 - 動作する
screen.getByRole('button', { name: /hello world/i })
```

### 9. 不要なアクセシビリティ属性を追加しない

セマンティックHTMLで十分な場合は、余分な属性を追加しない。

```javascript
// NG: 悪い例
render(<button role="button">Click me</button>)

// OK: 良い例
render(<button>Click me</button>)
```

### 10. `@testing-library/user-event` を使う

`fireEvent`よりも実際のユーザー操作に近い動作をシミュレートする。

```javascript
// NG: 悪い例
import { fireEvent } from '@testing-library/react-hooks'
fireEvent.change(input, { target: { value: 'hello world' } })

// OK: 良い例
import userEvent from '@testing-library/user-event'
const user = userEvent.setup()
await user.type(input, 'hello world')
```

### 11. `query*` は非存在チェックのみに使う

```javascript
// NG: 悪い例 - 要素が存在する場合
expect(screen.queryByRole('alert')).toBeInTheDocument()

// OK: 良い例 - 要素が存在する場合
expect(screen.getByRole('alert')).toBeInTheDocument()

// OK: 良い例 - 要素が存在しない場合
expect(screen.queryByRole('alert')).not.toBeInTheDocument()
```

### 12. `find*` を使って非同期要素を待つ

```javascript
// NG: 悪い例
const submitButton = await waitFor(() =>
  screen.getByRole('button', { name: /submit/i })
)

// OK: 良い例
const submitButton = await screen.findByRole('button', { name: /submit/i })
```

### 13. `waitFor` を適切に使う

**13-1. 空のコールバックを渡さない**

```javascript
// NG: 悪い例
await waitFor(() => {})
expect(window.fetch).toHaveBeenCalledWith('foo')

// OK: 良い例
await waitFor(() => expect(window.fetch).toHaveBeenCalledWith('foo'))
```

**13-2. 単一のアサーションを使う**


```javascript
// NG: 悪い例
await waitFor(() => {
  expect(window.fetch).toHaveBeenCalledWith('foo')
  expect(window.fetch).toHaveBeenCalledTimes(1)
})

// OK: 良い例
await waitFor(() => expect(window.fetch).toHaveBeenCalledWith('foo'))
expect(window.fetch).toHaveBeenCalledTimes(1)
```

**副作用を`waitFor`の外に置く**

```javascript
// NG: 悪い例
await waitFor(() => {
  fireEvent.keyDown(input, { key: 'ArrowDown' })
  expect(screen.getAllByRole('listitem')).toHaveLength(3)
})

// OK: 良い例
fireEvent.keyDown(input, { key: 'ArrowDown' })
await waitFor(() => {
  expect(screen.getAllByRole('listitem')).toHaveLength(3)
})
```

### 14. 明示的なアサーションを使う

```javascript
// NG: 悪い例 - アサーションなし
screen.getByRole('alert', { name: /error/i })

// OK: 良い例 - 明示的なアサーション
expect(screen.getByRole('alert', { name: /error/i })).toBeInTheDocument()
```
