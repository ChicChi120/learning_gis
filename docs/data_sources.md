# データ出典

## N03 行政区域(愛知県)
- 提供者: 国土交通省
- URL: https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03-2026.html
- 年度・基準日: 2026年（令和8年）版 (2026-01-01)
- 取得日: 2026-10-29
- 形式 / ファイル名: data/raw/N03-20260101_23_GML/N03-20260101_23.shp
- ライセンス: オープンデータ（CC_BY_4.0）
- クレジット表記: 「国土数値情報(行政区域データ)」(国土交通省)(https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N03-2026.html)を加工して作成
- 原データの CRS: JGD2011 地理座標系(EPSG:6668、単位: 度)。.shp / .geojson とも同じ
- 備考:

## OSM 変電所(愛知県)
- 提供者: OpenStreetMap contributors
- 取得方法: Overpass Turbo
- クエリ・条件: OSM クエリセクションを参照
- 取得日時: 2026-10-29T07:32:40Z
- 形式 / ファイル名: data/raw/osm_substation_aichi_20261029.geojson
- ライセンス: ODbL
- クレジット表記: © OpenStreetMap contributors(https://www.openstreetmap.org/copyright)
- 原データの CRS: WGS 84 地理座標系(EPSG:4326、単位: 度)。GeoJSON の仕様(RFC 7946)で WGS 84 と定められている
- 備考: 

## OSM クエリ
```SQL  
/* Overpass Turbo で実行した Overpass QL クエリ */
/* JSON 形式で返す。タイムアウトは 180 秒 */
[out:json][timeout:180]; 
/* 愛知県の範囲を探し、aichi という名前で保持する(JP-23 は愛知県の都道府県コード) */
area["ISO3166-2"="JP-23"]->.aichi;  
/* 愛知県の範囲内にある、power=substation を持つ node・way・relation をすべて取り出す */
nwr["power"="substation"](area.aichi);
/* 見つかった要素をタグ付きで出力する */
out body;
/* way を構成する node(輪郭の頂点)をたどって取得する */
>;
/* その頂点の座標を出力する。(これがないと面の形が作れない) */
out skel qt;
```
