---
marp: true
theme: gaia
class: invert
paginate: true
size: 16:9
---
<!-- _class: "invert lead" -->

# Claude Code で 600 テストケースを<br>書いて得た知見

2025/07/31
LINEDC Claude Code 実践 Meetup

---

# 自己紹介
<style scoped>
  .name {
    font-size: 100px;
    font-weight: bold;
  }

  li {
    line-height: 1.5;
  }
</style>

しらいし ゆうま
 <span class="name">白石 祐大</span>

- 株式会社よりそう テックリード
- 株式会社BOXIV AI エバンジェリスト
- 元ユニークビジョン株式会社
- X: @punksy2
![bg right:39%](profile.jpg)

---

## 概要
- Claude Code をフル活用してテストが無かったバックエンドの<br>プロジェクトに自動テストを導入
- 最終的に**全自動**で全コントローラのテストケースを<br>書けるようになった
- つまづいたポイントと解決方法を紹介

---

## 前提
- 技術スタック
    - Cloud Functions for Firebase
    - Cloud SQL
    - TypeScript
    - Prisma

---

## 課題: 毎回の承認が面倒
- CC に張り付いてコマンドを承認するのが面倒
- ポチポチしているうちに `rm -rf` してしまう可能性もある

### 解決方法
公式の DevContainer を導入して承認をスキップ

- 3 ファイルをコピペして VSCode 開くだけで OK
- 放置できるだけで体験が全然違う！おすすめ！


---

## 課題: 途中で作業を勝手に止める
- 長時間の作業中に処理が中断される
- 突然の `本日の作業はここまでとします。` 

### 解決方法
- TODO.md を作成し作業の状態を保持する
- シェルスクリプトでループ実行する
```bash
for i in {1..10}; do claude -p "/fill-todo"; done
```

---

## 課題: 余計な実装が無限増殖する
- 要求していない機能を勝手に追加する
- CLAUDE.md に実装方針を書いたが全然無視される

### 解決方法
- linter ルールで CC がやりがちな実装をエラーにする
- エラーメッセージで修正方法を明記するのが効果的


---
### 例: prisma.create の禁止
- `userFactory` を使って欲しいのに無限に `prisma.user.create` される

```
```

- prisma.create の禁止
- Factory 系クラスに追加のメソッド定義を禁止
- 記法の統一 (user.id! → user.id as string)
- 

CLAUDE.md に実装方針を書く → 効果は限定的

---

## 課題: 初手で生成されるコードの品質が低い
- 型エラーや linter エラーが多い状態で生成される
- 修正に時間がかかる

### 解決方法
hooks で即時フィードバック
- hooks で即時フォーマット
- hooks で即時型エラー通知

---

## まとめ

- 何度も実行すれば作業が終わるようにコマンドを設計する
- ルール < 既存のコード << プロンプト <<<< 静的解析
- 静的解析で自分の代わりに即時フィードバックしてくれる状態を作る


現在のプログラミングは、
- 自動テストループ
- 品質を担保するガード
が書ければ終わる。

参考:
https://aoai-ai-coding.mizchi.workers.dev/#5


## 番外編: 普段よく使うテクニック
- Kiro を参考にしたコマンドで最初に要件定義と設計をする
  - https://izanami.dev/post/11c5067c-d2f9-4945-8944-0d1c20c1263d
- git worktree で並行開発
  - PR まで上げさせる
  - PR 上でレビューする
  - CC に PR での指摘箇所を修正させる
  - 上記を繰り返す