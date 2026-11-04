# 学習計画と進捗

## 学習フェーズと進捗
- [x] Phase 0: 環境構築(QGIS 起動、`import geopandas` が通る)
- [ ] Phase 1: QGIS GUI(市区町村ごとの変電所数と 10km バッファのカバー率を出せる)
    ステップ	やること	学べる概念
    ① データ取得	N03(愛知県)と OSM 変電所を data/raw/ に置き、出典を docs/data_sources.md に記録する	データの出典とライセンス
    ② 読込・CRS確認	2つのレイヤの CRS を見る	地理座標系と投影座標系
    ③ 前処理	EPSG:6675 に変換し、面の変電所は点にする	再投影、代表点
        - [x] ③-1 変電所の点を EPSG:6675 に再投影(11/2)→ data/processed/substation_points_6675.gpkg(18 件)
        - [x] ③-2 市区町村を作る(11/3): 再投影 → 所属未定地を除外 → N03_007 で融合 → data/processed/n03_municipalities_6675.gpkg(69 件)
        - [x] ③-3 変電所を「1 か所 = 1 点」にするルールを決める(11/4)
            1. 面を点にする方法: 地物上の点(Point on surface)
            2. 二重カウント: 面の内側にある点は同じ施設とみなし、情報の少ない方を除く → node/1551659855 と way/566534529 を除く
            3. 境界をまたぐ面(9 件): 地物上の点が落ちた市区町村に 1 件として数える
            4. 対象: 鉄道用(substation=traction)と使われていないもの(disused=yes)を除く。種類が NULL のものは残す
        - [ ] ③-4 変電所の点レイヤを作る(③-3 のルールを適用)
            A. 変電所の面を EPSG:6675 に再投影(230 件)
            B. 地物上の点で面を点にする(「各パーツの点を作成」は外す)(230 件)
            C. B の結果と ③-1 の点をマージ(248 件)
            D. 式による抽出で除外 → data/processed/substations_6675.gpkg に保存
               `"id" NOT IN ('node/1551659855', 'way/566534529') AND coalesce("substation", '') <> 'traction' AND coalesce("disused", '') <> 'yes'`
        - [ ] ③-5 確認: D の総地物数を記録する / NULL の比較で件数が変わることを確かめる(任意)
    ④ 件数集計	市区町村ごとに変電所を数える	点が面に含まれるかの判定(point in polygon)
    ⑤ バッファ・カバー率	10km バッファ → 重なりを統合 → 市区町村と交差 → 面積比	バッファ、ディゾルブ、交差
    ⑥ 保存・記録	GPKG で data/processed/ に保存し、.qgz を保存する	再現性(Phase 2・3 の「正解」になる)
- [ ] Phase 2: PyQGIS(GUI と同じ結果をスクリプトで再現)
- [ ] Phase 3: GeoPandas(1本のノートブックで Phase 1 と同じ結果)
- [ ] Phase 4: ミニプロジェクト(送電線をテーマに「線」+「面」の処理を実施)
