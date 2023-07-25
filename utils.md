- [utils](#utils)
  - [コマンドライン引数](#コマンドライン引数)
  - [ファイル一覧を取得](#ファイル一覧を取得)
- [Formatter \& Linter](#formatter--linter)
- [setup.py](#setuppy)

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

### ファイル一覧を取得

```python
def get_image_list(path: str) -> list:
    image_names = []
    for maindir, _subdir, file_name_list in os.walk(path):
        for filename in file_name_list:
            apath = os.path.join(maindir, filename)
            ext = os.path.splitext(apath)[1]
            if ext in IMAGE_EXT:
                image_names.append(apath)
    return image_names
```

## Formatter & Linter

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
  paths = ["./"]
```

## setup.py

```python
from setuptools import find_packages, setup


def get_install_requirements():
    with open("requirements.txt", "r", encoding="utf-8") as f:
        reqs = [x.strip() for x in f.read().splitlines()]
    reqs = [x for x in reqs if not x.startswith("#")]
    return reqs


setup(
    name="library_name",
    version="0.0.1",
    packages=find_packages(),
    python_requires=">=3.8, <4",
    install_requires=get_install_requirements(),
    include_package_data=True,
    zip_safe=False,
)
```