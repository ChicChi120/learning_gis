# 学習ログ

## 2026-10-29 Phase 1: データ取得と読込・CRS 確認
- やったこと: N03(愛知県・2026-01-01 基準)と OSM 変電所(Overpass Turbo)を取得して data/raw/ に配置、出典・ライセンスを docs/data_sources.md に記録、QGIS で 3 レイヤ(N03 .shp、変電所の点・面)を読み込み CRS を確認
- 分かったこと: N03 は CC BY 4.0、OSM は ODbL でクレジット表記が異なる / OSM は取得日時(timestamp_osm_base)が版になる / OSM は node・way・relation で構成され、変電所は点と面が混在する(Polygon 230・Point 18)/ N03 は EPSG:6668(JGD2011)、OSM GeoJSON は EPSG:4326(WGS 84)で、どちらも単位が度の地理座標系
- つまずき/未解決: ODbL のシェアアライクが成果物(GPKG と地図画像)にどう関係するか / N03 が 1,033 件ある理由 / .shp は Polygon、.geojson は MultiPolygon と表示される理由
- 次回: QGIS のプロジェクト CRS とオンザフライ再投影を確認し、計測ツールで距離を測って単位と計算方法を確かめる
