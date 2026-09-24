_(English version follows below)_

# Lab 03 - エージェントに問いかける

**所要時間**：約 45 分  
**ゴール**：オントロジーに対して6つの問いを投げ、単一システムの検索では出せない答えが返ってくることを確認している状態  

## 0. このラボで行うこと

これまでに作成したオントロジーは、MCP エンドポイントを通じて、外部の AI エージェントから利用できます。

このラボでは、Visual Studio Code を MCP クライアントとしてオントロジーに接続し、自然言語による問い合わせを行います。人材・案件・顧客・ナレッジ・財務などの情報を関係性に沿って横断することで、単一のデータソースだけでは得られない回答やインサイトを導き出せることを確認します。

あわせて、オントロジーに設定した業務用語、類義語、プロパティの説明などのメタデータが、AI エージェントによる問い合わせの解釈や回答にどのように役立つかを確認します。

## 1. MCP サーバのエンドポイントを確認する

Microsoft Fabric のオントロジーは、MCP (Model Context Protocol) に対応しており、MCP サーバーとして外部の AI エージェントと MCP を通じて対話できます。  
つまり、公開されたオントロジーは、社内の Copilot をはじめ、Copilot Studio や GitHub Copilot、Claude Code など、様々な AI エージェントで、同一のオントロジーを扱うことができるということです。  

Microsoft Fabric における、オントロジー MCP サーバーのエンドポイント URL 形式は以下となっています。

> https://api.fabric.microsoft.com/v1/mcp/dataPlane/workspaces/{workspace-ID}/items/{ontology-item-ID}/ontologyEndpoint

また、上記に加え、Agent 365 Gateway を経由したオントロジー MCP サーバーのエンドポイントも存在します。

> https://agent365.svc.cloud.microsoft/agents/tenants/{tenant_id}/servers/mcp_FabricIQOntology/workspaces/{workspace-ID}/ontologies/{ontology-item-ID}

`workspace-ID` および `ontology-item-ID` は、環境ごとに一意の値となります。この 2 つの ID 値を確認し、エンドポイント URL を完成させる形となります。  

1. 作成済みのワークスペース画面を開き、`ont_its_asset` を開きます。 
2. ブラウザーのアドレスバーの URL を取得します。以下のような形になっているはずです。  
  > https://app.fabric.microsoft.com/groups/{workspace-ID}/ontologies/{ontology-item-ID}?experience=fabric-developer
3. `workspace-ID` と `ontology-item-ID` の位置にある値 (GUID) をメモ帳などに控えてください。  

Agent 365 Gateway 対応版の場合は、加えて `tenant_Id` も必要となります。こちらは、Entra ID のテナント ID の値となります。

## 2. MCP クライアントの設定を行う

Microsoft Fabric のオントロジーを MCP で利用するため、MCP クライアントの設定を行います。このラボでは、Visual Studio Code で MCP サーバーの設定を行います。

https://code.visualstudio.com/docs/agent-customization/mcp-servers

1. **Ctrl+Shift+P** を入力し、**MCP: Add Server** を選択します。
2. **HTTP (HTTP またはサーバ送信イベント)** を選択します。
3. サーバーの URL に、前述のオントロジー MCP サーバーのエンドポイントを入力します。
  `workspace-ID` と `ontology-item-ID` は取得したものに置き換えてください。
  > https://api.fabric.microsoft.com/v1/mcp/dataPlane/workspaces/{workspace-ID}/items/{ontology-item-ID}/ontologyEndpoint

  または  
  
  > https://agent365.svc.cloud.microsoft/agents/tenants/{tenant_id}/servers/mcp_FabricIQOntology/workspaces/{workspace-ID}/ontologies/{ontology-item-ID}
4. MCP ID を入力します。これは識別するための表示名なので、任意の名前を入力します。(`fabric-iq-ontology-its-asset` など)
5. インストール先で `グローバル` または `ワークスペース` のどちらかを選択します。特に何もなければグローバルを選択します。  
6. `mcp.json` ファイル画面が開き、新しく追加したオントロジー MCP の設定情報が確認できます。
  設定した MCP ID の値の上に表示されている **起動** を選択します。
7. Microsoft 認証の許可ダイアログが表示される場合は、**許可** を選択します。  
  Microsoft 認証のポップアップが表示されるので、Microsoft Fabric と同じアカウントでログインを行ってください。  
8. 認証が成功することを確認します。

設定がうまくいかない場合は、トラブルシュートの内容を確認してください。  


## 3. デモプロンプトを実行しオントロジーが使用されることを確認する

MCP クライアントの設定が完了したので、実際に MCP クライアントからオントロジーに自然言語で問い合わせを行ってみましょう。なお、オントロジー MCP を使用した回答の精度は、使用する AI エージェントの LLM によって変わる場合があります。  

### 3-1. スキーマ理解

※ `fabric-iq-ontology-its-asset` の部分は、作成した MCP ID に置き換えてください。  

```
fabric-iq-ontology-its-asset のオントロジーに接続して、エンティティ型・主要プロパティ・関係型の一覧を取得し、この会社がどんな企業資産をモデル化しているか要約して。
```

<u>**成功条件**</u>  

エンティティ型が 17 個、関係型が 19 個取得され、人材・案件・顧客・知識といった資産の分類についての説明が返ってくること。  

<u>**確認ポイント**</u>  

- AI エージェントがオントロジーの語彙を取得し、実データの Delta テーブル名ではなく、`Person` や `Project` といった業務概念で話ができること

### 3-2. 要員探索

```
製造業のお客様向けの生成AI案件を経験していて、Microsoft Fabric または Azure Databricks のスキルを持ち、2026年9月に30%以上の空き工数があるメンバーを探して。
名前・所属・役職・保有スキルのレベル・根拠になった案件・9月の空き工数を表にまとめて。
```

<u>**成功条件**</u>  

_3 名_ に絞り込まれ、それぞれに根拠案件と空き工数がついていること。  
検索結果が 1 名だけや、十数名のもので返ってくる場合は、LLM 側の推論により、どこかの条件が効いていない可能性があります。「製造業の顧客に限定してしるか」「案件の領域が AI のものなっているか」と、条件を 1 つずつ確認してみてください。  

<u>**確認ポイント**</u>  

- `Customer.Industry` → `Project` → `Assignment` → `Person` → `PersonSkill` → `Availability` と、5つの関係をまたいで初めて答えが出ているか

#### 追い込み質問（レベルと根拠）

```
その中で、Microsoft Fabric または Azure Databricks のレベルが上級以上の人だけに絞って。
絞られた人の Microsoft Fabric のスキルは、何を根拠にそう言えるのかを説明して。
```

出力された情報を従業員 (Person) 情報をベースに、`PersonSkill` の根拠案件と証拠種別をたどり、完了済みの案件実績から適切な要因絞り込みを行います。  

<u>**成功条件**</u>  

絞り込みの結果、_1 名 (松本 麻衣)_ さんのみになります。

### 3-3. 提案準備（類似案件と有識者検索）

```
大和精密工業様から、技能継承のためのナレッジ検索の仕組みを、設計ナレッジまで広げて全社に展開したい、と相談を受けました。提案の準備を手伝って。
進め方としては、次の4つを順番確認して。それぞれ「全部で何件あるか」を先に教えてから、中身を出して。

1. 他のお客様で、似たテーマの案件が過去にどれだけあるか。
  お客様名・案件名・いまどうなっているか（完了／進行中／これから）を一覧にして。
  まだ始まっていない案件は、実績とは分けて「参考」として扱うこと。

2. 1で挙げた案件のうち、完了と進行中のものすべてについて、そこで作られた資料が全部で何件あるか。
  他の案件でも使い回せそうなものを、使い回しやすさの評価とあわせて挙げて。使いにくいものは件数だけで構わない。

3. 同じ案件について、現場が書き残した気づき・反省点・うまくいったやり方が全部で何件あるか。
  そのうえで、特に参考になるものを挙げて。

4. 同じ案件について、責任者や技術リーダーを務めた人が全部で何人いるか。
  そのうえで、氏名・役職・どの案件で何を担当したかを表にして。

最後に、提案準備として最初にやるべきことをまとめて。
```

<u>**成功条件**</u>  

プロンプトで提示した 4 点についての回答が得られており、オントロジー経由で取得した情報が根拠として付加されていること。  

| # | 求めたもの | 回答 |
| --- | --- | --- |
| 1 | 他のお客様での類似案件 | `8 件`（実績: 7 件、参考: 1 件） |
| 2 | 完了・進行中案件で作成された資料 | `20 件`（高: 5 件、中: 9 件、低: 6 件） |
| 3 | 登録済みの気づき・反省点・成功パターン | `21 件`（BestPractice: 9 件、LessonsLearned: 7 件、Issue: 5 件） |
| 4 | 責任者・技術リーダー | `12 名`（担当実績: 14 件、複数案件の担当者: 2 名） |
| 5 | 提案の初動プラン | 提案に向けたプランが説明されていること |

案件テーマとしては、「技能継承ナレッジ検索」「設計レビュー支援」「保全ナレッジ検索」の 3 種類があります。案件名にこれらが含まれていることを確認します。
相談内容に「全社展開したい」と書いてあるので、生成 AI プラットフォームの全社展開案件（北斗重工業・信州電子工業）を含めてくるのは、オントロジーを使用するメリットを得られている良い読み取りになります。  

<u>**確認ポイント**</u>  

- `Project` → `Deliverable` / `Knowledge` / `Person` のリレーションを辿り、「誰に聞けばよいか」という暗黙知が構造化されて出力されるか
- 4 つの依頼内容それぞれが適切に集計されているか (冒頭の依頼で検索範囲の母数を固定していないか)
- これから始まる案件を実績として扱っていないか

#### 追い込み質問（相談先の提案）

```
いま挙げた類似案件それぞれについて、PM・アーキテクト・テックリードを務めた人を一覧で出して。
そのうえで、2件以上に名前が出てくる人を教えて。
案件が完了済みか進行中かも添えること。
```

先に一覧を出させてから絞り込みを行うようにしているのには、理由があります。いきなり「複数案件でリードを務めたことは誰か」と聞くと、直前の回答で名前を挙げた人の中からしか探さない場合があります。  
案件ベースでの一覧を経由させることで、もれなく情報をオントロジー経由で取得することができます。  

<u>**成功条件**</u>  

_2 名（佐藤 光・後藤 陽子）_ の回答が得られること。
2 名とも、完了案件と進行中案件の両方でリードを務めています。過去の実績を語りつつ、かつ今の現場感も持ち合わせている、という並びです。  
場合によっては、`坂本 直樹` も回答に含まれる場合があります。ただし、彼の持つ陸奥製鋼の案件は計画中でありまだ開始されていないので、実績として弱いことが添えられていれば問題ありません。

### 3-4. チーム編成（Opportunityベース）

```
信州電子工業様の「生産データ分析ツールの内製化支援」について、必要スキルと2026年10月に空き工数30%以上ある人を確認し、3〜5名のチーム案を作って。

まず必要スキルごとに、保有者数と条件を満たす候補者の全人数を出してから人選して。
候補者が0名のスキルは、無理に埋めず「不足スキル」として教えて。

各メンバーの推薦理由、スキルの根拠、10月の空き工数も付けて。
```

**「空き工数が30%以上」という条件を絶対に消さないでください。**   
Python 保有者 27 名の 10 月の空き工数は、26 名が 1〜29%、1 名が 0% という分布です。しきい値を書かずに「稼働を確保できる人」とだけ問うと、1〜29% の人が候補に含まれ、`不足スキル：なし` との回答になってしまいます。  
これは、エージェントの誤りではなく、条件が曖昧な点から起こりうる内容となります。「空いている人」ではなく「何 % 以上空いている人」と **数値で問う** のは、自社データ活用でも大切になる作法です。  

<u>**成功条件**</u>  

チームメンバーの選定が行われ、Python が「不足スキル」として回答されること。(チームメンバーの内容は実行ごとに変わる可能性があります)  

| 必要スキル | 保有者 | 2026 年 10 月に 30% 以上空いている人 |
| --- | --- | --- |
| Python | 27名 | **0名** ← 不足 |
| データモデリング | 70名 | 12名 |
| 製造業務知識(生産管理) | 42名 | 9名 |

この表の右端の列は、最終的にチームへ推薦された人数ではなく、各スキルについて `2026年10月` かつ `AvailablePercent >= 30` を満たす候補者の**全件数**です。チームはこの候補者全体から 3〜5 名に絞り込みます。そのため、回答でデータモデリングが 5 名、製造業務知識が 2 名などと示された場合、推薦メンバーとして挙げた人数が正しくても、候補者全体の取得または集計に漏れがある可能性があります。  

<u>**確認ポイント**</u>  

- 足りないものを「無し」と正直に回答されているか
  オントロジーは不足の可視化にもつながります。
- 必要スキルごとの候補者を全件取得したうえで、チームメンバーを 3〜5 名に絞り込んでいるか
- 推薦メンバー全員について、2026 年 10 月の空き工数が 30% 以上であることを確認できるか

推薦メンバーの組み合わせは実行ごとに変わっても問題ありません。ただし、選ばれた全員が商談のクローズ予定月である `2026年10月` に空き工数 30% 以上であることを確認します。また、スキルの証拠種別が `案件実績` か `自己申告` かも推薦理由に含めると、単にレベルが高い人を選ぶだけでなく、経験の確からしさを踏まえた人選になります。自己申告の候補者を選ぶこと自体は誤りではありませんが、同程度の条件で案件実績のある候補者がいる場合は、どちらを優先したかが説明されていることが望ましいです。  

#### 追い込み質問（将来のメンバーアサイン可否）

```
Python の要員はいつなら確保できますか。
2026年10月から2027年1月まで、月ごとに空き工数30%以上の人が何名いるか教えて。
結果を踏まえて、商談時期の調整や他案件からのどの要員を融通するかについて検討したい。
```

<u>**確認ポイント**</u>  

- 月ごとの人数が返ってくるか
  - 10月: 0 名
  - 11月: 0 名
  - 12月: 7 名
  - 1月: 16 名
- 要員状況を踏まえて、提案時期を動かすべきか、などの提案を行っているか

「10月クローズは要員的に無理。12月まで待つか、Python だけ他部門から融通するか」という、**提案時期そのものを動かす判断** についても、オントロジーを踏まえて AI エージェントが判断することができます。  

### 3-5. ナレッジ継承・リスク把握

```
生成AIの案件で、うまくいかなかったことの記録を横断で見て、繰り返し起きている失敗のパターンを上位3つ挙げて。
それぞれについて、何件あるか、何人が書いたものか、実際の記述例、そして次の提案書にリスクとして書くなら何と書くか、まで出して。

件数と人数は推測せず、該当する記録を実際に並べて数えること。
```

<u>**成功条件**</u>  

以下の 3 点が上位に上がること。  

| 失敗パターン | 件数 | 登録者 |
|---|---|---|
| 顧客側データオーナーとの合意形成が長期化した | **12件** | 11名 |
| 基幹システムのIF仕様が古く、結合テストで手戻りが出た | **11件** | 11名 |
| 現場用語の辞書整備が後手に回った | **11件** | 11名 |

<u>**確認ポイント**</u>  

- 「生成 AI の案件」「うまくいかなかったこと」という業務の言葉だけで、AI エージェントが自分でオントロジーの該当箇所を辿ることができるか

オントロジーのメタデータに、「この値は AI 案件を表す」「この 2 種別は失敗を表す」「この項目で束ねると、再発回数が数えられる」と設定してあり、これがコンテキストとして読み込まれているかを確認します。

#### 追い込み質問（発生状況の確認）

```
その失敗は、昔の話ですか。それとも今も起きていますか。
記録された時期と、案件が完了済みか進行中かで教えて。
```

<u>**確認ポイント**</u>  

- 完了の件数と、進行中の件数がそれぞれの失敗パターンで出力されること
- 2024 ~ 2026 年の間で、年単位で比較した推察が出ていること

件数だけでなく、そのナレッジ登録の状況まで確認し、現在のリスクについて判断することができます。  

#### 追い込み質問（失敗の繰り返し有無）

```
その失敗は、同じお客様で繰り返し起きていますか。
お客様ごとに何件あるか教えて。
```

<u>**確認ポイント**</u>  

- 顧客ごとの失敗件数についての分析が出ているか
- 以下の顧客に対しての洞察が出ているか
  - 浜名川県庁
  - 陸奥製鋼株式会社
  - 信州電子工業株式会社

ある案件で一度組織で経験した失敗について、ほかの案件でも繰り返していないかの洞察を得ることができます。  

### 3-6. 経営視点

```
完了案件の利益率を、ソリューション領域別と契約種別別に集計して。
利益率が最も低い領域について、

1. その領域の案件で登録された失敗の教訓
2. その領域の案件に必要なスキルの保有状況と、2026年10月〜12月に空き工数が30%以上ある人の数

も合わせて見て、採用・育成・案件の選び方のどれで対応すべきか、示唆を3つ出して。
```

<u>**成功条件**</u>  

完了 `39 件` のうち、**AI 領域が最も低い (平均 15.7%)** という点が特定されていること。  
そのうえで、**その領域の教訓と、その領域に必要なスキルの稼働見通しの両方** が言及されていること。  

<u>**確認ポイント**</u>  

- 財務・知識・人材の情報を辿り、_なぜ利益率が低いのかを教訓ベースで説明し、どう手を打つかを人材データで裏付ける_ ところまで行えるか

単純な BI レポートでは「AI 領域が低い」で止まる分析も、オントロジーを利用することで分析の深さが変わることを確認します。

#### 追い込み質問（契約手法）

```
AI 領域の完了案件の中で、利益率が高いものと低いものを分けているのは何ですか。
開発手法別と案件規模別に集計して比べて。
```

<u>**確認ポイント**</u> 

- `AI 領域の案件における契約種別` が論点であることを認識できるか
  - 開発手法（ウォーターフォール: 11.5% / アジャイル: 15.7% / ハイブリッド: 18.4%）
  - 案件規模（3 億以上: 12.3% / 1.5 億未満: 21.6%）

単純な「AI 領域の利益率が低い」という話から、オントロジーの関係性をたどって推論を行うことで、**「どのように AI 領域の案件を受注するべきか」** という契約の仕方と進め方の問題にたどり着けるかを確認します。  

### 3-7. メタデータ効果

```
2026年9月に bandwidth のあるメンバーを、空き工数の多い順に10名教えて。
```

<u>**成功条件**</u>  

空き工数が高い順に 10 名の従業員情報が得られること。    

<u>**確認ポイント**</u>  

- `bandwidth` という言葉から、`Availability` のエンティティにたどり着くことができるか

類義語 (シノニム) を登録していることで、AI エージェントが Abailability = bandwidth という関連を辿ることができます。  

```
2026年9月に slack のあるメンバーを教えて。
```

<u>**確認ポイント**</u> 

- AI エージェントが推論過程で戸惑うか / Slack (製品名) と誤解するか

`slack` (余力の意) は類義語として登録していないため、推論の過程が増えます。  
類義語としてメタデータ登録することの優位性を確認できます。  

```
Azure Databricks の上級以上の使い手を、案件実績で裏付けられた人だけに絞って探して。
自己申告だけの人が何名除外されたかも教えて。
```

<u>**成功条件**</u>  

_16 名_ (エキスパート: 6 名、上級: 10 名) に絞られます。  

<u>**確認ポイント**</u> 

- Azure Databricks の経験保有者 35 名のうち、以下の絞り込みが行われているか
  - 上級者の絞り込み: 35 → 23 名
  - 自己申告か否か: 23 → 16 名

`PersonSkill.EvidenceType` プロパティの説明に「案件実績は実績に基づき、自己申告は本人の申告のみ」
という趣旨が英語で書いてあります。そのため、上記の絞り込みが行えるようになります。  

```
2026年9月の稼働状況を、すぐ動かせる人・調整すれば動かせる人・手一杯の人の3つに分けて、それぞれ何名いるか教えて。
```

<u>**成功条件**</u>  

**空きあり: 81 名、調整可: 49 名、逼迫: 70 名** の結果が回答されること。  

<u>**確認ポイント**</u>  

- `調整可` というワードを第二候補として扱っているか

`Availability.AvailabilityStatus` プロパティの説明に、`空きあり`（30% 以上）・`調整可`（11〜29 %）・`逼迫`（10% 以下）の意味をメタデータとして明示してあります。これがない場合、「調整可」というものは単なる文字列として扱われてしまうリスクがあります。  

## 4. まとめ

このラボでは、作成したオントロジーを MCP 経由で利用することで、人材・案件・顧客・商談・ナレッジ・財務といった複数の業務データを、共通の業務概念と関係性に基づいて横断的に探索できることを実際に体験しました。単一のテーブルやシステムを検索するだけでは難しい、「どの案件で、誰が、どのような経験を積み、その知識を次の提案や人員配置にどう生かせるか」といった問いにも、一連の文脈を保ったまま答えられることを確認しました。

オントロジーの価値は、データをまとめて検索できることだけではありません。関係性をたどることで、利益率の低下と過去の失敗、人材のスキルと将来の空き工数などを結び付け、事実の提示から原因の分析、次に取るべき行動の提案へと分析を深められます。これにより、オントロジーは検索基盤ではなく、提案準備、人員配置、リスク管理、経営判断を支援するための知識基盤として活用できます。

また、類義語、プロパティの意味、値の判定基準などのメタデータを整備することで、AI エージェントは利用者の自然な表現を適切な業務概念へ結び付けられます。一方で、「空き工数が 30% 以上」のような条件を明確に指定することも重要です。信頼できる回答を得るためには、オントロジー側の意味定義と、利用者側の問いの具体性の両方が必要になります。

MCP を利用すれば、同じオントロジーを複数の AI エージェントから共通して利用できます。利用するツールや LLM が変わっても、業務データの意味と関係性を共有できるため、組織に散在するデータや暗黙知を、継続的に意思決定へ生かすための土台となります。

この後の流れとしては、実際の問い合わせで回答が不安定になる箇所を確認しながら、関係性、説明、類義語、判定基準を改善していくことになります。オントロジーは一度作って終わりにせず、利用結果をもとに育て続けてください。AI エージェントが組織の業務をより正確に理解し、根拠のある提案を継続して行えるようになります。

以上で、本ラボは終了です。今回確認したオントロジーの活用方法をもとに、自社データを用いたユースケースの検討などにお役立ていただければ幸いです。  

## トラブルシュート

| 症状 | 原因として多いもの | 対処 |
|---|---|---|
| オントロジー MCP の URL もアカウント認証も適切だが認証がエラー | Microsoft Edge の認証プロファイルにより認証先テナントが誤認識されてしまう | Microsoft Edge の既定プロファイルを認証で使用するものに変更して認証を再実施 |
| 検索が失敗する | エンティティが見つからず search_ontology が失敗する | Microsoft Foundry Agent など別のツールを使用して再実施 |
| 検索が失敗する | Fabric compute capacity has exceeded its limit と出る | Fabric 容量を上位のものに切り替え |
| 検索結果が 0 件になる | オントロジーに紐づくグラフデータベースの更新ができていない | nb_03_build_ontology にてグラフデータベースの取り込みが正常に完了しているか確認する |

---

# Lab 03 - Ask the Agent

**Estimated time**: Approximately 45 minutes  
**Goal**: Confirm that you can ask the ontology six questions and receive answers that cannot be produced by searching a single system  

## 0. What You Will Do in This Lab

The ontology you have created can be used by external AI agents through an MCP endpoint.

In this lab, you will connect Visual Studio Code to the ontology as an MCP client and submit natural-language queries. By traversing relationships across information such as people, projects, customers, knowledge, and finance, you will confirm that it can produce answers and insights unavailable from any single data source.

You will also examine how metadata configured in the ontology—such as business terms, synonyms, and property descriptions—helps an AI agent interpret queries and formulate answers.

## 1. Check the MCP Server Endpoint

Microsoft Fabric ontologies support MCP (Model Context Protocol) and can interact with external AI agents through MCP as an MCP server.
In other words, a published ontology can be used consistently by various AI agents, including your organization's Copilot, Copilot Studio, GitHub Copilot, and Claude Code.

The endpoint URL format for an ontology MCP server in Microsoft Fabric is as follows.

> https://api.fabric.microsoft.com/v1/mcp/dataPlane/workspaces/{workspace-ID}/items/{ontology-item-ID}/ontologyEndpoint

In addition to the endpoint above, an ontology MCP server endpoint routed through Agent 365 Gateway is also available.

> https://agent365.svc.cloud.microsoft/agents/tenants/{tenant_id}/servers/mcp_FabricIQOntology/workspaces/{workspace-ID}/ontologies/{ontology-item-ID}

The `workspace-ID` and `ontology-item-ID` values are unique to each environment. Check these two ID values and use them to complete the endpoint URL.

1. Open the workspace you created, and then open `ont_its_asset`.
2. Copy the URL from the browser's address bar. It should have the following format.
  > https://app.fabric.microsoft.com/groups/{workspace-ID}/ontologies/{ontology-item-ID}?experience=fabric-developer
3. Record the GUID values in the `workspace-ID` and `ontology-item-ID` positions in Notepad or a similar application.

For the Agent 365 Gateway version, you will also need `tenant_Id`. This is the tenant ID value from Entra ID.

## 2. Configure the MCP Client

Configure an MCP client so that you can use the Microsoft Fabric ontology through MCP. In this lab, you will configure the MCP server in Visual Studio Code.

https://code.visualstudio.com/docs/agent-customization/mcp-servers

1. Press **Ctrl+Shift+P**, and select **MCP: Add Server**.
2. Select **HTTP (HTTP or Server-Sent Events)**.
3. For the server URL, enter the ontology MCP server endpoint described above.
  Replace `workspace-ID` and `ontology-item-ID` with the values you obtained.
  > https://api.fabric.microsoft.com/v1/mcp/dataPlane/workspaces/{workspace-ID}/items/{ontology-item-ID}/ontologyEndpoint

  Or

  > https://agent365.svc.cloud.microsoft/agents/tenants/{tenant_id}/servers/mcp_FabricIQOntology/workspaces/{workspace-ID}/ontologies/{ontology-item-ID}
4. Enter an MCP ID. This is a display name used for identification, so you can enter any name (for example, `fabric-iq-ontology-its-asset`).
5. For the installation location, select either `Global` or `Workspace`. Select Global unless you have a particular reason not to.
6. The `mcp.json` file opens, where you can review the configuration information for the ontology MCP server you just added.
  Select **Start**, displayed above the MCP ID value you configured.
7. If a Microsoft authentication permission dialog appears, select **Allow**.
  A Microsoft authentication pop-up appears. Sign in using the same account that you use for Microsoft Fabric.
8. Confirm that authentication succeeds.

If configuration is unsuccessful, review the troubleshooting section.


## 3. Run the Demo Prompts and Confirm That the Ontology Is Used

Now that the MCP client configuration is complete, query the ontology from the MCP client using natural language. Note that the accuracy of answers produced using ontology MCP may vary depending on the LLM used by the AI agent.

### 3-1. Schema Understanding

Note: Replace `fabric-iq-ontology-its-asset` with the MCP ID you created.

```
Connect to the fabric-iq-ontology-its-asset ontology, retrieve a list of entity types, key properties, and relationship types, and summarize what kinds of enterprise assets this company models.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

The response retrieves 17 entity types and 19 relationship types and explains the asset categories, such as people, projects, customers, and knowledge.

<u>**Validation points**</u>

- The AI agent retrieves the ontology vocabulary and can discuss business concepts such as `Person` and `Project`, rather than the names of the underlying Delta tables

### 3-2. Resource Search

```
Find members who have experience with generative AI projects for manufacturing customers, have Microsoft Fabric or Azure Databricks skills, and have at least 30% availability in September 2026.
Summarize their names, departments, job titles, skill levels, supporting projects, and September availability in a table.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

The results are narrowed down to _3 people_, each with a supporting project and availability.
If the search returns only one person or more than ten people, one of the conditions may not have been applied because of the LLM's reasoning. Check each condition one at a time, such as whether the results are limited to manufacturing customers and whether the project domain is AI.

<u>**Validation points**</u>

- The answer is produced only after traversing five relationships: `Customer.Industry` → `Project` → `Assignment` → `Person` → `PersonSkill` → `Availability`

#### Follow-up Question (Level and Evidence)

```
Of those people, narrow the results to only those whose Microsoft Fabric or Azure Databricks level is Advanced or higher.
Explain the evidence supporting the selected people's Microsoft Fabric skills.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

Based on the employee (`Person`) information in the output, trace the supporting project and evidence type in `PersonSkill`, and appropriately narrow the results using completed project experience.

<u>**Success criteria**</u>

The results are narrowed down to only _1 person (松本 麻衣)_.

### 3-3. Proposal Preparation (Similar Projects and Expert Search)

```
大和精密工業 has asked us about expanding its knowledge-search system for skills transfer to include design knowledge and rolling it out across the entire company. Help me prepare a proposal.
Proceed by checking the following four items in order. For each item, first tell me the total number of records, and then show the details.

1. How many projects with similar themes have there been for other customers?
  List the customer name, project name, and current status (completed/in progress/upcoming).
  Separate projects that have not yet started from actual experience and treat them as "reference" projects.

2. Among the projects listed in item 1, how many deliverables in total were created for all completed and in-progress projects?
  List those that appear reusable for other projects, together with an assessment of how easy they are to reuse. For those that are difficult to reuse, only the count is needed.

3. For the same projects, how many observations, lessons learned, and successful approaches were recorded by the people working in the field?
  Then list the ones that are especially useful.

4. For the same projects, how many people in total served as the person responsible or technical lead?
  Then provide a table with their names, job titles, and what they were responsible for on each project.

Finally, summarize the first actions we should take to prepare the proposal.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

The response addresses the four points presented in the prompt and includes supporting information obtained through the ontology.

| # | Requested item | Answer |
| --- | --- | --- |
| 1 | Similar projects for other customers | `8 projects` (actual experience: 7, reference: 1) |
| 2 | Deliverables created in completed and in-progress projects | `20 deliverables` (High: 5, Medium: 9, Low: 6) |
| 3 | Recorded observations, lessons learned, and successful patterns | `21 records` (BestPractice: 9, LessonsLearned: 7, Issue: 5) |
| 4 | Persons responsible and technical leads | `12 people` (role assignments: 14, people assigned to multiple projects: 2) |
| 5 | Initial proposal plan | A plan for preparing the proposal is explained |

There are three project themes: "技能継承ナレッジ検索", "設計レビュー支援", and "保全ナレッジ検索". Confirm that project names include these phrases.
Because the request says that the customer wants to "roll it out across the entire company," including enterprise-wide generative AI platform rollout projects (北斗重工業 and 信州電子工業) is a good interpretation that demonstrates the benefit of using the ontology.

<u>**Validation points**</u>

- Whether the response follows the `Project` → `Deliverable` / `Knowledge` / `Person` relationships and structures the tacit knowledge of "who should we ask?"
- Whether each of the four requested items is aggregated appropriately (rather than fixing the search population based on the initial request)
- Whether upcoming projects are excluded from actual experience

#### Follow-up Question (Suggested Contacts)

```
For each similar project just listed, provide a list of the people who served as PM, architect, or tech lead.
Then identify the people whose names appear on two or more projects.
Also include whether each project is completed or in progress.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

There is a reason for asking the agent to produce the list first and then narrow it down. If you immediately ask, "Who served as a lead on multiple projects?", the agent may search only among the people named in its preceding answer.
By first producing a project-based list, you can retrieve the information comprehensively through the ontology.

<u>**Success criteria**</u>

The response identifies _2 people (佐藤 光 and 後藤 陽子)_.
Both served as leads on completed and in-progress projects. This combination means they can discuss past experience while also retaining a current perspective from active work.
In some cases, `坂本 直樹` may also be included in the answer. However, his project for 陸奥製鋼 is planned and has not yet started, so this is acceptable if the response notes that it is weaker as evidence of actual experience.

### 3-4. Team Formation (Opportunity-Based)

```
For 信州電子工業's "生産データ分析ツールの内製化支援", check the required skills and the people with at least 30% availability in October 2026, and propose a team of 3–5 people.

Before selecting people, show the number of skill holders for each required skill and the total number of candidates who meet the conditions.
If a skill has zero candidates, do not force a match; report it as a "missing skill."

Include each member's recommendation rationale, evidence for their skills, and October availability.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

**Do not remove the "at least 30% availability" condition under any circumstances.**
Among the 27 Python skill holders, October availability is distributed as follows: 26 people have 1–29%, and 1 person has 0%. If you ask only for "people whose availability can be secured" without stating the threshold, people with 1–29% availability will be included as candidates, resulting in an answer of `Missing skills: None`.
This is not an agent error; it can occur because the condition is ambiguous. Asking numerically for "people with at least what percentage of availability," rather than merely "people who are available," is also an important practice when using your organization's own data.

<u>**Success criteria**</u>

Team members are selected, and Python is reported as a "missing skill." (The selected team members may vary from run to run.)

| Required skill | Skill holders | People with at least 30% availability in October 2026 |
| --- | --- | --- |
| Python | 27 people | **0 people** ← Missing |
| データモデリング | 70 people | 12 people |
| 製造業務知識(生産管理) | 42 people | 9 people |

The rightmost column of this table is not the number of people ultimately recommended for the team. It is the **total number** of candidates for each skill who meet both `October 2026` and `AvailablePercent >= 30`. The team is narrowed down to 3–5 people from this complete candidate pool. Therefore, if the response shows five people for データモデリング or two people for 製造業務知識, the number of recommended members may be correct, but some candidates may have been missed during retrieval or aggregation of the complete candidate pool.

<u>**Validation points**</u>

- Whether missing resources are honestly reported as "None"
  The ontology also helps make shortages visible.
- Whether all candidates for each required skill are retrieved before narrowing the team down to 3–5 members
- Whether every recommended member can be confirmed to have at least 30% availability in October 2026

The combination of recommended members may vary from run to run. However, confirm that everyone selected has at least 30% availability in `October 2026`, the opportunity's planned closing month. Also, including whether the skill evidence type is `案件実績` or `自己申告` in the recommendation rationale enables selection based on confidence in a person's experience, rather than simply choosing people with high levels. Selecting a self-reported candidate is not itself an error, but when another candidate with project experience meets otherwise comparable conditions, the response should preferably explain which candidate was prioritized.

#### Follow-up Question (Future Member Assignment Feasibility)

```
When can we secure people with Python skills?
From October 2026 through January 2027, tell me how many people have at least 30% availability in each month.
Based on the results, I want to consider adjusting the opportunity timing or deciding which people could be reassigned from other projects.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Validation points**</u>

- Whether the number of people for each month is returned
  - October: 0 people
  - November: 0 people
  - December: 7 people
  - January: 16 people
- Whether the response makes suggestions based on resource availability, such as whether the proposal timing should be changed

Based on the ontology, the AI agent can also make a decision that **changes the proposal timing itself**, such as: "Closing in October is not feasible from a staffing perspective. Either wait until December or source only the Python resources from another department."

### 3-5. Knowledge Transfer and Risk Assessment

```
Review records of what went wrong across generative AI projects and list the top three recurring failure patterns.
For each one, provide the number of records, the number of people who wrote them, examples of the actual descriptions, and how it should be written as a risk in the next proposal.

Do not estimate the numbers of records or people; enumerate and count the applicable records.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

The following three items appear at the top.

| Failure pattern | Records | Contributors |
|---|---|---|
| 顧客側データオーナーとの合意形成が長期化した | **12 records** | 11 people |
| 基幹システムのIF仕様が古く、結合テストで手戻りが出た | **11 records** | 11 people |
| 現場用語の辞書整備が後手に回った | **11 records** | 11 people |

<u>**Validation points**</u>

- Whether the AI agent can independently navigate to the relevant parts of the ontology using only business-language phrases such as "generative AI projects" and "what went wrong"

The ontology metadata is configured to indicate that "this value represents an AI project," "these two types represent failures," and "grouping by this field makes it possible to count recurrences." Confirm that this information is loaded as context.

#### Follow-up Question (Check When Failures Occurred)

```
Are those failures only from the past, or are they still occurring?
Tell me by the period when they were recorded and whether the projects are completed or in progress.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Validation points**</u>

- Whether the counts for completed and in-progress projects are output for each failure pattern
- Whether the response includes year-by-year comparative observations for 2024–2026

You can assess current risk by examining not only the counts but also the circumstances under which the knowledge was recorded.

#### Follow-up Question (Whether Failures Recur)

```
Are those failures recurring for the same customers?
Tell me how many records there are for each customer.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Validation points**</u>

- Whether an analysis of failure counts by customer is provided
- Whether insights are provided for the following customers
  - 浜名川県庁
  - 陸奥製鋼株式会社
  - 信州電子工業株式会社

You can gain insight into whether failures experienced once by the organization on one project have also recurred on other projects.

### 3-6. Management Perspective

```
Aggregate the profit margins of completed projects by solution domain and contract type.
For the domain with the lowest profit margin, also examine:

1. Lessons from failures recorded in projects in that domain
2. The availability of the skills required for projects in that domain, and the number of people with at least 30% availability from October through December 2026

Based on this information, provide three insights on whether we should respond through hiring, training, or project selection.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

Of the `39 completed projects`, the response identifies that **the AI domain is the lowest (average 15.7%)**.
It then discusses **both the lessons from that domain and the availability outlook for the skills required in that domain**.

<u>**Validation points**</u>

- Whether the response can trace financial, knowledge, and people information to _explain why the profit margin is low based on lessons learned, and then support recommended actions with workforce data_

Confirm that using the ontology changes the depth of analysis beyond a simple BI report that stops at "the AI domain is low."

#### Follow-up Question (Contracting Approach)

```
What separates the high-margin and low-margin completed projects in the AI domain?
Aggregate and compare them by development methodology and project size.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Validation points**</u>

- Whether the agent can recognize that `contract type for projects in the AI domain` is the key issue
  - Development methodology (Waterfall: 11.5% / Agile: 15.7% / Hybrid: 18.4%)
  - Project size (300 million or more: 12.3% / less than 150 million: 21.6%)

Starting from the simple observation that "profit margins in the AI domain are low," confirm whether following the ontology's relationships and reasoning can lead to the issue of contracting and delivery approach: **"How should we win projects in the AI domain?"**

### 3-7. Metadata Effects

```
Tell me the 10 members with bandwidth in September 2026, ordered by highest availability.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

Information for 10 employees is returned in descending order of availability.

<u>**Validation points**</u>

- Whether the word `bandwidth` leads the agent to the `Availability` entity

Because a synonym is registered, the AI agent can follow the association Abailability = bandwidth.

```
Tell me the members with slack in September 2026.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Validation points**</u>

- Whether the AI agent hesitates during its reasoning process or mistakes Slack for the product name

`slack` (meaning spare capacity) is not registered as a synonym, so the reasoning process requires additional steps.
This demonstrates the advantage of registering synonyms as metadata.

```
Find Azure Databricks users at Advanced level or higher, limited to people whose skills are supported by project experience.
Also tell me how many people were excluded because they had only self-reported experience.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

The results are narrowed down to _16 people_ (Expert: 6, Advanced: 10).

<u>**Validation points**</u>

- Whether the following filtering is applied to the 35 people with Azure Databricks experience
  - Advanced-user filtering: 35 → 23 people
  - Whether the evidence is self-reported: 23 → 16 people

The description of the `PersonSkill.EvidenceType` property states in English that project experience is based on actual experience, while self-reported experience is based only on the person's own report.
This enables the filtering above.

```
Divide the September 2026 staffing status into three groups—people who can start immediately, people who can start with adjustments, and people who are fully occupied—and tell me how many people are in each group.

When searching the ontology, use Japanese search terms and match the Japanese values stored in OneLake. Provide the final answer in English.
```

<u>**Success criteria**</u>

The response returns **Available: 81 people, Adjustable: 49 people, Fully occupied: 70 people**.

<u>**Validation points**</u>

- Whether the term `調整可` is treated as the second-choice category

The description of the `Availability.AvailabilityStatus` property explicitly defines the meanings of `空きあり` (30% or more), `調整可` (11–29%), and `逼迫` (10% or less) as metadata. Without this metadata, there is a risk that `調整可` would be treated as nothing more than a string.

## 4. Summary

In this lab, you experienced how using the ontology you created through MCP enables you to explore multiple kinds of business data—including people, projects, customers, opportunities, knowledge, and finance—across a common set of business concepts and relationships. You confirmed that it can answer questions that are difficult to address by searching a single table or system, such as "On which projects did each person gain what experience, and how can that knowledge be applied to the next proposal or staffing decision?" while preserving the full context.

The value of an ontology is not limited to searching consolidated data. By following relationships, you can connect factors such as declining profit margins and past failures, or workforce skills and future availability, deepening the analysis from presenting facts to identifying causes and recommending the next action. This allows the ontology to serve not merely as a search platform, but as a knowledge foundation supporting proposal preparation, staffing, risk management, and management decisions.

In addition, by maintaining metadata such as synonyms, property meanings, and criteria for interpreting values, the AI agent can connect users' natural expressions to the appropriate business concepts. At the same time, it is important to specify conditions clearly, such as "at least 30% availability." Reliable answers require both semantic definitions in the ontology and specificity in the user's questions.

With MCP, the same ontology can be used consistently by multiple AI agents. Even when the tools or LLMs change, the meaning and relationships of business data can be shared, providing a foundation for continuously applying data and tacit knowledge distributed throughout the organization to decision-making.

As a next step, identify where answers to real queries are unstable and improve the relationships, descriptions, synonyms, and evaluation criteria. Do not treat the ontology as finished after creating it once; continue refining it based on usage results. This will enable the AI agent to understand the organization's business more accurately and consistently make evidence-based recommendations.

This concludes the lab. We hope that the ontology usage methods explored here will help you consider use cases involving your own organization's data.

## Troubleshooting

| Symptom | Common cause | Resolution |
|---|---|---|
| Authentication fails even though the ontology MCP URL and account authentication are correct | The authentication target tenant is misidentified because of the Microsoft Edge authentication profile | Change the default Microsoft Edge profile to the one used for authentication, and authenticate again |
| Search fails | `search_ontology` fails because the entity cannot be found | Try again using another tool, such as Microsoft Foundry Agent |
| Search fails | The message `Fabric compute capacity has exceeded its limit` appears | Switch to a higher Fabric capacity |
| Search results contain 0 records | The graph database associated with the ontology has not been updated | In nb_03_build_ontology, confirm that graph database ingestion completed successfully |
