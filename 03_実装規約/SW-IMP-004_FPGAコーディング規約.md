# FPGA コーディング規約

**文書番号:** SW-IMP-004  
**版数:** 1.0  
**作成日:** 2026-06-15  
**対象言語:** VHDL / Verilog HDL

---

## 1. 適用範囲

本規約は、東海事業所における FPGA 論理設計に適用する。主に VHDL を使用し、Verilog を使用する場合も同様の方針に従うこと。

---

## 2. 設計方針

1. **同期設計**: 非同期リセット・非同期セットは原則禁止。全て同期リセットを使用する
2. **シングルクロック**: 複数クロックドメインは最小化し、CDC (Clock Domain Crossing) は明示的に設計する
3. **ラッチ禁止**: 意図しないラッチが生成されないよう、組み合わせ回路の全条件分岐を網羅する
4. **タイミング制約**: 全クロックパスに Timing Constraint を設定し、タイミング違反を解消してからテープアウトする

---

## 3. ファイル・モジュール規約

### 3.1 ファイル命名

```
[module_name].vhd        # VHDL ソース (1ファイル = 1エンティティ原則)
[module_name]_tb.vhd     # テストベンチ
[module_name]_pkg.vhd    # パッケージ定義
```

### 3.2 エンティティ・モジュール構成テンプレート

```vhdl
-------------------------------------------------------------------------------
-- File    : ctrl_main.vhd
-- Brief   : メイン制御ロジック
-- Version : 1.0
-- Date    : 2026-06-15
-- Author  : [氏名]
--
-- Copyright (c) 2026 Nippon Radiation Engineering Co., Ltd.
-------------------------------------------------------------------------------
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity ctrl_main is
    generic (
        DATA_WIDTH : integer := 16;
        CLK_FREQ   : integer := 50_000_000  -- 50 MHz
    );
    port (
        clk_i   : in  std_logic;
        rst_n_i : in  std_logic;  -- 同期負論理リセット
        data_i  : in  std_logic_vector(DATA_WIDTH-1 downto 0);
        data_o  : out std_logic_vector(DATA_WIDTH-1 downto 0);
        valid_o : out std_logic
    );
end entity ctrl_main;
```

---

## 4. 命名規則

| 種別 | 規則 | サフィックス例 |
|---|---|---|
| クロック信号 | `clk_` プレフィックス | `clk_sys`, `clk_adc` |
| リセット (負論理) | `_n` サフィックス | `rst_n_i` |
| 入力ポート | `_i` サフィックス | `data_i` |
| 出力ポート | `_o` サフィックス | `valid_o` |
| 双方向ポート | `_io` サフィックス | `sda_io` |
| 内部信号 | `_s` サフィックス | `count_s` |
| レジスタ | `_r` サフィックス | `state_r` |
| 定数 | `C_` プレフィックス + 大文字 | `C_MAX_COUNT` |

---

## 5. 記述スタイル

### 5.1 レジスタ記述 (2プロセス方式)

```vhdl
-- 順序プロセス (FF のみ)
p_reg : process(clk_i)
begin
    if rising_edge(clk_i) then
        if rst_n_i = '0' then
            state_r <= ST_IDLE;
            count_r <= (others => '0');
        else
            state_r <= state_s;   -- 次状態
            count_r <= count_s;
        end if;
    end if;
end process p_reg;

-- 組み合わせプロセス (次状態・出力計算)
p_comb : process(state_r, count_r, data_i)
begin
    -- デフォルト代入 (ラッチ防止)
    state_s  <= state_r;
    count_s  <= count_r;
    valid_o  <= '0';
    data_o   <= (others => '0');

    case state_r is
        when ST_IDLE =>
            if data_i /= (data_i'range => '0') then
                state_s <= ST_PROCESS;
            end if;
        when ST_PROCESS =>
            data_o  <= data_i;
            valid_o <= '1';
            state_s <= ST_IDLE;
        when others =>
            state_s <= ST_IDLE;
    end case;
end process p_comb;
```

### 5.2 禁止事項

| 禁止事項 | 理由 |
|---|---|
| 非同期リセット | メタスタビリティリスク |
| 意図しないラッチ | 動作保証不可 |
| `wait` 文 (テストベンチ外) | 合成不可 |
| 組み合わせループ | 発振・不定動作 |
| ハードコードされた遅延 (`after`) | 合成後無効 |

---

## 6. シミュレーション・検証

- テストベンチは自動チェック (`assert`) を含めること
- コードカバレッジ (行・条件・状態遷移) を取得し、カバレッジ目標 ≥ 90% を達成すること
- フォーマル検証 (Lint / CDC チェック) を必ず実施すること

---

## 7. 改訂履歴

| 版数 | 日付 | 変更内容 | 担当 |
|---|---|---|---|
| 1.0 | 2026-06-15 | 初版作成 | SW開発グループ |
