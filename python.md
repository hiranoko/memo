- [開発環境](#開発環境)
  - [Jupyter Notebook](#jupyter-notebook)
  - [Virtual Environment](#virtual-environment)
  - [Formatter](#formatter)
- [画像処理](#画像処理)
  - [Video Capture](#video-capture)
  - [Template Matching](#template-matching)
  - [Matplotlib Figure を画像に変換](#matplotlib-figure-を画像に変換)
  - [動画書き出し](#動画書き出し)
  - [画像のリサイズ](#画像のリサイズ)
  - [色の入れ替え](#色の入れ替え)
  - [透明画像の生成](#透明画像の生成)


# 開発環境

## Jupyter Notebook

```
%matplotlib inline
%load_ext autoreload
%autoreload 2 # コードを修正した際、自動的に再読み込み
```

```
import isort
print(isort.code(_ih[1])
```

## Virtual Environment

- conda

```
$ conda env export --no-builds > env.yml
$ sed -i '/prefix:/d' env.yml
$ conda env create -f env.yml
$ conda install gdal poppler jsonschema-with-format-nongpl webcolors sqlite -c conda-forge
```

- venv

```
$ python -m venv .venv --system-site-packages
$ source .venv/bin/activate
```

- 自作のライブラリ

```
pip install -e . --config-settings editable_mode=strict
```

## Formatter

```
pip install "pysen[lint]"
```

```toml
[tool.pysen]
version = "0.10"

[tool.pysen.lint]
enable_black = true
enable_flake8 = true
enable_isort = true
enable_mypy = true
mypy_preset = "strict"
line_length = 88
py_version = "py38"

[[tool.pysen.lint.mypy_targets]]
paths = ["./src"]
```

# 画像処理

## Video Capture

OpenCV でカメラを開く

```python
import cv2

cap = cv2.VideoCapture(0)  # カメラデバイスID 0番を選択

cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)  
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)
cap.set(cv2.CAP_PROP_FPS, 30)  # 実際に反映されない場合もある

width  = cap.get(cv2.CAP_PROP_FRAME_WIDTH)  
height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
fps    = cap.get(cv2.CAP_PROP_FPS)

print(width, height, fps)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    cv2.imshow("camera", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

- cv2.CAP_PROP_FRAME_WIDTH - 幅
- cv2.CAP_PROP_FRAME_HEIGHT - 高さ
- cv2.CAP_PROP_FPS - フレームレート
- cv2.CAP_PROP_FOURCC - コーデック
- cv2.CAP_PROP_BRIGHTNESS - 明るさ
- cv2.CAP_PROP_CONTRAST - コントラスト
- cv2.CAP_PROP_SATURATION - 彩度
- cv2.CAP_PROP_HUE - 色合い

## Template Matching

```python
import torch
import torch.nn.functional as F

def calc_CC(template, image):
    """
    Cross-correlation (CC)
    Args:
        template: (C, H, W) テンプレート
        image: (C, H, W) 入力画像
    Returns:
        cc: CC結果 (conv2dで計算)
    """
    # 画像とテンプレートの平均を差し引いて正規化 などの追加処理をする場合は適宜実装
    cc = F.conv2d(image, template)
    cc = (cc - torch.min(cc)) / (torch.max(cc) - torch.min(cc) + 1e-8)  # [0,1]にスケーリング
    return cc

def calc_NCC(template, image):
    """
    Normalized Cross-correlation (NCC)
    """
    # テンプレートの大きさの計算
    template_norm = torch.sqrt(torch.sum(template ** 2))
    
    # イメージの各部分の大きさの計算
    image_patch_norms = torch.sqrt(F.conv2d(image ** 2, torch.ones_like(template)))

    # 畳み込みの実行と正規化
    ncc = F.conv2d(image, template) / (image_patch_norms * template_norm + 1e-8)
    return ncc

def pixel_corr(z, x):
    """
    Pixel-wise correlation (実装例: forループ + conv2d)
    速度的には高速とは言えないが、仕組み理解向けのサンプル
    """
    size = z.size()  # (bs, c, hz, wz)
    CORR = []
    for i in range(len(x)):
        ker = z[i:i + 1]  # (1, c, hz, wz)
        fea = x[i:i + 1]  # (1, c, hx, wx)
        ker = ker.view(size[1], size[2] * size[3]).transpose(0, 1)  # (hz * wz, c)
        ker = ker.unsqueeze(2).unsqueeze(3)  # (hz * wz, c, 1, 1)
        co = F.conv2d(fea, ker.contiguous())  # (1, hz * wz, hx, wx)
        CORR.append(co)
    corr = torch.cat(CORR, 0)  # (bs, hz * wz, hx, wx)
    return corr
```

## Matplotlib Figure を画像に変換

```python
import io
import numpy as np
import matplotlib.pyplot as plt
import cv2

# figureをメモリに保存し、そのバイナリをOpenCV画像として読み込み
buf = io.BytesIO()
plt.savefig(buf, format='png')
enc = np.frombuffer(buf.getvalue(), dtype=np.uint8)
dst = cv2.imdecode(enc, 1)  # cv2.IMREAD_COLOR と同等
```

## 動画書き出し

```python
import cv2

frames = [...]  # 書き出したい画像フレームのリスト (np.arrayのリストなど)

writer = cv2.VideoWriter(
    'test.mp4',
    cv2.VideoWriter_fourcc('m', 'p', '4', 'v'),
    30, 
    (640, 480)
)  # ライター作成

for frame in frames:
    writer.write(frame)

writer.release()
```

## 画像のリサイズ

```python
def resize_image(
    image: np.ndarray,
    width: Optional[int] = None,
    height: Optional[int] = None,
    inter: int = cv2.INTER_AREA,
) -> np.ndarray:
    (h, w) = image.shape[:2]

    if width is None and height is None:
        return image

    if width is None:
        if height is None:
            return image
        else:
            r = height / float(h)
            dim = (int(w * r), height)
    else:
        r = width / float(w)
        dim = (width, int(h * r))

    return cv2.resize(image, dim, interpolation=inter)
```

## 色の入れ替え

```python
import numpy as np

img = np.zeros((256, 512, 3), np.uint8)
black = [0, 0, 0]
white = [255, 255, 255]

# (0,0,0) → (255,255,255) に置換
img[np.all(img == black, axis=2)] = white
```

## 透明画像の生成

```python
import cv2
import numpy as np

img = np.zeros((256, 512, 3), np.uint8)
mask = np.zeros((256, 512, 1), np.uint8)

# 円を赤色で描画
img = cv2.circle(img, (256, 128), 100, (0, 0, 255), thickness=-1)
# 同じ領域をマスク (白部分が塗りつぶされ、そこは不透明)
mask = cv2.circle(mask, (256, 128), 100, (255), thickness=-1)

dst = np.dstack([img, mask])  # RGBA
cv2.imwrite("test.png", dst)

# 読み込むとき
img_png = cv2.imread("test.png", -1)  # -1指定でアルファチャネルも含む
```