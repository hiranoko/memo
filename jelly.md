[Home](./README.md)

- [kv260\_sample](#kv260_sample)
- [kv260\_blinking\_led\_ps](#kv260_blinking_led_ps)

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

# [kv260_blinking_led_ps](https://zenn.dev/ryuz88/articles/kv260_led_blinking_ps)

- address mapの表示

Window -> Address Editor

- `devmem2`で操作

```
$ sudo devmem2 0xa0000000 # 値を確認
$ sudo devmem2 0xa0000000 w 0x1  # LEDを点灯 16進数
$ sudo devmem2 0xa0000000 w 0x0  # LEDを消灯 16進数
```

- `c++`から操作

Linuxでは、システム全体の物理メモリ空間にアクセスする特殊デバイスファイルとして`/dev/mem`が用意されてる。
任意の物理アドレスを`mmap()`することで、そのアドレス領域をユーザ空間から読み書きできる。

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


- `python`からの操作

<details>

<summary>source code</summary>

```python
#!/usr/bin/env python3

import os
import mmap
import time

def main():
    fd = os.open("/dev/mem", os.O_RDWR | os.O_SYNC)
    if fd < 0:
        print("Error: open /dev/mem failed")
        return 1

    map_size = 0x10000
    target_addr = 0xA0000000

    try:
        mem = mmap.mmap(
            fileno=fd,
            length=map_size,
            flags=mmap.MAP_SHARED,
            prot=mmap.PROT_READ | mmap.PROT_WRITE,
            offset=target_addr
        )
    except ValueError as e:
        print("Error: mmap failed:", e)
        os.close(fd)
        return 1

    try:
        for i in range(10):
            current_val = mem[0]  # 先頭バイトを読み取り (0〜255のint)
            print(f"Address: 0x{target_addr:08X}, Current Value: 0x{current_val:02X}")

            # LSB(下位1ビット) をトグル
            new_val = current_val ^ 0x01
            # Python の mmap は単一バイトの書き込み時に int を期待する
            mem[0] = new_val & 0xFF

            # 書き込み後の値を再度読み出し
            updated_val = mem[0]
            print(f"Address: 0x{target_addr:08X}, New Value: 0x{updated_val:02X}")

            time.sleep(0.5)
    finally:
        mem.close()
        os.close(fd)

    return 0


if __name__ == "__main__":
    main()
```

</details>
