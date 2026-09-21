_(English version follows below)_

# IT Services Enterprise Assets

架空の SIer `Contoso` 社が持つ、人材・スキル・案件・顧客情報などをオントロジーとして定義し、MCP エンドポイント経由で外部の AI エージェントにオントロジーを提供します。  
このハンズオンでは、オントロジーの作成に加え、オントロジーに紐づくグラフデータベースの利用と、MCP プロトコルによる AI エージェントとの連携について学ぶことができます。  

## 目次

- Lab.1
  1. レイクハウスの作成
  2. サンプルデータのアップロード
  3. Bronze/Silver データの作成
- Lab.2  
  1. オントロジーの作成
  2. MCP エンドポイントの設定
- Lab.3
  1. デモプロンプトの実行
  2. (オプション) データ更新とデモプロンプトの再実行

## 事前準備

このワークショップの全内容を行うためには、有償の Microsoft Fabric 容量が必要です。必ず、F2 以上の Fabric 容量以上が割り当てられたワークスペースを使用してください。  

### 試用版容量の利用

試用版容量を使用する場合、一部のコンテンツは実行できないことに注意してください。作成した Microsoft Fabric のオントロジーを MCP サーバーとして利用する場合は、有料の F2 以上の Fabric 容量が必須となります。   

試用版容量の制約については、以下を参照ください。  

https://learn.microsoft.com/ja-jp/fabric/fundamentals/fabric-trial#whats-includedand-whats-not

### テナント設定

Fabric でオントロジーの機能を利用するためには、テナント設定にて指定の項目を有効化する必要があります。

https://learn.microsoft.com/ja-jp/fabric/iq/ontology/overview-tenant-settings

[Fabric 管理者](https://learn.microsoft.com/ja-jp/fabric/admin/roles#power-platform-and-fabric-admin-roles) の Entra ID 管理者ロールが付与されたアカウントにて、下記項目の **有効化** を必ず行ってください。  

- テナント設定 -> Microsoft Fabric
  - ユーザーは Ontology アイテムを作成できます

下記の設定については本ワークショップでは不要です。ただし、ワークショップを通じて作成したオントロジーを使用して Data Agent および Operations Agent などの検証を行いたい場合は、必要に応じて有効化が必要です。  

https://learn.microsoft.com/ja-jp/fabric/data-science/data-agent-tenant-settings

- テナント設定 -> Copilot and AI
  - Users can use Copilot, AI Agents and other AI experiences powered by Azure OpenAI
  - 容量は Fabric Copilot として指定できます
  - Azure OpenAI に送信されたデータは、容量の地理的リージョン、コンプライアンス境界、または国内クラウド インスタンスの外部で処理できます
  - Azure OpenAI に送信されたデータは、容量の地理的リージョン、コンプライアンス境界、または国内クラウド インスタンスの外部に格納できます

## サンプルデータ

社員情報や顧客情報、案件情報や稼働情報など、計 18 個の CSV ファイルが存在します。各 CSV ファイルに含まれる人名や企業名、案件名などのデータは生成 AI によって作成された架空の情報であり、実在する企業などとの関係は一切ありません。  

サンプルデータは `/data` ディレクトリにあります。実運用のデータレイクを想定し、ソースシステム単位にディレクトリが分かれています。  
ディレクトリ名に使用されている製品名は、実在する製品の名称を用いていますが、ディレクトリ内に含まれるデータは前述のとおり、架空のものです。実際のシステムから出力されるデータとは異なります。  

### データ規模

サンプルデータは、オントロジー作成時に以下の規模となるよう想定の上、作成がされています。  

| エンティティ | 件数 | 補足 |
| :-- | :-- | :-- |
| 社員 | 200 名 | 7 部門・7 職位のピラミッド構成 |
| 顧客 | 29 社 | 製造 8 社を中心に 7 業界 |
| 案件 | 93 件 | 完了 39・進行中 39・計画中 15 |
| 案件アサイン | 520 件 | 稼働はこのアサインから計算される |
| 保有スキル | 1,066 件 | 案件実績による裏付けの有無を持つ |
| 月次稼働 | 1,200 件 | 200 名 × 6 か月（2026-08〜2027-01） |
| 教訓・ナレッジ | 279 件 | 同じ論点でも書き手により文面が異なる |
| 成果物 | 216 件 | 再利用価値の 3 段階評価つき |
| 商談 | 13 件 | 必要スキルと想定クローズ時期を持つ |

### データ取得元

サンプルデータの各種 CSV ファイルは、以下に示すシステムから必要な情報を取得していることを想定したものとなっています。各ファイル内の列情報や値については、ワークショップ向けに仮想的に作成されたものであり、現実のシステムから取得できるデータ形式と同一であることを保証しません。  

| データソース | 種別 | 取得できる情報 |
| :-- | :-- | :-- |
| Entra ID | IdP | サインイン識別子やアカウントの状態。 |
| SAP SuccessFactors | 人事データ (HCM) | 従業員に関する記録情報 (社員番号や組織情報、保有資格/スキルなど)。 |
| Dynamics 365 Project Operations | 案件管理 (PSA) | 要員の稼働状況や案件へのアサイン状況に関する情報。 |
| SharePoint Online | 文書管理 | 成果物の情報を保持。 |
| Salesforce | 顧客管理 (CRM) | 顧客情報や顧客担当者情報、商談内容など。 |
| Dataverse | マスタ | スキル体系・資格・技術のマスタ情報。 |
| Confluence | ナレッジ | 現場が書き溜める知識・教訓・ベストプラクティスなどを蓄積。 |
| ServiceNow | プロジェクトポートフォリオ管理 (PPM) | 案件内容や使用技術、必要スキルなどの情報。 |

本リポジトリにおけるサンプルデータは、MDM (マスタデータ管理) に関わる内容は割愛された値となっています。実際には、各種システムごとに発行される内部 ID などのデータを紐づける作業が追加で必要となる場合がありますが、コンテンツの本題ではないため、本ワークショップでは割愛します。  

## 注意

- オントロジー (Fabric IQ) については、随時内容が更新される場合があります。
- サンプルデータの人名、企業名、案件、金額、その他すべての情報は架空のものです。  

---

This scenario defines the people, skills, projects, customers, and other information of a fictional systems integrator named `Contoso` as an ontology, and makes the ontology available to external AI agents through an MCP endpoint.
In this hands-on workshop, you will learn how to create an ontology, use the graph database associated with it, and integrate AI agents through the MCP protocol.

## Table of contents

- Lab 1
  1. Create lakehouses
  2. Upload the sample data
  3. Create the Bronze and Silver data layers
- Lab 2
  1. Create the ontology
  2. Configure the MCP endpoint
- Lab 3
  1. Run the demonstration prompts
  2. (Optional) Update the data and rerun the demonstration prompts

## Prerequisites

A paid Microsoft Fabric capacity is required to complete all parts of this workshop. Make sure that you use a workspace assigned to an F2 or higher Fabric capacity.

### Using a trial capacity

Some content cannot be run with a trial capacity. A paid F2 or higher Fabric capacity is required to use the Microsoft Fabric ontology created in this workshop as an MCP server.

For information about trial capacity limitations, see:

https://learn.microsoft.com/en-us/fabric/fundamentals/fabric-trial#whats-includedand-whats-not

### Tenant settings

To use ontology features in Fabric, you must enable the relevant tenant setting.

https://learn.microsoft.com/en-us/fabric/iq/ontology/overview-tenant-settings

Using an account assigned the Entra ID administrator role for a [Fabric administrator](https://learn.microsoft.com/en-us/fabric/admin/roles#power-platform-and-fabric-admin-roles), make sure that you enable the following setting:

- Tenant settings -> Microsoft Fabric
  - Users can create Ontology items

The following settings aren't required for this workshop. However, if you want to use the ontology created during the workshop to evaluate Data Agent, Operations Agent, or similar features, enable them as needed.

https://learn.microsoft.com/en-us/fabric/data-science/data-agent-tenant-settings

- Tenant settings -> Copilot and AI
  - Users can use Copilot, AI Agents and other AI experiences powered by Azure OpenAI
  - Capacities can be designated as Fabric Copilot capacities
  - Data sent to Azure OpenAI can be processed outside your capacity's geographic region, compliance boundary, or national cloud instance
  - Data sent to Azure OpenAI can be stored outside your capacity's geographic region, compliance boundary, or national cloud instance

## Sample data

The sample contains 18 CSV files covering employees, customers, projects, availability, and other information. All names of people, companies, projects, and other data in these CSV files are fictional and generated using generative AI. They have no relationship to any real organization.

The sample data is located in the `/data` directory. To resemble a production data lake, its directories are separated by source system.
The directory names use the names of real products, but the data in those directories is fictional, as described above, and differs from data exported by the actual systems.

### Data volume

The sample data is designed to produce an ontology of approximately the following size.

| Entity | Count | Notes |
| :-- | :-- | :-- |
| Employees | 200 | Pyramid structure across seven departments and seven job levels |
| Customers | 29 | Seven industries, centered on eight manufacturing companies |
| Projects | 93 | 39 completed, 39 in progress, and 15 planned |
| Project assignments | 520 | Availability is calculated from these assignments |
| Employee skills | 1,066 | Includes whether each skill is supported by project experience |
| Monthly availability | 1,200 | 200 employees x six months (2026-08 through 2027-01) |
| Lessons and knowledge | 279 | Wording varies by author, even for the same topic |
| Deliverables | 216 | Includes a three-level reusability rating |
| Opportunities | 13 | Includes required skills and expected close dates |

### Data sources

The sample CSV files assume that the required information is obtained from the systems listed below. The columns and values in each file were created specifically for this workshop and aren't guaranteed to match the formats available from the actual systems.

| Data source | Type | Information provided |
| :-- | :-- | :-- |
| Entra ID | IdP | Sign-in identifiers and account status |
| SAP SuccessFactors | Human capital management (HCM) | Employee records, including employee IDs, organizational information, certifications, and skills |
| Dynamics 365 Project Operations | Professional services automation (PSA) | Employee availability and project assignment information |
| SharePoint Online | Document management | Deliverable information |
| Salesforce | Customer relationship management (CRM) | Customer information, customer contacts, and opportunities |
| Dataverse | Master data | Skill, certification, and technology master data |
| Confluence | Knowledge management | Knowledge, lessons learned, and best practices recorded by project teams |
| ServiceNow | Project portfolio management (PPM) | Project details, technologies used, and required skills |

The sample data in this repository omits master data management (MDM) considerations. In a real implementation, additional work might be required to associate internal IDs issued by the different systems. Because this isn't the main subject of the workshop, that process is outside its scope.

## Notes

- Ontology features in Fabric IQ can change as the service is updated.
- All names, companies, projects, monetary amounts, and other information in the sample data are fictional.
