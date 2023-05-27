- [Algorithms](#algorithms)
  - [Complexity](#complexity)
  - [Cumulative sum](#cumulative-sum)
  - [Binary search](#binary-search)
  - [Dynamic programming](#dynamic-programming)
  - [Mathematical problems](#mathematical-problems)
  - [Consideration techniques](#consideration-techniques)
  - [Heuristics](#heuristics)
  - [Data structures and query processing](#data-structures-and-query-processing)
  - [Graph algorithms](#graph-algorithms)
  - [Others](#others)
    - [Atcoder](#atcoder)

# Algorithms
## Complexity
## Cumulative sum
## Binary search

- [bisect](https://docs.python.org/ja/3/library/bisect.html)

## Dynamic programming
## Mathematical problems
## Consideration techniques
## Heuristics
## Data structures and query processing
## Graph algorithms

```python
import numpy as np
from collections import deque

def count_paths(adjacency_matrix):
    # グラフのサイズを取得
    num_rows, num_cols = adjacency_matrix.shape

    # 出発点と目的地の座標
    start = (0, 0)
    goal = (num_rows - 1, num_cols - 1)

    # 最短経路の数を保持する配列
    path_counts = np.zeros((num_rows, num_cols), dtype=int)

    # 幅優先探索のためのキューを初期化
    queue = deque()
    queue.append(start)

    # スタート地点の最短経路数を初期化
    path_counts[start] = 1

    # 幅優先探索
    while queue:
        current = queue.popleft()

        # ゴールに到達した場合、終了
        if current == goal:
            break

        # 上下左右の移動
        moves = [(0, 1), (0, -1), (1, 0), (-1, 0)]
        for move in moves:
            next_pos = (current[0] + move[0], current[1] + move[1])

            # マスが範囲内かつ白いマスである場合
            if 0 <= next_pos[0] < num_rows and 0 <= next_pos[1] < num_cols and adjacency_matrix[next_pos] == 0:
                # 未訪問の場合、キューに追加し最短経路数を更新
                if path_counts[next_pos] == 0:
                    queue.append(next_pos)
                    path_counts[next_pos] = path_counts[current]

                # 既に訪問済みの場合、最短経路数を累積して更新
                else:
                    path_counts[next_pos] += path_counts[current]

    return path_counts[goal]

# 隣接行列を作成
adjacency_matrix = np.array([
    [0, 1, 0, 0],
    [0, 0, 0, 1],
    [0, 0, 1, 0],
    [0, 0, 0, 0]
])

# 最短経路の数を計算
result = count_paths(adjacency_matrix)
print("最短経路の数:", result)

```

```python
import numpy as np

def adjacency_matrix_to_list(adj_matrix):
    num_vertices = len(adj_matrix)
    adj_list = []

    for vertex in range(num_vertices):
        neighbors = []
        for neighbor in range(num_vertices):
            weight = adj_matrix[vertex, neighbor]
            if weight != 0:
                neighbors.append((neighbor, weight))
        adj_list.append(neighbors)

    return adj_list

# グラフの隣接行列を作成
adjacency_matrix = np.array([[0, 2, 0, 4, 0],
                             [2, 0, 1, 0, 0],
                             [0, 1, 0, 3, 0],
                             [4, 0, 3, 0, 1],
                             [0, 0, 0, 1, 0]])

# 隣接行列を隣接リストに変換
adjacency_list = adjacency_matrix_to_list(adjacency_matrix)

# 変換された隣接リストを表示
for vertex, neighbors in enumerate(adjacency_list):
    print(f"頂点 {vertex}: {neighbors}")
```

```python
import numpy as np
from heapq import heappop, heappush

def dijkstra(graph, start_vertex):
    num_vertices = len(graph)
    distances = np.full(num_vertices, np.inf)  # 初期距離を無限大に設定
    distances[start_vertex] = 0  # 開始頂点の距離を0に設定
    visited = np.zeros(num_vertices, dtype=bool)  # 訪れた頂点のフラグ
    queue = [(0, start_vertex)]  # (距離, 頂点) のタプルを格納する優先度キュー

    while queue:
        current_distance, current_vertex = heappop(queue)  # 距離が最小の頂点を取り出す
        if visited[current_vertex]:
            continue
        visited[current_vertex] = True

        for neighbor, weight in graph[current_vertex]:  # 隣接する頂点と重みを取得
            if current_distance + weight < distances[neighbor]:
                distances[neighbor] = current_distance + weight
                heappush(queue, (distances[neighbor], neighbor))  # 更新された頂点をキューに追加

    return distances

# グラフの隣接リストを作成
graph = [[(1, 2), (3, 4)],
         [(0, 2), (2, 1)],
         [(1, 1), (3, 3)],
         [(0, 4), (2, 3), (4, 1)],
         [(3, 1)]]

# ダイクストラ法を実行
start_vertex = 0
distances = dijkstra(graph, start_vertex)

# 開始頂点から各頂点への最短距離を表示
for i in range(len(distances)):
    if i != start_vertex:
        print(f"頂点 {i} までの最短距離: {distances[i]}")
```

## Others

### Atcoder

| Packages | Version |
|:--------:|:-------:|
|  Python   |  3.8.2  |
|  NumPy    |      1.18.2 |
|  SciPy    |      1.4.1 |
|  scikit-learn |  0.22.2.post1 |
|  Numba    |      0.48.0 |
|  1NetworkX    |   2.4 |
