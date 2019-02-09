---
title: Golangの文字列操作
tags:
- golang
- go
id: golang-strings
---

golang の文字列操作をまとめる。

- strings パッケージ
- strconv パッケージ

<!-- more -->

https://qiita.com/tchnkmr/items/b3d0b884db8d7d91fb1b

# strings パッケージ

以下のようなメソッドがある。

- func Contains(s, substr string) bool
    - s 内に substr が含まれるか
- func Count(s, sep string) int
    - s 内の sep をカウント
- func Fields(s string) []string
    - 文字列 s をひとつ以上の連続したホワイトスペースで分割
- func FieldsFunc(s string, f func(rune) bool) []string
    - 関数 `f(c)` が true を返す( c は Unicode コードポイント)位置で文字列 s を分割
- func Join(a []string, sep string) string
    - 文字列スライス a の要素を sep で繋げる
- func Replace(s, old, new string, n int) string
    - 文字越 s が含む old を new で前から n 個置き換える（n=-1 は全て）
- func Split(s, sep string) []string
    - s を sep で分割する
- func ToLower(s string) string
    - 文字列を小文字に変換する
- func ToUpper(s string) string
    - 文字列を大文字に変換する
- func Trim(s string, cutset string) string
    - 文字列 s 内の cutset をトリムする

# strconv パッケージ

以下のようなメソッドがある。

- 文字列から各型へ変換する
    - func Atoi(s string) (int, error)
    - func ParseBool(str string) (bool, error)
    - func ParseFloat(s string, bitSize int) (float64, error)
    - func ParseInt(s string, base int, bitSize int) (i int64, err error)
    - func ParseUint(s string, base int, bitSize int) (uint64, error)
- 数値かた文字列へ変換する
    - func Itoa(i int) string