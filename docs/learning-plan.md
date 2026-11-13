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
            1. 面を点にする方法: 内部保証点(Point on surface)
            2. 二重カウント: 面の内側にある点は同じ施設とみなし、情報の少ない方を除く → node/1551659855 と way/566534529 を除く
            3. 境界をまたぐ面(9 件): 内部保証点が落ちた市区町村に 1 件として数える
            4. 対象: 鉄道用(substation=traction)と使われていないもの(disused=yes)を除く。種類が NULL のものは残す
        - [x] ③-4 変電所の点レイヤを作る(③-3 のルールを適用)
            A. 変電所の面を EPSG:6675 に再投影(230 件)✅ 11/5
            B. 内部保証点で面を点にする(「パーツ」を含むチェックボックスは外す)(230 件)✅ 11/5
            C. B の結果と ③-1 の点をマージ(248 件)✅ 11/5
            D. 式による抽出で除外 → data/processed/substations_6675.gpkg に保存(228 件)✅ 11/5
               `"id" NOT IN ('node/1551659855', 'way/566534529') AND coalesce("substation", '') <> 'traction' AND coalesce("disused", '') <> 'yes'`
        - [x] ③-5 確認(11/5): 除外 20 件 = id 2 + traction 17 + disused 1(重なりなし)/ `"substation" <> 'traction'` は 100 件、coalesce ありは 231 件(差 131 = substation が NULL の行)
    ④ 件数集計	市区町村ごとに変電所を数える	点が面に含まれるかの判定(point in polygon)
        - [x] ④-1 検算の基準を決める(QGIS 操作なし)
            - 69 市区町村の件数の合計は何件になるべきかを考える(③-4 の 228 件のうち、どの市区町村にも入らない点はあり得るか)
            - 点がちょうど境界線の上にあると、2 つの市区町村で数えられる可能性があることを知っておく
        - [x] ④-2 市区町村ごとに変電所を数える
            A. ツールボックスで「ポリゴン内の点の数をカウント」(英語名 Count points in polygon)を開く(出なければ `count` で検索)
            B. ポリゴン = n03_municipalities_6675、点 = substations_6675、カウントのフィールド名 = NUMPOINTS(初期値のまま)
            C. 出力を data/processed/n03_substation_count_6675.gpkg に保存
        - [x] ④-3 結果を確認する
            A. 出力の地物数が 69 件か(市区町村の行が増減していないか)
            B. 統計パネルで NUMPOINTS の合計が ④-1 の基準と一致するか
            C. NUMPOINTS が 0 の市区町村の件数を数え、地図で位置を見て理由を考える
            D. 境界をまたぐ変電所の面(9 件)のうち 1 件を選び、内部保証点が落ちた市区町村で数えられているかを確かめる
    ★ 寄り道: GeoServer を試す(⑤ の前に実施。準備 約 15 分 + 試し作業 約 30 分)
        目的: QGIS(手元で分析するデスクトップ GIS)と GeoServer(データを Web で配信するサーバー)の役割の違いを知る
        学べる概念: OGC 標準(WMS = 地図を画像で配信、WFS = 地物をデータで配信)、ワークスペース・ストア・レイヤ、サーバー側の再投影
        - [x] G-0 準備(11/12): openjdk 17.0.21 / geoserver-3.0.2-bin.zip を ~/Applications/geoserver に展開 / admin でログイン(サンプルの 5 ワークスペース・28 レイヤ)
            A. Java 17 を入れる(GeoServer は Java 17 か 21 が必要。現在この Mac には Java が入っていない)。`brew install openjdk@17` か Adoptium の Temurin 17 のインストーラ
            B. GeoServer 公式サイトから安定版(3.0 系)の Platform Independent Binary(zip)を取得し、プロジェクトの外(例: ~/Applications/geoserver)に展開する。リポジトリには入れない
            C. `bin/startup.sh` で起動し、ブラウザで http://localhost:8080/geoserver を開き、admin / geoserver(初期パスワード)でログインできることを確かめる
            - Docker を使う方法(`docker run -it -p 8080:8080 docker.osgeo.org/geoserver:3.0.x`)もあるが、Docker Desktop の導入が必要になるため今回は使わない
        - [x] G-1 データを公開する(約 10 分)(11/13: aichi:n03_substation_count_6675、Native / Declared とも EPSG:6675)
            A. ワークスペースを作る(名前: aichi。ワークスペース名は 10 文字以内・空白なしの制約があるため learning_gis は使えない)
            B. ストアを追加する: 種類 = GeoPackage、データベース = data/processed/n03_substation_count_6675.gpkg(絶対パス)、Read only にチェック
            C. レイヤ n03_substation_count_6675 を公開する。宣言された CRS が EPSG:6675 か確認し、境界ボックスを「データから計算」で埋めて保存する
        - [x] G-2 配信された地図とデータを見る(約 10 分)(11/13: 豊橋市 NUMPOINTS 17 / EPSG:4326 の画像は縦に縮む)
            A. Browse Layers(3.0 で Layer Preview から改名。上のメニューの GeoServer の中)の OpenLayers で地図を表示し、市区町村をクリックして NUMPOINTS が見えるか確かめる
            B. WMS の GetMap の URL をブラウザで開き、返ってくるのが「画像(PNG)」であることを確かめる。srs を EPSG:4326 に変えても表示できる理由を考える
            C. WFS の GetFeature(outputFormat=application/json)の URL を開き、返ってくるのが「属性付きの地物データ」であることを確かめる
        - [x] G-3 QGIS から GeoServer に接続して比べる(約 10 分)(11/13: WFS で 69 件・NUMPOINTS 合計 228 が一致 / WMS は属性テーブル不可、地物情報表示は HTML)
            A. QGIS で WMS レイヤと WFS レイヤとして同じレイヤを追加する
            B. WMS レイヤと WFS レイヤで、地物情報表示・属性テーブルで何ができて何ができないかを比べる
            C. WFS レイヤの地物数が 69 件、NUMPOINTS の合計が 228 で、④ の結果と一致するか確かめる
        - 片付け: ターミナルで Ctrl+C(または `bin/shutdown.sh`)で停止する。GeoServer は localhost だけで使い、外部に公開しない(OSM 由来のデータは ODbL のため、公開するならクレジット表記などの条件を確かめる必要がある)
    ⑤ バッファ・カバー率	10km バッファ → 重なりを統合 → 市区町村と交差 → 面積比	バッファ、ディゾルブ、交差
    ⑥ 保存・記録	GPKG で data/processed/ に保存し、.qgz を保存する	再現性(Phase 2・3 の「正解」になる)
- [ ] Phase 2: PyQGIS(GUI と同じ結果をスクリプトで再現)
- [ ] Phase 3: GeoPandas(1本のノートブックで Phase 1 と同じ結果)
- [ ] Phase 4: ミニプロジェクト(送電線をテーマに「線」+「面」の処理を実施)
