# 概要

本リポジトリは、YoctoProject の勉強記録用のリポジトリです。

## 開発環境

- OS

  - Ubuntu-22.04.5-desktop

- 開発ボード
  - Raspberry Pi 4 Model B 4GB

## コマンド

### 1\_事前準備

- パッケージの更新

```bash
sudo apt update -y && sudo apt upgrade -y
```

- YoctoProject のビルド環境必要ツール

```bash
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential \
     chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils \
     iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev \
     pylint xterm python3-subunit mesa-common-dev zstd liblz4-tool \
     ninja-build lzop
```

- お好みで

```bash
sudo apt install -y openssh-server vim
```

- pylint 環境のパッケージをインストール

```bash
pip3 install --user pylint
```

### 2\_セッアップ

- ビルドのディレクトリを作成

```bash
mkdir rpi4
```

```bash
cd rpi4/
```

- git で開発環境をクローンする

```bash
git clone -b mickledore https://git.yoctoproject.org/poky.git
```

- セットアップスクリプトの実行
  **(環境立ち上げ直しの度に必要)**

```bash
. poky/oe-init-build-env
```

### 3_BSP レイヤの取得

- Raspberry Pi 用 BSP レイヤを使用するために、`meta-openembedded` と `meta-raspberrypi` を取得し、  
  依存関係のある順番で `bitbake-layers` に追加

```bash
git clone -b mickledore https://git.openembedded.org/meta-openembedded ~/rpi4/poky/meta-openembedded
```

```bash
git clone -b mickledore https://git.yoctoproject.org/git/meta-raspberrypi.git ~/rpi4/poky/meta-raspberrypi
```

```bash
cd ~/rpi4/build
```

#### 依存レイヤを先に追加し、最後に `meta-raspberrypi` を追加します。

#### 依存レイヤを追加

```bash
bitbake-layers add-layer ../poky/meta-openembedded/meta-oe
```

```bash
bitbake-layers add-layer ../poky/meta-openembedded/meta-python
```

```bash
bitbake-layers add-layer ../poky/meta-openembedded/meta-networking
```

#### Raspberry Pi BSP レイヤを追加（最後に追加）

```bash
bitbake-layers add-layer ../poky/meta-raspberrypi
```

#### 追加の作業が完了したら、以下を実行してレイヤが正しく登録されているか確認

```bash
bitbake-layers show-layers
```

```bash
cd ~/rpi4/
```

- lolal.conf の設定

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

IMAGE_INSTALL:append = " \
    systemd-bootchart \
    openssh vim \
    libgpiod-tools i2c-tools minicom \
    lsb-release file which tree \
    connman connman-client \
    kernel-devsrc git cmake \
    python3 python3-pip \
    python3-setuptools python3-wheel \
"

EXTRA_IMAGE_FEATURES ?= "debug-tweaks tools-sdk dev-pkgs tools-debug"

# Accept for the synaptics license
LICENSE_FLAGS_ACCEPTED = "synaptics-killswitch"
```

- 補足 `IMAGE_INSTALL:append = " ... "`</br>
  追加でインストールするパッケージ群を指定します。</br>
  例: systemd-bootchart, openssh, vim, python3 python3-pip 各種開発ツールなど。</br>

### 4_Linux イメージの構築

- ビルドに必要なソースをダウンロードだけ実施</br>
  **ビルド時のネットワークトラブル回避可能**

```bash
bitbake core-image-base --runall=fetch
```

- ビルド

```bash
bitbake core-image-base
```

- 成果物の出力パス</br>
  ビルド後のイメージファイルや bmap ファイルの保存場所（例: build/tmp/deploy/images/raspberrypi4-64/）

### 5_SD に書き込み

- すでに書き込みデータがある場合

```bash
sudo umount /dev/mmcblk0p1
sudo umount /dev/mmcblk0p2
```

- bmaptool で書き込み</br>
  **同じディレクトリに{XXX.wic.bmap}ファイルが必要**

```bash
sudo bmaptool copy {core-image-base-raspberrypi4-64-XXXXXXXX.rootfs.wic.bz2} /dev/mmcblk0
```

#### wi-fi 接続時

connmanctl を使用する。</br>
**参考リンク:** [https://wiki.archlinux.jp/index.php/ConnMan](https://wiki.archlinux.jp/index.php/ConnMan)
