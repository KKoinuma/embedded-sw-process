# PLC コーディング規約

**文書番号:** SW-IMP-003  
**版数:** 1.0  
**作成日:** 2026-06-15  
**適用規格:** IEC 61131-3

---

## 1. 適用範囲

本規約は、IEC 61131-3 に準拠した PLC プログラム開発に適用する。対象言語は Structured Text (ST) を主とし、Ladder Diagram (LD)、Function Block Diagram (FBD) を含む。

---

## 2. プロジェクト構成

### 2.1 プログラム構成

```
PLC_Project/
├── GlobalVars/          # グローバル変数宣言
│   ├── GVL_Inputs.gvl
│   ├── GVL_Outputs.gvl
│   └── GVL_Constants.gvl
├── DataTypes/           # ユーザ定義型
├── FunctionBlocks/      # ファンクションブロック
├── Functions/           # ファンクション
└── Programs/            # プログラム (POUs)
    ├── PRG_Main
    ├── PRG_Safety
    └── PRG_Comm
```

### 2.2 タスク設定

| タスク名 | 種別 | 周期 | 優先度 | 割り当てプログラム |
|---|---|---|---|---|
| Safety_Task | 周期 | 1 ms | 1 (最高) | PRG_Safety |
| Control_Task | 周期 | 10 ms | 2 | PRG_Main |
| Comm_Task | 周期 | 100 ms | 3 | PRG_Comm |

---

## 3. 命名規則

### 3.1 変数

| 種別 | プレフィックス | 例 |
|---|---|---|
| 入力変数 | `I_` | `I_StartButton` |
| 出力変数 | `Q_` | `Q_AlarmLamp` |
| メモリ変数 | `M_` | `M_MotorRunning` |
| タイマ | `T_` | `T_StartDelay` |
| カウンタ | `C_` | `C_CycleCount` |
| 定数 | `K_` | `K_MaxSpeed` |
| ファンクションブロックインスタンス | `FB_` | `FB_PIDController` |

### 3.2 データ型プレフィックス (ハンガリアン記法)

| 型 | プレフィックス |
|---|---|
| BOOL | `b` |
| INT / DINT | `i` / `di` |
| REAL | `r` |
| TIME | `t` |
| STRING | `s` |
| 配列 | `a` |

**例:** `bMotorEnabled`, `rFlowRate`, `tCycleTime`

---

## 4. Structured Text 記述規則

### 4.1 インデントとフォーマット

```pascal
(* 正しい例 *)
IF bStartCmd AND NOT bFault THEN
    bMotorEnable := TRUE;
    tStartDelay(IN := TRUE, PT := T#500MS);
    IF tStartDelay.Q THEN
        bMotorRunning := TRUE;
    END_IF
ELSE
    bMotorEnable := FALSE;
END_IF
```

### 4.2 禁止事項

| 禁止事項 | 代替手段 |
|---|---|
| GOTO 文 | 構造化制御フロー |
| 再帰的 FB 呼び出し | 反復ループ |
| 未初期化変数の参照 | 初期値を必ず設定 |
| マジックナンバー | 定数 (K_xxx) で定義 |

### 4.3 コメント

```pascal
(* ================================================================
   Function: モータ起動シーケンス
   Safety:   SIL1
   ================================================================ *)

(* 起動条件チェック *)
bCanStart := bRemoteReady AND NOT bEStop AND NOT bFault;
```

---

## 5. 安全プログラム規則

### 5.1 SIL 対応プログラム

- SIL 対応 POU は通常タスクと物理的に分離されたセーフティタスクで実行すること
- セーフティ変数への書き込みは専用 API (メーカ提供の安全認証ライブラリ) 経由のみ許可
- セーフティ出力はデュアルチャンネル構成を原則とする

### 5.2 フォルト検出

```pascal
(* 例: 二重化センサ不一致検出 *)
bSensorFault := ABS(rSensor1 - rSensor2) > K_SensorTolerance;
IF bSensorFault THEN
    Q_SafetyOutput := FALSE;  (* フェールセーフ: 出力 OFF *)
END_IF
```

---

## 6. 改訂履歴

| 版数 | 日付 | 変更内容 | 担当 |
|---|---|---|---|
| 1.0 | 2026-06-15 | 初版作成 | SW開発グループ |
