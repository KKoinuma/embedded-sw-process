# 詳細設計書

**文書番号:** SW-DDS-001  
**版数:** [x.x]  
**対象モジュール:** [モジュール名]  
**作成日:** [YYYY-MM-DD]  
**作成者:** [氏名]  
**承認者:** [氏名]  
**機密区分:** 社外秘

---

## 1. 目的・概要

本書は、[モジュール名] の詳細設計を定義する。

### 1.1 参照文書

| 文書番号 | 文書名 |
|---|---|
| SW-ARC-001 | ソフトウェアアーキテクチャ設計書 |
| SW-REQ-001 | ソフトウェア要件仕様書 |

### 1.2 対応要件

| 要件ID | 要件名 |
|---|---|
| REQ-F-001 | [要件名] |

---

## 2. モジュール構成

### 2.1 ファイル構成

| ファイル名 | 種別 | 役割 |
|---|---|---|
| [module].c | 実装ファイル | [役割] |
| [module].h | ヘッダファイル | 外部公開インタフェース定義 |
| [module]_priv.h | 内部ヘッダ | 内部型・マクロ定義 |

### 2.2 公開インタフェース

```c
/**
 * @brief  [関数の説明]
 * @param  [param1] [説明]
 * @param  [param2] [説明]
 * @return [戻り値の説明]
 */
RetCode_t MODULE_Init(const ModuleConfig_t *config);
RetCode_t MODULE_Process(void);
void      MODULE_GetStatus(ModuleStatus_t *status);
```

---

## 3. 関数詳細設計

### 3.1 MODULE_Init()

**目的:** モジュールの初期化を行う

**処理フロー:**

```
START
  │
  ├─[config == NULL]─▶ return ERR_NULL_PTR
  │
  ├─ パラメータ範囲チェック
  │   └─[範囲外]──▶ return ERR_INVALID_PARAM
  │
  ├─ 内部バッファ初期化 (memset)
  ├─ ハードウェア初期化 (HAL 呼び出し)
  ├─ 内部状態を STATE_INITIALIZED へ遷移
  │
  └─ return ERR_OK
END
```

**エラー処理:**

| 条件 | 戻り値 |
|---|---|
| config が NULL | ERR_NULL_PTR |
| パラメータ範囲外 | ERR_INVALID_PARAM |
| HW 初期化失敗 | ERR_HW_FAULT |
| 正常完了 | ERR_OK |

### 3.2 MODULE_Process()

**目的:** [処理の目的]

**処理フロー:**

```
[フローチャートを記述]
```

---

## 4. データ設計

### 4.1 内部データ

```c
/* モジュール内部状態 */
typedef struct {
    ModuleState_t state;
    uint32_t      error_count;
    uint32_t      last_update_ms;
    /* [その他フィールド] */
} ModuleContext_t;

static ModuleContext_t s_ctx;  /* モジュール内部コンテキスト */
```

### 4.2 定数・マクロ

```c
#define MODULE_TIMEOUT_MS    (100U)   /* タイムアウト時間 */
#define MODULE_RETRY_MAX     (3U)     /* 最大リトライ回数 */
#define MODULE_BUF_SIZE      (256U)   /* バッファサイズ */
```

---

## 5. 状態遷移

```
[UNINIT] ──MODULE_Init() OK──▶ [IDLE]
[IDLE]   ──MODULE_Start()────▶ [RUNNING]
[RUNNING]──エラー検出──────────▶ [FAULT]
[FAULT]  ──MODULE_Reset()────▶ [IDLE]
[任意]   ──MODULE_Deinit()───▶ [UNINIT]
```

---

## 6. テスト観点

| 観点 | テスト内容 | 対応テストケース |
|---|---|---|
| 正常系 | 正常パラメータでの動作 | TC-001 |
| 異常系 | NULL ポインタ渡し | TC-002 |
| 異常系 | 境界値テスト | TC-003 |
| 異常系 | HW エラー発生時 | TC-004 |

---

## 7. 改訂履歴

| 版数 | 日付 | 変更内容 | 担当 |
|---|---|---|---|
| 1.0 | | 初版作成 | |
