+++
date = "2018-12-19T00:00:00Z"
title = "Android: RecyclerView Adapter比較"
tags = ["android", "recyclerview", "jetpack"]
blogimport = true
type = "post"
draft = true
+++

## Groupie

|||
|-----|-----|
| Data Binding | ⭕ |
| Kotlin Android Extensions | ⭕ |
| Paging | ❌ |
| DiffUtil | ⭕ |

## Epoxy

|||
|-----|-----|
| Data Binding | ⭕ |
| Kotlin Android Extensions | ⭕ |
| Paging | ⭕ |
| DiffUtil | ⭕ |

- changed payloadを自動的にやってくれる。
- EpoxyModelにview情報を定義し、それに追従する形でViewがdisplayされる
    - Modelがベース
