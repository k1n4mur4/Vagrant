# Kali Linux ペネトレーションテスト環境 - セットアップガイド

## 構成概要

| 項目 | 設定値 |
|------|--------|
| OS | Kali Linux (Rolling) |
| RAM | 12GB |
| CPU | 6コア |
| ディスク | 80GB |
| GUI | XFCE デスクトップ |
| ネットワーク | ブリッジ + ホストオンリー (192.168.56.100) |
| USB | パススルー対応（WiFiアダプタ用） |

## ファイル構成

```
C:\vagrant-kali\
├── Vagrantfile       # VM定義ファイル
└── README.md         # このファイル

C:\pentest-share\     # ホスト ↔ VM 共有フォルダ
```

---

## 事前準備（Windows 11側）

### 1. VirtualBox のインストール

https://www.virtualbox.org/wiki/Downloads からダウンロード。

**Extension Pack も必須**（USB 2.0/3.0 パススルーに必要）。
同じページからダウンロードし、VirtualBox を開いて
「ファイル > ツール > Extension Pack Manager」から追加。

### 2. Vagrant のインストール

https://www.vagrantup.com/downloads から Windows AMD64 版をダウンロード。
インストール後、PCを再起動。

### 3. Vagrant プラグインのインストール

```powershell
vagrant plugin install vagrant-disksize
vagrant plugin install vagrant-reload
vagrant plugin install vagrant-cachier
vagrant plugin install vagrant-hostmanager
```

確認:

```powershell
vagrant plugin list
```

以下の4つが表示されればOK:

```
vagrant-cachier
vagrant-disksize
vagrant-hostmanager
vagrant-reload
```

### 4. 共有フォルダの作成

```powershell
mkdir C:\pentest-share
```

---

## プラグイン一覧と役割

| プラグイン | 役割 |
|---|---|
| **vagrant-disksize** | VMのディスクサイズを80GBに拡張 |
| **vagrant-reload** | プロビジョニング完了後にVM内から安全に再起動 |
| **vagrant-cachier** | apt等のパッケージキャッシュをホスト側に保存。`destroy` → `up` し直しても再ダウンロード不要 |
| **vagrant-hostmanager** | Windows側とVM側の `hosts` ファイルを自動更新。`kali-pentest` というホスト名でアクセス可能に |

---

## 起動手順

### 1. Vagrantfile を配置

```powershell
mkdir C:\vagrant-kali
cd C:\vagrant-kali
# Vagrantfile, README.md を配置
```

### 2. 起動

```powershell
vagrant up
```

初回の流れ:
1. Kali Box のダウンロード（数GB）
2. VM 作成・起動
3. ブリッジNIC選択（プロンプトで使用中のNIC番号を入力）
4. プロビジョニング実行（ツール一括インストール / 30〜60分）
5. 自動再起動（vagrant-reload）
6. GUI デスクトップが表示される

### 3. ログイン

| 項目 | 値 |
|------|-----|
| ユーザー名 | `vagrant` |
| パスワード | `vagrant` |

---

## ブリッジNICの固定（任意）

毎回起動時にNIC選択のプロンプトが出るのが面倒な場合、
Vagrantfile に直接NIC名を書いて固定できます。

### NIC名の確認

```powershell
Get-NetAdapter
```

### Vagrantfile を編集

```ruby
# 変更前
config.vm.network "public_network"

# 変更後（NIC名を指定）
config.vm.network "public_network", bridge: "Intel(R) Wi-Fi 6 AX201 160MHz"
```

---

## インストールされるツール

### Web アプリケーション

Burp Suite, OWASP ZAP, nikto, dirb, gobuster, feroxbuster,
sqlmap, wfuzz, ffuf, whatweb, wafw00f, sublist3r, amass

### ネットワーク / インフラ

nmap, masscan, Wireshark, tcpdump, netdiscover, arp-scan,
enum4linux, smbclient, smbmap, CrackMapExec, Responder,
Impacket, Evil-WinRM, proxychains4, chisel, ligolo-ng

### WiFi / 無線

aircrack-ng, Kismet, Wifite, Reaver, Bully, Pixiewps,
hostapd-wpe, mdk4, hcxdumptool, hcxtools

### Active Directory

BloodHound, Neo4j, Kerbrute, Rubeus, Mimikatz,
PowerShell, ldapdomaindump

### 共通・ユーティリティ

Metasploit Framework, SecLists, wordlists, ExploitDB,
John the Ripper, Hashcat, Hydra, Docker, tmux,
Flameshot, Terminator

---

## USB WiFi アダプタの設定

### 1. Vendor ID / Product ID を調べる

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list usbhost
```

### 2. Vagrantfile の USB フィルタを編集

出力された `vendorid` と `productid` を使って、Vagrantfile 内の
USB フィルタセクションのコメントを外し、値を書き換える。

```ruby
vb.customize ["usbfilter", "add", "0",
  "--target", :id,
  "--name", "My WiFi Adapter",
  "--vendorid", "XXXX",    # ← 調べた値
  "--productid", "YYYY"]   # ← 調べた値
```

### 3. 反映

```powershell
vagrant reload
```

---

## よく使うコマンド

| コマンド | 説明 |
|----------|------|
| `vagrant up` | 起動 |
| `vagrant halt` | シャットダウン |
| `vagrant reload` | 再起動（Vagrantfile 変更後） |
| `vagrant suspend` | サスペンド（状態保存して停止） |
| `vagrant resume` | サスペンドから復帰 |
| `vagrant ssh` | SSH接続（CLI） |
| `vagrant provision` | プロビジョニング再実行 |
| `vagrant snapshot save <名前>` | スナップショット保存 |
| `vagrant snapshot restore <名前>` | スナップショット復元 |
| `vagrant snapshot list` | スナップショット一覧 |
| `vagrant destroy` | VM完全削除 |
| `vagrant plugin list` | インストール済みプラグイン一覧 |
| `vagrant plugin update` | 全プラグインを更新 |

---

## スナップショットの活用（おすすめ）

プロビジョニング完了直後にスナップショットを取っておくと、
いつでもクリーンな状態に戻せます。

```powershell
# セットアップ完了後に保存
vagrant snapshot save fresh-install

# テスト後にクリーンな状態へ戻す
vagrant snapshot restore fresh-install

# 一覧確認
vagrant snapshot list
```

---

## vagrant-cachier のキャッシュ管理

キャッシュは Box ごとにホスト側に保存されます。

```
# キャッシュの場所（Windows）
C:\Users\<ユーザー名>\.vagrant.d\cache\

# キャッシュを削除したい場合
# → 上記フォルダを手動で削除
```

`vagrant destroy` してもキャッシュは残るので、
次の `vagrant up` 時にパッケージの再ダウンロードをスキップできます。

---

## vagrant-hostmanager について

起動後、Windows 側の `hosts` ファイルに以下が自動追加されます:

```
192.168.56.100  kali-pentest
```

これにより、ホスト PC から以下のようにアクセス可能:

```powershell
# SSH
ssh vagrant@kali-pentest

# ブラウザ（VM内でWebサーバーを立てた場合）
http://kali-pentest:8080
```

※ 管理者権限のプロンプトが出る場合があります（hosts 書き換えのため）

---

## トラブルシューティング

### ブリッジ接続で IP が取得できない

- Vagrantfile の bridge 指定が正しいNIC名か確認
- `Get-NetAdapter` の出力と一致しているか確認
- VirtualBox のネットワーク設定でブリッジ先NICが正しいか確認

### USB デバイスが認識されない

- VirtualBox Extension Pack がインストールされているか確認
- `VBoxManage list usbhost` でデバイスが見えるか確認
- Vagrantfile の USB フィルタの `vendorid` / `productid` が正しいか確認
- デバイスがホスト側で使用中でないか確認

### GUI が重い

- Vagrantfile の `--accelerate3d` を `off` に変更して `vagrant reload`
- VirtualBox の「設定 > ディスプレイ」でビデオメモリを確認
- ホスト側で重いアプリを閉じてリソースを確保

### プロビジョニングが途中で失敗した

```powershell
# プロビジョニングだけ再実行
vagrant provision
```

### hostmanager で hosts が更新されない

```powershell
# 手動で hosts を更新
vagrant hostmanager
```

### プラグインが壊れた

```powershell
# 修復
vagrant plugin repair

# それでもダメなら再インストール
vagrant plugin expunge --reinstall
```
