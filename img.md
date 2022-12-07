
## 画像処理

### make movie

```python
```

### 画像生成
```
# 1chのブランク画像を生成
img = np.zeros((height, width), np.uint8)

# 1chの画像をスタックして3chのブランク画像を生成
img = np.dstack([img, img, img])

# 3chのブランク画像を生成
img = np.zeros((height, width, 3), np.uint8)

# 4chのブランク画像を生成,png形式の透明画像で使用する
img = np.zeros((height, width, 4), np.uint8)

# 画像を出力
cv2.imwrite("./test.jpg", img)
```

---
### 空画像の生成 
```
# 1chのブランク画像を生成
img = np.zeros((height, width), np.uint8)

# 1chの画像をスタックして3chのブランク画像を生成
img = np.dstack([img, img, img])

# 3chのブランク画像を生成
img = np.zeros((height, width, 3), np.uint8)

# 4chのブランク画像を生成,png形式の透明画像で使用する
img = np.zeros((height, width, 4), np.uint8)

# 画像を出力
cv2.imwrite("./test.jpg", img)
```

---
### 画像のリサイズ
```
path = "./test.jpg"
img = cv.imread(path)
print(img.shape) # 返り値はheight width ch
img = img.resize(img, (height, width))
print(img.shape)
```
---
### 色の入れ替え
```
img = np.zeros((256, 512, 3), np.uint8)
black, white = [0, 0, 0], [255, 255, 255]
img[np.where((img == black).all(axis = 2))] = white
```
---
### 透明画像の生成
```
img = np.zeros((256, 512, 3), np.uint8)
mask = np.zeros((256, 512, 1), np.uint8)

img = cv2.circle(img, (256, 128), 100, (0, 0, 255), thickness=-1)
mask = cv2.circle(mask, (256, 128), 100, (255), thickness=-1) # 0の部分は透明となる

dst = np.dstack([img, mask])
cv2.imwrite("test.png", dst)

img = cv2.imread("./test.png", -1) # 透明画像を読み込むときの引数は-1
```

---
### CPUコア数をカウント
```
import multiprocessing
multiprocessing.cpu_count()
```

## utils

### コマンドライン引数
```
def get_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--aaa", type=float, default=-1.0)
    parser.add_argument("--bbb", type=str, default='sample.png')
    parser.add_argument("--ccc", type=str, default=None)
    parser.add_argument('--ddd', action='store_true', default=False, help='')
    args = parser.parse_args()
    return args
```
---
### 