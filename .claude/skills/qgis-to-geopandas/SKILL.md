---
name: qgis-to-geopandas
description: QGIS で行った処理(バッファ、空間結合、クリップ等)を GeoPandas で再現し、QGIS の結果と件数・面積で突き合わせる。Phase 3 のノートブック作成時や、結果が QGIS と合わないときに使う。
---

# QGIS → GeoPandas 再現と突合

## 手順
1. `qgis/` の処理履歴(Processing → 履歴)で、使ったアルゴリズムと入力パラメータを確認する。
2. 下表で対応する GeoPandas の処理を選び、ノートブックに 1 処理 = 1 セルで書く。
3. 各セルの直後に、件数(と面積)を表示する。
4. QGIS の出力(`data/processed/*.gpkg`)を読み込み、`compare()` で突き合わせる。

## 対応表
| QGIS アルゴリズム | GeoPandas |
|---|---|
| `native:reprojectlayer` | `gdf.to_crs("EPSG:6675")` |
| `native:extractbyexpression` | `gdf[gdf["col"] == v]` / `gdf.query(...)` |
| `native:buffer` | `gdf.buffer(10000, resolution=5)` |
| `native:joinattributesbylocation` | `gpd.sjoin(left, right, how="left", predicate="intersects")` |
| `native:countpointsinpolygon` | `sjoin` → `groupby(...).size()` |
| `native:clip` | `gpd.clip(gdf, mask)` |
| `native:dissolve` | `gdf.dissolve(by="col")` |
| `native:intersection` | `gpd.overlay(a, b, how="intersection")` |
| `native:fieldcalculator` | `gdf.assign(new=...)` |

## 突合コード
```python
import geopandas as gpd

def compare(name, gdf_gp, qgis_path, layer=None, crs="EPSG:6675", tol=1e-3):
    q = gpd.read_file(qgis_path, layer=layer).to_crs(crs)
    g = gdf_gp.to_crs(crs)
    print(f"[{name}] count  QGIS={len(q)}  GeoPandas={len(g)}")
    assert len(q) == len(g), "件数が不一致"
    if q.geom_type.str.contains("Polygon").all():
        aq, ag = q.area.sum(), g.area.sum()
        rel = abs(ag - aq) / aq
        print(f"[{name}] area   QGIS={aq:,.1f}  GeoPandas={ag:,.1f}  rel={rel:.2e}")
        assert rel < tol, "面積が許容誤差を超過"
```

## 不一致のときの確認順
1. CRS が揃っているか(`crs-check` を使う)
2. バッファの分割数: QGIS の既定は 5 → `resolution=5` に合わせる
3. `sjoin` の `predicate`: 境界上の点で `intersects` と `within` の結果が変わる。1 対多で件数が増えていないかも確認する
4. 不正ジオメトリ: `gdf.is_valid.all()`、必要なら `make_valid()`
5. 文字化け: 国土数値情報の Shapefile は `encoding="cp932"` が必要なことがある
6. QGIS 側の選択地物のみ処理していないか(「選択地物のみ」オプション)
