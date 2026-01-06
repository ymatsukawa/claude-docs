# カスタムフックのテスト: 応用原則

./custom_hooks_basic の基本原則に加え、応用原則例を示す

## 非同期（Async）フックのテスト

フックが非同期的な更新を行う場合、その更新は即座に `result.current` に反映されないことがある。一方で、`renderHook` はテストがフックの更新を待つためのユーティリティを返す。

最も基本的な非同期ユーティリティは **`waitForNextUpdate`** である。これを使うと、`async/await` 構文でフックの更新を待つことができる。

例として、`useCounter` フックに100ミリ秒後にカウントを増やす `incrementAsync` 関数を追加する。

```javascript
// useCounter.js
import { useState, useCallback } from 'react'

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  const increment = useCallback(() => setCount((c) => c + 1), [])

  const incrementAsync = useCallback(() => {
    setTimeout(() => {
      setCount((c) => c + 1)
    }, 100)
  }, [])

  return { count, increment, incrementAsync }
}
```

この `incrementAsync` をテストする場合は次のとおり

```javascript
// useCounter.test.js
import { renderHook } from '@testing-library/react-hooks'
import { useCounter } from './useCounter'

test('遅延後にカウンターがインクリメントされるべきである', async () => {
  const { result, waitForNextUpdate } = renderHook(() => useCounter())

  result.current.incrementAsync()

  await waitForNextUpdate()

  expect(result.current.count).toBe(1)
})
```

`waitForNextUpdate` を `await` する際、非同期の `act` ラッパーが自動的に適用されるため、`incrementAsync` の呼び出しを `act` で囲む必要はない。

**より複雑な非同期のシナリオ（例：特定の条件が満たされるまで待機する）では、`waitFor` ユーティリティが有効である。**

## エラーハンドリング

フックが意図的にエラーをスローするかのテストは、`result.error` プロパティを使用して、直前のレンダーでスローされたエラーへアクセス可能。

例えば、`useCounter` フックが特定のカウント数に達した場合にエラーをスローするように変更する。

```javascript
// useCounter.js
import { useState, useCallback } from 'react'

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  if (count > 2) {
    throw new Error('Count exceeded')
  }

  const increment = useCallback(() => setCount((c) => c + 1), [])

  return { count, increment }
}
```

このエラーをテストする場合は次の通り

```javascript
// useCounter.test.js
import { renderHook, act } from '@testing-library/react-hooks'
import { useCounter } from './useCounter'

test('カウンターが2を超えた場合にエラーをスローすべきである', () => {
  const { result } = renderHook(() => useCounter())

  act(() => {
    result.current.increment() // count is 1
    result.current.increment() // count is 2
    result.current.increment() // count is 3, throws error
  })

  expect(result.error).toEqual(new Error('Count exceeded'))
})
```

**React 18以降のバージョンでは、Testing Libraryの `renderHook` の仕様が変更され、エラーの捕捉方法が変わる場合がある。Jest でいう `.toThrow()` マッチャーなど、テストランナーの機能を使ってエラーを検証する方法も一般的である。**

## コンテキスト（Context）の利用

フックがコンテキストから値を取得する場合、`renderHook` の **`wrapper`** オプションを使用する。これにより、テスト対象のフックを任意のコンポーネント（通常は `Provider`）でラップできる。

例として、カウンターの増分ステップをコンテキストから取得する `useCounter` フックを考える。

```javascript
// counter.js
import React, { useState, useContext, useCallback } from 'react'

const CounterStepContext = React.createContext(1)

export function CounterStepProvider({ step, children }) {
  return (
    <CounterStepContext.Provider value={step}>
      {children}
    </CounterStepContext.Provider>
  )
}

export function useCounter(initialValue = 0) {
  const step = useContext(CounterStepContext)
  const [count, setCount] = useState(initialValue)

  const increment = useCallback(() => setCount((c) => c + step), [step])

  return { count, increment }
}
```

このフックをテストするには次の通り。

```javascript
// counter.test.js
import { renderHook, act } from '@testing-library/react-hooks'
import { CounterStepProvider, useCounter } from './counter'

test('カスタムステップでインクリメントされるべきである', () => {
  const wrapper = ({ children }) => <CounterStepProvider step={2}>{children}</CounterStepProvider>

  const { result } = renderHook(() => useCounter(), { wrapper })

  act(() => {
    result.current.increment()
  })

  expect(result.current.count).toBe(2)
})
```

これにより、フックが必要とするコンテキストをテスト環境内で提供し、コンテキストに依存したロジックを正確に検証することが可能になる。
