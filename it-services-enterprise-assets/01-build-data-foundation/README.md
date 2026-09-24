_(English version follows below)_

# Lab 01 - データ基盤をつくる

**所要時間** : 約 20 分  
**ゴール** : `lh_sier_asset_silver` に、オントロジーがバインドできる `sv_*` テーブルが 17 本そろっている状態  

---

## 0. このラボで作るもの

| アイテム | 名前 | 役割 |
|---|---|---|
| ワークスペース | `(任意のワークスペース名)` | ワークショップで作成する各種アイテムを配置 |
| レイクハウス | `lh_its_asset_bronze` | CSV を取り込み、Delta テーブルとして保持するブロンズ層のレイクハウス |
| レイクハウス | `lh_its_asset_silver` | オントロジーのバインド先として、ブロンズ層のレイクハウスに存在するデータから加工されたものを保持するシルバー層のレイクハウス |
| ノートブック | `nb_01_ingest_bronze` | 取り込んだ CSV ファイルを Delta テーブル化するノートブック。 |
| ノートブック | `nb_02_build_silver` | ブロンズ層の Delta テーブルの内容を取得し、シルバー層の Delta テーブルへ整形するノートブック。 |

## 1. ワークスペースを作成する

Fabric で新しいワークスペースを作成します。ワークスペースはわかりやすいものにしてください。(例: `ws_its_asset_Ontology_demo` など)

## 2. レイクハウスを作成する

ブロンズ層とシルバー層のレイクハウスを作成します。

1. 作成済みのワークスペース画面を開きます。  
2. `+ 新しい項目` から、**レイクハウス** を選択します。
3. 名前に `lh_its_asset_bronze` を入力します。  
  場所は 1. で作成したワークスペース名が選択されていることを確認します。  
  レイクハウス スキーマのチェックボックスは **オン** にします。
4. **作成** を選択して、レイクハウスを作成します。  
5. 同じ手順で、`lh_its_asset_silver` も作成します。  

## 3. サンプルデータをダウンロードする

本リポジトリの `data` ディレクトリ以下に格納されているサンプルデータ群をローカルにダウンロードします。  

## 4. サンプルデータをアップロードする

ダウンロードした `data` フォルダ以下に格納されているサンプルデータをブロンズ層のレイクハウスにアップロードします。  

1. `lh_its_asset_bronze` を開きます。
2. エクスプローラーの **Files** を右クリックします。
3. **アップロード -> フォルダーのアップロード** を選択します。
4. ダウンロードした `data` フォルダ内にある `confluence` フォルダを選択し、**アップロード** を選択します。  
5. ファイルアップロードの確認ポップアップが表示される場合は、**アップロード** を選択します。  
6. **アップロード** を選択し、ファイル/フォルダをアップロードします。
7. 同じ手順で、残りの以下フォルダも Files ディレクトリ配下にアップロードします。  
  - d365_project_operations
  - dataverse
  - entra_id
  - salesforce
  - servicenow
  - sharepoint_online
  - successfactors

最終的に、Files 配下が以下の形になれば OK です。  

```
Files/
├── confluence/
│   └── knowledge/2026/08/05/knowledge_20260805.csv
├── d365_project_operations/
│   ├── assignment/2026/08/05/assignment_20260805.csv
│   └── availability/2026/08/05/availability_20260805.csv
├── dataverse/
│   ├── certification/2026/08/05/certification_20260805.csv
│   ├── skill/2026/08/05/skill_20260805.csv
│   └── technology/2026/08/05/technology_20260805.csv
├── entra_id/
│   └── person/2026/08/05/person_20260805.csv
├── salesforce/
│   ├── customer/2026/08/05/customer_20260805.csv
│   ├── opportunity/2026/08/05/opportunity_20260805.csv
│   └── stakeholder/2026/08/05/stakeholder_20260805.csv
├── servicenow/
│   ├── project/2026/08/05/project_20260805.csv
│   ├── project_required_skill/
│   │   └── 2026/08/05/project_required_skill_20260805.csv
│   └── project_technology/
│   │   └── 2026/08/05/project_technology_20260805.csv
├── sharepoint_online/
│   └── deliverable/2026/08/05/deliverable_20260805.csv
└── successfactors/
    ├── employee_profile/2026/08/05/employee_profile_20260805.csv
    ├── organization/2026/08/05/organization_20260805.csv
    ├── person_certification/
    │   └── 2026/08/05/person_certification_20260805.csv
    └── person_skill/2026/08/05/person_skill_20260805.csv
```

## 5. ブロンズ層の Delta テーブルを作成する

Microsoft Fabric ノートブックを使用して、アップロードした CSV ファイルをベースにブロンズ層の Delta テーブルを `lh_its_asset_bronze` 上に作成します。  

1. [nb_01_ingest_bronze.ipynb](./nb_01_ingest_bronze.ipynb) ファイルをダウンロードします。  
2. 作成済みのワークスペース画面を開きます。  
3. ワークスペース画面の上部にある **インポート -> ノートブック -> コンピューターから** を選択します。   
4. ダウンロードしたノートブックファイルを選択し、**アップロード** します。
5. `nb_01_ingest_bronze` を選択して開きます。
6. 画面左にあるエクスプローラーから、**データ項目の追加 -> OneLake カタログから** を選択します。  
7. `lh_its_asset_bronze` を選択し、**追加** を選択します。
8. 実行言語が **PySpark (Python)**、環境が **ワークスペースの既定値** になっていることを確認し、**すべて実行** を選択します。
9. `lh_its_asset_bronze` の Tables 配下に、`bz_` で始まる 18 個の Delta テーブルが作成されることを確認します。  

このノートブックは、CSV の列名から BOM と前後空白などの情報を除去したうえで、

- `ingest_date` 
- `_source_system`
- `_source_entity`
- `_ingested_at`

の列を追加します。
ingest_date 列は、ファイルパスの `<yyyy>/<MM>/<dd>` ディレクトリ情報を参照し、最新データの取り込み日時が入ります。この列は、後続で実施するシルバー層の Delta テーブル作成でも使用されます。  
アンダーバー (_) から始まる列は監査列です。 

## 6. シルバー層の Delta テーブルを作成する

ブロンズ層のレイクハウスに対して行った処理と同様に、Microsoft Fabric ノートブックを使用して、シルバー層の Delta テーブルを `lh_its_asset_silver` 上に作成します。  

1. [nb_02_build_silver.ipynb](./nb_02_build_silver.ipynb) ファイルをダウンロードします。  
2. 作成済みのワークスペース画面を開きます。  
3. ワークスペース画面の上部にある **インポート -> ノートブック -> コンピューターから** を選択します。   
4. ダウンロードしたノートブックファイルを選択し、**アップロード** します。
5. `nb_02_build_silver` を選択して開きます。
6. 画面左にあるエクスプローラーから、**データ項目の追加 -> OneLake カタログから** を選択します。  
7. `lh_its_asset_silver` を選択し、**追加** を選択します。
8. 同じ手順で、`lh_its_asset_bronze` も追加します。  
9. `lh_its_asset_silver` が既定のレイクハウスとして設定されていることを確認します。(名前の右側にピンが付いていれば既定になっています。)
  もし既定のレイクハウスになっていない場合は、右クリックから既定のレイクハウスに設定を行います。  
10. 実行言語が **PySpark (Python)**、環境が **ワークスペースの既定値** になっていることを確認し、**すべて実行** を選択します。
11. `lh_its_asset_silver` の Tables 配下に `sv_` で始まる 17 個の Delta テーブルが作成されることを確認します。  

## 7. まとめ

このラボでは、複数の業務システムから収集したサンプルデータをブロンズ層へ取り込み、オントロジーから利用しやすい形に整えたシルバー層を作成しました。

- `lh_its_asset_bronze` に、取り込んだデータを保持する `bz_*` テーブルを 18 個作成
- `lh_its_asset_silver` に、最新化・重複排除・キー生成などを行った `sv_*` テーブルを17個作成
- データの取り込み元や取り込み日時を追跡する監査列を付与
- データ品質と鮮度を `dq_silver_status` で確認できる状態を構築

ブロンズ層ではソースシステムごとのデータをそのまま保持し、シルバー層では、それらを「人」「組織」「プロジェクト」「顧客」「スキル」といった業務上の実体として扱えるように整えています。

これで、オントロジーのエンティティ型や関係型を実データへバインドするための準備が整いました。

次の [Lab 02 - オントロジーをつくり、つなぐ](../02-model-and-connect-ontology/README.md) では、今回作成した `lh_its_asset_silver` のテーブルを使用してオントロジーを作成し、分散していた業務データを意味と関係性でつないでいきます。

## トラブルシューティング

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 確認クエリが0件 | CSV のアップロード漏れ | ブロンズ層のテーブル数が18本あるか確認する |
| `SILVER_BUILD_STATUS = WARN` | 同一取り込み日の中で業務キーが重複している（ソース側の実データ異常） | `dq_*_duplicates` の中身を確認。二重登録ならソース側を修正、そうでなければ `nb_02` の `ENTITIES` の業務キーに列を足す |
| 日付が判定できない旨の警告 | フォルダー階層が崩れている | `Files/<system>/<entity>/<yyyy>/<MM>/<dd>/` の4階層になっているか確認する |
| 列が `_c0`, `_c1` になる | ヘッダーが読まれていない | `option("header", "true")` が有効か、CSV が空でないか確認する |
| 列名の先頭に見えない文字が付く | BOM が残っている | ノートブックの列名クレンジング処理が実行されているか確認する |
| `TABLE_OR_VIEW_NOT_FOUND` | テーブル参照のスキーマ指定漏れ、またはレイクハウス未アタッチ | Fabric のテーブル参照は3階層（`<レイクハウス>.<スキーマ>.<テーブル>`）。`nb_02` の `BRONZE_SCHEMA` を スキーマ有効なら `'dbo'`、スキーマ無効（レガシー）なら `None` に設定する。あわせて `nb_02` に bronze / silver の両方がアタッチされているか確認する |

---

# Lab 01 - Build the Data Foundation

**Estimated time**: Approximately 20 minutes  
**Goal**: Have all 17 `sv_*` tables available in `lh_sier_asset_silver` so that the ontology can bind to them  

---

## 0. What You Will Build in This Lab

| Item | Name | Purpose |
|---|---|---|
| Workspace | `(any workspace name)` | Hosts the various items created during the workshop |
| Lakehouse | `lh_its_asset_bronze` | Bronze-layer lakehouse that ingests CSV files and stores them as Delta tables |
| Lakehouse | `lh_its_asset_silver` | Silver-layer lakehouse that stores data transformed from the bronze-layer lakehouse for use as ontology binding targets |
| Notebook | `nb_01_ingest_bronze` | Notebook that converts the ingested CSV files into Delta tables. |
| Notebook | `nb_02_build_silver` | Notebook that retrieves the contents of the bronze-layer Delta tables and transforms them into silver-layer Delta tables. |

## 1. Create a Workspace

Create a new workspace in Fabric. Give the workspace an easy-to-understand name. (For example, `ws_its_asset_Ontology_demo`.)

## 2. Create the Lakehouses

Create the bronze-layer and silver-layer lakehouses.

1. Open the workspace you created.
2. From `+ New item`, select **Lakehouse**.
3. Enter `lh_its_asset_bronze` as the name.
  Confirm that the workspace created in step 1 is selected as the location.
  Turn **on** the Lakehouse schemas checkbox.
4. Select **Create** to create the lakehouse.
5. Follow the same steps to create `lh_its_asset_silver`.

## 3. Download the Sample Data

Download the sample data stored under the `data` directory in this repository to your local machine.

## 4. Upload the Sample Data

Upload the sample data stored under the downloaded `data` folder to the bronze-layer lakehouse.

1. Open `lh_its_asset_bronze`.
2. Right-click **Files** in the Explorer.
3. Select **Upload -> Upload folder**.
4. Select the `confluence` folder inside the downloaded `data` folder, and then select **Upload**.
5. If a file upload confirmation pop-up appears, select **Upload**.
6. Select **Upload** to upload the file/folder.
7. Follow the same steps to upload the remaining folders listed below under the Files directory.
  - d365_project_operations
  - dataverse
  - entra_id
  - salesforce
  - servicenow
  - sharepoint_online
  - successfactors

When complete, the structure under Files should look like this:

```
Files/
├── confluence/
│   └── knowledge/2026/08/05/knowledge_20260805.csv
├── d365_project_operations/
│   ├── assignment/2026/08/05/assignment_20260805.csv
│   └── availability/2026/08/05/availability_20260805.csv
├── dataverse/
│   ├── certification/2026/08/05/certification_20260805.csv
│   ├── skill/2026/08/05/skill_20260805.csv
│   └── technology/2026/08/05/technology_20260805.csv
├── entra_id/
│   └── person/2026/08/05/person_20260805.csv
├── salesforce/
│   ├── customer/2026/08/05/customer_20260805.csv
│   ├── opportunity/2026/08/05/opportunity_20260805.csv
│   └── stakeholder/2026/08/05/stakeholder_20260805.csv
├── servicenow/
│   ├── project/2026/08/05/project_20260805.csv
│   ├── project_required_skill/
│   │   └── 2026/08/05/project_required_skill_20260805.csv
│   └── project_technology/
│   │   └── 2026/08/05/project_technology_20260805.csv
├── sharepoint_online/
│   └── deliverable/2026/08/05/deliverable_20260805.csv
└── successfactors/
    ├── employee_profile/2026/08/05/employee_profile_20260805.csv
    ├── organization/2026/08/05/organization_20260805.csv
    ├── person_certification/
    │   └── 2026/08/05/person_certification_20260805.csv
    └── person_skill/2026/08/05/person_skill_20260805.csv
```

## 5. Create the Bronze-Layer Delta Tables

Using a Microsoft Fabric notebook, create bronze-layer Delta tables in `lh_its_asset_bronze` based on the uploaded CSV files.

1. Download the [nb_01_ingest_bronze.ipynb](./nb_01_ingest_bronze.ipynb) file.
2. Open the workspace you created.
3. At the top of the workspace screen, select **Import -> Notebook -> From this computer**.
4. Select the downloaded notebook file and **Upload** it.
5. Select and open `nb_01_ingest_bronze`.
6. From the Explorer on the left side of the screen, select **Add data items -> From OneLake catalog**.
7. Select `lh_its_asset_bronze`, and then select **Add**.
8. Confirm that the runtime language is **PySpark (Python)** and the environment is **Workspace default**, and then select **Run all**.
9. Confirm that 18 Delta tables whose names begin with `bz_` have been created under Tables in `lh_its_asset_bronze`.

This notebook removes information such as the BOM and leading or trailing whitespace from the CSV column names, and then adds the following columns:

- `ingest_date`
- `_source_system`
- `_source_entity`
- `_ingested_at`

The ingest_date column refers to the `<yyyy>/<MM>/<dd>` directory information in the file path and contains the ingestion date of the latest data. This column is also used later when creating the silver-layer Delta tables.
Columns whose names begin with an underscore (_) are audit columns.

## 6. Create the Silver-Layer Delta Tables

As with the processing performed for the bronze-layer lakehouse, use a Microsoft Fabric notebook to create the silver-layer Delta tables in `lh_its_asset_silver`.

1. Download the [nb_02_build_silver.ipynb](./nb_02_build_silver.ipynb) file.
2. Open the workspace you created.
3. At the top of the workspace screen, select **Import -> Notebook -> From this computer**.
4. Select the downloaded notebook file and **Upload** it.
5. Select and open `nb_02_build_silver`.
6. From the Explorer on the left side of the screen, select **Add data items -> From OneLake catalog**.
7. Select `lh_its_asset_silver`, and then select **Add**.
8. Follow the same steps to add `lh_its_asset_bronze`.
9. Confirm that `lh_its_asset_silver` is set as the default lakehouse. (It is the default if a pin appears to the right of its name.)
  If it is not the default lakehouse, right-click it and set it as the default lakehouse.
10. Confirm that the runtime language is **PySpark (Python)** and the environment is **Workspace default**, and then select **Run all**.
11. Confirm that 17 Delta tables whose names begin with `sv_` have been created under Tables in `lh_its_asset_silver`.

## 7. Summary

In this lab, you ingested sample data collected from multiple business systems into the bronze layer and created a silver layer shaped for convenient use by the ontology.

- Created 18 `bz_*` tables in `lh_its_asset_bronze` to store the ingested data
- Created 17 `sv_*` tables in `lh_its_asset_silver` with processing such as selecting the latest data, removing duplicates, and generating keys
- Added audit columns to track the source and ingestion time of the data
- Established a state in which data quality and freshness can be checked using `dq_silver_status`

The bronze layer retains the data from each source system as-is, while the silver layer prepares it so that it can be handled as business entities such as "people," "organizations," "projects," "customers," and "skills."

You are now ready to bind ontology entity types and relationship types to actual data.

In the next lab, [Lab 02 - Build and Connect the Ontology](../02-model-and-connect-ontology/README.md), you will use the `lh_its_asset_silver` tables created in this lab to build an ontology and connect the previously distributed business data through meaning and relationships.

## Troubleshooting

| Symptom | Common Cause | Resolution |
|---|---|---|
| Validation query returns 0 rows | Some CSV files were not uploaded | Confirm that there are 18 tables in the bronze layer |
| `SILVER_BUILD_STATUS = WARN` | Business keys are duplicated within the same ingestion date (an actual source-data error) | Check the contents of `dq_*_duplicates`. If records were registered twice, correct the source; otherwise, add a column to the business key in `ENTITIES` in `nb_02` |
| Warning that the date cannot be determined | The folder hierarchy is incorrect | Confirm that it has the four-level structure `Files/<system>/<entity>/<yyyy>/<MM>/<dd>/` |
| Columns are named `_c0`, `_c1` | The header was not read | Confirm that `option("header", "true")` is enabled and that the CSV is not empty |
| An invisible character appears at the beginning of a column name | The BOM remains | Confirm that the notebook's column-name cleansing process was executed |
| `TABLE_OR_VIEW_NOT_FOUND` | The schema was omitted from the table reference, or the lakehouse is not attached | Fabric table references have three levels (`<lakehouse>.<schema>.<table>`). Set `BRONZE_SCHEMA` in `nb_02` to `'dbo'` when schemas are enabled, or to `None` when schemas are disabled (legacy). Also confirm that both bronze and silver are attached to `nb_02` |
