# ZMKファームウェア 完全ガイド

このドキュメントは、ZMK（Zephyr Mechanical Keyboard）ファームウェアの詳細な技術情報をまとめたものです。

## 目次

1. [概要](#概要)
2. [コアアーキテクチャ](#コアアーキテクチャ)
3. [分割キーボードアーキテクチャ](#分割キーボードアーキテクチャ)
4. [Bluetooth実装](#bluetooth実装)
5. [設定システム](#設定システム)
6. [ビルドシステム](#ビルドシステム)
7. [開発ワークフロー](#開発ワークフロー)

---

## 概要

ZMKファームウェアは、Zephyr™ Project Real Time Operating System (RTOS)をベースに構築されたオープンソースのキーボードファームウェアです。

### 主な特徴

- **電力効率**: 低消費電力設計でワイヤレスキーボードに最適
- **柔軟性**: モジュラー設計により多様なハードウェアに対応
- **幅広いハードウェアサポート**: 有線・無線両方の入力デバイスに対応
- **Zephyr RTOSの活用**: ハードウェア抽象化、強力なBluetoothスタック
- **コンパイル時設定**: すべての設定はビルド時に決定され、ファームウェアの再ビルドが必要

---

## コアアーキテクチャ

ZMKのアーキテクチャは、Zephyr RTOSの機能を最大限に活用し、3つの主要システムで構成されています。

### 1. キーマップシステム

キーマップシステムは、物理的なキー位置を論理的な「キー位置」に変換し、特定のビヘイビアにバインドします。

#### レイヤー構造

- キーマップは複数の**レイヤー**で構成
- 最低1つのデフォルトレイヤー（レイヤー0）が必須
- レイヤーは動的に切り替え可能
- 上位レイヤーが優先され、イベントは下位レイヤーに「フォールスルー」可能

#### キーマップファイル形式

```dts
/ {
    keymap {
        compatible = "zmk,keymap";

        layer_0 {
            bindings = <
                &kp A  &kp B  &kp C
                &kp D  &kp E  &kp F
            >;
        };

        layer_1 {
            display-name = "Function";
            bindings = <
                &kp F1  &kp F2  &kp F3
                &trans  &trans  &trans
            >;
        };
    };
};
```

#### キーイベント処理フロー

1. キー押下・リリース時に**Position State Changed Event**が生成
2. Keymap Systemが最上位レイヤーから順に処理
3. 各レイヤーでキー位置に対応するビヘイビアバインディングを取得
4. ビヘイビアを実行（イベント消費 or 下位レイヤーにフォールスルー）

#### レイヤー管理

キーマップシステムは内部的にビットマップ（`_zmk_keymap_layer_state`）でアクティブレイヤーを管理します。

**レイヤー制御関数:**
- `zmk_keymap_layer_activate()`: レイヤーを有効化
- `zmk_keymap_layer_deactivate()`: レイヤーを無効化
- `zmk_keymap_layer_toggle()`: レイヤーをトグル
- `zmk_keymap_layer_to()`: 指定レイヤーに切り替え

### 2. ビヘイビアシステム

ビヘイビアは、キー位置が押された・離された時、またはセンサーイベント（エンコーダー回転など）が発生した時のアクションを定義します。

#### 基本ビヘイビア

| ビヘイビア | 説明 | 例 |
|----------|------|-----|
| `&kp` | Key Press - キーコード送信 | `&kp A` |
| `&mt` | Mod-Tap - ホールドで修飾キー、タップでキー | `&mt LSHIFT D` |
| `&lt` | Layer-Tap - ホールドでレイヤー、タップでキー | `&lt 1 SPACE` |
| `&mo` | Momentary Layer - ホールド中のみレイヤー有効 | `&mo 2` |
| `&to` | To Layer - 指定レイヤーに切り替え | `&to 1` |
| `&tog` | Toggle Layer - レイヤーをトグル | `&tog 2` |
| `&sl` | Sticky Layer - 次のキー押下までレイヤー有効 | `&sl 1` |

#### マウス制御ビヘイビア

| ビヘイビア | 説明 | 例 |
|----------|------|-----|
| `&mkp` | Mouse Key Press - マウスボタン | `&mkp LCLK` |
| `&mmv` | Mouse Move - カーソル移動 | `&mmv MOVE_UP` |
| `&msc` | Mouse Scroll - スクロール | `&msc SCRL_DOWN` |

#### Bluetoothビヘイビア

| ビヘイビア | 説明 | 例 |
|----------|------|-----|
| `&bt` | Bluetooth - プロファイル管理 | `&bt BT_CLR` |

**BTコマンド:**
- `BT_CLR`: 現在のプロファイルのボンド削除
- `BT_NXT`: 次のプロファイルに切り替え
- `BT_PRV`: 前のプロファイルに切り替え
- `BT_SEL <n>`: プロファイルnを選択

#### ビヘイビアバインディング構文

ビヘイビアバインディングは、ビヘイビアと最大2つのパラメータで構成されます：

```dts
&behavior_name PARAMETER1 PARAMETER2
```

例:
- `&kp A` - ビヘイビア: kp, パラメータ1: A
- `&mt LSHIFT D` - ビヘイビア: mt, パラメータ1: LSHIFT, パラメータ2: D

### 3. HID実装

ZMKはHuman Interface Device (HID)レポートを実装し、ホストデバイスに入力イベントを送信します。

#### HIDレポートタイプ

**N-Key Rollover (NKRO):**
```conf
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y
```
- 同時押し対応
- より多くのキーを同時に認識

**#-Key Rollover (HKRO):**
```conf
CONFIG_ZMK_HID_REPORT_TYPE_HKRO=y
```
- 互換性重視
- 一部のBIOSやレガシーシステムで必要

#### HIDレポートの種類

- **キーボードレポート**: 標準のキー押下
- **コンシューマーレポート**: メディアキー（音量、再生/停止など）
- **HIDインジケーター**: Caps Lock、Num Lockなどのステータス

#### HIDディスクリプタのリフレッシュ

ZMKの機能や動作を変更すると、ホストに送信されるHIDディスクリプタが変更される場合があります：

- **USB接続**: 起動時に自動更新
- **BLE接続**: ホストがキャッシュするため、手動更新が必要
  1. ホスト側でキーボードを削除
  2. キーボード側で該当プロファイルをクリア（`&bt BT_CLR`）
  3. 再ペアリング

---

## 分割キーボードアーキテクチャ

ZMKは、各物理部分が独自のマイクロコントローラーとZMKファームウェアを実行する分割キーボードをサポートします。

### Central-Peripheral モデル

分割キーボードは、Central（中央）とPeripheral（周辺）の役割に分かれています。

#### Central（中央）の役割

通常は左側が担当します：

- ✅ Peripheralからキー・センサーイベントを受信
- ✅ キーマップロジックの処理
- ✅ レイヤー状態の管理
- ✅ HIDレポートの生成
- ✅ ホストデバイスとの通信（USB/BLE）
- ✅ **すべてのビヘイビア処理**

#### Peripheral（周辺）の役割

通常は右側（または追加の部分）が担当します：

- ✅ キーマトリックスのスキャン
- ✅ センサーデータの読み取り
- ✅ イベントをCentralに送信
- ❌ ホストデバイスと直接通信しない
- ❌ キーマップ処理を行わない

### BLE通信プロトコル

#### カスタムGATTサービス

ZMKは、キーボード間通信用のカスタムBLE GATTサービスを使用します。

**主要なCharacteristic:**

| Characteristic | 用途 | 方向 |
|---------------|------|------|
| Position State | キーイベント | Peripheral → Central |
| Sensor State | センサーデータ | Peripheral → Central |
| HID Indicators | LED状態 | Central → Peripheral |

#### スロットベース接続管理

Centralは複数のPeripheralとの接続を**スロット**で管理：

- 各スロットは1つのPeripheralに対応
- 接続状態を追跡
- Peripheralサービスを自動検出

### ペアリングプロセス

分割キーボードの内部ペアリングは、ホストとのペアリングとは別です。

#### 初回ペアリング

1. **Central**: BLEアドバタイジングを開始
2. **Peripheral（未ペアリング）**: Centralに接続
3. **ペアリング完了**: ボンド情報を相互に保存
4. **自動再接続**: 次回起動時に自動的に再接続

#### ボンド情報の管理

- 各デバイスのBLEアドレスを使用してボンド情報を保存
- 電源ON時に自動再接続
- `&bt BT_CLR`でボンド情報をクリア可能

### Central/Peripheral設定

#### Kconfig設定

```conf
# Central側（通常は左側）
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Peripheral側（通常は右側）
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
```

#### CMake引数での指定

```bash
# Central
west build -b board_name -- -DSHIELD=keyboard_left -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Peripheral
west build -b board_name -- -DSHIELD=keyboard_right
```

---

## Bluetooth実装

ZMKのBluetooth機能により、キーボードはBLE (Bluetooth Low Energy)でホストに接続でき、分割キーボードの半分間の通信にも使用されます。

### システム要件

- **Bluetooth 4.2以上**が必須
- **Secure Connection機能**を実装（高度なセキュリティ）
- 既知の脆弱性を回避

### プロファイル管理

ZMKは**5つのプロファイル**をサポートし、複数のボンド済みホストデバイスを管理できます。

#### プロファイルの仕組み

- 各プロファイルに1つのホストデバイスをペアリング可能
- ペアリング時、選択中のプロファイルにボンド情報を保存
- `&bt`ビヘイビアでプロファイル管理

#### プロファイル操作

```dts
// プロファイル0を選択
&bt BT_SEL 0

// 次のプロファイルに切り替え
&bt BT_NXT

// 前のプロファイルに切り替え
&bt BT_PRV

// 現在のプロファイルのボンドをクリア
&bt BT_CLR
```

### BLE設定オプション

```conf
# Bluetoothを有効化
CONFIG_ZMK_BLE=y

# 実験的なBLE設定（接続間隔など）
CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y

# 優先接続間隔（ミリ秒単位、×1.25ms）
CONFIG_ZMK_BLE_PREF_INT=6

# 接続レイテンシ
CONFIG_ZMK_BLE_PREF_LATENCY=30

# スーパービジョンタイムアウト（×10ms）
CONFIG_ZMK_BLE_PREF_TIMEOUT=400
```

### HIDディスクリプタキャッシュの問題

BLEホストはHIDディスクリプタをキャッシュするため、ファームウェアの変更が反映されない場合があります。

**対処方法:**
1. ホストデバイス側でキーボードを「削除」
2. キーボード側で該当プロファイルをクリア（`&bt BT_CLR`）
3. キーボードを再ペアリング

---

## 設定システム

ZMKの設定システムは、**Kconfig**と**Devicetree**の2つの主要コンポーネントで構成されています。両方ともZephyr RTOSから継承されています。

### すべての設定はコンパイル時

⚠️ **重要**: ZMKのすべての設定はコンパイル時に決定されます。設定を変更するには、新しいファームウェアをビルドしてフラッシュする必要があります。

### 1. Kconfig

Kconfigは、グローバル設定と機能の有効化/無効化を管理します。

#### ファイル形式

拡張子: `.conf`

構文:
```conf
CONFIG_OPTION_NAME=value
```

#### 値の型

- **Boolean**: `y`（有効）または `n`（無効）
- **Integer**: `123`
- **String**: `"text"`

#### ファイルの配置場所

優先順位順（後のファイルが前のファイルを上書き）：

1. **ボード固有**: `boards/*/boardname.conf`
2. **シールド固有**: `boards/shields/*/*.conf`
3. **ユーザー設定**: `zmk-config/config/*.conf` ⭐ 最優先

#### 主要な設定カテゴリ

**基本設定:**
```conf
# キーボード名
CONFIG_ZMK_KEYBOARD_NAME="My Keyboard"
```

**HID設定:**
```conf
# NKROを有効化
CONFIG_ZMK_HID_REPORT_TYPE_NKRO=y
```

**接続:**
```conf
# Bluetooth有効化
CONFIG_ZMK_BLE=y

# USB有効化
CONFIG_ZMK_USB=y
```

**分割キーボード:**
```conf
# 分割キーボードを有効化
CONFIG_ZMK_SPLIT=y

# Central役割
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# BLE接続設定
CONFIG_ZMK_SPLIT_BLE_PREF_INT=6
CONFIG_ZMK_SPLIT_BLE_PREF_LATENCY=30
CONFIG_ZMK_SPLIT_BLE_PREF_TIMEOUT=400
```

**電力管理:**
```conf
# スリープを有効化
CONFIG_ZMK_SLEEP=y

# アイドルスリープタイムアウト（ミリ秒）
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```

**ディスプレイ & LED:**
```conf
# RGB Underglowを有効化
CONFIG_ZMK_RGB_UNDERGLOW=y
```

**ビヘイビア:**
```conf
# マウスキーを有効化
CONFIG_ZMK_MOUSE=y
```

**デバッグ:**
```conf
# ロギングレベル
CONFIG_ZMK_LOG_LEVEL=4
```

#### 設定の確認方法

ビルド後、最終的なKconfig設定を確認できます：

1. **ローカルビルド**: `build/zephyr/.config`ファイル
2. **GitHub Actions**: ビルドログ内

### 2. Devicetree

Devicetreeは、ハードウェアの記述とキーマップの定義に使用されます。

#### ファイル拡張子と用途

| 拡張子 | 用途 |
|--------|------|
| `.dts` | ベースハードウェア定義 |
| `.overlay` | 定義の追加/上書き |
| `.keymap` | キーマップ + ユーザー固有のハードウェア設定 |
| `.dtsi` | `#include`用の共通定義 |

#### ファイルの配置場所

検索順序：

1. **ユーザー設定**: `zmk-config/config/<name>.keymap` ⭐
2. **ボード定義**: `boards/<arch>/<board>/<board>.dts`, `<board>.keymap`
3. **シールド定義**: `boards/shields/<shield>/<shield>.overlay`, `<shield>.keymap`

#### Devicetreeの構造

Devicetreeは**ツリー構造**でハードウェアを表現します：

```dts
/ {
    chosen {
        zmk,kscan = &kscan0;
        zmk,matrix-transform = &default_transform;
    };

    kscan0: kscan {
        compatible = "zmk,kscan-gpio-matrix";

        col-gpios = <&gpio0 1 GPIO_ACTIVE_HIGH>,
                    <&gpio0 2 GPIO_ACTIVE_HIGH>;

        row-gpios = <&gpio0 3 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>,
                    <&gpio0 4 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>;
    };

    default_transform: keymap_transform_0 {
        compatible = "zmk,matrix-transform";
        columns = <2>;
        rows = <2>;
        map = <
            RC(0,0) RC(0,1)
            RC(1,0) RC(1,1)
        >;
    };
};
```

#### 重要なDevicetreeノード

**`chosen` ノード:**
システムが使用するデバイスを指定

```dts
chosen {
    zmk,kscan = &kscan0;              // キースキャンデバイス
    zmk,matrix-transform = &transform; // マトリックス変換
    zmk,physical-layout = &layout;     // 物理レイアウト
    zmk,battery = &vbatt;              // バッテリー監視
};
```

**`kscan` ノード:**
キーマトリックススキャンの定義

```dts
kscan0: kscan {
    compatible = "zmk,kscan-gpio-matrix";
    wakeup-source;  // スリープから復帰可能

    col-gpios = <...>;  // 列のGPIOピン
    row-gpios = <...>;  // 行のGPIOピン
};
```

**`matrix-transform` ノード:**
物理位置から論理位置へのマッピング

```dts
transform: keymap_transform {
    compatible = "zmk,matrix-transform";
    columns = <14>;
    rows = <5>;
    map = <
        RC(0,0) RC(0,1) RC(0,2) ...
        RC(1,0) RC(1,1) RC(1,2) ...
    >;
};
```

#### プロパティの定義

Devicetreeプロパティは、特定のノードに適用されます（グローバルではない）。

プロパティは`.yaml`ファイル（`dts/bindings`フォルダ）で定義されています。

#### 設定の確認方法

ビルド後、最終的なDevicetreeを確認できます：

1. **ローカルビルド**: `build/zephyr/zephyr.dts`ファイル
2. **GitHub Actions**: ビルドログ内

### Kconfig、Devicetree、Keymapの統合

これら3つのシステムは、ビルドプロセス中に統合されます：

```
┌─────────────┐
│   Kconfig   │ ← グローバル設定、機能の有効化
│   (.conf)   │   例: CONFIG_ZMK_USB=y
└──────┬──────┘
       │
       ├──────→ ビルドシステム (West/CMake)
       │
┌──────┴──────┐
│ Devicetree  │ ← ハードウェア記述
│(.dts/.overlay)│  例: GPIO定義、kscan設定
└──────┬──────┘
       │
┌──────┴──────┐
│   Keymap    │ ← レイヤー、ビヘイビアバインディング
│  (.keymap)  │   例: &kp A, &lt 1 SPACE
└─────────────┘
```

**例:**
1. Kconfig: `CONFIG_ZMK_RGB_UNDERGLOW=y` でRGB機能を有効化
2. Devicetree: LED接続とピンを定義
3. Keymap: `&rgb_ug RGB_TOG` でLED制御のビヘイビア使用

---

## ビルドシステム

ZMKのビルドシステムは、Zephyr RTOSとWestメタツールを基盤としています。

### West、Zephyr、ZMKの関係

```
┌──────────┐
│   West   │ ← メタツール（依存関係管理、ビルド実行）
└─────┬────┘
      │
      ├─── manages ───┐
      │                │
┌─────▼────┐     ┌────▼─────┐
│  Zephyr  │ ← ─ │   ZMK    │
│   RTOS   │ base │ Firmware │
└──────────┘     └──────────┘
```

- **West**: Zephyrプロジェクトのメタツール。リポジトリ管理、依存関係取得、ビルド実行
- **Zephyr**: ZMKの基盤となるRTOS
- **ZMK**: Zephyr上に構築されたキーボードファームウェア

### west.yml マニフェストファイル

`app/west.yml`（または`config/west.yml`）は、Zephyrバージョンと必要な依存関係を定義します。

**例:**
```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.2
      import: app/west.yml
    - name: zephyr
      # Zephyrのバージョンはzmkのwest.ymlでインポート
```

### ビルドワークフロー

#### 1. 初回セットアップ

```bash
# ワークスペースの初期化
west init -l config/

# 依存関係の取得
west update

# Zephyr CMakeパッケージのエクスポート
west zephyr-export
```

#### 2. ビルド

**基本構文:**
```bash
west build -b <board_name> [-- <cmake_args>]
```

**ボード内蔵MCUの場合:**
```bash
west build -b planck_rev6
```

**シールド使用の場合:**
```bash
west build -b proton_c -- -DSHIELD=kyria_left
```

**追加のCMake引数:**
```bash
west build -b nice_nano_v2 -- -DSHIELD=corne_left -DCONFIG_ZMK_SLEEP=y
```

#### 3. 分割キーボードのビルド

各半分を別ディレクトリにビルドすることを推奨：

```bash
# 左側（Central）
west build -d build/left -b nice_nano_v2 -- \
  -DSHIELD=corne_left \
  -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# 右側（Peripheral）
west build -d build/right -b nice_nano_v2 -- \
  -DSHIELD=corne_right
```

#### 4. フラッシュ

**UF2サポートボード:**
```bash
# ビルド後、build/zephyr/zmk.uf2をUSBマスストレージにコピー
```

**DFUサポートボード:**
```bash
west flash
```

### BoardsとShields

ZMKは、Zephyrの「Boards」と「Shields」の概念を使用してハードウェアを定義します。

#### Board（ボード）

**定義**: MCU（マイクロコントローラーユニット）を搭載したPCB

**種類:**
- 完全なキーボードPCB（MCU内蔵）
- 小型のMCUボード（他のキーボードコンポーネントと組み合わせ）

**例**: `nice_nano_v2`, `proton_c`, `planck_rev6`

**ディレクトリ構造:**
```
<board_root>/boards/<arch>/<board_name>/
├── Kconfig.board
├── Kconfig.defconfig
├── <board_name>_defconfig
├── <board_name>.dts
├── <board_name>.keymap (オプション)
├── board.cmake
└── <board_name>.zmk.yml
```

#### Shield（シールド）

**定義**: MCUなしのPCBまたはハードワイヤードコンポーネント。MCU専用ボードと組み合わせて使用。

**利点**: 異なる互換性のあるコントローラーと混在可能

**例**: `corne_left`, `kyria_right`, `lily58_left`

**ディレクトリ構造:**
```
<board_root>/boards/shields/<shield_name>/
├── Kconfig.shield
├── Kconfig.defconfig
├── <shield_name>.overlay
├── <shield_name>.keymap
└── <shield_name>.zmk.yml
```

### モジュールシステム

ZMKは外部モジュールをサポートし、追加のソースコードや設定ファイルをツリー外で管理できます。

#### モジュールのビルド

```bash
west build -b nice_nano_v2 -- \
  -DSHIELD=vendor_shield \
  -DZMK_EXTRA_MODULES="C:/path/to/my-module"
```

#### module.yml

各モジュールは`zephyr/module.yml`ファイルで定義します：

```yaml
name: my-custom-module
build:
  cmake: .
  kconfig: Kconfig
  settings:
    board_root: .
    dts_root: .
```

### ビルド出力

ビルドが成功すると、以下のファイルが生成されます：

- **`build/zephyr/zmk.uf2`**: フラッシュ用のUF2ファイル
- **`build/zephyr/zephyr.dts`**: 最終的なDevicetree
- **`build/zephyr/.config`**: 最終的なKconfig設定

---

## 開発ワークフロー

### 1. zmk-config リポジトリのセットアップ

ZMKは、コアファームウェアから分離されたユーザー設定リポジトリ（`zmk-config`）を使用します。

#### リポジトリ作成

1. GitHubで新しいリポジトリ`zmk-config`を作成
2. ZMKのテンプレートをクローンまたはフォーク
3. セットアップスクリプトを実行してボード・シールドを設定

#### ディレクトリ構造

```
zmk-config/
├── config/
│   ├── west.yml              # ZMK依存関係の定義
│   ├── <keyboard>.conf       # Kconfig設定
│   └── <keyboard>.keymap     # キーマップ定義
├── build.yaml                # GitHub Actionsビルド設定
└── .github/
    └── workflows/
        └── build.yml          # CI/CDワークフロー
```

### 2. キーマップのカスタマイズ

`config/<keyboard>.keymap`を編集：

```dts
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>

/ {
    keymap {
        compatible = "zmk,keymap";

        default_layer {
            bindings = <
                &kp Q  &kp W  &kp E  &kp R  &kp T
                &kp A  &kp S  &kp D  &kp F  &kp G
            >;
        };
    };
};
```

### 3. 機能の有効化/無効化

`config/<keyboard>.conf`を編集：

```conf
# マウスキーを有効化
CONFIG_ZMK_MOUSE=y

# RGB Underglowを有効化
CONFIG_ZMK_RGB_UNDERGLOW=y

# スリープを有効化
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```

### 4. ビルドとデプロイ

#### GitHub Actionsを使用（推奨）

1. 変更をコミット
2. GitHubにプッシュ
3. GitHub Actionsが自動的にビルド
4. Artifactsからファームウェアをダウンロード

#### ローカルビルド

```bash
# 依存関係の更新
west update

# ビルド
west build -b <board> -- -DSHIELD=<shield>

# フラッシュ
# UF2ファイルをコピー、またはwest flash
```

### 5. デバッグとテスト

#### ログの有効化

```conf
CONFIG_ZMK_LOG_LEVEL=4
CONFIG_ZMK_USB_LOGGING=y
```

#### シリアルコンソールでログ表示

```bash
# Linuxの場合
screen /dev/ttyACM0 115200

# macOSの場合
screen /dev/cu.usbmodem* 115200
```

### 6. カスタムハードウェアの開発

#### 新しいボードの作成

1. `boards/<arch>/<board_name>/`ディレクトリを作成
2. 必要なファイルを追加：
   - `Kconfig.board`
   - `Kconfig.defconfig`
   - `<board>.dts`
   - `<board>.zmk.yml`

#### 新しいシールドの作成

1. `boards/shields/<shield_name>/`ディレクトリを作成
2. 必要なファイルを追加：
   - `Kconfig.shield`
   - `Kconfig.defconfig`
   - `<shield>.overlay`
   - `<shield>.keymap`
   - `<shield>.zmk.yml`

### 7. トラブルシューティング

#### ビルドエラー

- `west update`を実行して依存関係を更新
- `.config`と`zephyr.dts`で設定を確認
- GitHub Actionsのビルドログを確認

#### Bluetooth接続の問題

- 両側のボンド情報をクリア（`&bt BT_CLR`）
- ホストデバイスでキーボードを削除
- 再ペアリング

#### 分割キーボードの接続問題

- Central/Peripheralの役割設定を確認
- 両側のファームウェアが最新か確認
- バッテリーレベルを確認

---

## まとめ

ZMKファームウェアは、Zephyr RTOSの強力な基盤の上に構築された、柔軟で電力効率の高いキーボードファームウェアです。

### 重要なポイント

1. **モジュラー設計**: Boards/Shields/Behaviorsの組み合わせ
2. **コンパイル時設定**: Kconfig + Devicetree
3. **強力な分割キーボードサポート**: Central-Peripheralモデル
4. **BLE実装**: 5プロファイル、Secure Connection
5. **柔軟なキーマップシステム**: レイヤーとビヘイビアバインディング
6. **Westビルドシステム**: Zephyrの統合されたツールチェーン

### 次のステップ

- ZMK公式ドキュメント: https://zmk.dev/
- ZMK GitHubリポジトリ: https://github.com/zmkfirmware/zmk
- Zephyrドキュメント: https://docs.zephyrproject.org/
- ZMK Discordコミュニティ

---

**このドキュメントの作成日**: 2025年10月9日
**ZMKバージョン**: v0.2ベース
**参照**: zmkfirmware/zmk GitHub リポジトリ、DeepWiki
