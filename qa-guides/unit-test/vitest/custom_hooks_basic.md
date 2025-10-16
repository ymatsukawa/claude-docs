# カスタムフックテスト: 基本原則

## フックのレンダリング

カスタムフックをテストするには、`renderHook` 関数を使用する。

この関数は、フックの現在の戻り値を持つ `result` オブジェクトを返す。

まず、テスト対象となるシンプルなカウンターフック `useCounter` を用意する。

```javascript
// useCounter.js
import { useState, useCallback } from 'react'

export default function useCounter() {
  const [count, setCount] = useState(0)

  const increment = useCallback(() => setCount((x) => x + 1), [])

  return { count, increment }
}
```

このフックをテストするために `renderHook` を呼び出す。`result.current` を通じてフックの戻り値にアクセスし、初期状態を検証する。

```javascript
// useCounter.test.js
import { renderHook } from '@testing-library/react-hooks'
import useCounter from './useCounter'

test('カウンターフックが正しく利用される', () => {
  const { result } = renderHook(() => useCounter())

  expect(result.current.count).toBe(0)
  expect(typeof result.current.increment).toBe('function')
})
```

-----

## 更新のテスト

フック内部の状態更新（例: `increment` 関数の呼び出し）をテストするには、`act` ユーティリティを使用する。
状態更新の操作を `act` でラップすることにより、ブラウザ環境動作を正確にシミュレートし、アサーションを実行できる。

```javascript
// useCounter.test.js
import { renderHook, act } from '@testing-library/react-hooks'
import useCounter from './useCounter'

test('カウンターがインクリメントされる', () => {
  const { result } = renderHook(() => useCounter())

  act(() => {
    result.current.increment()
  })

  expect(result.current.count).toBe(1)
})
```

**注意点:** `result.current` はミュータブル（変更可能）なオブジェクトである。そのため、分割代入 (`const { count, increment } = result.current`) を使用すると、更新前の古い値を参照し続けてしまう。従って、常に `result.current` プロパティ経由でアクセスする必要がある。

-----

## Propsの提供

フックがPropsを受け取る場合、`renderHook` のコールバック内で引数を渡すことでテストできる。例えば、`useCounter` フックが `initialValue` を受け取るように変更する。

```javascript
// useCounter.js
import { useState, useCallback } from 'react'

export default function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  const increment = useCallback(() => setCount((x) => x + 1), [])

  return { count, increment }
}
```

テストでは、`renderHook` の第一引数であるコールバック関数内で、フックに初期値を渡す。

```javascript
// useCounter.test.js
import { renderHook, act } from '@testing-library/react-hooks'
import useCounter from './useCounter'

test('カスタム初期値からカウンターがインクリメントされる', () => {
  const { result } = renderHook(() => useCounter(9000))

  act(() => {
    result.current.increment()
  })

  expect(result.current.count).toBe(9001)
})
```

-----

## Props変更の処理

テスト中にフックへ渡すPropsを変更したい場合、`renderHook` が返す `rerender` 関数を使用する。`rerender` を呼び出すことで、フックが新しいPropsで再レンダリングされる。

これには2つの方法がある。

### 1. `initialProps` を使用する方法

`renderHook` の第二引数オプションで `initialProps` を指定し、`rerender` 関数に新しいPropsを渡す方法が推奨される。

```javascript
// useCounter.test.js
import { renderHook } from '@testing-library/react-hooks'
import useCounter from './useCounter'

test('更新された初期値にカウンターがリセットされる', () => {
  // initialValueをpropsとして渡す
  const { result, rerender } = renderHook(
    ({ initialValue }) => useCounter(initialValue),
    {
      initialProps: { initialValue: 0 }
    }
  )

  // initialValueを10にして再レンダリング
  rerender({ initialValue: 10 })

  // ここでリセット関数などをテストする（例）
  expect(result.current.count).toBe(10)
})
```

### 2. 変数を使用する方法

コールバックの外で変数を定義し、その変数を更新した後に引数なしで `rerender` を呼び出す方法もある。

```javascript
// useCounter.test.js
import { renderHook } from '@testing-library/react-hooks'
import useCounter from './useCounter'

test('更新された初期値にカウンターがリセットされる', () => {
  let initialValue = 0
  const { result, rerender } = renderHook(() => useCounter(initialValue))

  initialValue = 10
  rerender() // 引数なしで呼び出す

  // ここでリセット関数などをテストする（例）
  expect(result.current.count).toBe(10)
})
```
