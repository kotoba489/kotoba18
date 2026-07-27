# ⌨️ kgrid18

36キーの自作分割キーボード用 ZMK ファームウェア設定です。Seeed Studio XIAO nRF52840（seeeduino_xiao_ble）を左右に使い、左手を Central（親機）、右手を Peripheral（子機）として動作させます。

> kgrid20をベースに、最下段の両端4キーを省いた19 mmピッチのコンパクト版です。

## 構成

| 項目 | 内容 |
| --- | --- |
| MCU | Seeed Studio XIAO nRF52840 |
| キー数 | 36キー（左右各18キー） |
| 物理配線 | 各手 4行 × 5列 |
| 有効キー | 1〜3行目は各10キー、最下段は中央6キー |
| 分割接続 | Bluetooth |
| 左手 | Central：PC / Mac / モバイル端末との接続を担当 |
| 右手 | Peripheral：左手との分割接続を担当 |
| ファームウェア | ZMK v0.3系、ZMK Studio対応 |

## レイアウト

最下段（row 3）の両端4キーを省いています。配線マトリクスは4行×10列のまま、`matrix_transform` で36キーだけを有効にしています。

```text
row 0:  L0 L1 L2 L3 L4       R0 R1 R2 R3 R4
row 1:  L0 L1 L2 L3 L4       R0 R1 R2 R3 R4
row 2:  L0 L1 L2 L3 L4       R0 R1 R2 R3 R4
row 3:        L2 L3 L4       R0 R1 R2
```

`config/boards/shields/k_grid18/k_grid18.dtsi` の変換定義：

```dts
map = <
    RC(0,0) RC(0,1) RC(0,2) RC(0,3) RC(0,4)  RC(0,5) RC(0,6) RC(0,7) RC(0,8) RC(0,9)
    RC(1,0) RC(1,1) RC(1,2) RC(1,3) RC(1,4)  RC(1,5) RC(1,6) RC(1,7) RC(1,8) RC(1,9)
    RC(2,0) RC(2,1) RC(2,2) RC(2,3) RC(2,4)  RC(2,5) RC(2,6) RC(2,7) RC(2,8) RC(2,9)
    RC(3,2) RC(3,3) RC(3,4)  RC(3,5) RC(3,6) RC(3,7)
>;
```

基本レイヤーの詳しい割り当てとコンボは [`config/k_grid18.keymap`](config/k_grid18.keymap) を参照してください。

## 配線

- ダイオード方向: `col2row`
- スキャン設定: `GPIO_ACTIVE_HIGH` / `GPIO_PULL_DOWN`

| 信号 | XIAOピン | GPIO (ZMK) | 用途 |
| --- | --- | --- | --- |
| Row 0–3 | D0, D1, D2, D3 | P0.02, P0.03, P0.28, P0.29 | 行スキャン |
| Col 0–4（左） | D4, D5, D10, D9, D8 | P0.04, P0.05, P1.15, P0.09, P0.08 | 列スキャン |

## リポジトリ構成

```text
config/
├── k_grid18.keymap              # 36キーの各レイヤー・コンボ
├── k_grid18.conf                # 共通設定
├── k_grid18_left.conf           # 左手（Central）固有設定
├── k_grid18_right.conf          # 右手（Peripheral）固有設定
├── settings_reset.conf          # 設定領域初期化用
└── boards/shields/k_grid18/
    ├── Kconfig.defconfig
    ├── Kconfig.shield
    ├── k_grid18.dtsi            # matrix_transform / physical layout
    ├── k_grid18_left.overlay
    └── k_grid18_right.overlay
```

## 接続と機能

### Bluetoothの安定化

共通設定では分割キーボード、Bluetooth、USB、バッテリー報告、ポインティング、ZMK Studioを有効にしています。

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
```

左右それぞれに Settings/NVS 保存を設定しているため、電源を切っても分割接続情報を保持できます。左手だけにはPC／MacとのBLE再接続を安定させるGATT設定と、Central側のバッテリー取得設定を入れています。

ファームウェアのスリープは無効です。長時間使わないときは、電源スイッチでOFFにします。

### ZMK Studioとポインティング

- `physical-layout` を定義済みで、ZMK Studioからキーマップを編集できます。
- マウス移動・クリック・スクロール用に、共通設定と左右の設定で `CONFIG_ZMK_POINTING=y` を有効にしています。
- 分割構成でポインティングが動かない場合は、左・右両方の `.conf` にこの設定があるか確認します。

## キーマップを変更する

1. [`config/k_grid18.keymap`](config/k_grid18.keymap) を編集する。各レイヤーは36スロットです。
2. mainへコミット・プッシュする。
3. GitHub Actionsのビルド完了後、生成された左手用UF2をダウンロードする。
4. 左手をブートローダーモードで接続し、UF2を書き込む。

通常のキーマップ変更では、左手用UF2だけを書き込めばよい運用です。左右の設定やシールド定義を変更した場合、初回セットアップ時、または接続不良を直す場合は、左右両方のファームウェアを書き込みます。

## 接続が不安定なとき

1. PC／Mac／モバイル端末のBluetooth設定から **kgrid18** を削除する。
2. 左右両方に `settings_reset.uf2` を書き込み、保存済み設定を初期化する。
3. 左右それぞれに通常ファームウェア（left / right）を書き込む。
4. 左右を近くに置いて同時に起動し、分割接続の確立を待つ。
5. PC／Macとあらためてペアリングする。

## ライセンス

[MIT License](LICENSE)
