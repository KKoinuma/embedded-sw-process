# C コーディング規約

**文書番号:** SW-IMP-001  
**版数:** 1.0  
**作成日:** 2026-06-15  
**適用規格:** MISRA C:2012

---

## 1. 適用範囲

本規約は、日本放射線エンジニアリング株式会社 東海事業所における組込み C 言語 (C99) ソフトウェア開発全般に適用する。

---

## 2. ファイル規約

### 2.1 ファイル命名

```
[モジュール名].[c|h]          # 例: ctrl_main.c, hal_gpio.h
[モジュール名]_priv.h         # モジュール内部ヘッダ
[モジュール名]_test.c         # 単体テストファイル
```

### 2.2 ファイルヘッダ

全ファイルの先頭に以下のブロックを記述すること。

```c
/**
 * @file    [ファイル名]
 * @brief   [ファイルの概要]
 * @version [x.x]
 * @date    [YYYY-MM-DD]
 * @author  [氏名]
 *
 * Copyright (c) [年] Nippon Radiation Engineering Co., Ltd.
 * Tokai Works. All rights reserved.
 *
 * SPDX-License-Identifier: Proprietary
 */
```

---

## 3. 命名規則

### 3.1 識別子

| 種別 | 規則 | 例 |
|---|---|---|
| マクロ定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 型定義 (typedef) | PascalCase + `_t` | `SensorData_t` |
| 列挙子 | UPPER_SNAKE_CASE | `SYS_STATE_NORMAL` |
| グローバル変数 | `g_` + lowerCamelCase | `g_sysState` |
| モジュール静的変数 | `s_` + lowerCamelCase | `s_rxBuffer` |
| ローカル変数 | lowerCamelCase | `retryCount` |
| 関数 (公開) | `MODULE_FunctionName()` | `CTRL_Init()` |
| 関数 (内部) | `lowerCamelCase()` | `calcChecksum()` |

### 3.2 禁止事項

- 1文字変数名の使用禁止 (ループカウンタ `i`, `j`, `k` は例外)
- `_` 先頭・`__` 含む識別子の使用禁止 (処理系予約)
- 意味不明な略語の使用禁止

---

## 4. 型使用規則

### 4.1 整数型

**必須**: `<stdint.h>` の固定幅型を使用する。`int`, `long` 等の処理系依存型は原則禁止。

```c
/* 正しい例 */
uint8_t  byteVal;
uint16_t wordVal;
int32_t  signedVal;
uint32_t timestamp_ms;

/* 禁止例 */
int   val;   /* NG: サイズ不明 */
long  cnt;   /* NG: サイズ不明 */
```

### 4.2 真偽値

```c
#include <stdbool.h>
bool isReady = false;  /* true / false を使用 */
```

### 4.3 浮動小数点

- 安全機能で浮動小数点演算を使用する場合は、NaN・Inf を必ずチェックすること
- 固定小数点演算への代替を検討すること

---

## 5. コード構造規則

### 5.1 関数

- 1関数の行数: **50行以内** を目標。100行を超えてはならない
- 引数の数: **最大 5個**
- return 文の数: **1個** を原則とする (MISRA C Rule 15.5)
- 循環的複雑度 (McCabe): **10以下**

```c
/* 良い例: 単一 return */
RetCode_t CTRL_SetParam(uint8_t param, uint32_t value) {
    RetCode_t ret = ERR_OK;
    if (param >= PARAM_MAX) {
        ret = ERR_INVALID_PARAM;
    } else if (value > PARAM_VALUE_MAX) {
        ret = ERR_INVALID_PARAM;
    } else {
        s_params[param] = value;
    }
    return ret;
}
```

### 5.2 禁止構文 (MISRA C 必須ルール)

| 禁止事項 | 理由 |
|---|---|
| `goto` 文 | 制御フロー不明瞭化 |
| `malloc` / `free` | ヒープ断片化・タイミング不定 |
| 再帰呼び出し | スタック使用量が確定不能 |
| 可変長引数 (`va_list`) | 型安全性がない |
| 未定義動作 (UB) を起こす演算 | 移植性・安全性欠如 |

### 5.3 マジックナンバー禁止

```c
/* 禁止例 */
if (temp > 85) { ... }

/* 正しい例 */
#define TEMP_LIMIT_DEGC  (85U)
if (temp > TEMP_LIMIT_DEGC) { ... }
```

---

## 6. コメント規則

### 6.1 ファイル・関数コメント

Doxygen 形式を使用する。

```c
/**
 * @brief  センサ値を読み取り、校正済み値を返す
 * @param  ch     ADC チャンネル番号 (0 ～ ADC_CH_MAX-1)
 * @param  value  [out] 校正済み値の格納先
 * @retval ERR_OK           正常
 * @retval ERR_INVALID_PARAM ch が範囲外
 * @retval ERR_HW_FAULT     ADC 読み出し失敗
 */
RetCode_t SENSOR_Read(uint8_t ch, float *value);
```

### 6.2 インラインコメント

- 「何をするか」ではなく「なぜするか」をコメントする
- コメントはコードと同期させること (古いコメントは削除)

---

## 7. ヘッダファイル規則

```c
/* インクルードガード必須 */
#ifndef HAL_GPIO_H
#define HAL_GPIO_H

#ifdef __cplusplus
extern "C" {
#endif

/* ... 宣言 ... */

#ifdef __cplusplus
}
#endif

#endif /* HAL_GPIO_H */
```

---

## 8. 改訂履歴

| 版数 | 日付 | 変更内容 | 担当 |
|---|---|---|---|
| 1.0 | 2026-06-15 | 初版作成 | SW開発グループ |
