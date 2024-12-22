- [KV260](#kv260)
  - [OSイメージ](#osイメージ)
  - [電源の起動後](#電源の起動後)
  - [環境構築](#環境構築)
    - [bootgen](#bootgen)
  - [`xmutil` コマンドのユーティリティ一覧と説明](#xmutil-コマンドのユーティリティ一覧と説明)
    - [Boot Farmwareの確認方法](#boot-farmwareの確認方法)
- [Demo](#demo)
  - [Lチカ](#lチカ)
    - [プロジェクト作成](#プロジェクト作成)
  - [KV260のPSからのLチカ](#kv260のpsからのlチカ)
    - [デバッグ方法](#デバッグ方法)

# KV260

## OSイメージ

KV260はwikiに使用可能なOSが[配布](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/1641152513/Kria+SOMs+Starter+Kits#K26-Starter-Kit-Linux)されている。  
リンク先からpetalinuxのイメージファイルが入手する。Ubuntu-serverを例に操作方法を示す。

```
iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img.xz
```

入手したファイルはxz圧縮されているので解凍が必要である。
   
```
$ xz -dk iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img.xz
$ ls iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img
```

SDカードをPCに接続して`disk`を開く。  
ボリュームを空領域とした状態でデバイスの場所`/dev/sda`を確認する。  
デバイスの場所が**問題なければ**書き込み。

```
sudo dd if=iot-limerick-kria-classic-server-2404-classic-24.04-x05-20241114.img of=/dev/sda bs=4M status=progress
```

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
        inet hoge.hoge.hoge.1  netmask 255.255.255.0
```

KV260側の設定

```
$ sudo ifconfig eth0 hoge.hoge.hoge.2 netmask 255.255.255.0
$ sudo route add default gw hoge.hoge.hoge.1
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

### Boot Farmwareの確認方法

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

# Demo

## [Lチカ](https://github.com/ryuz/jelly/tree/master/projects/kv260/kv260_sample)

### プロジェクト作成

プロジェクトファイルを開く際にブロックデザイン(.bd)と論理合成後に作れられるチェックポイント(.dcp)ファイルは削除しておく。


<details>

<summary>具体例</summary>

```diff
- <File Path="$PSRCDIR/sources_1/bd/design_1/design_1.bd">
-        <FileInfo>
-          <Attr Name="UsedIn" Val="synthesis"/>
-          <Attr Name="UsedIn" Val="implementation"/>
-          <Attr Name="UsedIn" Val="simulation"/>
-        </FileInfo>
-        <CompFileExtendedInfo CompFileName="design_1.bd" FileRelPathName="ip/design_1_zynq_ultra_ps_e_0_0_1/design_1_zynq_ultra_ps_e_0_0.xci">
-          <Proxy FileSetName="design_1_zynq_ultra_ps_e_0_0"/>
-        </CompFileExtendedInfo>
-      </File>    
+
```

```diff
-      <File Path="$PPRDIR/project_1/project_1.srcs/utils_1/imports/synth_1/kv260_sample.dcp">
-        <FileInfo>
-          <Attr Name="UsedIn" Val="synthesis"/>
-          <Attr Name="UsedIn" Val="implementation"/>
-          <Attr Name="UsedInSteps" Val="synth_1"/>
-          <Attr Name="AutoDcp" Val="1"/>
-        </FileInfo>
-      </File>
+
```

AutoIncrementalCheckpointが有効になっている部分は、AutoIncrementalDirが異なるとエラーになる。
`AutoIncrementalDir`がある項目は一通り、空白に変えておく。

```diff
-     AutoIncrementalCheckpoint="true" IncrementalCheckpoint="$PPRDIR/project_1/project_1.srcs/utils_1/imports/synth_1/kv260_sample.dcp"
+     AutoIncrementalCheckpoint="true" IncrementalCheckpoint=""
```

</details>

## [KV260のPSからのLチカ](https://zenn.dev/ryuz88/articles/kv260_led_blinking_ps)

### デバッグ方法

```
$ sudo devmem2 0xa0000000 # 値を確認
$ sudo devmem2 0xa0000000 w 0x1  # LEDを点灯
$ sudo devmem2 0xa0000000 w 0x0  # LEDを消灯
```

<details>

<summary>source code</summary>

```cpp
#include <iostream>
#include <iomanip> // std::hex, std::dec
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    // デバイスオープン
    auto fd = open("/dev/mem", O_RDWR | O_SYNC);
    if (fd < 0) {
        std::cerr << "Error: open /dev/mem failed" << std::endl;
        return 1;
    }

    size_t map_size = 0x10000;
    off_t target_addr = 0xa0000000;

    // メモリをマップ
    void* iomap = mmap(0, map_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, target_addr);
    if (iomap == MAP_FAILED) {
        std::cerr << "Error: mmap failed" << std::endl;
        close(fd);
        return 1;
    }

    volatile unsigned char* reg = static_cast<volatile unsigned char*>(iomap);

    // LEDの状態とアドレスをデバッグ表示
    for (int i = 0; i < 10; ++i) {
        // 現在の値を表示
        std::cout << "Address: 0x" << std::hex << target_addr
                  << ", Current Value: 0x" << static_cast<int>(*reg) << std::endl;

        // 値をトグルして書き込み
        *reg ^= 1;

        // 書き込み後の値を表示
        std::cout << "Address: 0x" << std::hex << target_addr
                  << ", New Value: 0x" << static_cast<int>(*reg) << std::endl;

        usleep(500000);
    }

    // マップ解除とクローズ
    munmap(iomap, map_size);
    close(fd);

    return 0;
}
```

</details>