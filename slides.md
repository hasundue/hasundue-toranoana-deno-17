---
author: Chiezo (@hasundue)
date: 2024-07-31
---

# Deno の依存アップデータを作っている話

---

## 自己紹介

*アカデミアをドロップアウトしてのんびりしている工学博士おじさん*

- GitHub: hasundue
- Bluesky: @chiezo.bsky.social

### 気ままに色々作ってます

- denopendabot ★40 - Dependabot for Deno projects
- **molt         ★63 - Update dependencies the Deno way**
- molt-action  ★ 8 - GitHub Action for molt
- amber        ★ 8 - Mocking Deno APIs with ease
- ...

### たまに本家にコントリビュートもしてます

- denoland/std
  - feat(testing/mock): enable spy to accept a class constructor (#3766)
- denoland/wasmbuild
  - fix: make rustup optional (#141)
- ...

---

## モチベーション

- deno-udd ★327 の更新停止 (2022.12)

### Fork する？

- レジストリ毎の正規表現をメンテするのしんどそう…
- インタフェースが自分のニーズに合わない…

### 全く違う設計で新しく作ってみよう

- ソースコードから依存を探す → deno_graph
- 最新のバージョンを取得する → レジストリのリダイレクトを利用

→ めちゃシンプルに書けるのでは？

---

## リダイレクトを利用した最新バージョンの取得

更新前のURL:

```typescript
"https://deno.land/std@0.222.0/bytes/copy.ts"
```

バージョン部分の削除:

```typescript
"https://deno.land/std/bytes/copy.ts"
```

レジストリにリクエスト:

```typescript
const res = await fetch("https://deno.land/std/bytes/copy.ts", {
  method: "HEAD" 
});
await res.arrayBuffer(); // body を確実に捨てる

if (!res.redirected) {
  return;
}
return res.url;
```

結果:

```typescript
"https://deno.land/std@0.224.0/bytes/copy.ts"
```

---

## 対応できたレジストリ

### 公式

- [deno.land/std](https://deno.land/std)
- [deno.land/x](https://deno.land/x)

### サードパーティ

- [esm.sh](https://esm.sh)
- [unpkg.com](https://unpkg.com)

## 対応できなかったレジストリ

- [cdn.jsdelivr.net](https://cdn.jsdelivr.net)
- [cdn.skypack.dev](https://cdn.skypack.dev)
- [esm.run](https://esm.run)
- [denopkg.com](https://denopkg.com)
- [ga.jspm.io](https://ga.jspm.io)
- [pax.deno.dev](https://pax.deno.dev)
- [raw.githubusercontent.com](https://github.com)
- [x.nest.land](https://x.nest.land)

---

## しかし…

### JSR の登場

```json
{
  "imports": {
    "@luca/flag": "jsr:@luca/flag@^1.0.0",
  }
}
```

先ほどの手法は使えない...

### どうする？

なんやかんや正攻法で対応

---

## Molt の現在

- フォーマット
    - `http:`, `https:`
    - `jsr:`
    - `npm:`
- 場所
    - ソースコード (`.ts` 他)
    - 設定ファイル (`deno.json`, `deno.jsonc`)
    - ロックファイル (`deno.lock`)
- バージョン
    - 固定 (`0.1.0`)
    - 範囲 (`^0.1.0`, `~0.1.0`, ...)

全部対応！

### イチオシ

- ロックファイルを依存毎に更新できます

---

## デモ (Molt CLI)

---

## まとめ
