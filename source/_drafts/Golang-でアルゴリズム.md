---
title: Golang でアルゴリズム
tags:
- Golang
- algorithm
id: golang-algorithm
---

アルゴリズムの複数兼ねて golang でまとめる。

- 全検索

# 全検索

リストを検索する際の以下について。

- for 全検索
- bit 全検索
- DFS （ Depth First Search ： 深さ優先探索）
- BFS （ Breadth-First Search ： 幅優先探索）
- IDDFS （ Iterative Deepening Depth First Search ： 反復深化探索）
- 順列
- 動的計画法・メモ化
- 探索範囲を狭めるアルゴリズム

## bit 全検索

所謂、要素の **組合せ** を求める際に利用する。  
リストに対して **ビットマスク** をあてることで実現する。  
ビット列の組合せを作ることができれば、リストの要素の組合せも求めることができる。  
例えば、 `01011` というビットマスクを `abcde` にあてると ` b de` が得られるといった具合だ。

```go
package main

import "fmt"

func main() {
	var n uint = 2                        // ビットの桁数
	for bit := 0; bit < (1 << n); bit++ { // 1 から n ビットまでの組合せ
		fmt.Printf("%b\n", bit)
	}
	fmt.Println("---")
	for bit := 1 << (n - 1); bit < (1 << n); bit++ { // n ビットの組合せ
		fmt.Printf("%b\n", bit)
	}
}
```

[参考](https://qiita.com/drken/items/7c6ff2aa4d8fce1c9361#bit-%E5%85%A8%E6%8E%A2%E7%B4%A2f)

## 順列

Golang には C++ でいう next_permutation はないので自作した。

```go
package main

import "fmt"

// リストから指定のインデッックスの要素を削除
func delEle(list []int, i int) []int {
	result := []int{}
	result = append(result, list[:i]...)
	result = append(result, list[i+1:]...)
	return result
}

func permutation(list []int) [][]int {
	var result [][]int            // 結果の格納
	var inner func(in, out []int) // 内部の再帰関数
	// in を捜査して、 out へ足していく
	inner = func(in, out []int) {
		if len(in) == 1 { // len(in) が 1 の時は結果を格納して終了
			result = append(result, append(out, in[0]))
		} else { // in から 1 つ取り出して out に付け足して再帰
			for i, n := range in {
				inner(delEle(in, i), append(out, n)) // 勘違いしやすいが参照渡しではないので引数は関数内部へコピー（clone_）される
			}
		}
	}
	inner(list, []int{}) // 初回呼び出しの out は空
	return result
}

func main() {
	fmt.Println(permutation([]int{1, 2, 3}))
}
```

# ソート

- 挿入ソート
- バブルソート
- 選択ソート
- 安定なソート
- シェルソート

# データ構造

- スタック
- キュー
- 連結リスト

# 探索

- 線形探索
- 二分探索

# 参考

- [AtCoder に登録したら次にやること ～ これだけ解けば十分闘える！過去問精選 10 問 ～](https://qiita.com/drken/items/fd4e5e3630d0f5859067)
    - 最後の参考書のまとめがいい
- 書籍 [プログラミングコンテスト攻略のためのアルゴリズムとデータ構造](https://book.mynavi.jp/ec/products/detail/id=35408)
    - もくじが参考になる
- 書籍 [世界で闘うプログラミング力を鍛える150問](https://www.amazon.co.jp/%E4%B8%96%E7%95%8C%E3%81%A7%E9%97%98%E3%81%86%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E5%8A%9B%E3%82%92%E9%8D%9B%E3%81%88%E3%82%8B150%E5%95%8F-%E3%83%88%E3%83%83%E3%83%97IT%E4%BC%81%E6%A5%AD%E3%81%AE%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9E%E3%81%AB%E3%81%AA%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AE%E6%9C%AC-Gayle-Laakmann-McDowell/dp/4839942390/ref=cm_cr_arp_d_product_top?ie=UTF8)
    - プログラミングによる採用テストにもふれている
- [競技プロ的なアルゴリズムのスライドのまとめ](https://yukicoder.me/wiki/slide)