# GIS 学習プロジェクト

## 目的
QGIS(GUI) → PyQGIS → GeoPandas の順に、**同じ処理を3段階で再現**しながら GIS の基礎を学ぶ。

共通テーマ:「愛知県の変電所を市区町村ごとに集計し、10km バッファ(仮想の IoT 通信範囲)でカバー率を見る」

## 学習者
- Python 経験あり、GIS は初学者
- GIS について基礎的な内容を習得するための伴走をしてほしいです。
- 質問に対して直接的な回答ではなく、考え方を教えるように回答してください。

## 環境
- macOS 26.6.2 / VSCode
- Python: `.venv`(venv)。QGIS 同梱の Python とは別物
- 主要パッケージ: geopandas, pyogrio, shapely, pyproj, folium, matplotlib, jupyter
- QGIS: LTR 版。PyQGIS はバージョン差があるため、使用中の版の公式ドキュメントを確認する
- 実行: `source .venv/bin/activate` → `jupyter lab` または VSCode の Jupyter 拡張

## ディレクトリ
```
data/raw/        元データ(読み取り専用。編集・上書き禁止)
data/processed/  処理結果(GPKG 推奨)
qgis/            QGIS プロジェクト(.qgz)、処理履歴
scripts/         PyQGIS / Python スクリプト
notebooks/       GeoPandas + Folium ノートブック
docs/            data_sources.md(出典)、learning-log.md(学習記録)
```

## 学習フェーズと進捗
`docs/learning-plan.md` で管理する。ユーザーが「完了」と言ったときだけチェックを更新する。

## ルール
1. **編集指示のあった箇所以外のコードは変更しない。** 気づいた問題は修正せず、指摘のみ行う
2. `data/raw/` は変更しない。加工結果は `data/processed/` に出す
3. **CRS を必ず明示する。**
   - 読込直後に `.crs` を確認
   - 距離・面積は投影座標系(愛知県は EPSG:6675)で計算
   - Web 表示(Folium)は EPSG:4326
4. **QGIS の結果を正解とする。** PyQGIS / GeoPandas の結果は、件数と面積合計で突き合わせる
5. 説明は「概念 → 最小コード → 確認方法」の順にする
6. パスは相対パスにする。ノートブック冒頭で CRS・パス・乱数シードを明示する
7. データの出典・取得日・ライセンスを `docs/data_sources.md` に記録する
8. 国土数値情報などは利用規約の同意が必要なため、**自動ダウンロードしない**。ユーザーが `data/raw/` に置く
9. 新しい API を使う前に、公式ドキュメント(Context7 MCP)で現行の仕様を確認する
10. ユーザーが「自分で試す」と言った場合は、答えを出さずヒントのみ示す

## 突合の目安
- フィーチャ件数: 完全一致
- 面積合計: 相対誤差 0.1% 未満(バッファの分割数の差に注意)

## 使える Skills(`.claude/skills/`)
- `crs-check`: CRS・単位・範囲の確認
- `qgis-to-geopandas`: QGIS 処理の GeoPandas 変換と結果の突合
- `pyqgis-standalone`: VSCode から PyQGIS スクリプトを実行
- `learning-log`: セッション終了時の学習記録

## MCP
- `context7`: geopandas / folium / shapely などの最新ドキュメント参照(`.mcp.json`)
