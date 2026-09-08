# Lab 03 - エージェントに問いかける

**所要時間**：約 45 分
**ゴール**：オントロジーに対して6つの問いを投げ、単一システムの検索では出せない答えが返ってくることを確認している状態

## 0. このラボで行うこと


## 1. MCP サーバのエンドポイントを確認する

Microsoft Fabric のオントロジーは、MCP (Model Context Protocol) に対応しており、MCP サーバーとして外部の AI エージェントと MCP を通じて対話できます。  
つまり、公開されたオントロジーは、社内の Copilot をはじめ、Copilot Studio や GitHub Copilot、Claude Code など、様々な AI エージェントで、同一のオントロジーを扱うことができるということです。  

Microsoft Fabric における、オントロジー MCP サーバーのエンドポイント URL 形式は以下となっています。

> https://api.fabric.microsoft.com/v1/mcp/dataPlane/workspaces/<workspace-ID>/items/<ontology-item-ID>/ontologyEndpoint

`workspace-ID` および `ontology-item-ID` は、環境ごとに一意の値となります。この 2 つの ID 値を確認し、エンドポイント URL を完成させます。  

1. 作成済みのワークスペース画面を開き、`ont_its_asset` を開きます。 
2. ブラウザーのアドレスバーの URL を取得します。以下のような形になっているはずです。  
  > https://app.fabric.microsoft.com/groups/<workspace-ID>/ontologies/<ontology-item-ID>?experience=fabric-developer
3. `workspace-ID` と `ontology-item-ID` の位置にある値 (GUID) をメモ帳などに控えてください。  

## 2. MCP クライアントの設定を行う

Microsoft Fabric のオントロジーを MCP で利用するため、MCP クライアントの設定を行います。このラボでは、Visual Studio Code で MCP サーバーの設定を行います。

https://code.visualstudio.com/docs/agent-customization/mcp-servers

1. **Ctrl+Shift+P** を入力し、**MCP: Add Server** を選択します。
2. **HTTP (HTTP またはサーバ送信イベント)** を選択します。
3. サーバーの URL に、前述のオントロジー MCP サーバーのエンドポイントを入力します。
  `workspace-ID` と `ontology-item-ID` は取得したものに置き換えてください。
  > https://api.fabric.microsoft.com/v1/mcp/dataPlane/workspaces/<workspace-ID>/items/<ontology-item-ID>/ontologyEndpoint
4. MCP ID を入力します。これは識別するための表示名なので、任意の名前を入力します。(`fabric-iq-ontology-its-asset` など)
5. インストール先で `グローバル` または `ワークスペース` のどちらかを選択します。特に何もなければグローバルを選択します。  
6. `mcp.json` ファイル画面が開き、新しく追加したオントロジー MCP の設定情報が確認できます。
  設定した MCP ID の値の上に表示されている **起動** を選択します。
7. Microsoft 認証の許可ダイアログが表示される場合は、**許可** を選択します。  
  Microsoft 認証のポップアップが表示されるので、Microsoft Fabric と同じアカウントでログインを行ってください。  
8. 認証が成功することを確認します。

## 3. デモプロンプトを実行しオントロジーが使用されることを確認する

MCP クライアントの設定が完了したので、実際に MCP クライアントからオントロジーに自然言語で問い合わせを行ってみましょう。  

### 3-1. スキーマを理解できるか

**確認ポイント**  
AI エージェントがオントロジーの語彙を取得し、実データの Delta テーブル名ではなく、`Person` や `Project` といった業務概念で話ができること。

**成功条件**  
エンティティ型が 17 個、関係型が 19 個取得され、人材・案件・顧客・知識といった資産の分類についての説明が返ってくること。  


