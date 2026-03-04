# VM 環境構成まとめ

> Ubuntu 22.04 LTS (jammy) の開発環境を Vagrant + VirtualBox で複製したもの。

---

## 基本情報

| 項目 | 値 |
|------|-----|
| Box | `ubuntu/jammy64` (Ubuntu 22.04.5 LTS) |
| ホスト名 | `jammy-dev` |
| プロバイダ | VirtualBox |

---

## VM スペック

| 項目 | 値 | 備考 |
|------|-----|------|
| CPU | 2 コア | 1 / 2 / 4 / 8 に変更可 |
| メモリ | 4096 MB | GNOME GUI 推奨値 |
| VRAM | 128 MB | GUI描画用 |
| 3D アクセラレーション | 有効 | |
| クリップボード共有 | 双方向 | |
| ドラッグ&ドロップ | 双方向 | |

---

## ネットワーク

| 項目 | 値 |
|------|-----|
| モード | `public_network` (bridged) |
| 備考 | 起動時にホスト側NICを選択するプロンプトが表示される |

---

## GUI

| 項目 | 値 |
|------|-----|
| デスクトップ環境 | GNOME (ubuntu-desktop) |
| ディスプレイマネージャ | GDM3 |
| 表示方法 | VirtualBox 仮想ディスプレイ (`vb.gui = true`) |
| Guest Additions | `virtualbox-guest-dkms` / `virtualbox-guest-utils` / `virtualbox-guest-x11` |

---

## インストール済みパッケージ

### 開発ツール

| パッケージ | バージョン |
|-----------|-----------|
| build-essential | 12.9ubuntu3 |
| gcc / g++ | 10 / 11 / 12 (マルチバージョン) |
| make | 4.3 |
| cmake | 3.22.1 |
| clang / clang-tidy / clang-tools | 12 |
| rake | 13.0.6 |
| xxd | 8.2.3995 |

### ネットワーク / SSH

| パッケージ | バージョン |
|-----------|-----------|
| git | 2.34.1 |
| curl | 7.81.0 |
| wget | 1.21.2 |
| netcat-openbsd | 1.218 |
| openssh-client / server | 8.9p1 |
| ufw | 0.36.1 |

### 言語ランタイム

| 言語 | バージョン |
|------|-----------|
| Python | 3.10.12 |
| Ruby | 3.0.2 |
| Node.js | 12.22.9 |
| npm | 8.5.1 |
| PHP | 8.1 (+ php8.1-curl) |

### snap パッケージ

| パッケージ | 備考 |
|-----------|------|
| chromium | latest/stable |
| firefox | latest/stable |

### pip パッケージ (主要なもの)

| パッケージ | バージョン | 用途 |
|-----------|-----------|------|
| norminette | 3.3.58 | 42School コードスタイルチェック |
| c-formatter-42 | 0.2.8 | 42School コードフォーマッタ |
| GitPython | 3.1.43 | Gitリポジトリ操作 |
| rich | 13.9.4 | ターミナル表示 |
| supervisor | 4.2.1 | プロセス管理 |
| speedtest-cli | 2.1.3 | 回線速度計測 |
| halo | 0.0.31 | スピナー表示 |
| tqdm | 4.57.0 | プログレスバー |
| termcolor | 2.5.0 | ターミナル色付け |
| PyYAML | 5.4.1 | YAML パース |

---

## プロビジョニング手順

```
[1/8] システムアップデート          apt-get update && upgrade
[2/8] 開発ツール                    gcc / clang / cmake / make
[3/8] ネットワーク / SSH ツール      curl / wget / git / openssh
[4/8] 言語ランタイム                 Python / Ruby / Node.js
[5/8] PHP 8.1
[6/8] GNOME デスクトップ環境        ubuntu-desktop / gdm3
[7/8] snap パッケージ               chromium / firefox
[8/8] pip パッケージ                norminette / c-formatter-42 など
```

---

## 基本操作コマンド

```bash
vagrant up        # 起動 + プロビジョニング（初回は30〜60分）
vagrant ssh       # SSH ログイン
vagrant halt      # シャットダウン
vagrant reload    # 再起動
vagrant provision # プロビジョニングのみ再実行
vagrant destroy   # VM を完全削除
```

---

## 注意事項

- `ubuntu-desktop` は容量が大きいため、初回 `vagrant up` は **30〜60分** かかる場合がある
- `virtualbox-guest-x11` のバージョンが VirtualBox 本体と一致しない場合、dkms ビルドエラーが発生することがある
- snap パッケージのインストールはプロビジョニング中に snapd が起動している必要があり、失敗した場合は `vagrant provision` で再実行する
- `public_network` (bridged) はホスト側のネットワークに直接接続されるため、IP アドレスはルータの DHCP から払い出される