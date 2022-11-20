# Title

TorchBeast: A PyTorch Platform for Distributed RL

## Author

Heinrich Küttler*

## Abstruct

- PyTorchのIMPALA実装
- 純粋なPython実装（"MonoBeast"）と、マルチマシン対応の高性能版（"PolyBeast"）の両方を提供している
- 環境はOpenAI Gymインターフェース

## Memo

### MonoBeastの実装

1台のマシンで実行することを想定しており、PyTorchのtensorを共有メモリで保持する機能を活用した実装となっている。  

- *num_buffers*を作成、*rollout buffer*はバッチ次元を除いたものを共有メモリに格納する

> buffers[0]['frame'] = torch.empty(T, *obs_shape, torch.uint8)

- 2つのキューを作成、free_queueとfull_queueはUNIX pipesで通信する

- num_actorsの開始、*actor process*数分のenvironmentを起動する。各アクターはfree_queueからindexをデキューして、ロールアウトデータを含むバッチスライスをbuffers[index]に書き込む。そのとき、indexをfull_queueにエンキューし、次のインデックスをデキューする。

- メインプロセスはいくつかの*learner threads*を持つ

> 1. full_queueからbatch_size-manyのインデックスをdequeue、それらをバッチにスタックしてGPUに移動させて、インデックスをfree_queueに戻す。  
> 2. バッチをモデルに送り、損失を計算し、バックワードパスを行い、重みをhogwild-updatesする。

共有メモリは大量のデータを保持しており、GPUではなくCPUでアクターのモデル評価を行い、厳密には必要ないテンソルコピーも多数含んでいる。

### 

[hogwild!](https://qiita.com/KazukiOsawa/items/3854eaac63db40146e3c)

### baseline

https://gitlab.aicrowd.com/neural-mmo/ijcai2022-nmmo-baselines

https://gitlab.aicrowd.com/nethack/neurips-2021-the-nethack-challenge