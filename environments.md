# 環境構築

## Conda

```
$ conda env export --no-builds > env.yml
$ sed -i '/prefix:/d' env.yml
$ conda env create -f env.yml
```

## Micromanba

```
conda install micromamba -c conda-forge
```

## Gdal

```
micromamba install gdal poppler jsonschema-with-format-nongpl webcolors -c conda-forge
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
  paths = ["./src/"]
```
