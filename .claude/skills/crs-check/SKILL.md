---
name: crs-check
description: ベクターデータの CRS・単位・座標範囲を確認する。データ読込直後、距離/面積/バッファ計算の前、sjoin や overlay の前、結果が不自然なときに使う。
---

# CRS チェック

## 手順
1. `gpd.read_file()` 直後に以下を確認する。
   - `gdf.crs`(None なら警告)
   - 地理座標系か投影座標系か(`gdf.crs.is_geographic`)
   - 単位(`gdf.crs.axis_info[0].unit_name`)
   - `gdf.total_bounds`
2. 範囲が日本として妥当か確認する。
   - 地理座標系: 経度 122〜154、緯度 20〜46
   - 経度・緯度が逆なら軸順の取り違えを疑う
3. 距離・面積・バッファの前に、投影座標系(愛知県は EPSG:6675)であることを確認する。
4. 複数レイヤを結合・重ね合わせる前に、CRS が一致しているか確認する。

## 確認用コード
```python
def crs_report(gdf, name="layer"):
    crs = gdf.crs
    print(f"[{name}] CRS={crs.to_string() if crs else None}")
    if crs:
        print(f"  geographic={crs.is_geographic}, unit={crs.axis_info[0].unit_name}")
    print(f"  bounds={gdf.total_bounds}")
```

## 報告形式
- 問題なし / 要対応を 1 行で示す
- 要対応なら原因と対処(`to_crs` か `set_crs`)を示す。`set_crs` は CRS が未設定のときだけ使う
