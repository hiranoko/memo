- [画像処理](#画像処理)
  - [Video Capture](#video-capture)
  - [Template Matching](#template-matching)
  - [save figure](#save-figure)
  - [make movie](#make-movie)
  - [画像生成](#画像生成)
  - [空画像の生成](#空画像の生成)
  - [画像のリサイズ](#画像のリサイズ)
  - [色の入れ替え](#色の入れ替え)
  - [透明画像の生成](#透明画像の生成)
  - [CPUコア数をカウント](#cpuコア数をカウント)
  - [描画](#描画)

## 画像処理

### Video Capture

- cv2.CAP_PROP_FRAME_WIDTH - 幅
- cv2.CAP_PROP_FRAME_HEIGHT - 高さ
- cv2.CAP_PROP_FPS - フレームレート
- cv2.CAP_PROP_FOURCC - コーデック
- cv2.CAP_PROP_BRIGHTNESS - 明るさ
- cv2.CAP_PROP_CONTRAST - コントラスト
- cv2.CAP_PROP_SATURATION - 彩度
- cv2.CAP_PROP_HUE - 色合い

```python
cap = cv2.VideoCapture(0)  # カメラを選択

cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)  
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)
cap.set(cv2.CAP_PROP_FPS, 30)

cap.get(cv2.CAP_PROP_FRAME_WIDTH)  
cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
cap.get(cv2.CAP_PROP_BUFFERSIZE)
cap.set(cv2.CAP_PROP_FPS)
```

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

def pixel_corr(z, x):
    """Pixel-wise correlation (implementation by for-loop and convolution)
    The speed is slower because the for-loop
    
    https://github1s.com/researchmm/LightTrack/blob/main/lib/models/connect.py#L6-L19
    
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

### 描画

```python
def draw_cross(
    image: np.array, bboxes: np.array, color: tuple = (255, 0, 0)
) -> np.array:
    for bbox in bboxes:
        x = int((bbox[0] + bbox[2]) / 2)
        y = int((bbox[1] + bbox[3]) / 2)
        # バウンディングボックス
        image = cv2.drawMarker(image, (x, y), color, 0, 40, 3)
    return image

def draw_bbox(
    image: np.array, bboxes: np.array, color: tuple = (255, 0, 0)
) -> np.array:
    for bbox in bboxes:
        x1, y1, x2, y2 = int(bbox[0]), int(bbox[1]), int(bbox[2]), int(bbox[3])
        # バウンディングボックス
        image = cv2.rectangle(
            image,
            (x1, y1),
            (x2, y2),
            color,
            thickness=2,
        )
    return image
```

## calibration

```python
import bpy
import math
import random

# シーンの初期化
def clear_scene():
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

# チェッカーボードの作成
def create_checkerboard(rows=7, cols=8, square_size=0.1):
    collection = bpy.data.collections.new("Checkerboard")
    bpy.context.scene.collection.children.link(collection)
    for row in range(rows):
        for col in range(cols):
            if (row + col) % 2 == 0:
                x = col * square_size
                y = row * square_size
                bpy.ops.mesh.primitive_plane_add(size=square_size, location=(x, y, 0))
                square = bpy.context.object
                square.name = f"Square_{row}_{col}"
                collection.objects.link(square)
                bpy.context.scene.collection.objects.unlink(square)
    return collection

# カメラの設定
def setup_camera(location=(2, 2, 2), rotation=(math.radians(60), 0, math.radians(45))):
    bpy.ops.object.camera_add(location=location, rotation=rotation)
    camera = bpy.context.object
    camera.name = "CalibrationCamera"
    camera.data.lens = 35
    camera.data.sensor_width = 36
    camera.data.sensor_height = 24
    return camera

# チェッカーボードの変換
def transform_checkerboard(collection, move_range=1.0, rotate_range=math.radians(15)):
    for obj in collection.objects:
        obj.location.x += random.uniform(-move_range, move_range)
        obj.location.y += random.uniform(-move_range, move_range)
        obj.location.z += random.uniform(-move_range, move_range)
        obj.rotation_euler.x += random.uniform(-rotate_range, rotate_range)
        obj.rotation_euler.y += random.uniform(-rotate_range, rotate_range)
        obj.rotation_euler.z += random.uniform(-rotate_range, rotate_range)

# ワールド座標からカメラ座標への変換
def world_to_camera(camera, world_coords):
    camera_matrix = camera.matrix_world.inverted()
    return [camera_matrix @ coord for coord in world_coords]

# カメラ座標から画像座標への変換
def camera_to_image(camera, camera_coords):
    lens = camera.data.lens
    sensor_width = camera.data.sensor_width
    sensor_height = camera.data.sensor_height
    resolution_x = bpy.context.scene.render.resolution_x
    resolution_y = bpy.context.scene.render.resolution_y
    image_coords = []
    for coord in camera_coords:
        if coord.z > 0:
            x = (lens * coord.x / coord.z + sensor_width / 2) * resolution_x / sensor_width
            y = (lens * coord.y / coord.z + sensor_height / 2) * resolution_y / sensor_height
            image_coords.append((x, y))
        else:
            image_coords.append(None)
    return image_coords

# 頂点座標の取得
def get_checkerboard_vertices(collection):
    vertices = []
    for obj in collection.objects:
        if obj.type == 'MESH':
            obj_mesh = obj.data
            for vertex in obj_mesh.vertices:
                world_coord = obj.matrix_world @ vertex.co
                vertices.append(world_coord)
    return vertices

# 画像のレンダリング
def render_image(filepath="render.png"):
    bpy.context.scene.render.image_settings.file_format = 'PNG'
    bpy.context.scene.render.filepath = filepath
    bpy.ops.render.render(write_still=True)

# メイン処理
def main():
    clear_scene()
    checkerboard = create_checkerboard()
    camera = setup_camera()
    transform_checkerboard(checkerboard)
    world_coords = get_checkerboard_vertices(checkerboard)
    camera_coords = world_to_camera(camera, world_coords)
    image_coords = camera_to_image(camera, camera_coords)
    render_image()
    print("3Dワールド座標と対応する2D画像座標:")
    for world, image in zip(world_coords, image_coords):
        if image:
            print(f"3D: {world}, 2D: {image}")

if __name__ == "__main__":
    main()
```
