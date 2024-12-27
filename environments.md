# 環境構築

## Conda

```
$ conda env export --no-builds > env.yml
$ sed -i '/prefix:/d' env.yml
$ conda env create -f env.yml
```

## venv

```
$ python -m venv .venv --system-site-packages
$ source .venv/bin/activate
```

## Micromanba

```
conda install micromamba -c conda-forge
```

## Gdal

```
conda install gdal poppler jsonschema-with-format-nongpl webcolors sqlite -c conda-forge
```

## Pysen

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
  paths = ["./- editable install
```

## 編集可能なインストール

自作のライブラリでもvscodeで補完ができるように

```
pip install -e . --config-settings editable_mode=strict
```


