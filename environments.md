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

## Docker
