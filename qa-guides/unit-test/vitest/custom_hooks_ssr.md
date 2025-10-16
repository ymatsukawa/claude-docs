# カスタムフック: SSRテスト方向性

## SSRでのフックのレンダリング

サーバーサイドでフックをテストする場合、クライアントサイドとは異なるレンダリング結果の検証が必要となる。特に、クライアントサイドで実行される `useEffect` や `useLayoutEffect` のようなエフェクトフックは、サーバーサイドでは実行されない。

`renderHook` 関数は、第一引数にテスト対象のフックを、第二引数にオプションを受け取る。SSRのテストを行うには、このオプションオブジェクトに `hydrate` プロパティを含める。

- **`hydrate: false`**: デフォルトの動作であり、通常のクライアントサイドレンダリングをシミュレートする。
- **`hydrate: true`**: フックがまずサーバーでレンダリングされ、その後クライアントで「ハイドレーション」されるというSSRの動作をシミュレートする

### テストのシナリオ

例として、`useEffect` を使用してマウント時にカウンターをインクリメントするフックを考える。

```javascript
// useCounter.js
import { useState, useEffect } from 'react'

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)

  useEffect(() => {
    // このエフェクトはクライアントでのみ実行される
    setCount((c) => c + 1)
  }, [])

  return { count }
}
```

このフックをSSR環境でテストする場合、`renderHook` は2つのレンダリングサイクルをシミュレートする。

1.  **サーバーレンダリング**: 最初のレンダリング。`useEffect` は実行されない。
2.  **クライアントハイドレーション**: `hydrate: true` の場合に行われる再レンダリング。ここで `useEffect` が実行される。

### テストコードの実装

テストでは、まずサーバー上での初期レンダリング結果を検証し、次にクライアントでのハイドレーション後の結果を検証する。`rerender` 関数を呼び出すことで、ハイドレーションのプロセスをトリガーできる。

```javascript
// useCounter.test.js
import { renderHook } from '@testing-library/react-hooks'
import { useCounter } from './useCounter'

test('SSRにおいてカウンターが正しくハイドレーションされるべきである', () => {
  // サーバーでの初期レンダリングをシミュレート
  const { result, rerender } = renderHook(() => useCounter())

  // サーバーではuseEffectが実行されないため、countは初期値の0である
  expect(result.current.count).toBe(0)

  // クライアントでのハイドレーションをシミュレート
  rerender()

  // ハイドレーション後にuseEffectが実行され、countは1になる
  expect(result.current.count).toBe(1)
})
```

`@testing-library/react-hooks/server` エントリポイントからインポートした `renderHook` を使用すると、自動的にハイドレーションを無効化し、`rerender` を呼び出しても何もしなくなる。これにより、サーバー環境のみのテストが容易になる。

```javascript
// useCounter.ssr.test.js
import { renderHook } from '@testing-library/react-hooks/server'
import { useCounter } from './useCounter'

test('SSRにおいてカウンターがインクリメントされないことを確認する', () => {
  const { result, rerender } = renderHook(() => useCounter())

  // サーバーではuseEffectは実行されない
  expect(result.current.count).toBe(0)

  // /server からインポートした場合、rerenderは何も行わない
  rerender()

  // したがって、countは変わらず0のままである
  expect(result.current.count).toBe(0)
})
```

このように `hydrate` オプションやサーバー用のエントリポイントを使い分けることで、SSR環境におけるフックの動作を正確にテストすることが可能となる。
