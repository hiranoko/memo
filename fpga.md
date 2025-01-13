[Home](./README.md)

- [KV260](#kv260)
  - [OSイメージ](#osイメージ)
  - [動作確認](#動作確認)
  - [電源の起動後](#電源の起動後)
  - [環境構築](#環境構築)
    - [bootgen](#bootgen)
  - [`xmutil` コマンドのユーティリティ一覧と説明](#xmutil-コマンドのユーティリティ一覧と説明)

# KV260

## OSイメージ

KV260はxilinx wikiにOSのイメージが[配布](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/1641152513/Kria+SOMs+Starter+Kits#K26-Starter-Kit-Linux)されてる。  

Ubuntu Server 24.04をダンロードすると
`iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img.xz`がある。

入手したファイルはxz圧縮されているので解凍が必要である。
   
```
$ xz -dk iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img.xz
```

diskのパーティションを整理して空領域とした状態でデバイスの場所`/dev/sda`を確認する。  

```
sudo dd if=iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img of=/dev/sda bs=4M status=progress
```

## 動作確認

```
$ sudo apt search armsoc
[sudo] password for ubuntu: 
Sorting... Done
Full Text Search... Done
xserver-xorg-video-armsoc-endlessm/noble,now 1.4.1-0ubuntu10 arm64 [installed]
  X.Org X server -- ARM SoC display driver
$ sudo apt install xserver-xorg-video-armsoc-endlessm
$ cat ~/.xinitrc 
xterm & 
exec openbox-session
$ startx
```

```
cmake \
-D CMAKE_C_COMPILER=/usr/bin/gcc-10 \
-D CMAKE_CXX_COMPILER=/usr/bin/g++-10 \
-D CMAKE_BUILD_TYPE=Release \
-D WITH_GTK=ON \
-D OPENCV_EXTRA_MODULES_PATH=/home/ubuntu/Projects/opencv_build/opencv_contrib/modules/ \
-D PYTHON3_EXECUTABLE=/home/ubuntu/Work/image-show/.venv/bin/python \
-D OPENCV_PYTHON3_INSTALL_PATH=/home/ubuntu/Work/image-show/.venv/lib/python3.12/site-packages/ \
-D BUILD_opencv_python3=ON \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D OPENCV_GENERATE_PKGCONFIG=ON ..
```

/home/ubuntu/Projects/opencv_build/opencv_contrib/modules/

## 電源の起動後

電源はACアダプタを抜き差しするだけ。ホストとの通信は以下の通り。

- USBを介した方法

```
$ sudo dmesg | grep tty
$ sudo minicom -D /dev/ttyUSB1 -b 115200
```

- sshを介した方法

Petalinuxの場合は、[FPGAの部屋](https://marsee101.blog.fc2.com/blog-entry-5725.html)が参考になった

```
$ sudo dnf search ssh
$ sudo dnf install openssh-sftp-server.cortexa72_cortexa53           
```

sftp-server のパスを確認

```                                
$ sudo find /usr /lib -name sftp-server
/usr/libexec/sftp-server 
```

- ホストPCから共有する方法

ホスト側のアドレスを確認

```
$ ifconfig # ethのネットワークアドレスを確認する
enp4s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet XXXX.XXXX.XXXX.1  netmask 255.255.255.0
```

KV260側の設定

```
$ sudo ifconfig eth0 XXXX.XXXX.XXXX.2 netmask 255.255.255.0
$ sudo route add default gw XXXX.XXXX.XXXX.1
$ ping 8.8.8.8
```

## 環境構築

### bootgen

```
git clone https://github.com/Xilinx/bootgen  
cd bootgen/  
make  
sudo cp bootgen /usr/bin/
```

- 使い方

```bash
bootgen -image <イメージ設定ファイル>.bif -o <出力ファイル名>.bin
```

<イメージ設定ファイル>.bif の中に、FSBL・bitstream・U-Boot・Linux などを並べて書き込み、最終的に 1 つのイメージにまとめることができます。

## `xmutil` コマンドのユーティリティ一覧と説明

| **Utility Name**            | **Description**                                                                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `xmutil boardid`            | ボードのEEPROM内容を読み取り、情報の概要をCLIに出力します。                                                                                                   |
| `xmutil bootfw_status`      | プライマリブートデバイス情報を読み取り、A/Bステータス情報、イメージID、およびチェックサムをCLIに出力します。                                                  |
| `xmutil bootfw_update`      | 非アクティブなパーティションに新しいブートイメージを更新するツールです。                                                                                      |
| `xmutil getpkgs`            | ボードID情報に基づいて、アクティブなプラットフォーム向けの関連パッケージをデバッグインターフェースに提供します。 **Kria Ubuntu ではサポートされていません。** |
| `xmutil listapps`           | プラットフォームで利用可能なプリビルトアプリケーションを対象ハードウェアリソースマネージャデーモンに問い合わせ、概要をデバッグインターフェースに提供します。  |
| `xmutil loadapp`            | ビットストリームを含む統合されたHW+SWアプリケーションをロードし、対応するプリビルトアプリケーションソフトウェアを起動します。                                 |
| `xmutil unloadapp`          | アクセラレーションアプリケーションをアンロードし、ビットストリームもアンロードします。                                                                        |
| `xmutil xlnx_platformstats` | CPU周波数、RAM使用量、温度、電力情報などの性能関連情報を読み取り、概要を出力します。                                                                          |
|                             | **注意:** PL SysMonアクセスが利用できない場合、PL温度は`-280°C`と表示されます。詳細はUG1085を参照してください。                                               |
| `xmutil ddrqos`             | PS DDRのQoS設定を変更するユーティリティです。初期実装ではPS DDRメモリコントローラーのトラフィッククラス設定に焦点を当てています。                             |
| `xmutil axiqos`             | PS/PL AXIインターフェースのQoS設定を変更するユーティリティです。初期実装ではAXIポートの読み書き優先順位設定に焦点を当てています。                             |
| `xmutil pwrctl`             | PLの電力制御およびステータス確認のユーティリティです。                                                                                                        |
| `xmutil desktop_disable`    | デスクトップ環境を無効化します。 **Kria Ubuntu Server ではサポートされていません。**                                                                          |
| `xmutil desktop_enable`     | デスクトップ環境を有効化します。 **Kria Ubuntu Server ではサポートされていません。**                                                                          |
| `xmutil dp_bind`            | ディスプレイドライバーをバインドします。                                                                                                                      |
| `xmutil dp_unbind`          | ディスプレイドライバーをアンバインドします。                                                                                                                  |

- Boot Farmwareの確認方法

```terminal
$ sudo xmutil bootfw_status
Image A: Bootable
Image B: Bootable
Requested Boot Image: Image A
Last Booted Image: Image A
XilinxSom_QspiImage_v2.0_07250622
ImageA Revision Info: XilinxSOM_BootFW_20220725
ImageB Revision Info: XilinxSOM_BootFW_20220725
```