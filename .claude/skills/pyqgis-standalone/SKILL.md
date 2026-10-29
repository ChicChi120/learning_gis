---
name: pyqgis-standalone
description: VSCode やターミナルから、QGIS を起動せずに PyQGIS スクリプトを実行する。Phase 2 で scripts/ 配下のスクリプトを作る・動かすときに使う。
---

# PyQGIS スクリプト(スタンドアロン実行)

## 前提
- PyQGIS は QGIS.app 同梱の Python で動かす。`.venv` の Python では `import qgis` できない
- アプリ名(`QGIS.app` / `QGIS-LTR.app`)とバージョンは環境で異なる。**最初に確認する**
  ```bash
  ls /Applications | grep -i qgis
  /Applications/QGIS.app/Contents/MacOS/bin/python3 --version
  ```
- パスが違う場合は `QGIS_APP` を実際のアプリ名に直す。バージョン固有の API は公式ドキュメント(Context7 / PyQGIS Developer Cookbook)で確認する

## 実行方法
```bash
QGIS_APP=/Applications/QGIS.app
PYTHONPATH=$QGIS_APP/Contents/Resources/python \
  $QGIS_APP/Contents/MacOS/bin/python3 scripts/buffer_shelters.py
```
- GUI で作業する場合は、QGIS の Python コンソール → エディタ(`Show Editor`)でも実行できる
- VSCode のインタープリタは `.venv` のままにし、PyQGIS スクリプトだけ上記コマンドで実行する

## スクリプトの雛形
```python
import sys
from qgis.core import QgsApplication, QgsVectorLayer

QGIS_APP = "/Applications/QGIS.app"
QgsApplication.setPrefixPath(f"{QGIS_APP}/Contents/MacOS", True)
qgs = QgsApplication([], False)
qgs.initQgis()

sys.path.append(f"{QGIS_APP}/Contents/Resources/python/plugins")
import processing
from processing.core.Processing import Processing
from qgis.analysis import QgsNativeAlgorithms

Processing.initialize()
QgsApplication.processingRegistry().addProvider(QgsNativeAlgorithms())

# --- 処理本体 ---
result = processing.run("native:buffer", {
    "INPUT": "data/processed/substations_6675.gpkg",
    "DISTANCE": 10000,
    "SEGMENTS": 5,
    "DISSOLVE": False,
    "OUTPUT": "data/processed/substations_buffer_pyqgis.gpkg",
})
layer = QgsVectorLayer(result["OUTPUT"], "buffer", "ogr")
print("valid:", layer.isValid(), "count:", layer.featureCount())

qgs.exitQgis()
```

## 進め方
1. GUI で実行 → Processing 履歴の「Python コマンドとしてコピー」を貼り付ける
2. 入出力パスを相対パスに直す(`data/raw/` へは書き込まない)
3. 実行し、件数が GUI の結果と一致するか確認する
4. 一致したら Phase 3 の `qgis-to-geopandas` で再現する
