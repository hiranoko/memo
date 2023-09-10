# 環境構築

## Conda

```
$ conda env export --no-builds > env.yml
$ sed -i '/prefix:/d' env.yml
$ conda env create -f env.yml
```

## Docker