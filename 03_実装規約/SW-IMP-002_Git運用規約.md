# Git 運用規約

**文書番号:** SW-IMP-002  
**版数:** 1.0  
**作成日:** 2026-06-15

---

## 1. ブランチ戦略

本プロジェクトは **Git Flow** ベースのブランチ戦略を採用する。

```
main          ─── 本番リリース済みコード (タグ付き)
develop       ─── 統合ブランチ (次期リリース向け)
feature/xxx   ─── 機能開発ブランチ (develop から派生)
fix/xxx       ─── バグ修正ブランチ (develop から派生)
release/x.x   ─── リリース準備ブランチ
hotfix/xxx    ─── 緊急修正ブランチ (main から派生)
```

### 1.1 ブランチ命名規則

| ブランチ種別 | 命名規則 | 例 |
|---|---|---|
| 機能開発 | `feature/<チケットID>-<概要>` | `feature/SW-123-uart-driver` |
| バグ修正 | `fix/<チケットID>-<概要>` | `fix/SW-456-wdt-timeout` |
| リリース | `release/<version>` | `release/1.2.0` |
| 緊急修正 | `hotfix/<version>-<概要>` | `hotfix/1.1.1-adc-overflow` |

---

## 2. コミット規約

### 2.1 コミットメッセージ形式

[Conventional Commits](https://www.conventionalcommits.org/) 形式を採用する。

```
<type>(<scope>): <summary>

[body]

[footer]
```

**type 一覧:**

| type | 用途 |
|---|---|
| `feat` | 新機能追加 |
| `fix` | バグ修正 |
| `refactor` | 機能変更を伴わないコード改善 |
| `test` | テストコード追加・修正 |
| `docs` | 文書のみの変更 |
| `chore` | ビルド設定・ツール変更 |
| `perf` | 性能改善 |

**例:**

```
feat(hal_uart): UART 受信割り込みハンドラを実装

- DMA 転送完了割り込みで受信バッファへ書き込む
- オーバーランエラー検出時は ERR フラグをセット

Refs: SW-123
```

### 2.2 コミット粒度

- 1コミット = 1論理的変更単位
- 「動作する状態」でコミットすること (WIP コミットは禁止)
- ビルドエラーが出る状態でのコミット禁止

---

## 3. マージ規則

### 3.1 Pull Request / マージリクエスト

- `develop` および `main` への直接プッシュ禁止
- マージには **最低1名** のコードレビュー承認が必要
- CI (ビルド・静的解析・単体テスト) が全て Pass していること
- コンフリクト解消は PR 作成者が行うこと

### 3.2 マージ方式

| ブランチ | マージ方式 | 理由 |
|---|---|---|
| feature → develop | Squash merge または Rebase | 履歴の簡潔化 |
| release → main | Merge commit | リリース境界を明確化 |
| hotfix → main | Merge commit | 緊急修正の追跡容易化 |

---

## 4. タグ規則

### 4.1 タグ命名

```
v<major>.<minor>.<patch>[-<YYYYMMDD>]

例:
v1.0.0-20260615     # 正式リリース
v1.0.0-rc1          # リリース候補
```

### 4.2 タグ付け手順

1. `release/x.x` ブランチでリリース確認テスト完了
2. `main` へマージ
3. SW リーダが署名付きタグを作成: `git tag -s v1.0.0 -m "Release v1.0.0"`
4. タグをリモートへプッシュ: `git push origin v1.0.0`

---

## 5. コミット前チェック (Pre-commit)

以下を全て Pass してからコミットすること。

```bash
# 静的解析
make lint

# ビルド確認
make all

# 単体テスト
make test
```

.git/hooks/pre-commit に自動チェックスクリプトを配置することを推奨する。

---

## 6. 禁止事項

- `git push --force` を `main` / `develop` ブランチへ実行することの禁止
- バイナリファイル (`.bin`, `.hex`, `.elf`) を直接コミットすることの禁止
  - ビルド成果物は CI アーティファクトとして管理すること
- 認証情報・秘密鍵をコミットすることの禁止
  - `.gitignore` に設定し、シークレット管理ツールを使用すること

---

## 7. 改訂履歴

| 版数 | 日付 | 変更内容 | 担当 |
|---|---|---|---|
| 1.0 | 2026-06-15 | 初版作成 | SW開発グループ |
