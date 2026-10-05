_(English version follows below)_

# Lab 02 - オントロジーをつくり、つなぐ

**所要時間** : 約 50 分  
**ゴール** : `ont_its_asset` に、エンティティ型17個・関係型19個が定義され、`sv_*` テーブルにバインドされている状態  

---

## 0. このラボで作るもの

| アイテム | 名前 | 役割 |
|---|---|---|
| オントロジー | `ont_its_asset` | ワークショップで作成する各種アイテムを配置 |
| ノートブック | `nb_03_build_ontology` | シルバー層の Delta テーブルを元に、新 TMDL 形式のオントロジー定義を設定・検証する。 |

このラボを進めるにあたっては、[事前準備に記載の設定](../README.md#事前準備) を行う必要があります。確認を行ってから実施するようにしてください。  

## 1. オントロジーを作成する

Microsoft Fabric では、オントロジーを扱う場合、ワークスペース上で専用の項目 (アイテム) を作成する必要があります。

1. 作成済みのワークスペース画面を開きます。  
2. `+ 新しい項目` から、**Ontology** を選択します。
3. 名前に `ont_its_asset` を入力します。  
  場所は 1. で作成したワークスペース名が選択されていることを確認します。  
4. **作成** を選択して、オントロジーを作成します。 

新エクスペリエンスのオントロジーは、定義とデータソースへのバインドを保持する意味付けの層です。グラフ実行はオプションであり、データが自動的にすべてグラフへコピーされるわけではありません。このラボでは、ノートブックで全定義を作成した後、[7章のWeb画面の手順](#enable-graph-ja)でグラフをマテリアライズ（検索用のグラフにデータを読み込むこと）します。サービスが管理する関連アイテムは、手動で削除しないでください。

参考: [Ontology overview — Optional graph execution](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview#optional-graph-execution)

Microsoft Fabric のオントロジーでは、大きく分けて以下の 4 つを定義することになります。

- エンティティ (Entity)
- プロパティ (Property)
- リレーションシップ (Relationship)
- メタデータ (Metadata)

これらを定義することにより、データの関係性の定義や関係性をたどって得られる情報をコンテキストとして AI エージェントに提供することが可能になります。以降の章では、これら 4 つ（エンティティ・プロパティ・リレーションシップ・メタデータ）を実際に定義していきます。  

## 2. GUI でエンティティを作成する

エンティティを Fabric のブラウザ画面から作成していきます。まずは、`Person` という、自社で採用しているエンジニアやコンサルタントの従業員情報をオントロジーのエンティティとして定義します。  

1. ホーム画面にて、**エンティティ型の追加** を選択します。  
2. エンティティ型名に `Person` を入力し、**エンティティ型の追加** を選択します。
3. エクスプローラーに `Person` が表示されることを確認します。
4. 同じ手順で、以下 4 つのエンティティも作成します。
  - Project
  - Customer
  - Organization
  - Assignment

エンティティ名の入力については、十分注意をお願いします。(誤りがある場合、後続の手順で影響が出てしまいます)

## 3. プロパティを作成し、データバインドを行う

実際に作成したエンティティを確認すると一目瞭然だと思いますが、エンティティを作っても、中身は空っぽのままになっています。オントロジーとして AI エージェントにコンテキストとして情報を与えるためには、プロパティの作成と実データとのバインドが必要です。  
そのため、前段で作成した 5 つのエンティティに対し、シルバー層のレイクハウスに作成した Delta テーブルに紐づける形でプロパティの設定を行います。  

1. ホーム画面にて、エクスプローラー上に表示されている `Person` を選択し、**エンティティ型の詳細を表示** を選択します。
2. **構成** タブを選択します。
3. `プロパティ` 欄にある **プロパティ バインドの管理 -> バインドとプロパティの追加** を選択します。
4. `バインディングの選択` 欄にて、ソース欄にある **追加** を選択します。
5. `lh_its_asset_silver` が表示されることを確認し、展開します。
  dbo -> `sv_person` を選択し、**テーブルの選択** を選択します。
6. **エンティティ型のプロパティ** を選択します。
7. プロパティ欄に表示されている `_valid_as_of` の行の横に表示されているごみ箱アイコンを選択し、**_valid_as_of 行を削除** します。
8. **作成する** を選択し、設定が保存されることを確認します。
9. **キャンセル** を選択し、Person エンティティのプロパティ画面に遷移します。  
  設定したデータバインドの内容が画面に表示されることを確認します。
10. **インスタンス** タブを選択し、Person エンティティにバインドした実データが表示されることを確認します。
  確認出来たら、画面左上にあるパンくずリストの **ホーム** を選択し、ホーム画面に戻ります。
11. 同じ手順で、残りのエンティティに対しても同じようにプロパティの設定と実データへのバインドを行います。

  | エンティティ | データバインド先 | プロパティの追加/削除 |
  | :-- | :-- | :-- |
  | `Project` | `sv_project` | (削除) `_valid_as_of` |
  | `Customer` | `sv_customer` | (削除) `_valid_as_of` |
  | `Organization` | `sv_organization` | (削除) `_valid_as_of` |
  | `Assignment` | `sv_assignment` | **(削除) `AssignmentId`**<br/>(削除) `_valid_as_of` |

### (解説) 一部のプロパティを削除した理由

`_valid_as_of` や `AssignmentId` について、プロパティ情報から削除を行いました。これがなぜかについて解説します。  

理由はズバリ、オントロジーは Delta テーブルの写しではないからです。オントロジーは **業務語彙** の層を形成するものであり、バインドされたプロパティは AI エージェントが使用するコンテキスト (スキーマ) として使用されます。そのため、例えば `_valid_as_of` をプロパティで残してしまうと、レイクハウス上の Delta テーブルの情報鮮度を人間が目視で確認するために追加した情報が、AI エージェントにとって推論を行う際のノイズ (コンテキスト汚染やハルシネーションリスク) につながるものとなってしまいます。  

AI エージェントは、エンティティのプロパティから意味を推論する動きを採ります。そのため、AI エージェントがオントロジーをコンテキストとして使用する際、推論家庭に使用されるとノイズとなるような内容が含まれないよう、オントロジーの設計を行うことが重要になります。  

## 4. リレーションを作成し、データマッピングを行う

ここまでの内容で、オントロジーのエンティティとプロパティを作成し、実データに対して組織内の業務語彙を結びつけることができました。ですが実際の画面を見てわかる通り、作成したエンティティはどれも孤立して存在しており、エンティティ同士の関係性が表現できていません。  
ここでは、業務語彙をベースに、実データを使用したマッピングする形で、作成したエンティティ同士の関係情報を付与する設定を行います。  

1. ホーム画面にて、**リレーションシップの追加** を選択します。  
2. `リレーションシップの種類の名前` に **belongsTo** を入力します。
3. `元のエンティティ型` のプルダウンリストより、**Person** を選択します。
4. `ターゲット エンティティ型` のプルダウンリストより、**Organization** を選択します。
5. **作成** を選択します。
6. `belongsTo` のリレーションが作成され、Person エンティティと Organization エンティティがつながっていることを確認します。
7. Person から Organization に対して表示されている矢印上の **belongsTo** を選択します。
8. **マッピング テーブルを使用する** をオフにし、`元のエンティティ型` のプロパティにて、**OrganizationId** を選択します。
9. `ターゲット エンティティ型` のプロパティにて、**OrganizationId** を選択します。
10. **保存** を選択します。
11. `リレーションシップの種類が正常に更新されました` のバナー表示を確認の上、**ホーム** を選択します。
12. 同じ手順で、2 つのリレーション設定を行います。
  設定の詳細については、以下の表を参照してください。

  | リレーションシップ名 | 元のエンティティ型 | ターゲット エンティティ型 | 元のエンティティ型の列 | ターゲット エンティティ型の列 |
  | :-- | :-- | :-- | :-- | :-- |
  | `performedBy` | `Assignment` | `Person` | `PersonId` | `PersonId` |
  | `deliveredFor` | `Project` | `Customer` | `CustomerId` | `CustomerId` |

これで、エンティティ同士の関係 (リレーション) を辿るための結合列の設定が完了しました。

オントロジーにおけるリレーションの設定での重要ポイントは、以下の 2 点です。

- リレーションシップ名は、AI エージェントがエンティティ同士の関係性を適切に理解できる一意な名称にすること
- エンティティ同士の関係を設定する際、ターゲットエンティティ側のプロパティにはキープロパティとなるものを設定すること

**エンティティ自身のキーと、リレーションの結合列は別です。** 新モデルの直接リレーションでは、外部キーに相当する起点側の列と、終点側のキー列を結びます。例えば `performedBy` は `Assignment.PersonId` と `Person.PersonId` で結合しますが、Assignment 自身を識別するキーは `AssignmentKey` のままです。`AssignmentKey` と `PersonId` は値が異なるため、直接結合には使いません。

REST API では、この結合をテーブル間の TOM `relationship` として定義し、業務上の関係を表す `entityRelationship` の `backingConfiguration.relationship` から参照します。専用のマッピングテーブルを使う形式もありますが、本ワークショップでは使用しません。

これらは、終点側のキーが一意であることを前提とした **N対1** の結合です。起点側の外部キーは重複していて構いません。例えば、複数の Assignment が同じ Person に結び付いても、それぞれの Assignment は `AssignmentKey` で区別されます。終点側のキーの一意性はデータ側で維持してください（`nb_03` はデータ行の一意性検査は行いません）。

## 5. メタデータを設定する

ここまでで、エンティティとプロパティ、エンティティ同士を結ぶリレーションを作成しました。しかし、今の状態では組織固有のビジネス概念やルール、用語の意味といった知識までは表現されていません。   
従来の GraphRAG が生成する Knowledge Graph は、文書などから抽出されたエンティティやリレーションを中心に構成されており、それぞれの概念が組織内でどのような意味を持つのかを事前に定義しているわけではありませんでした。そのため、同じグラフ上の構造や関係性は表現できるものの、それらに対する組織共通の意味や制約が保証されているわけではありませんでした。  
オントロジーでは、エンティティやリレーションの構造に加え、「Person/Customer とは何か」「Assignment とは何か」「どのような関係のみを許可するのか」といったビジネス上の意味や制約を明示的に定義します。これにより、グラフ上のデータを組織全体で一貫した意味で解釈できるようになり、AI エージェントが推論を行う際の基準を統一することができます。  

Microsoft Fabric のオントロジーでは、以下の種類のメタデータを設定することができます。  

| 種類 | 内容 | 付与できる対象 |
| :-- | :-- | :-- |
| 説明 (Descriptions) | その概念が何を表すか (1 ~ 3 文) | エンティティ<br/>プロパティ<br/>リレーション |
| 類義語 (Synonyms) | 別名や略称、業界用語など | 新 TMDL ではエンティティ・プロパティ・リレーションに対応。本ラボではエンティティに設定 |
| 追加のメタデータ (Additional metadata) | 単位や機密区分、業務オーナーなどの情報を Key-Value の形式で表現 (Data quality: Incomplete) | エンティティ<br/>プロパティ<br/>リレーション |

ここでは、`Person` エンティティと `belongsTo` リレーションシップに対して、メタデータの設定を行います。  

### Person エンティティのメタデータ設定

1. ホーム画面にて、エクスプローラー上に表示されている `Person` を選択し、**エンティティ型の詳細を表示** を選択します。
2. **構成** タブを選択します。
3. `エンティティ メタデータ` 欄にて、以下の情報を入力し、**更新** を選択します。
  類義語については、1 つずつ入力を行ってください。

| 項目 | 入力値 |
| :-- | :-- |
| 説明 | `An engineer or consultant employed by the company. The starting point for staffing searches and team formation.` |
| 類義語 | `employee` , `staff` , `member` , `engineer` |
| 追加のメタデータ | キー : `sensitivity`<br/>値 : `Contains personal data` |

### belongsTo リレーションシップのメタデータ設定

1. ホーム画面にて、エクスプローラ上に表示されている **belongsTo** を選択します。
2. `メタデータ` 欄にある **編集** を選択します。
3. 以下の情報を入力し、**更新** を選択します。
  追加のメタデータは設定不要です。
4. **保存** を選択します。  
5. `リレーションシップの種類が正常に更新されました` のバナー表示を確認します。

| 項目 | 入力値 |
| :-- | :-- |
| 説明 | `Links an engineer to the organization they belong to.` |
| 追加のメタデータ | - (設定なし) |

## 6. コードでオントロジーを設定する

ここまでオントロジーの各種設定（エンティティ・プロパティ・リレーションシップ・メタデータ）を画面上で行ってきました。正直、かなり大変だと思います。そして、実際に設定を行った方の中には、以下のような思いを抱く方もいらっしゃると思います。

- オントロジーは全部、手動で設定するしかできないの？
- コマンドや REST API などを使用してオントロジーを設定できないの？
- CI/CD や IaC など、バージョン管理や設定値管理はどうしたらいいの？

Microsoft Fabric の [Ontology REST API](https://learn.microsoft.com/en-us/rest/api/fabric/ontology/items) を使用すると、オントロジー定義をコードで管理できます。

ここでは、残りのオントロジー設定を REST API 経由で行います。  

> **対応形式**: 新エクスペリエンスの TMDL（互換性レベル `1000000`）専用です。旧 JSON 形式の自動変換は行いません。互換性レベルの根拠は [公式仕様の database.tmdl](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/ontology-definition#databasetmdl-database-file) を参照してください。

1. [nb_03_build_ontology.ipynb](./nb_03_build_ontology.ipynb) ファイルをダウンロードします。  
2. 作成済みのワークスペース画面を開きます。  
3. ワークスペース画面の上部にある **インポート -> ノートブック -> コンピューターから** を選択します。   
4. ダウンロードしたノートブックファイルを選択し、**アップロード** します。
5. 画面左にあるエクスプローラーから、**データ項目の追加 -> OneLake カタログから** を選択します。  
6. `lh_its_asset_silver` を選択し、**追加** を選択して、既定のレイクハウスに設定します。
7. 実行言語が **PySpark (Python)**、環境が **ワークスペースの既定値** になっていることを確認し、**すべて実行** を選択します。

ノートブックの実行が完了したら、`ont_its_asset` のオントロジー画面に戻り、手動で設定したもの以外にエンティティやリレーションが追加されていることを確認してください。ノートブックはグラフモデルへの直接書き込みやデータ取り込みを行いません。続けて[7章](#enable-graph-ja)を実施してください。

### 設定

| 設定 | 既定値・用途 |
|---|---|
| `ONTOLOGY_NAME` | `ont_its_asset` |
| `SILVER_LAKEHOUSE_NAME` | `lh_its_asset_silver`（既定のレイクハウスに設定する） |
| `SILVER_SCHEMA` | SQL 側のスキーマ名。通常 `dbo`。スキーマ無効レイクハウスでも `None` は指定しない |
| `APPLY_METADATA` | `True`：説明・類義語・annotation を設定する |
| `OVERWRITE_EXISTING_METADATA` | `False`：既存の値を保持し、未設定項目だけを補完する |

### ノートブックの処理

| 手順 | 内容 |
|---|---|
| 1. アイテムの解決 | Ontology、Workspace、Silver Lakehouse の識別情報・OneLake URL・SQL 分析エンドポイントを取得 |
| 2. 読み込み | 現在の TMDL と Silver の実スキーマを取得 |
| 3. 構築 | バインドを検証し、不足する型・関係・プロパティ・キー・メタデータを補完。変更パートを表示 |
| 4. 更新 | 同時編集がないことを再確認し、変更があれば全パートを一度に送信 |
| 5. 読み戻し | 必要な17エンティティ型・19関係型とバインド・メタデータを確認 |

既存パート・接続式・lineageTag・手動設定を保持します。既存のキー・型・結合列が想定と異なる場合は、上書きせず停止します。UI で作成した5型・3関係に不足分を追加するほか、空の新形式のアイテムから全定義を作ることもできます。

新規テーブルは DirectLake partition を持ち、`let Source = AzureStorage.DataLake("https://<OneLake ホスト>/<Workspace ID>/<Lakehouse ID>", [HierarchicalNavigation=true]) in Source` 形式の共有 M 式で Silver を参照します。一致する UI 作成の接続式を再利用し、なければ `nb03_SilverLakehouse` を追加します。partition には、UI の保存済み定義で確認した `ONT_WorkspaceId`・`ONT_ItemId`・`ONT_ItemKind = Lakehouse`・名前・SQL 接続情報・追加時刻を設定します。`ONT_ItemId` は Lakehouse の ID、`ONT_SqlDatabase` は Lakehouse 名で、SQL エンドポイント ID とは区別します。これらの接続情報は `APPLY_METADATA` に関係なく検証・補完します。

プロパティの型は Silver の実スキーマから決定します。説明は `///`、類義語は `synonym`、追加メタデータは `annotation` として出力します。

関係は、起点の外部キー列と終点のキー列を結ぶ TOM `relationship` と、それを参照する `entityRelationship` で表現します。新規 TOM relationship は `isActive: false` とし、複数経路の自動フィルター伝播を避けます。

### 完了と再実行

最終セルで次の表示を確認します。

```text
Verified required entity types: 17 / 17
Verified required relationship types: 19 / 19
Verified Lakehouse source bindings: 17 / 17
```

変更がなければ `No definition changes are needed.` と表示し、更新を送信しません。実行中の UI 編集は避けてください。同時編集検出はロックではありません。

ノートブックを差し替えた場合は、**すべて実行**して古い変数・関数を残さないでください。UI の定義だけを修正した場合は、手順2からやり直します。タイムアウトなどで更新結果が不明な場合は、API の操作状態を確認してから再実行してください。

旧 nb_03 で作成済みの場合も再実行で補修できます。`nb03_SilverSource` を参照し、SQL 接続先と生成時の table lineageTag が一致するテーブルだけを OneLake 接続へ切り替え、欠けた Lakehouse 識別情報を追加します。UI の正常なバインドと既存の時刻は保持し、別 Lakehouse の識別情報や変更済みの接続式は上書きせず停止します。旧共有式は他テーブルが参照している可能性があるため削除しません。読み戻しでは17テーブルすべての接続式と識別情報を検証しますが、UI 表示や MCP の実データ検索成功までは保証しません。

<a id="enable-graph-ja"></a>

## 7. グラフの探索を有効化する（Web画面）

本ワークショップでは、Lab 03 の MCP による実データ検索の準備としてグラフを有効化します。

1. オントロジー画面で、画面の上部にある **グラフの探索** を選択します。
2. 表示内容を確認し、グラフモデルを有効化します。

## 8. まとめ

このラボでは、作成したシルバー層のレイクハウスに存在するデータを使用して、オントロジーの作成と実データへのバインド設定を行いました。  

オントロジー定義の更新とグラフの検索準備完了は別です。7章のWeb操作でグラフへの取り込みを完了し、実際の値と関係を検索できることを確認してから MCP に進みます。

次の [Lab 03 - エージェントに問いかける](../03-ask-the-agent/README.md) では、今回作成した `ont_its_asset` オントロジーを使用して、MCP クライアントとの連携をためします。

## トラブルシューティング

### オントロジーの作成・アイテムの解決

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| オントロジーアイテムを作成できない | テナントで Ontology（プレビュー）が未有効、または容量が F2 未満 | テナント設定で Ontology アイテムを有効にし、F2 以上の容量にワークスペースを割り当てる |
| `Ontology 'ont_its_asset' が見つかりません` | 名前の綴り違い、または別ワークスペースにある | 2-1 で作成した名前とワークスペースが、ノートブックの `ONTOLOGY_NAME` と一致しているか確認する |
| オントロジー名を保存できない | 名前にスペースやハイフンが含まれている | 数字・英字・アンダースコアのみにする |
| 認証エラー | 実行ユーザーにオントロジーの書き込み権限がない | ワークスペースのロールを確認する（共同作成者以上） |

### エンティティ型とデータバインド

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| グラフにエンティティ型が出てこない | バインドが未保存、投影対象が未選択、または取り込みが未完了 | バインドを確認し、[7章](#enable-graph-ja)の Manage graph で適格性・選択対象・取り込み完了を確認する |
| キーを利用する処理が失敗する | 本ラボで想定するキーが未設定 | `nb_03` はキーがない場合に補完する。新モデル自体はキーなしのエンティティもサポートする |
| インスタンスを参照できない | データソースの権限・可用性・バインドが未確認 | Silver と SQL 分析エンドポイントの状態、接続権限を確認する。グラフの自動再構築を前提にしない |
| 同上 | シルバー層のテーブルが空 | Step 1 の `nb_02` が正常終了しているか確認する |
| `CorruptedPayload` / 定義の検証エラー | TMDL と既存のバインドが不整合 | エラーで指定された型・プロパティ・結合列を確認する。`SILVER_SCHEMA` は SQL 側のスキーマ名（通常 `dbo`） |
| `Project` / `Assignment` の件数が異様に多い | 時系列バインドを追加してしまった | 本キットは全エンティティ型が静的バインドのみ（2-2 参照）。時系列バインドを削除する |
| エージェントが取り込み日で「最近入社した人」を答える | `_valid_as_of` をバインドしたまま | プロパティ一覧から削除する（2-2 / `docs/property-binding.md`） |

### 関係型

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 設定は成功したのに関係が空 | 結合列または値が一致していない | 4章の表どおりに起点の外部キー列と終点のキー列を結ぶ |
| `existing TOM relationship columns differ` | 保存済みの結合列が想定と異なる | 4章の表どおりに修正して保存し、手順2から再実行する。`performedBy` は両側 `PersonId` |
| `keyProperty must be ...` | 想定と異なるキーが設定されている | 下記「キーと既存バインドの検証」を参照 |
| 線は見えているのに結果が出ない | 関係のデータバインドがずれている | シルバー層のテーブル名や列名を変更していないか確認する |

### メタデータ

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 手動設定したメタデータが消えた | `OVERWRITE_EXISTING_METADATA` が `True` | `False` に戻す（既定は `False`） |
| 追加メタデータでエラーになる | 同じ対象の中でキーが重複している | エンティティ型・プロパティ・関係型それぞれの単位でキーを一意にする |
| 類義語が追加されない | 手動設定済みの類義語を保持している | 既定では既存の類義語一覧を保持する。意図的に置き換える場合のみ `OVERWRITE_EXISTING_METADATA = True` |

### ノートブック（`nb_03`）

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 数が 17 / 19 にならない | 手動作成した名前の綴り違い | エンティティ型 `Person` / `Project` / `Customer` / `Organization` / `Assignment`、関係型 `belongsTo` / `performedBy` / `deliveredFor`（大文字小文字も一致させる） |
| `Old JSON ontology detected` | 旧エクスペリエンスのアイテムを指定した | 新エクスペリエンスで作成したオントロジーを使用する。旧形式のパートを混在させない |
| `The ontology changed after it was read` | 実行中に別の編集が保存された | UI の編集を止め、定義の読み込みからやり直す |
| `Readback still needs changes` | サービス側で一部の定義・メタデータが保持されなかった | 読み戻した TMDL と表示されたパートを調査する。更新の自動再送はしない |
| MCP でスキーマは見えるがデータ検索が失敗する | データのアクセス権やバインドが未確認 | 定義更新の完了だけで判断せず、Fabric 側でインスタンス・関係を参照できるか確認する |
| 実行が途中で失敗した | 検証エラー・通信エラーなど | 原因を修正し、上記「完了と再実行」に従う。更新結果が不明な場合は状態確認を先に行う |

### キーと既存バインドの検証

新モデルでは `keyProperty` で単一のプロパティをエンティティのキーとして指定します。`nb_03` は未設定の場合だけ `ENTITY_TYPES` に従って補います。例えば Assignment は `AssignmentKey` です。

backing table に列が存在し、entity 側に対応する `property` 宣言がない場合は、実スキーマと列の型を検証してから不足する宣言を補います。既存のテーブル名・列名・接続・メタデータは保持します。

既存のキーが異なる、必要な backing column がない、実スキーマと型が違う、結合列が異なる場合は書き込み前に停止します。表示された対象を UI で修正してから再実行してください。複合キー・継承・時系列などを追加した独自モデルへの自動変換は行いません。

### ローカルでの検証

Fabric に接続せず、TMDL 生成、既存定義の保持、再実行、REST エラー処理を確認できます。

```powershell
python -m unittest discover -s .\02-model-and-connect-ontology\tests -v
```

---

# Lab 02 - Build and Connect an Ontology

**Estimated time**: Approximately 50 minutes  
**Goal**: Define 17 entity types and 19 relationship types in `ont_its_asset`, with bindings to the `sv_*` tables  

---

## 0. What You Will Build in This Lab

| Item | Name | Purpose |
|---|---|---|
| Ontology | `ont_its_asset` | Contains the various items created in this workshop |
| Notebook | `nb_03_build_ontology` | Configures and validates a new-format TMDL ontology definition from the Silver-layer Delta tables. |

Before proceeding with this lab, you must complete the [settings described in Prerequisites](../README.md#prerequisites). Make sure you have verified them before continuing.

## 1. Create an Ontology

In Microsoft Fabric, working with an ontology requires creating a dedicated item in a workspace.

1. Open the workspace you created.
2. From `+ New item`, select **Ontology**.
3. Enter `ont_its_asset` as the name.
  Confirm that the workspace name created in step 1 is selected as the location.
4. Select **Create** to create the ontology.

The new-experience ontology stores definitions and bindings to data sources. Graph execution is optional; data is not automatically copied into a graph. In this lab, first build all definitions with the notebook, then follow the [web steps in section 7](#enable-graph-en) to materialize the graph (load data into a graph for querying). Do not manually delete service-managed related items.

Reference: [Ontology overview — Optional graph execution](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview#optional-graph-execution)

Microsoft Fabric ontologies broadly consist of the following four types of definitions:

- Entity
- Property
- Relationship
- Metadata

Defining these makes it possible to provide an AI agent with context consisting of information obtained by defining and traversing data relationships. In the following sections, you will define each of these four elements: entities, properties, relationships, and metadata.

## 2. Create Entities in the GUI

You will create entities from the Fabric browser interface. First, define `Person` as an ontology entity representing employee information for engineers and consultants employed by your company.

1. On the home screen, select **Add entity type**.
2. Enter `Person` as the entity type name, then select **Add entity type**.
3. Confirm that `Person` appears in Explorer.
4. Follow the same steps to create the following four entities:
  - Project
  - Customer
  - Organization
  - Assignment

Please take great care when entering entity names. (Any errors will affect subsequent steps.)

## 3. Create Properties and Bind Data

As you can clearly see by inspecting the entities you created, creating an entity alone leaves it empty. To provide information as context to an AI agent through the ontology, you must create properties and bind them to actual data.
Therefore, configure properties for the five entities created in the previous section by linking them to the Delta tables created in the Silver-layer Lakehouse.

1. On the home screen, select `Person` in Explorer, then select **View entity type details**.
2. Select the **Configuration** tab.
3. In the `Properties` section, select **Manage property bindings -> Add binding and properties**.
4. In the `Select binding` section, select **Add** next to the source.
5. Expand `lh_its_asset_silver`, select dbo -> `sv_person`, then select **Select table**.
6. Select **Entity type properties**.
7. Select the trash icon next to `_valid_as_of` and **remove that row**.
8. Select **Create** and confirm that the settings are saved.
9. Select **Cancel** to return to Person's properties and confirm that the data binding is displayed.
10. On the **Instances** tab, confirm that the bound data is visible, then return to **Home**.
11. Repeat for the remaining entities.

  | Entity | Data binding target | Add/remove properties |
  | :-- | :-- | :-- |
  | `Project` | `sv_project` | (Remove) `_valid_as_of` |
  | `Customer` | `sv_customer` | (Remove) `_valid_as_of` |
  | `Organization` | `sv_organization` | (Remove) `_valid_as_of` |
  | `Assignment` | `sv_assignment` | **(Remove) `AssignmentId`**<br/>(Remove) `_valid_as_of` |

### (Explanation 1) The Assignment Entity

Only the Assignment entity differs from the others in that its entity type key does not use an Id column. This section explains why.

`Assignment` is an associative entity representing who is assigned to which project between an employee and a project. In this lab, the combination of `PersonId` and `ProjectId` is treated as the business key that identifies one assignment.

The `AssignmentId` in the source data is an ID assigned by the source system. Its value may change when data is re-registered or re-exported, so if it were used as the ontology's entity type key, the same business assignment could be recognized as a different entity. Conversely, if the same ID were reused in a different environment or at a different point in time, different assignments could be treated as the same entity.

Therefore, `nb_02_build_silver`, which creates the Silver layer, deterministically generates `AssignmentKey` from the business key as follows.

```text
AssignmentKey = SHA-256(PersonId + "|" + ProjectId)
```

Because the same employee and project combination always generates the same value, the ontology can identify it as the same Assignment entity even if the source-side `AssignmentId` changes. In addition, the `performedBy` relationship identifies the Assignment using this `AssignmentKey` and connects it to the responsible Person entity using `PersonId`.

This design is based on the principle that the **entity type key should represent the conditions under which something continues to be the same business entity**, rather than simply using a column whose name contains ID. Note that this lab assumes that "an employee has one current assignment to a project." If you need to retain and distinguish multiple assignments between the same employee and project, you must add attributes such as role or validity period to the business key.

### (Explanation 2) Time-Series Data

In a Fabric ontology, regular properties and time-series properties are defined separately. **Only the column representing the observation time of a time-series property should be specified as the timestamp column.** The supported source column types are `datetime`, `date`, and `timestamp`, but this does not mean every column of a supported type should be used as the timestamp column.

For example, `Project.StartDate` / `Project.EndDate` and `Assignment.StartDate` / `Assignment.EndDate` are regular properties representing the duration of a project or assignment, not observation times for time-series properties.

In the new TMDL format, these ordinary date properties use `dataType: dateTime`. Time-series properties instead use a `TimeSeries<T>` type and a time-series backing configuration. `nb_03` reads the actual Silver schema to determine each ordinary property's type.

The five entities created through the GUI steps in this lab bind only static data, so you will not add time-series properties or time-series bindings.

> Reference (Microsoft Learn)
> - [Bind data to your ontology](https://learn.microsoft.com/ja-jp/fabric/iq/ontology/how-to-bind-data)
> - [Tutorial part 2: Enrich your ontology with more data](https://learn.microsoft.com/ja-jp/fabric/iq/ontology/tutorial-2-enrich-ontology)

### (Explanation 3) Why Some Properties Were Removed

You removed `_valid_as_of` and `AssignmentId` from the property information. This section explains why.

The simple reason is that an ontology is not a copy of a Delta table. An ontology forms a layer of **business vocabulary**, and its bound properties are used as the context (schema) consumed by AI agents. For example, if `_valid_as_of` were retained as a property, information added so that humans can visually check the freshness of a Delta table in the Lakehouse would become noise for the AI agent's reasoning, potentially causing context pollution and increasing hallucination risk.

AI agents infer meaning from entity properties. Therefore, when designing an ontology that an AI agent will use as context, it is important to ensure that it does not include content that would become noise if used in the reasoning process.

## 4. Create Relationships and Map Data

Up to this point, you have created ontology entities and properties and linked your organization's business vocabulary to actual data. However, as you can see in the interface, each created entity exists in isolation, and the relationships between entities have not yet been represented.
In this section, you will configure relationship information between the created entities by mapping actual data based on the business vocabulary.

1. On the home screen, select **Add relationship**.
2. Enter **belongsTo** for `Relationship type name`.
3. From the `Source entity type` dropdown, select **Person**.
4. From the `Target entity type` dropdown, select **Organization**.
5. Select **Create**.
6. Confirm that the `belongsTo` relationship has been created and that the Person and Organization entities are connected.
7. Select **belongsTo** on the arrow between Person and Organization.
8. Turn off **Use mapping table?**.
9. Under the origin entity's **Property**, select **OrganizationId**.
10. Under the target entity's **Property**, select **OrganizationId**.
11. Select **Save**.
12. After confirming that the `Relationship type updated successfully` banner appears, select **Home**.
13. Follow the same steps to configure two relationships.
  Refer to the following table for configuration details.

  | Relationship name | Source entity type | Target entity type | Source entity type column | Target entity type column |
  | :-- | :-- | :-- | :-- | :-- |
  | `performedBy` | `Assignment` | `Person` | `PersonId` | `PersonId` |
  | `deliveredFor` | `Project` | `Customer` | `CustomerId` | `CustomerId` |

You have now configured the join columns needed to traverse relationships between entities.
The following two points are important when configuring ontology relationships:

- Give each relationship a unique name that allows an AI agent to understand the relationship between entities correctly
- When defining a relationship between entities, set the target entity's property to one that matches its key property

**An entity's identity key and a relationship's join columns serve different purposes.** A direct relationship in the new model joins the origin's foreign-key column to the target's key column. For example, `performedBy` joins `Assignment.PersonId` to `Person.PersonId`, while Assignment's own identity key remains `AssignmentKey`. Do not directly join `AssignmentKey` to `PersonId`, because their values differ.

The REST definition represents this join as a TOM table `relationship`, referenced by the business-level `entityRelationship` through `backingConfiguration.relationship`. A separate mapping-table form is also available, but this workshop does not use it.

These are **many-to-one** joins, assuming that the target key is unique. The origin's foreign-key values may repeat. For example, multiple Assignments can reference the same Person while remaining distinct through `AssignmentKey`. Maintain target-key uniqueness in the data; `nb_03` does not scan data rows to check uniqueness.

## 5. Configure Metadata

So far, you have created entities, properties, and relationships that connect entities. However, the current state does not yet express knowledge such as organization-specific business concepts, rules, or the meanings of terms.
Knowledge Graphs generated by traditional GraphRAG are primarily composed of entities and relationships extracted from documents and other sources, without predefining what each concept means within the organization. Therefore, although structures and relationships could be represented on the same graph, organization-wide shared meanings and constraints for them were not guaranteed.
In an ontology, in addition to entity and relationship structures, you explicitly define business meanings and constraints, such as "What is a Person/Customer?", "What is an Assignment?", and "Which relationships are permitted?" This allows graph data to be interpreted consistently across the organization and establishes a unified basis for AI-agent reasoning.

Microsoft Fabric ontologies support the following types of metadata.

| Type | Description | Applicable to |
| :-- | :-- | :-- |
| Descriptions | What the concept represents (1–3 sentences) | Entity<br/>Property<br/>Relationship |
| Synonyms | Alternate names, abbreviations, industry terms, and so on | New TMDL supports entities, properties, and relationships. This lab configures entity synonyms |
| Additional metadata | Information such as units, sensitivity classifications, and business owners, represented as Key-Value pairs (Data quality: Incomplete) | Entity<br/>Property<br/>Relationship |

Here, you will configure metadata for the `Person` entity and the `belongsTo` relationship.

### Configure Metadata for the Person Entity

1. On the home screen, select `Person` in Explorer, then select **View entity type details**.
2. Select the **Configuration** tab.
3. In the `Entity metadata` section, enter the following information, then select **Update**.
  Enter synonyms one at a time.

| Field | Input value |
| :-- | :-- |
| Description | `An engineer or consultant employed by the company. The starting point for staffing searches and team formation.` |
| Synonyms | `employee` , `staff` , `member` , `engineer` |
| Additional metadata | Key: `sensitivity`<br/>Value: `Contains personal data` |

### Configure Metadata for the belongsTo Relationship

1. On the home screen, select `Organization` displayed in Explorer editing, then select **belongsTo** in the diagram.
2. In the `Metadata` section, select **Edit**.
3. Enter the following information, then select **Update**.
  You do not need to configure additional metadata.
4. Select **Save**.
5. Confirm that the `Relationship type updated successfully` banner appears.

| Field | Input value |
| :-- | :-- |
| Description | `Links an engineer to the organization they belong to.` |
| Additional metadata | - (Not configured) |

## 6. Configure the Ontology with Code

Up to this point, you have configured the ontology's various elements (entities, properties, relationships, and metadata) through the user interface. To be honest, this is probably quite a lot of work. Some of you who have actually performed the configuration may also be wondering:

- Is manual configuration the only way to configure an entire ontology?
- Can an ontology be configured using commands, REST APIs, or other methods?
- How should version control and configuration management be handled with CI/CD or IaC?

The Microsoft Fabric [Ontology REST API](https://learn.microsoft.com/en-us/rest/api/fabric/ontology/items) lets you manage ontology definitions as code.

Here, you will complete the remaining ontology configuration through the REST API.

> **Supported format**: New-experience TMDL at compatibility level `1000000` only. Old JSON-format items are not converted. See the [official database.tmdl specification](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/ontology-definition#databasetmdl-database-file) for the compatibility-level requirement.

1. Download the [nb_03_build_ontology.ipynb](./nb_03_build_ontology.ipynb) file.
2. Open the workspace you created.
3. At the top of the workspace screen, select **Import -> Notebook -> From this computer**.
4. Select the downloaded notebook file and **upload** it.
5. In Explorer on the left side of the screen, select **Add data item -> From OneLake catalog**.
6. Select `lh_its_asset_silver`, select **Add**, and make it the default lakehouse.
7. Confirm that the run language is **PySpark (Python)** and the environment is **Workspace default**, then select **Run all**.

After running the notebook, return to `ont_its_asset` and confirm that entities and relationships have been added alongside those configured manually. The notebook does not write directly to the graph model or trigger graph ingestion. Continue with [section 7](#enable-graph-en).

### Configuration

| Setting | Default / purpose |
|---|---|
| `ONTOLOGY_NAME` | `ont_its_asset` |
| `SILVER_LAKEHOUSE_NAME` | `lh_its_asset_silver` (attach as the default lakehouse) |
| `SILVER_SCHEMA` | SQL-side schema, normally `dbo`. Do not use `None`, even for schema-disabled lakehouses |
| `APPLY_METADATA` | `True`: configure descriptions, synonyms, and annotations |
| `OVERWRITE_EXISTING_METADATA` | `False`: retain existing values and fill missing metadata only |

### What the Notebook Does

| Step | Action |
|---|---|
| 1. Resolve items | Retrieve the Ontology, workspace and Silver Lakehouse identities, OneLake URL, and SQL analytics endpoint |
| 2. Read | Retrieve the current TMDL and actual Silver schemas |
| 3. Build | Validate bindings, fill missing types, relationships, properties, keys, and metadata, and list changed parts |
| 4. Update | Recheck for concurrent edits and submit all parts in one call only when changes exist |
| 5. Read back | Verify the required 17 entity types, 19 relationships, bindings, and metadata |

Existing parts, connection expressions, lineage tags, and manual settings are retained. Incompatible keys, types, or join columns stop the update rather than being overwritten. The notebook supplements the five types and three relationships created in the UI, or builds all definitions in an empty new-format item.

New tables use DirectLake partitions and a shared M expression of the form `let Source = AzureStorage.DataLake("https://<OneLake host>/<workspace ID>/<Lakehouse ID>", [HierarchicalNavigation=true]) in Source`. A matching UI-authored expression is reused; otherwise, `nb03_SilverLakehouse` is added. Partitions receive the source annotations observed in the saved UI definition: `ONT_WorkspaceId`, `ONT_ItemId`, `ONT_ItemKind = Lakehouse`, names, SQL connection details, and a timestamp. `ONT_ItemId` is the Lakehouse ID and `ONT_SqlDatabase` is the Lakehouse name, not the SQL endpoint ID. Source metadata is validated and supplemented regardless of `APPLY_METADATA`.

Property types come from the actual Silver schema. Descriptions use `///`, synonyms use `synonym`, and additional metadata uses `annotation`.

Relationships consist of a TOM `relationship` joining the origin's foreign-key column to the target's key column, referenced by an `entityRelationship`. New TOM relationships use `isActive: false` to avoid automatic filter propagation along multiple paths.

### Completion and Reruns

Confirm that the final cell displays:

```text
Verified required entity types: 17 / 17
Verified required relationship types: 19 / 19
Verified Lakehouse source bindings: 17 / 17
```

When nothing changes, the notebook prints `No definition changes are needed.` and skips the update. Avoid UI edits during execution; the concurrent-edit check is not a lock.

After replacing the notebook, use **Run all** to redefine its functions and variables. If only the UI definition changed, rerun from step 2. If a timeout or another error leaves the update outcome unknown, check the API operation status before rerunning.

Rerunning also repairs tables created by the previous nb_03: only tables referencing `nb03_SilverSource` with the expected SQL source and generated table lineage tag are switched to OneLake and receive missing Lakehouse identification metadata. Valid UI bindings and existing timestamps are preserved. Conflicting Lakehouse identifiers or modified source expressions stop the update rather than being overwritten. The legacy shared expression is retained because other tables might reference it. Readback checks all 17 tables' expressions and source identities; it does not guarantee UI rendering or successful MCP data queries.

<a id="enable-graph-en"></a>

## 7. Enable Graph Exploration in the Web UI

Complete `nb_03` and verify its 17 entity types, 19 relationships, and Lakehouse bindings first. This workshop enables the graph before Lab 03's MCP data searches. **Opening the explorer alone is insufficient: select the scope and finish loading the data.**

> **Product capability versus lab procedure:** The official documentation describes graph as optional for the new ontology. The Ontology MCP prerequisites do not explicitly require it either. These steps prepare this lab's graph searches; they do not establish a universal graph requirement for every MCP data query.

1. Reopen `ont_its_asset` in Fabric and confirm that the notebook's added types appear.
2. Select **Manage graph** on the Home ribbon.
3. In **Configure Graph**, check **Eligible** status and select all 17 lab entity types. Expand them and check that the preview includes the 19 relationships. For an existing graph, use **Select entities** to revise the scope.
4. Select **Continue** and review **Projection Summary** for omissions.
5. Select **Materialize** and wait for ingestion to finish. Depending on data volume, this can take minutes to hours.
6. Open **Explore graph**. Inspect **Components**, use **Add a node** to select `Person`, and expand related nodes. Select **Run** and confirm that results contain actual employee values and relationships, not just type names, before starting Lab 03.

**Ineligible types:** Hover over `Ineligible` for the reason. Correct missing keys or bindings. Graph projection supports Delta tables in Lakehouses or Mirrored Databases; types with multiple backing tables are among its limitations. Do not assume all demos will work with types omitted.

**Later changes:** Revisit **Manage graph** after adding types or relationships. After changing Silver data rows, open the associated graph model's **… → Schedule → Refresh now** in the workspace and verify the results. Rerunning `nb_03` alone does not confirm that graph data is current.

Official references, checked October 5, 2026: [Graph materialization, exploration, refresh, and limitations](https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-use-ontology-graph) and [Ontology MCP prerequisites and connection](https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-use-ontology-mcp-server).

## 8. Summary

In this lab, you used data in the Silver-layer Lakehouse you created to build an ontology and configure bindings to actual data.

Updating the ontology definition and preparing graph queries are separate steps. Complete the web-based ingestion in section 7 and confirm that actual values and relationships are queryable before proceeding to MCP.

In the next section, [Lab 03 - Ask the Agent](../03-ask-the-agent/README.md), you will use the `ont_its_asset` ontology created in this lab to try integration with an MCP client.

## Troubleshooting

### Ontology Creation and Item Resolution

| Symptom | Common cause | Resolution |
|---|---|---|
| Unable to create an ontology item | Ontology (Preview) is not enabled for the tenant, or the capacity is below F2 | Enable Ontology items in the tenant settings and assign the workspace to an F2 or higher capacity |
| `Ontology 'ont_its_asset' not found` | The name is misspelled, or it is in another workspace | Confirm that the name and workspace created in 2-1 match the notebook's `ONTOLOGY_NAME` |
| Unable to save the ontology name | The name contains spaces or hyphens | Use only numbers, letters, and underscores |
| Authentication error | The executing user does not have write permission for the ontology | Check the workspace role (Contributor or higher) |

### Entity Types and Data Bindings

| Symptom | Common cause | Resolution |
|---|---|---|
| An entity type does not appear in the graph | Unsaved binding, missing projection selection, or incomplete ingestion | Check the binding, then verify eligibility, scope, and ingestion completion through Manage graph in [section 7](#enable-graph-en) |
| A key-dependent operation fails | The key expected by this lab has not been set | `nb_03` fills missing keys. The new model itself also supports keyless entities |
| Instances cannot be queried | Source permissions, availability, or bindings have not been checked | Check Silver, its SQL analytics endpoint, and connection permissions; do not assume an automatic graph rebuild |
| Same as above | The Silver-layer table is empty | Confirm that `nb_02` in Step 1 completed successfully |
| `CorruptedPayload` / definition validation error | TMDL and existing bindings are inconsistent | Inspect the named type, property, or join column. `SILVER_SCHEMA` is the SQL-side schema, normally `dbo` |
| The number of `Project` / `Assignment` records is unusually large | A time-series binding was added | This kit uses static bindings only for every entity type (see 2-2). Remove the time-series binding |
| The agent answers "people who joined recently" based on the ingestion date | `_valid_as_of` is still bound | Remove it from the property list (2-2 / `docs/property-binding.md`) |

### Relationship Types

| Symptom | Common cause | Resolution |
|---|---|---|
| The relationship is empty even though configuration succeeded | Join columns or their values do not match | Join the origin's foreign-key column to the target's key column as shown in section 4 |
| `existing TOM relationship columns differ` | Saved join columns differ from the expected mapping | Correct and save them as shown in section 4, then rerun from step 2. `performedBy` uses `PersonId` on both sides |
| `keyProperty must be ...` | A different entity key is configured | See "Key and Existing Binding Validation" below |
| Lines are visible, but no results are returned | The relationship data binding is misaligned | Confirm that the Silver-layer table or column names have not been changed |

### Metadata

| Symptom | Common cause | Resolution |
|---|---|---|
| Manually configured metadata disappeared | `OVERWRITE_EXISTING_METADATA` is `True` | Change it back to `False` (the default is `False`) |
| Additional metadata produces an error | A key is duplicated within the same target | Make keys unique within each entity type, property, and relationship type |
| Synonyms are not added | A manually configured synonym list is being retained | Existing lists are preserved by default. Set `OVERWRITE_EXISTING_METADATA = True` only to replace them intentionally |

### Notebook (`nb_03`)

| Symptom | Common cause | Resolution |
|---|---|---|
| The counts do not reach 17 / 19 | A manually created name is misspelled | Match the capitalization of entity types `Person` / `Project` / `Customer` / `Organization` / `Assignment` and relationship types `belongsTo` / `performedBy` / `deliveredFor` |
| `Old JSON ontology detected` | The selected item uses the old experience | Use an ontology created with the new experience. Do not mix old-format parts into it |
| `The ontology changed after it was read` | Another edit was saved while the notebook was running | Stop UI edits and rerun from the definition-reading step |
| `Readback still needs changes` | The service did not retain some definition or metadata content | Inspect the returned TMDL and listed parts. Do not automatically resubmit the update |
| The schema is visible through MCP, but data search fails | Data permissions or bindings have not been checked | Do not rely only on a successful definition update. Check access to entity instances and relationships in Fabric |
| Execution failed midway | Validation, communication, or another error | Fix the cause and follow "Completion and Reruns" above. If the update outcome is unknown, check its status first |

### Key and Existing Binding Validation

The new model uses `keyProperty` to name a single entity identity property. `nb_03` fills it from `ENTITY_TYPES` only when it is absent. For example, Assignment uses `AssignmentKey`.

When a backing column exists but the corresponding entity `property` declaration is absent, the notebook checks its type against the source schema and adds the missing declaration. Existing table names, column names, connections, and metadata are retained.

The notebook stops before writing if an existing key differs, a required backing column is missing, types do not match the source schema, or join columns differ. Correct the named object in the UI and rerun. It does not automatically convert customized models with composite keys, inheritance, or time-series extensions.

### Local Validation

Check TMDL generation, preservation of existing definitions, reruns, and REST error handling without connecting to Fabric:

```powershell
python -m unittest discover -s .\02-model-and-connect-ontology\tests -v
```
