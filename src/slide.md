---
marp: true
paginate: true
---

# Bitcoin Cash Upgrade 2022-05-15

---

## 自己紹介
- [haryu703](https://twitter.com/haryu703)
- コインチェック株式会社 (2020~)

---

## 今回のハードフォークの概要
- 2022年5月15日にBitcoin Cashはハードフォークによるアップグレードを行った
  - ハードフォークによるアップグレードは2020年11月15日以来
  - 2021年5月にはハードフォークを伴わないアップグレードがあった
- Bitcoin Scriptのランタイムに関する下記の2点が更新される
  - [CHIP-2021-02: Native Introspection Opcodes](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Add-Native-Introspection-Opcodes.md)
    トランザクションの情報を取得するOpCodeを追加する
  - [CHIP-2021-03: Bigger Script Integers](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md)
    扱える整数を32-bitから64-bitに拡張し、`OP_MUL`を再有効化する

- CHIP(Cash Improvement Proposal)はBitcoin Cashのネットワークに関する変更案
  - CHIPはBIPに比べスコープが広いらしい

---

## Native Introspection Opcodes
- [`OP_CHECKDATASIG`](https://tech.coincheck.blog/entry/2020/02/28/114652)により可能になったcovenantsの課題を改善する
- [今までのcovenantsの実装](https://github.com/haryu703/ccstudy-33/blob/master/examples/mecenas/mecenas.spedn)
- 検証したいトランザクション中の情報に直接アクセスできるOpCodeを追加する
- 追加されるOpCodeは`0xc0`-`0xcf`の15個

---

### 追加されるOpCode (1/5)

| Operation | Codepoint | Description |
| - | - | - |
| OP_INPUTINDEX     | `0xc0` (192) | 評価中のinputのindexをスタックにプッシュする |
| OP_ACTIVEBYTECODE | `0xc1` (193) | 評価中のbytecodeをスタックにプッシュする。もし`OP_CODESEPARATOR`が実行されていれば最後に実行されたところから後の部分がプッシュされる。P2SHの場合はredeem script、それ以外ならlocking scriptがプッシュされることになる |

---

### 追加されるOpCode (2/5)

| Operation | Codepoint | Description |
| - | - | - |
| OP_TXVERSION      | `0xc2` (194) | トランザクションのバージョンをスタックにプッシュする |
| OP_TXINPUTCOUNT   | `0xc3` (195) | トランザクションのinputの数をスタックにプッシュする |
| OP_TXOUTPUTCOUNT  | `0xc4` (196) | トランザクションのoutputの数をスタックにプッシュする |
| OP_TXLOCKTIME     | `0xc5` (197) | トランザクションのlocktimeをスタックにプッシュする |

---

### 追加されるOpCode (3/5)

| Operation | Codepoint | Description |
| - | - | - |
| OP_UTXOVALUE           | `0xc6` (198) | スタックの先頭をinputのindexとしてポップし、消費するUTXOの金額をsatoshi単位でスタックにプッシュする |
| OP_UTXOBYTECODE        | `0xc7` (199) | スタックの先頭をinputのindexとしてポップし、消費するUTXOのlocking scriptをスタックにプッシュする |
| OP_OUTPOINTTXHASH      | `0xc8` (200) | スタックの先頭をinputのindexとしてポップし、消費するUTXOのTXIDをスタックにプッシュする |

---

### 追加されるOpCode (4/5)

| Operation | Codepoint | Description |
| - | - | - |
| OP_OUTPOINTINDEX       | `0xc9` (201) | スタックの先頭をinputのindexとしてポップし、消費するUTXOのindexをスタックにプッシュする |
| OP_INPUTBYTECODE       | `0xca` (202) | スタックの先頭をinputのindexとしてポップし、そのinputのunlocking scriptをスタックにプッシュする |
| OP_INPUTSEQUENCENUMBER | `0xcb` (203) | スタックの先頭をinputのindexとしてポップし、そのinputのsequence numberをスタックにプッシュする |

---

### 追加されるOpCode (5/5)

| Operation | Codepoint | Description |
| - | - | - |
| OP_OUTPUTVALUE         | `0xcc` (204) | スタックの先頭をoutputのindexとしてポップし、そのoutputの金額をsatoshi単位でスタックにプッシュする |
| OP_OUTPUTBYTECODE      | `0xcd` (205) | スタックの先頭をoutputのindexとしてポップし、そのoutputのlocking scriptをスタックにプッシュする |

---

### 参考: トランザクションの構造
![](https://en.bitcoin.it/w/images/en/e/e1/TxBinaryMap.png)
https://en.bitcoin.it/wiki/File:TxBinaryMap.png

---

### 実装への影響
- [preimage版](https://github.com/haryu703/ccstudy-65/blob/master/example/out/mecenas_preimage.json)
  ```bash
  $ yarn preimage -s -c
  yarn run v1.22.15
  $ yarn cashc-060 mecenas_preimage.cash -s -c
  $ ./node_modules/cashc-060/dist/main/cashc-cli.js mecenas_preimage.cash -s -c
  Opcode count: 99
  Bytesize: 158
  Done in 0.31s.
  ```
- [native introspection版](https://github.com/haryu703/ccstudy-65/blob/master/example/out/mecenas_native.json)
  ```bash
  $ yarn native -s -c
  yarn run v1.22.15
  $ yarn cashc-070 mecenas_native.cash -s -c
  $ ./node_modules/cashc-070/dist/main/cashc-cli.js mecenas_native.cash -s -c
  Opcode count: 60
  Bytesize: 90
  Done in 0.29s.
  ```

---

## Bigger Script Integers
- Bitcoin Scriptで扱える整数を32-bitから64-bitに拡張する
- signed 32-bit integerで表現できる金額は約21 BCH(2147483647 satoshi)まで
  - 擬似的に上限以上の数値を表現するには複雑な実装が必要
- この制限は初期のBitcoin実装でオーバーフロー処理を考慮して導入されたもの
- 標準的なオーバーフロー処理を導入したうえでこの制限を64-bitに拡張する
  - 内部的な実装ではすでに64-bit integerの計算は使われている
- `OP_MUL`(`0x95`/`149`)の課題も解決されるため再有効化する

---

#### Script Number の範囲

||下限|上限|
|-|-|-|
|~2022/05/15|`0xffffffff`<br>(`-2147483647`)|`0xffffff7f`<br>(`2147483647`)|
|2022/05/15~|`0xffffffffffffffff`<br>(`-9223372036854775807`)|`0xffffffffffffff7f`<br>(`9223372036854775807`)|

Bitcoin Script上の整数の表現に使われる`Script Number`のフォーマットはC言語などのsigned 64-bit integerと比べ表現できる最小値が`1`少ない。

---

#### オーバーフロー処理
- 算術演算はオーバーフローチェック検知付きsigned 64-bit integerで計算する
  - [X86-64 GNU C Integer Overflow Builtins](https://gcc.gnu.org/onlinedocs/gcc/Integer-Overflow-Builtins.html)など
- オーバーフローが発生した場合、スクリプトは失敗と評価される

---

#### OP_MUL
- 乗算演算子の`OP_MUL`を再有効化する
- 乗算以外の`OP_DIV`や`OP_MOD`は2018年5月のハードフォークで有効化されていた
- Bigger Script Integerで導入されたオーバーフロー処理により`OP_MUL`も再有効化できるようになった

---

## 終わりに
- Bitcoin ABCの分裂を受け、今までに比べ時間をかけてプロトコル更新への合意が行われた
- Bitcoin Scriptの改善により今まで以上に便利で低コストなコントラクトが期待できる
- [いろいろなCHIP](https://bitcoincashresearch.org/c/chips/17)が登場しており今後もプロトコルの更新は続く
