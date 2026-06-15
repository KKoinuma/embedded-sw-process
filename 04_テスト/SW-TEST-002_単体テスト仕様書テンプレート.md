# 単体テスト仕様書

**文書番号:** SW-TEST-002  
**版数:** [x.x]  
**対象モジュール:** [モジュール名]  
**作成日:** [YYYY-MM-DD]  
**作成者:** [氏名]  
**承認者:** [氏名]

---

## 1. 目的・対象

本仕様書は、[モジュール名] の単体テストのテストケースを定義する。

### 1.1 対象ファイル

| ファイル | 版数/コミットハッシュ |
|---|---|
| [module].c | |
| [module].h | |

### 1.2 テスト環境

| 項目 | 内容 |
|---|---|
| テストフレームワーク | Unity |
| カバレッジツール | gcov |
| 実行環境 | Host PC (x86-64) / Target Board |

---

## 2. テストケース一覧

| TC-ID | 対象関数 | テスト観点 | 分類 | 合否基準 |
|---|---|---|---|---|
| TC-001 | MODULE_Init() | 正常初期化 | 正常系 | ERR_OK を返す |
| TC-002 | MODULE_Init() | NULL ポインタ渡し | 異常系 | ERR_NULL_PTR を返す |
| TC-003 | MODULE_Init() | パラメータ上限値 | 境界値 | ERR_OK を返す |
| TC-004 | MODULE_Init() | パラメータ上限+1 | 境界値 | ERR_INVALID_PARAM を返す |
| TC-005 | MODULE_Process() | 正常処理 | 正常系 | 期待値と一致 |
| TC-006 | MODULE_Process() | 未初期化状態での呼び出し | 異常系 | ERR_NOT_INIT を返す |

---

## 3. テストケース詳細

### TC-001: MODULE_Init() 正常初期化

| 項目 | 内容 |
|---|---|
| **目的** | 正常なパラメータで初期化が成功することを確認 |
| **前提条件** | モジュール未初期化状態 |
| **入力** | `config = {.param1 = 10, .param2 = 100}` |
| **期待結果** | 戻り値 `ERR_OK`、内部状態 `STATE_INITIALIZED` |
| **実施手順** | 1. config 構造体を設定 2. `MODULE_Init(&config)` 呼び出し 3. 戻り値確認 |
| **合否基準** | 戻り値 == ERR_OK |

```c
void test_ModuleInit_Normal(void) {
    ModuleConfig_t config = {
        .param1 = 10U,
        .param2 = 100U
    };
    RetCode_t ret = MODULE_Init(&config);
    TEST_ASSERT_EQUAL(ERR_OK, ret);
}
```

### TC-002: MODULE_Init() NULL ポインタ渡し

| 項目 | 内容 |
|---|---|
| **目的** | NULL ポインタ渡し時にエラーが返ることを確認 |
| **前提条件** | モジュール未初期化状態 |
| **入力** | `config = NULL` |
| **期待結果** | 戻り値 `ERR_NULL_PTR` |

```c
void test_ModuleInit_NullPtr(void) {
    RetCode_t ret = MODULE_Init(NULL);
    TEST_ASSERT_EQUAL(ERR_NULL_PTR, ret);
}
```

---

## 4. カバレッジ結果

| 項目 | 目標 | 実績 | 判定 |
|---|---|---|---|
| 行カバレッジ (C0) | ≥ 90% | | |
| 分岐カバレッジ (C1) | ≥ 90% | | |
| 条件カバレッジ (C2) | ≥ 80% | | |

---

## 5. テスト結果サマリ

| 項目 | 件数 |
|---|---|
| 総テストケース数 | |
| Pass | |
| Fail | |
| Skip | |

**合否判定:** [ Pass / Fail ]

**特記事項:**

[特記事項があれば記載]

---

## 6. 改訂履歴

| 版数 | 日付 | 変更内容 | 担当 |
|---|---|---|---|
| 1.0 | | 初版作成 | |
