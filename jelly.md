- [kv260\_sample](#kv260_sample)
  - [KV260のPSからのLチカ](#kv260のpsからのlチカ)
    - [デバッグ方法](#デバッグ方法)

# [kv260_sample](https://github.com/hiranoko/jelly/tree/master/projects/kv260/kv260_sample)

プロジェクトファイルを開く際、ブロックデザイン(.bd)とチェックポイント(.dcp)ファイルが確認される。
指定の部分を削除する、もしくはインクリメンタルフローを無効化する。

1. Flow Navigator -> PROJECT MANAGER -> Settings -> Project Settings
-> Synthesis -> Settings -> Inremental synthesis -> `Not set`

<details>

<summary>2. 削除するとき</summary>

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