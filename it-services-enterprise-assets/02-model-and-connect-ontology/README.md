_(English version follows below)_

# Lab 02 - オントロジーをつくり、つなぐ

**所要時間** : 約 50 分  
**ゴール** : `ont_its_asset` に、エンティティ型17個・関係型19個が定義され、`sv_*` テーブルにバインドされている状態  

---

## 0. このラボで作るもの

| アイテム | 名前 | 役割 |
|---|---|---|
| オントロジー | `ont_its_asset` | ワークショップで作成する各種アイテムを配置 |
| ノートブック | `nb_03_build_ontology` | シルバー層のレイクハウスにある Delta テーブルの内容を元にオントロジーの作成とグラフモデルの設定を行うノートブック。 |

このラボを進めるにあたっては、[事前準備に記載の設定](../README.md#事前準備) を行う必要があります。確認を行ってから実施するようにしてください。  

## 1. オントロジーを作成する

Microsoft Fabric では、オントロジーを扱う場合、ワークスペース上で専用の項目 (アイテム) を作成する必要があります。

1. 作成済みのワークスペース画面を開きます。  
2. `+ 新しい項目` から、**Ontology** を選択します。
3. 名前に `ont_its_asset` を入力します。  
  場所は 1. で作成したワークスペース名が選択されていることを確認します。  
4. **作成** を選択して、オントロジーを作成します。 

オントロジーの作成が完了したら、ワークスペースの画面を開いてみてください。`ont_its_asset` の項目に紐づく子項目として、以下の 3 つが作成されていることを確認できます。  

- グラフモデル (ont_its_asset_graph_xxxxxxxx)
- レイクハウス (ont_its_asset_lh_xxxxxxxx)
- SQL 分析エンドポイント (ont_its_asset_lh_xxxxxxxxx)

これらはオントロジーを利用するにあたって裏で必要となる管理アーティファクトになるため、誤って削除しないようにしてください。  

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
4. `バインディングの選択` 欄にて **データ バインディングの追加 -> レイクハウス テーブル** を選択します。
5. `lh_its_asset_silver` を選択し、**次へ** を選択します。
6. OneLake -> lh_its_asset_silver -> Tables -> dbo を選択し、**sv_person** を選択します。
7. **エンティティ型キー -> エンティティ型キーの定義** を選択します。
8. プロパティ一覧のプルダウンリストより、**PersonId** を選択します。
9. キーとして使用する選択済みプロパティに `PersonId` のみが表示されていることを確認し、**保存** を選択します。
10. プロパティ欄に、レイクハウスの Delta テーブルの列名と同じ数のプロパティが表示されていることを確認します。(表示されていない場合は、作業を中断し、1 の手順からやり直してください)
11. プロパティ欄に表示されている `_valid_as_of` の行の横に表示されているごみ箱アイコンを選択し、**_valid_as_of 行を削除** します。
12. **保存** を選択します。
13. `エンティティ型が正常に更新されました` のバナー表示を確認の上、**キャンセル** を選択します。
14. **インスタンス** タブを選択し、Person エンティティにバインドした実データが表示されることを確認します。
15. 同じ手順で、残りのエンティティに対しても同じようにプロパティの設定と実データへのバインドを行います。
  設定の詳細については、以下の表を参照してください。`Project` と `Assignment` では日時型のプロパティがタイムスタンプ列の候補として表示されますが、このラボでは時系列プロパティを定義しないため `なし` を選択します。

  | エンティティ | データバインド先 | エンティティ型キー | 時系列データ -> タイムスタンプ列 | プロパティの追加/削除 |
  | :-- | :-- | :-- | :-- | :-- |
  | `Project` | `sv_project` | `ProjectId` | `なし` | (削除) `_valid_as_of` |
  | `Customer` | `sv_customer` | `CustomerId` | — | (削除) `_valid_as_of` |
  | `Organization` | `sv_organization` | `OrganizationId` | — | (削除) `_valid_as_of` |
  | `Assignment` | `sv_assignment` | **`AssignmentKey`** | `なし` | **(削除) `AssignmentId`**<br/>(削除) `_valid_as_of` |

### (解説1) Assignment エンティティ

Assignment エンティティのみ、エンティティ型キーの設定が Id 列を指定せず他と異なっています。なぜそうなっているのかについて解説します。  

`Assignment` は、従業員とプロジェクトの間にある「誰が、どの案件にアサインされているか」を表す関連実体です。このラボでは、`PersonId` と `ProjectId` の組み合わせを一つのアサインを識別する業務キーとして扱います。

元データの `AssignmentId` はソースシステムが採番した ID です。データの再登録や再エクスポートによって値が変わる可能性があるため、これをオントロジーのエンティティ型キーにすると、業務上は同じアサインであっても別のエンティティとして認識されるおそれがあります。反対に、異なる環境や時点で同じ ID が再利用された場合は、別のアサインを同一のエンティティとして扱うおそれがあります。

そこで、シルバー層を作成する `nb_02_build_silver` では、次のように業務キーから決定論的な `AssignmentKey` を生成しています。

```text
AssignmentKey = SHA-256(PersonId + "|" + ProjectId)
```

同じ従業員とプロジェクトの組み合わせからは常に同じ値が生成されるため、ソース側の `AssignmentId` が変わっても、オントロジー上では同じ Assignment エンティティとして識別できます。また、`performedBy` リレーションでは、この `AssignmentKey` で Assignment を特定し、`PersonId` を使って担当者の Person エンティティへ接続します。

この設計は、単に ID という名前の列をキーにするのではなく、**業務上、同じ実体であり続けるための条件をエンティティ型キーにする**という考え方に基づいています。なお、このラボでは「一人の従業員が一つのプロジェクトに持つ現在のアサインは一つ」という前提です。同じ従業員とプロジェクトの間に複数のアサインを区別して保持する必要がある場合は、役割や有効期間などを業務キーに追加する必要があります。

### (解説2) 時系列データ

Fabric オントロジーでは、通常のプロパティと時系列プロパティは区別して定義されます。**タイムスタンプ列に指定するのは、時系列プロパティの観測時刻を表す列だけです。** ソース列の型としては `datetime`、`date`、`timestamp` がサポートされていますが、対応する型の列をすべてタイムスタンプ列にするわけではありません。

たとえば、`Project.StartDate` / `Project.EndDate` や `Assignment.StartDate` / `Assignment.EndDate` は、プロジェクトやアサインの期間を表す通常のプロパティです。時系列プロパティの観測時刻ではないため、タイムスタンプ列には指定せず `なし` を選択します。

時系列データをバインドする場合は、先に静的データのバインドとエンティティ型キーの定義を完了させます。その後、同じエンティティ型に時系列データのソースを追加し、静的データのキーと一致する列でエンティティを対応付け、時系列データ内の観測時刻を表す列をタイムスタンプ列として選択します。時系列データには OneLake または Eventhouse のテーブルを使用できます。

このラボの GUI 手順で作成する 5 つのエンティティは静的データのみをバインドするため、時系列プロパティおよび時系列バインドは追加しません。

> 参考（Microsoft Learn）
> - [オントロジーへのデータのバインド](https://learn.microsoft.com/ja-jp/fabric/iq/ontology/how-to-bind-data)
> - [チュートリアル パート 2: オントロジーを追加データで強化する](https://learn.microsoft.com/ja-jp/fabric/iq/ontology/tutorial-2-enrich-ontology)

### (解説3) 一部のプロパティを削除した理由

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
7. **belongsTo** を選択します。
8. `マッピング テーブル` にて、**使用可能なソースの参照 -> sv_person** を選択します。  
  (sv_person が一覧に表示されない場合は、使用可能なソースの参照から選択を行ってください)
9. `一致した Person: PersonId` 項目にて、**PersonId** を選択します。  
10. `一致した Organization: OrganizationId` 項目にて、**OrganizationId** を選択します。
11. **保存** を選択します。
12. `リレーションシップの種類が正常に更新されました` のバナー表示を確認の上、**ホーム** を選択します。
13. 同じ手順で、2 つのリレーション設定を行います。
  設定の詳細については、以下の表を参照してください。

  | リレーションシップ名 | 元のエンティティ型 | ターゲット エンティティ型 | マッピング テーブル | 元のエンティティ型の列 | ターゲット エンティティ型の列 |
  | :-- | :-- | :-- | :-- | :-- | :-- |
  | `performedBy` | `Assignment` | `Person` | `sv_assignment` | `AssignmentKey` | `PersonId` |
  | `deliveredFor` | `Project` | `Customer` | `sv_project` | `ProjectId` | `CustomerId` |

これで、エンティティ同士の関係 (リレーション) を辿るためのキー設定が完了しました。  
オントロジーにおけるリレーションの設定での重要ポイントは、以下の 2 点です。

- リレーションシップ名は、AI エージェントがエンティティ同士の関係性を適切に理解できる一意な名称にすること
- エンティティ同士の関係を設定する際、ターゲットエンティティ側のプロパティにはキープロパティと一致するものを設定すること

リレーショナルデータベース (RDB) の経験がある方は、この設定が `外部キー` (Foreign Key) の設定に似ているということにお気づきかもしれません。確かに似ているのですが、オントロジーのリレーション設定では、RDB の外部キーとはすこし内容が異なります。  
オントロジーのリレーション設定では、エンティティのキープロパティを指定します。これにより、オントロジーが裏側で使用するグラフモデルも、エンティティ（ノード）をリレーション（エッジ）で紐づける処理が行われます。例えば、`performedBy` リレーションにおける Assignment と Person の設定を見てみると、キープロパティは同じ PersonId になっていません。両方とも PersonId で結んでしまうこと、エッジの始点にあるノード (Assignment) がどのデータを使用しているのか、特定ができなくなってしまいます。そのため、外部キーとは異なり、どちらのエンティティにおいても、自身を一意で識別できるキープロパティで指定がされているわけです。  

## 5. メタデータを設定する

ここまでで、エンティティとプロパティ、エンティティ同士を結ぶリレーションを作成しました。しかし、今の状態では組織固有のビジネス概念やルール、用語の意味といった知識までは表現されていません。   
従来の GraphRAG が生成する Knowledge Graph は、文書などから抽出されたエンティティやリレーションを中心に構成されており、それぞれの概念が組織内でどのような意味を持つのかを事前に定義しているわけではありませんでした。そのため、同じグラフ上の構造や関係性は表現できるものの、それらに対する組織共通の意味や制約が保証されているわけではありませんでした。  
オントロジーでは、エンティティやリレーションの構造に加え、「Person/Customer とは何か」「Assignment とは何か」「どのような関係のみを許可するのか」といったビジネス上の意味や制約を明示的に定義します。これにより、グラフ上のデータを組織全体で一貫した意味で解釈できるようになり、AI エージェントが推論を行う際の基準を統一することができます。  

Microsoft Fabric のオントロジーでは、以下の種類のメタデータを設定することができます。  

| 種類 | 内容 | 付与できる対象 |
| :-- | :-- | :-- |
| 説明 (Descriptions) | その概念が何を表すか (1 ~ 3 文) | エンティティ<br/>プロパティ<br/>リレーション |
| 類義語 (Synonyms) | 別名や略称、業界用語など | エンティティ |
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

1. ホーム画面にて、エクスプローラ編集に表示されている `Organization` を選択し、図上にある **belongsTo** を選択します。
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

Microsoft Fabric では、オントロジーやグラフモデルに関する REST API が提供されています。そのため、本番環境でオントロジーを利用する際は、コードベースの管理や CI/CD といった運用を行うことが可能になっています。  

https://learn.microsoft.com/en-us/rest/api/fabric/ontology/items
https://learn.microsoft.com/en-us/rest/api/fabric/graphmodel/items

ここでは、残りのオントロジー設定を REST API 経由で行います。  

1. [nb_03_build_ontology.ipynb](./nb_03_build_ontology.ipynb) ファイルをダウンロードします。  
2. 作成済みのワークスペース画面を開きます。  
3. ワークスペース画面の上部にある **インポート -> ノートブック -> コンピューターから** を選択します。   
4. ダウンロードしたノートブックファイルを選択し、**アップロード** します。
5. 画面左にあるエクスプローラーから、**データ項目の追加 -> OneLake カタログから** を選択します。  
6. `lh_its_asset_silver` を選択し、**追加** を選択します。
7. 実行言語が **PySpark (Python)**、環境が **ワークスペースの既定値** になっていることを確認し、**すべて実行** を選択します。

ノートブックの実行が完了したら、`ont_its_asset` のオントロジー画面に戻ってみてください。手動で設定したもの以外に、新しくエンティティやリレーションが追加されています。また、グラフモデルについても、ノードとエッジが追加されていることを確認できます。  

### ノートブックが実行したこと

**フェーズ 1: 構造をつくる**

1. 既存のオントロジー定義を取得する（手動で作った分を壊さない）
2. まだ無いエンティティ型 12 個を定義に追加する
3. まだ無い関係型 16 個を定義に追加する

**フェーズ 2: 意味を与える**

4. 各定義の `semanticEnrichment` に、説明・シノニム・追加メタデータを書き込む
5. 手動で手動設定した内容は上書きしない（`OVERWRITE_EXISTING_METADATA = False`）

**フェーズ 3: 反映する**

6. Update Item Definition API で、**一度に** 定義を更新する

構造とメタデータを別々に投入せず 1 回の更新にまとめているのは、API 呼び出しを減らすためと、途中で失敗して半端な状態が残るのを避けるためです。

**フェーズ 4: グラフを構築する**

7. オントロジーの定義から、グラフのノードとエッジを生成する
8. グラフにデータを取り込む

**取り込みには数分かかります。** デモ中にライブで実行する場合は、冒頭の
`WAIT_FOR_REFRESH = False` にすると完了を待たずに次へ進めます（取り込み自体は
バックグラウンドで続きます）。事前に一度通しで実行しておくのが確実です。

## 7. まとめ

このラボでは、作成したシルバー層のレイクハウスに存在するデータを使用して、オントロジーの作成と実データへのバインド設定を行いました。  

これで、オントロジーを介して MCP クライアントにコンテキストを提供する準備が整いました。

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
| グラフにエンティティ型が出てこない | データバインドが未保存 | エンティティ型を作っただけでは出ない。バインドを保存する |
| 同上 | エンティティ型キーが未設定 | キーが未定義だとノードを構成できない。2-2 の表のキーを設定する |
| 同上 | 保存直後で再構築中 | 数分待ってから再確認する |
| 同上 | シルバー層のテーブルが空 | Step 1 の `nb_02` が正常終了しているか確認する |
| `CorruptedPayload` | `SILVER_SCHEMA` がレイクハウスの種別と合っていない | スキーマ有効なら `'dbo'`、スキーマ無効（レガシー）なら `None` |
| `Project` / `Assignment` の件数が異様に多い | 時系列バインドを追加してしまった | 本キットは全エンティティ型が静的バインドのみ（2-2 参照）。時系列バインドを削除する |
| エージェントが取り込み日で「最近入社した人」を答える | `_valid_as_of` をバインドしたまま | プロパティ一覧から削除する（2-2 / `docs/property-binding.md`） |

### 関係型

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 設定は成功したのに関係が空 | 突き合わせ列が相手のキーと一致していない | 起点・終点とも、相手側エンティティ型のキーと**値が一致する列**を選ぶ（2-3） |
| `Contextualization sourceKeyRefBindings count (1) must match the number of EntityIdParts (0)` | 起点エンティティ型にキーが未設定（`Assignment` で起きやすい） | 下記「`EntityIdParts (0)` のエラー」を参照 |
| `Edge source/destination key column count does not match` | 結合列のフィールド名が違う | `sourceNodeKeyColumns` / `destinationNodeKeyColumns` を使う |
| 線は見えているのに結果が出ない | 関係のデータバインドがずれている | シルバー層のテーブル名や列名を変更していないか確認する |

### メタデータ

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 手動設定したメタデータが消えた | `OVERWRITE_EXISTING_METADATA` が `True` | `False` に戻す（既定は `False`） |
| 追加メタデータでエラーになる | 同じ対象の中でキーが重複している | エンティティ型・プロパティ・関係型それぞれの単位でキーを一意にする |
| シノニムを設定できない | プロパティまたは関係型に設定しようとしている | シノニムはエンティティ型のみ対応 |

### ノートブック（`nb_03`）とグラフ

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| 数が 17 / 19 にならない | 手動作成した名前の綴り違い | エンティティ型 `Person` / `Project` / `Customer` / `Organization` / `Assignment`、関係型 `belongsTo` / `performedBy` / `deliveredFor`（大文字小文字も一致させる） |
| グラフが5ノードのまま増えない | フェーズ4が実行されていない | `BUILD_GRAPH = True` になっているか確認し、「ノード 17 種類 / エッジ 19 種類」の出力を確認する |
| 同上 | 取り込みが完了していない | 「取り込みが完了しました」まで出たか確認する。`WAIT_FOR_REFRESH = False` なら数分後に再確認 |
| `GraphNotRefreshable` | グラフが空の状態でリフレッシュしようとした | フェーズ4でグラフ定義を書き込んでから取り込む（ノートブックはこの順で実行する） |
| MCP でスキーマは見えるがデータ検索が失敗する | グラフの取り込みが未完了 | フェーズ4の出力を確認する。オントロジーとグラフは別アイテムで、スキーマだけ先に見えることがある |
| 実行が途中で失敗した | — | そのまま再実行してよい。何度実行しても結果は同じになるよう作ってある |

### `EntityIdParts (0)` のエラー

関係型は、起点エンティティの**キー**で突き合わせます。UI ではエンティティ型キーを選ばずに保存できてしまうため、キーが0個のエンティティ型が残ることがあります。
そのエンティティ型を起点にした関係型を追加しようとすると、突き合わせ列は 1 個なのにキーが 0 個で数が合わず、定義の更新が 400 で失敗します。

本キットで起きやすいのは `Assignment` です。エンティティ型キーに `AssignmentKey` を選ぶ手順（2-2）を飛ばすと、この状態になります。

`nb_03_build_ontology.ipynb` は実行時にキーの有無を点検し、未設定のエンティティ型があれば `ENTITY_TYPES` の定義に従ってキーを補ってから関係型を作ります。エラーが出た場合は、ノートブックを最新版に差し替えて再実行してください。UI に戻る必要はありません。

キーにするプロパティ自体がバインドされていない場合だけは補えないため、その場合はどのプロパティが足りないかを名指ししたメッセージで停止します。

---

# Lab 02 - Build and Connect an Ontology

**Estimated time**: Approximately 50 minutes  
**Goal**: Define 17 entity types and 19 relationship types in `ont_its_asset`, with bindings to the `sv_*` tables  

---

## 0. What You Will Build in This Lab

| Item | Name | Purpose |
|---|---|---|
| Ontology | `ont_its_asset` | Contains the various items created in this workshop |
| Notebook | `nb_03_build_ontology` | A notebook that creates the ontology and configures the graph model based on the contents of the Delta tables in the Silver-layer Lakehouse. |

Before proceeding with this lab, you must complete the [settings described in Prerequisites](../README.md#prerequisites). Make sure you have verified them before continuing.

## 1. Create an Ontology

In Microsoft Fabric, working with an ontology requires creating a dedicated item in a workspace.

1. Open the workspace you created.
2. From `+ New item`, select **Ontology**.
3. Enter `ont_its_asset` as the name.
  Confirm that the workspace name created in step 1 is selected as the location.
4. Select **Create** to create the ontology.

After the ontology has been created, open the workspace screen. You can confirm that the following three child items have been created and are associated with the `ont_its_asset` item.

- Graph model (ont_its_asset_graph_xxxxxxxx)
- Lakehouse (ont_its_asset_lh_xxxxxxxx)
- SQL analytics endpoint (ont_its_asset_lh_xxxxxxxxx)

These are management artifacts required behind the scenes to use the ontology, so be careful not to delete them accidentally.

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
4. In the `Select binding` section, select **Add data binding -> Lakehouse table**.
5. Select `lh_its_asset_silver`, then select **Next**.
6. Select OneLake -> lh_its_asset_silver -> Tables -> dbo, then select **sv_person**.
7. Select **Entity type key -> Define entity type key**.
8. From the property list dropdown, select **PersonId**.
9. Confirm that only `PersonId` appears under the selected properties used as the key, then select **Save**.
10. Confirm that the Properties section displays the same number of properties as there are columns in the Lakehouse Delta table. (If they are not displayed, stop and repeat the process from step 1.)
11. Select the trash icon next to the `_valid_as_of` row displayed in the Properties section and **delete the _valid_as_of row**.
12. Select **Save**.
13. After confirming that the `Entity type updated successfully` banner appears, select **Cancel**.
14. Select the **Instances** tab and confirm that the actual data bound to the Person entity is displayed.
15. Follow the same steps to configure properties and bind actual data for the remaining entities.
  Refer to the following table for configuration details. For `Project` and `Assignment`, date/time properties appear as candidates for the timestamp column, but because this lab does not define time-series properties, select `None`.

  | Entity | Data binding target | Entity type key | Time-series data -> Timestamp column | Add/remove properties |
  | :-- | :-- | :-- | :-- | :-- |
  | `Project` | `sv_project` | `ProjectId` | `None` | (Remove) `_valid_as_of` |
  | `Customer` | `sv_customer` | `CustomerId` | — | (Remove) `_valid_as_of` |
  | `Organization` | `sv_organization` | `OrganizationId` | — | (Remove) `_valid_as_of` |
  | `Assignment` | `sv_assignment` | **`AssignmentKey`** | `None` | **(Remove) `AssignmentId`**<br/>(Remove) `_valid_as_of` |

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

For example, `Project.StartDate` / `Project.EndDate` and `Assignment.StartDate` / `Assignment.EndDate` are regular properties representing the duration of a project or assignment. Because they do not represent the observation time of a time-series property, do not specify them as the timestamp column; select `None`.

When binding time-series data, first complete the static data binding and define the entity type key. Then add a time-series data source to the same entity type, associate entities using a column that matches the static data key, and select the column representing the observation time in the time-series data as the timestamp column. OneLake or Eventhouse tables can be used for time-series data.

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
7. Select **belongsTo**.
8. Under `Mapping table`, select **Browse available sources -> sv_person**.
  (If sv_person does not appear in the list, select it through Browse available sources.)
9. Under `Matched Person: PersonId`, select **PersonId**.
10. Under `Matched Organization: OrganizationId`, select **OrganizationId**.
11. Select **Save**.
12. After confirming that the `Relationship type updated successfully` banner appears, select **Home**.
13. Follow the same steps to configure two relationships.
  Refer to the following table for configuration details.

  | Relationship name | Source entity type | Target entity type | Mapping table | Source entity type column | Target entity type column |
  | :-- | :-- | :-- | :-- | :-- | :-- |
  | `performedBy` | `Assignment` | `Person` | `sv_assignment` | `AssignmentKey` | `PersonId` |
  | `deliveredFor` | `Project` | `Customer` | `sv_project` | `ProjectId` | `CustomerId` |

You have now completed the key configuration required to traverse the relationships between entities.
The following two points are important when configuring ontology relationships:

- Give each relationship a unique name that allows an AI agent to understand the relationship between entities correctly
- When defining a relationship between entities, set the target entity's property to one that matches its key property

If you have experience with relational databases (RDBs), you may notice that this configuration resembles a `foreign key` setting. Although they are similar, ontology relationship configuration differs somewhat from an RDB foreign key.
Ontology relationship configuration specifies the key property of each entity. This enables the graph model used behind the scenes by the ontology to link entities (nodes) through relationships (edges). For example, in the Assignment and Person configuration for the `performedBy` relationship, the key properties are not both the same PersonId. If both sides were joined using PersonId, it would be impossible to identify which data is used by the node at the start of the edge (Assignment). Therefore, unlike a foreign key, each entity is specified using a key property that uniquely identifies that entity itself.

## 5. Configure Metadata

So far, you have created entities, properties, and relationships that connect entities. However, the current state does not yet express knowledge such as organization-specific business concepts, rules, or the meanings of terms.
Knowledge Graphs generated by traditional GraphRAG are primarily composed of entities and relationships extracted from documents and other sources, without predefining what each concept means within the organization. Therefore, although structures and relationships could be represented on the same graph, organization-wide shared meanings and constraints for them were not guaranteed.
In an ontology, in addition to entity and relationship structures, you explicitly define business meanings and constraints, such as "What is a Person/Customer?", "What is an Assignment?", and "Which relationships are permitted?" This allows graph data to be interpreted consistently across the organization and establishes a unified basis for AI-agent reasoning.

Microsoft Fabric ontologies support the following types of metadata.

| Type | Description | Applicable to |
| :-- | :-- | :-- |
| Descriptions | What the concept represents (1–3 sentences) | Entity<br/>Property<br/>Relationship |
| Synonyms | Alternate names, abbreviations, industry terms, and so on | Entity |
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

Microsoft Fabric provides REST APIs for ontologies and graph models. Therefore, when using an ontology in a production environment, you can adopt practices such as code-based management and CI/CD.

https://learn.microsoft.com/en-us/rest/api/fabric/ontology/items
https://learn.microsoft.com/en-us/rest/api/fabric/graphmodel/items

Here, you will complete the remaining ontology configuration through the REST API.

1. Download the [nb_03_build_ontology.ipynb](./nb_03_build_ontology.ipynb) file.
2. Open the workspace you created.
3. At the top of the workspace screen, select **Import -> Notebook -> From this computer**.
4. Select the downloaded notebook file and **upload** it.
5. In Explorer on the left side of the screen, select **Add data item -> From OneLake catalog**.
6. Select `lh_its_asset_silver`, then select **Add**.
7. Confirm that the run language is **PySpark (Python)** and the environment is **Workspace default**, then select **Run all**.

When notebook execution is complete, return to the `ont_its_asset` ontology screen. In addition to the items you configured manually, new entities and relationships have been added. You can also confirm that nodes and edges have been added to the graph model.

### What the Notebook Does

**Phase 1: Build the Structure**

1. Retrieve the existing ontology definition (without breaking what was created manually)
2. Add the 12 entity types that do not yet exist to the definition
3. Add the 16 relationship types that do not yet exist to the definition

**Phase 2: Add Meaning**

4. Write descriptions, synonyms, and additional metadata to each definition's `semanticEnrichment`
5. Do not overwrite content configured manually (`OVERWRITE_EXISTING_METADATA = False`)

**Phase 3: Apply the Changes**

6. Update the definition **all at once** using the Update Item Definition API

The structure and metadata are combined into a single update rather than submitted separately to reduce API calls and avoid leaving a partially configured state if the process fails midway.

**Phase 4: Build the Graph**

7. Generate graph nodes and edges from the ontology definition
8. Load data into the graph

**Loading takes several minutes.** When running it live during a demo, set
`WAIT_FOR_REFRESH = False` at the beginning to proceed without waiting for completion (the load itself
continues in the background). The safest approach is to run the notebook through once in advance.

## 7. Summary

In this lab, you used data in the Silver-layer Lakehouse you created to build an ontology and configure bindings to actual data.

You are now ready to provide context to an MCP client through the ontology.

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
| An entity type does not appear in the graph | The data binding has not been saved | Creating the entity type alone is not enough. Save the binding |
| Same as above | The entity type key has not been set | A node cannot be constructed without a defined key. Set the key shown in the table in 2-2 |
| Same as above | The graph is rebuilding immediately after saving | Wait several minutes, then check again |
| Same as above | The Silver-layer table is empty | Confirm that `nb_02` in Step 1 completed successfully |
| `CorruptedPayload` | `SILVER_SCHEMA` does not match the Lakehouse type | Use `'dbo'` when schemas are enabled, or `None` when schemas are disabled (legacy) |
| The number of `Project` / `Assignment` records is unusually large | A time-series binding was added | This kit uses static bindings only for every entity type (see 2-2). Remove the time-series binding |
| The agent answers "people who joined recently" based on the ingestion date | `_valid_as_of` is still bound | Remove it from the property list (2-2 / `docs/property-binding.md`) |

### Relationship Types

| Symptom | Common cause | Resolution |
|---|---|---|
| The relationship is empty even though configuration succeeded | The matching column does not correspond to the other entity's key | For both the source and destination, select columns whose **values match** the key of the entity on the other side (2-3) |
| `Contextualization sourceKeyRefBindings count (1) must match the number of EntityIdParts (0)` | The source entity type has no key configured (common with `Assignment`) | See "`EntityIdParts (0)` Error" below |
| `Edge source/destination key column count does not match` | The join-column field names are incorrect | Use `sourceNodeKeyColumns` / `destinationNodeKeyColumns` |
| Lines are visible, but no results are returned | The relationship data binding is misaligned | Confirm that the Silver-layer table or column names have not been changed |

### Metadata

| Symptom | Common cause | Resolution |
|---|---|---|
| Manually configured metadata disappeared | `OVERWRITE_EXISTING_METADATA` is `True` | Change it back to `False` (the default is `False`) |
| Additional metadata produces an error | A key is duplicated within the same target | Make keys unique within each entity type, property, and relationship type |
| Unable to configure synonyms | You are attempting to configure them on a property or relationship type | Synonyms are supported only for entity types |

### Notebook (`nb_03`) and Graph

| Symptom | Common cause | Resolution |
|---|---|---|
| The counts do not reach 17 / 19 | A manually created name is misspelled | Match the capitalization of entity types `Person` / `Project` / `Customer` / `Organization` / `Assignment` and relationship types `belongsTo` / `performedBy` / `deliveredFor` |
| The graph remains at five nodes and does not grow | Phase 4 was not executed | Confirm that `BUILD_GRAPH = True` and check for the output "17 node types / 19 edge types" |
| Same as above | Loading has not completed | Confirm that "Loading completed" was displayed. If `WAIT_FOR_REFRESH = False`, check again after several minutes |
| `GraphNotRefreshable` | An attempt was made to refresh an empty graph | Write the graph definition in Phase 4 before loading it (the notebook runs in this order) |
| The schema is visible through MCP, but data search fails | Graph loading has not completed | Check the Phase 4 output. The ontology and graph are separate items, so the schema may become visible first |
| Execution failed midway | — | You can rerun it as-is. It is designed to produce the same result no matter how many times it runs |

### `EntityIdParts (0)` Error

Relationship types match against the source entity's **key**. Because the UI allows you to save without selecting an entity type key, an entity type with zero keys may remain.
If you try to add a relationship type whose source is that entity type, the definition update fails with a 400 error because there is one matching column but zero keys, so the counts do not match.

In this kit, this most commonly occurs with `Assignment`. Skipping the step in 2-2 that selects `AssignmentKey` as the entity type key causes this state.

At runtime, `nb_03_build_ontology.ipynb` checks whether keys are present. If an entity type has no configured key, it supplements the key according to the `ENTITY_TYPES` definition before creating relationship types. If you encounter this error, replace the notebook with the latest version and rerun it. You do not need to return to the UI.

The only case it cannot correct is when the property to be used as the key is not itself bound. In that case, it stops with a message explicitly naming the missing property.
