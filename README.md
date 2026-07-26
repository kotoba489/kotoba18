
# ⌨️ kgrid18
## 🛠️ Current Configuration 
18キー（左右各9キー、最下段中央3キー×2）分割レイアウト。  
kgrid20をベースに左右各2キー（row3の両端）を削除した19ピッチコンパクト版。

## 📂 Branch Strategy
- **main**: 安定版

## ✨ Special Features
- **Split Layout**: `k_grid.json` の `"split": true` 設定と座標調整により、中央に1列分のスペースを配置。
- **18キー構成**: row3（最下段）から左端 `RC(3,0)(3,1)` と右端 `RC(3,8)(3,9)` を削除。マトリクス配線（4行×5列）はkgrid20から変更なし。

## 🔨 Hardware Mod
- **Keycaps**: 19ピッチ対応キーキャップ
- **PCB**: kgrid20と同一ピン配列のまま、左右各2スイッチ穴を削減した新設計

---

## 1. 現在の物理的な状態

| 部位 | 役割 | 接続先 / 設定 |
|:---:|:---:|:---|
| **左手** | 親機 (Central) | `ZMK_SPLIT_ROLE_CENTRAL=y` / PC接続担当 |
| **右手** | 子機 (Peripheral) | `ZMK_SPLIT_ROLE_CENTRAL=n` / 左手接続担当 |

- **ペアリング状態**: 左右間の同期 ✅ / PCとのBluetooth接続 ✅
- **マトリクス**: 左右それぞれ5列×4行（物理配線）、有効キーは左右各9キー＋最下段左右各3キー＝計36キー。

---

## 2. kgrid20からの主な変更点

### `config/boards/shields/k_grid/k_grid.dtsi`
- matrix_transformのmapからrow3両端4キーを削除（40→36キー）。
- マトリクスの `columns` / `rows` 設定はkgrid20と同一（配線変更なし）。

```dts
map = <
    RC(0,0) RC(0,1) RC(0,2) RC(0,3) RC(0,4)  RC(0,5) RC(0,6) RC(0,7) RC(0,8) RC(0,9)
    RC(1,0) RC(1,1) RC(1,2) RC(1,3) RC(1,4)  RC(1,5) RC(1,6) RC(1,7) RC(1,8) RC(1,9)
    RC(2,0) RC(2,1) RC(2,2) RC(2,3) RC(2,4)  RC(2,5) RC(2,6) RC(2,7) RC(2,8) RC(2,9)
                    RC(3,2) RC(3,3) RC(3,4)  RC(3,5) RC(3,6) RC(3,7)
>;
```

### `config/k_grid.json`
- layoutエントリを40→36に削減。row3はcol2〜4（左）とcol5〜7（右）のみ。

### `config/k_grid.keymap`
- 全レイヤーのバインディングを40スロット→**36スロット**に変更。
- row3の割り当て：左 `LALT / LGUI / SPACE`、右 `SPACE / ENTER / BSPC`。

---

## 3. GitHub 上の設定ファイル構成

### `config/boards/shields/k_grid/Kconfig.defconfig`
左手を「親」、右手を「子」として明確に役割分担させる設定を導入。これによりBluetoothの認識率が大幅に向上しました。

---
### Bluetooth 再接続安定化設定 (`k_grid18.conf`)
Mac や Mobile 端末との自動接続安定化およびポインティング（pointing）機能有効化設定です。

```properties
CONFIG_ZMK_KEYBOARD_NAME="kgrid18"
CONFIG_ZMK_SPLIT=y
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
CONFIG_ZMK_BLE=y
CONFIG_ZMK_USB=y
CONFIG_ZMK_BATTERY_REPORTING=y
CONFIG_BT_BAS=y
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
CONFIG_ZMK_SLEEP=n  

---

##. 今後のキーマップ変更の流れ
基本的に**左手（親機）の設定を書き換えるだけ**で、左右両方のキー配置を更新可能です。

1. GitHub 上の `.keymap` ファイルを編集（全**36スロット**分）
2. Commit & Push → 自動ビルド
3. `k_grid_left.uf2` をダウンロードして左手に書き込む
4. 完了！（2回目以降は右手は変更しなくても配置が変わります）

---

## 5. ポインティングデバイス（マウス機能）の有効化
このキーボードでは、キー入力に加えてマウス操作を行うために  
ZMKのポインティング機能を有効化しています。

```conf
CONFIG_ZMK_POINTING=y
```

詳細なキー割り当ては `k_grid.keymap` を参照してください。

---

## 6. メンテナンス・トラブルシューティング

### ✅ 通常のキーマップ変更
- `.keymap` を修正してPush。
- 生成された **`left.uf2` だけを左手に書き込む**。
- （右手は1回目の変更以外は不要）

### 🔄 接続が不安定になった場合（リセット儀式）
1. PC側のBluetooth設定から「kgrid18」を一度削除。
2. 左手・右手両方に `settings_reset.uf2` を書き込む。
3. 左手に `left.uf2`、右手に `right.uf2` をそれぞれ書き込む。
4. 左右を至近距離に置いて同時起動し、ペアリングを待つ。

---


## ZMK Studio 対応
ZMK v0.3 固定・physical-layout 定義済み。Studio からキーマップをリアルタイム編集できます。

---

## Bluetooth再接続の安定化

電源OFF/ON後もBluetoothのペアリング情報（bond/profile）を保持できるように、左右それぞれへZMKのSettings/NVS保存設定を追加しています。<br>
これにより、MacやWindowsで一度ペアリングした後、毎回デバイス削除や再ペアリングをしなくても再接続しやすくなります。

PC/Mac/Windows向けのGATT互換設定やCentral側バッテリーproxy/fetchingは左手Central側だけに入れています。右手Peripheral側は、左手Centralとの分割接続情報を保持するためのSettings/NVS保存設定だけを持たせています。

長時間のdeep sleepから復帰できないことがあったため、ファームウェア側のsleepは無効化しています。30分以上使わない場合は、電源スイッチでOFFにする運用を推奨します。

あわせて、`settings_reset.uf2` でも保存領域を正しく初期化できるように `config/settings_reset.conf` を追加しています。  
接続が不安定な場合は、PC/Mac側の古いペアリングを削除し、`settings_reset.uf2` → 通常ファームウェア → 新規ペアリングの順でやり直してください。

---

## 分割キーボードのポインティング機能

レイヤー内のマウス移動・クリック・スクロールを確実に有効化するため、共通設定に加えて左右それぞれの `.conf` にも `CONFIG_ZMK_POINTING=y` を明示しています。  
分割構成でポインティング操作が効かない場合は、`*_left.conf` / `*_right.conf` の両方にこの設定が入っているか確認してください。

---

## MIT License
This project is licensed under the MIT License.
