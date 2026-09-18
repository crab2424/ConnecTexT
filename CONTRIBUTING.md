# 開発の進め方（2人用の運用ルール）

GitHub での事故（相手の作業を消す・壊す、秘密情報を公開する）を防ぐための最小ルール。
慣れてきたら見直してよい。変更するときはこのファイルを PR で更新する。

## 1. ブランチの使い方

- **`main` は「動く・合意済み」の状態だけを置く場所。直接 push しない。**
- 作業は必ず自分のブランチを切って行い、Pull Request（PR）で `main` に取り込む。
- ブランチ名は `種類/内容` の形で、英数字とハイフンを使う（日本語名は一部ツールで文字化けするため避ける）。

  | 種類 | 用途 | 例 |
  |---|---|---|
  | `feature/` | 機能追加 | `feature/sentence-splitter` |
  | `fix/` | 不具合修正 | `fix/edge-arrow-direction` |
  | `docs/` | 設計メモ・README | `docs/relation-types` |
  | `chore/` | 設定・依存関係 | `chore/setup-vite` |

- 1つのブランチには1つの目的だけ入れる。大きくなりそうなら分ける（PR が小さいほど確認が楽）。

## 2. 作業の流れ

```bash
# 1. 最新の main から始める
git switch main
git pull

# 2. 自分のブランチを切る
git switch -c feature/xxx

# 3. 作業してコミット（こまめに、意味のある単位で）
git add -A
git commit -m "文分割: 「。」「！」「？」で分割する関数を追加"

# 4. push して PR を作る
git push -u origin feature/xxx
# → GitHub 上で「Compare & pull request」を押す
```

- PR を作ったら相手に一言知らせる。相手が目を通して問題なければマージする（自分でマージしてもよいが、**相手が一度は見る**）。
- マージ後はブランチを削除してよい（GitHub の "Delete branch" ボタン）。
- 長く作業するブランチは、ときどき `main` を取り込んで差分を小さく保つ:

  ```bash
  git switch feature/xxx
  git merge main
  ```

## 3. やらないこと

| 禁止 | 理由 |
|---|---|
| `main` への直接 push | 相手が知らないうちに main が変わる。GitHub のブランチ保護で物理的に止める |
| `git push --force` / `-f`（共有ブランチに対して） | 相手のコミットを消す。自分だけのブランチでも原則使わない |
| `git reset --hard` / `git checkout -- .` を確認せずに実行 | 未コミットの作業が消える。実行前に `git status` で確認 |
| API キー・パスワードのコミット | 公開リポジトリでは即座に悪用される。`.env` は `.gitignore` 済み。誤ってコミットしたら**キーを無効化**してから履歴を消す |
| `node_modules/` `dist/` のコミット | 巨大で差分が読めなくなる。`.gitignore` 済み |
| 相手のブランチへの直接コミット | 変更したいときは PR か、相手に一言 |

## 4. コミットメッセージ

- 1行目に「何をしたか」を日本語で。例: `接続表現辞書に「したがって」「ゆえに」を追加`
- 「なぜ」が自明でないときは空行を挟んで本文に書く。
- `fix`, `update`, `修正` だけのメッセージは避ける（後から履歴を追えない）。

## 5. 衝突（コンフリクト）が起きたら

```bash
git switch feature/xxx
git merge main
# CONFLICT と出たファイルを開き、<<<<<<< ======= >>>>>>> の部分を手で直す
git add <直したファイル>
git commit
```

- 相手の変更を消してよいか迷ったら、消さずに聞く。
- 設計メモ（`crab_memory/`）は同じファイルを2人で触ると衝突しやすい。大きく書き換えるときは先に一言。

## 6. GitHub 側の設定（オーナーが1回だけ行う）

リポジトリの **Settings → Branches → Add branch ruleset**（または Branch protection rule）で `main` に対して:

- [x] Require a pull request before merging
- [x] Block force pushes
- [x] Restrict deletions

これで「うっかり main に push」「うっかり force push」が GitHub 側で拒否される。

## 7. 各自の git 設定（1回だけ）

```bash
# pull したとき、分岐していればマージで解決する（謎のエラーで止まらない）
git config pull.rebase false

# 名前とメールが GitHub アカウントと一致しているか確認
git config user.name
git config user.email
```

## 8. 設計メモの扱い

- `crab_memory/` は「決めたこと・決めていないこと・理由」を残す場所（[crab_memory/README.md](crab_memory/README.md) 参照）。
- 方針が変わったら該当ファイルを書き換える。履歴は git に残るので古い記述を残す必要はない。
- 決定事項は `03_未決事項と質問.md` の決定記録に追記する。
