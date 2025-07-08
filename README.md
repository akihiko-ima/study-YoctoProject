# 概要
本リポジトリは、YoctoProject の勉強記録用のリポジトリです。

## 開発環境
- OS
  - Ubuntu-22.04.5-desktop

- 開発ボード
  - Raspberry Pi 4 Model B 4GB

## コマンド
### 1_事前準備
- パッケージの更新
```bash
sudo apt update -y && sudo apt upgrade -y
```
- YoctoProjectのビルド環境必要ツール
```bash
sudo apt install -y gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev xterm python3-subunit mesa-common-dev zstd liblz4-tool
```
- お好みで
```bash
sudo apt install -y openssh-server vim
```
- pylint環境のパッケージをインストール
```bash
pip3 install --user pylint
```

### 2_セッアップ
- ビルドのディレクトリを作成
```bash
mkdir rpi4
```
```bash
cd rpi4/
```
- gitで開発環境をクローンする
```bash
git clone -b mickledore https://git.yoctoproject.org/poky.git
```
- セットアップスクリプトの実行
**(環境立ち上げ直しの度に必要)**
```bash
. poky/oe-init-build-env
```

### 3_BSPレイヤの取得
- gitで取得
```bash
git clone -b mickledore https://git.yoctoproject.org/git/meta-raspberrypi.git ~/rpi4/poky/meta-raspberrypi
```
```bash
cd ~/rpi4/build
```
```bash
bitbake-layers add-layer ../poky/meta-raspberrypi
```
```bash
cd ~/rpi4/
```

- lolal.confの設定
```bash
vim ~/rpi4/build/conf/local.conf
```
- 追加する内容
```
MACHINE = "raspberrypi4-64"

INHERIT += "ccache"
CCACHE_DIR = "${TOPDIR}/ccache"
DL_DIR = "${TOPDIR}/../downloads"
SSTATE_DIR ?= "${TOPDIR}/../sstate-cache"

# enable uart
ENABLE_UART = "1"

# systemd
DISTRO_FEATURES:append = " systemd"
VIRTUAL-RUNTIME_init_manager = "systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscripts = ""


# Accept for the synaptics license
LICENSE_FLAGS_ACCEPTED = "synaptics-killswitch"
```
### 4_Linuxイメージの構築
- ビルドに必要なソースをダウンロードだけ実施</br>
  **ビルド時のネットワークトラブル回避可能**
```bash
bitbake core-image-base --runall=fetch
```
- ビルド
```bash
bitbake core-image-base
```

### 5_SDに書き込み
- すでに書き込みデータがある場合
```bash
sudo umount /dev/mmcblk0p1
sudo umount /dev/mmcblk0p2
```
- bmaptoolで書き込み</br>
 **同じディレクトリに{XXX.wic.bmap}ファイルが必要**
```bash
sudo bmaptool copy {core-image-base-raspberrypi4-64-XXXXXXXX.rootfs.wic.bz2} /dev/mmcblk0
```

#### wi-fi接続時
connmanctlを使用する。</br>
**参考リンク:** [https://wiki.archlinux.jp/index.php/ConnMan](https://wiki.archlinux.jp/index.php/ConnMan)






