- [画像処理](#画像処理)
  - [Template Matching](#template-matching)
  - [save figure](#save-figure)
  - [make movie](#make-movie)
  - [画像生成](#画像生成)
  - [空画像の生成](#空画像の生成)
  - [画像のリサイズ](#画像のリサイズ)
  - [色の入れ替え](#色の入れ替え)
  - [透明画像の生成](#透明画像の生成)
  - [CPUコア数をカウント](#cpuコア数をカウント)
- [utils](#utils)
  - [コマンドライン引数](#コマンドライン引数)

## 画像処理

### Template Matching

```python
def calc_CC(template, image):
    # 平均を引いて正規化
    cc = F.conv2d(image, template)
    # 範囲を[0,1]にスケーリング
    cc = (cc - torch.min(cc)) / (torch.max(cc) - torch.min(cc) + 1e-8)
    return cc

def calc_NCC(template, image):
    # テンプレートの大きさの計算
    template_norm = torch.sqrt(torch.sum(template ** 2))
    
    # イメージの各部分の大きさの計算
    image_patch_norms = torch.sqrt(F.conv2d(image ** 2, torch.ones_like(template)))

    # 畳み込みの実行と正規化
    ncc = F.conv2d(image, template) / (image_patch_norms * template_norm + 1e-8)
    return ncc
```

### save figure

```python
buf = io.BytesIO()
plt.savefig(buf, format='png')
enc = np.frombuffer(buf.getvalue(), dtype=np.uint8)
dst = cv2.imdecode(enc, 1)
```

### make movie

```python
writer = cv2.VideoWriter(
    'test.mp4',
    cv2.VideoWriter_fourcc('m', 'p', '4', 'v'),
    30, 
    (640, 480)
) # ライター作成

for frame in frames:
    writer.write(frame)

writer.release()
```

### 画像生成

```python
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

### 空画像の生成 

```python
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

### 画像のリサイズ

```python
path = "./test.jpg"
img = cv.imread(path)
print(img.shape) # 返り値はheight width ch
img = img.resize(img, (height, width))
print(img.shape)
```

### 色の入れ替え

```python
img = np.zeros((256, 512, 3), np.uint8)
black, white = [0, 0, 0], [255, 255, 255]
img[np.where((img == black).all(axis = 2))] = white
```

### 透明画像の生成

```python
img = np.zeros((256, 512, 3), np.uint8)
mask = np.zeros((256, 512, 1), np.uint8)

img = cv2.circle(img, (256, 128), 100, (0, 0, 255), thickness=-1)
mask = cv2.circle(mask, (256, 128), 100, (255), thickness=-1) # 0の部分は透明となる

dst = np.dstack([img, mask])
cv2.imwrite("test.png", dst)

img = cv2.imread("./test.png", -1) # 透明画像を読み込むときの引数は-1
```

### CPUコア数をカウント

```python
import multiprocessing
multiprocessing.cpu_count()
```

## utils

### コマンドライン引数

```pyhton
def get_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--aaa", type=float, default=-1.0)
    parser.add_argument("--bbb", type=str, default='sample.png')
    parser.add_argument("--ccc", type=str, default=None)
    parser.add_argument('--ddd', action='store_true', default=False, help='')
    args = parser.parse_args()
    return args
```
