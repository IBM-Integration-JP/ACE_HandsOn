# 演習 7（オプション）REST API アプリケーションの開発

> [!NOTE]
> **注意：**
> この演習はオプションです。時間に余裕のある方のみ実施ください。
> 演習に当たっては、追加の構成が必要となりますので[演習4](../演習/4-ファイル連携とデータベース・アクセス.md)と[演習5](../演習/5-（オプション）追加演習の準備.md)を実施してから始めてください。

<a id="ex7"></a>

## 目次

- [7.1 REST APIの解説](#ex7.1)
- [7.2 OpenAPIについての解説](#ex7.2)
- [7.3 シナリオ](#ex7.3)
- [7.4 演習用ライブラリーの定義](#ex7.4)
- [7.5 REST APIアプリケーションの作成](#ex7.5)
- [7.6 操作の実装](#ex7.6)
- [7.7 デプロイ](#ex7.7)
- [7.8 REST APIアプリケーションのテスト](#ex7.8)

<a id="ex7.1"></a>

## 7.1 REST APIの解説

近年企業内のアプリケーションやデータをREST APIとして外部に公開し、外部の開発者から利用してもらおうとする試みが増えています。ACEは既存のアプリケーションやデータをREST APIとして簡単に公開することが可能です。この演習ではACEのREST APIアプリケーション機能を使ってデータベースの操作をAPIとして公開する方法を見ていきます。

演習に入る前に簡単にREST APIの概念を整理します。
REST APIを理解する上で重要な下記の概念について解説します。
下記の図はREST APIの構成を示しています。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/1.png)

APIは相対パスで識別される複数の「リソース」を持ちます。上の例の場合、customerdbというAPIには/customersという相対パスで識別される顧客の集合を表すリソースと、/customers/{customerId}という相対パスで識別される顧客個人を表す2つのリソースが含まれています。

各リソースは「操作」を持ち、対応するHTTPメソッドが関連付けられます。

例えば、顧客情報の更新を行う操作”updateCustomer”は/customers/{customerId}に対して、HTTPのPUT処理が関連付けられています。同様に顧客情報の削除を行う操作”deleteCustomer”に対しては、HTTPのDELETEメソッドが関連付けられています。

また、各操作にはHTTPのボディで渡される情報以外に「パラメーター」を定義することができます。

パラメーターはURLのパスの一部として値を渡す「パス・パラメーター」、URL末尾にkey=valueの形で組込む「クエリ・パラメーター」、HTTP要求のヘッダーに設定される「ヘッダー・パラメーター」等があります。
REST APIを利用した連携を行う際はAPIのリソースや操作、操作に対するパラメーターについてAPIのプロバイダーとコンシューマーの間で合意されている必要があります。

[戻る](#ex7)

<a id="ex7.2"></a>

## 7.2 OpenAPIについての解説

REST API は広く普及してきましたが、長い間そのインターフェース仕様を記述するための標準的な形式は存在しませんでした。近年では OpenAPI Specification（OAS） と呼ばれる仕様が、REST API のインターフェース定義の事実上の標準として急速に広がっています。

OpenAPI は、以前 Swagger と呼ばれていた仕様を発展させたもので、現在では業界全体で利用されている共通仕様です。OpenAPI 文書（YAML または JSON 形式）には、API のリソース、操作、パラメーター、リクエスト／レスポンスの形式などが定義されます。

OpenAPI 文書を基にして、さまざまなプログラミング言語向けの クライアントコード や サーバースタブ を自動生成することができ、API の開発やテストを効率化できます。また、Swagger UI や他のツールを用いて API ドキュメントを自動生成して可視化することも可能です。

### 7.2.1 OpenAPI 文書の例

以下は、この演習で利用する OpenAPI 3.0 形式の API 仕様の抜粋です。
OpenAPI 文書には、REST API の構造を理解するための次のような情報が記述されています。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/2.png)

① API の基本情報
- タイトル（例：Staff Table API）
- バージョン
- サーバー情報（例：http://localhost/staffinfo/v1 ）
- 利用されるプロトコル（HTTP/HTTPS）

② リソース（エンドポイント） 
- 相対パスで表されるリソース（例：/staff/{id}）

③ リソースに対する操作（Operation）
- 各リソースに対して、HTTP メソッド（GET、POST、PUT、DELETE など）が関連付けられる
- 操作には概要、説明、入力・出力の形式、レスポンスコードなどが定義される

④ API で使用されるデータ構造（Schema）  
- リクエストボディやレスポンスとして利用されるオブジェクトの定義（例：Staff オブジェクト、Error オブジェクト）

### 7.2.2　OpenAPI仕様と関連ツールについて

OpenAPI（旧 Swagger 仕様）は、REST API を記述するための **オープンな仕様（Spec）** であり、OpenAPI 自体はツールを提供していません。
しかし、OpenAPI/Swagger 仕様を扱うための外部ツールが広く利用されており、API の設計・可視化・テストを支援します。

代表的なツールとしては、下記のような Swagger プロジェクトやコミュニティが提供するツールがあります。

> **Swagger UI**  
> &nbsp;&nbsp;&nbsp;&nbsp;ドキュメントをブラウザ上で可視化し、API の呼び出しを直接テストできるツールです。 

> **Swagger Editor**  
> &nbsp;&nbsp;&nbsp;&nbsp;YAML/JSON 形式で API 仕様を記述しながら、同時にプレビューできるブラウザベースのエディタです。

> **Swagger Codegen / OpenAPI Generator**   
> &nbsp;&nbsp;&nbsp;&nbsp;OpenAPI ドキュメントから、各種プログラミング言語向けのクライアント SDK やサーバースタブを自動生成するツール群です。

IBM App Connect Enterprise（ACE）では、V12 以降で OpenAPI 3.0 のサポートが追加され、ACE 13 においても継続・強化されています。
ACE Toolkit では、OpenAPI 3.0 ドキュメントをインポートし、リソース・操作・モデルを含む REST API を自動生成できます。また、OpenAPI エディタを利用して API を新規作成することも可能です。
生成された REST API は、ACE のメッセージフロー（サブフロー）として実装することができ、ツールキットに統合された OpenAPI/REST API エディタから、各操作に対してサブフローを作成・編集できます。

[戻る](#ex7)

<a id="ex7.3"></a>

## 7.3 シナリオ

この演習では従業員を管理するデータベース・テーブル（STAFFテーブル）に対する操作をREST APIとして公開します。演習で実装する操作は、下記の2つです。
 
**テーブルに対するレコードの挿入**

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/3.png)

**テーブルからのデータの参照**  

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/4.png)

また、この演習ではバックエンド側のデータベース操作をGUIマッピングで行います。

### 7.3.1 アプリケーションの開発手順概要

今回構築するREST API連携アプリケーションの構築手順は下記の通りです。

  - 演習用ライブラリーの準備
  - REST APIアプリケーションの作成
  - 操作の実装
  - デプロイ
  - テスト

[戻る](#ex7)

<a id="ex7.4"></a>

## 7.4 演習用ライブラリーの定義

### 7.4.1 REST APIアプリケーションから参照するデータベース定義を含むライブラリーの準備

演習6を実施していない場合には [6.5．データベース定義の作成](../演習/6-（オプション）データベース連携シナリオ.md#ex6.5)の手順に従いデータベース定義を含むライブラリーを作成してください。

[戻る](#ex7)

<a id="ex7.5"></a>

## 7.5 REST APIアプリケーションの作成

REST APIアプリケーションはREST APIサービスの公開に特化した特殊なアプリケーションです。OpenAPI文書をインポートすることで作成できます。

### 7.5.1 REST APIアプリケーションの作成

① ツールキットの「アプリケーション開発」ビューより「新規…」→「REST API」を選択します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/5.png)

② 「名前」に”RESTDemo”を設定し、「REST APIドキュメントで定義されたリソースと操作をインポートする」を選択して、「次へ」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/6.png)

③ インポートするOpenAPI文書を指定します。「ファイル・システムから選択」を選択し、「ロケーション」に”C:\students\OpenAPI\Staff.json”を入力し、「次へ」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/7.png)

④ 下記の画面で「終了」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/8.png)

REST APIアプリケーションが作成され、REST API 記述文書が開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/9.png)

### 7.5.2 ライブラリー参照の設定

REST APIアプリケーションから参照するライブラリーを設定します。

① REST APIアプリケーション「RESTDemo」を右クリックし、「ライブラリー参照の管理」を選択します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/10.png)

② 「Lib_Staff」を選択し、「OK」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/11.png)

[戻る](#ex7)

<a id="ex7.6"></a>

## 7.6 操作の実装

REST APIアプリケーションでは操作はサブ・フローとして実装されます。
この演習で実装する操作は下記の二つです。

| 操作名           | リソース・パス        | HTTPメソッド | パラメーター                           | 説明             |
|-----------------|----------------------|-------------|---------------------------------------|------------------|
| createStaffData | /staff/createStaff   | POST        | なし。Body に JSON で従業員情報を設定   | 従業員情報の作成   |
| getStaffData    | /staff/{id}          | GET         | パス・パラメーターとして従業員 ID を設定 | 従業員情報の参照   |

### 7.6.1 操作の実装（従業員情報の作成）

ここでは従業員情報の作成用API操作を実装していきます。

① API記述文書を開き、リソースとオペレーション「/staff/createStaff」配下のPOSTの「サブフローの作成」アイコンをクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/12.png)

サブ・フローが開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/13.png)

② 下記のとおり、Compute ノードと Mapping ノードを配置し、下記のようにワイヤリングを行います。Compute ノードには「generateStaffId」、Mapping ノードには「InsertRecord」という名称を設定します。

| 接続元ノード (Source Node) | 接続元端子 (Source Port)   | → | 接続先ノード (Target Node)   | 接続先端子 (Target Port)    |
|---------------------------|---------------------------|---|-----------------------------|----------------------------|
| Input                     | out                       | → | generateStaffId             | in                         |
| generateStaffId           | out                       | → | InsertRecord                | in                         |
| InsertRecord              | out                       | → | Output                      | in                         |

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/14.png)

③ プロパティー・ビューでComputeノードのプロパティーに下表の値を設定します。

|  タブ |      項目    |  値  |
|------|--------------|-------|
| 基本 | データ・ソース | mydb  |

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/15.png)

④ Computeノードをダブルクリックして、ESQLエディターを開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/16.png)

下記のようにESQLを編集します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/17.png)

ESQLのコードはC:\students\ESQL\createStaffData_Compute.esqlにも準備されていますので、時間のない方はコピーしてお使いください。

```JAVA
CREATE COMPUTE MODULE createStaffData_Compute
	CREATE FUNCTION Main() RETURNS BOOLEAN
	BEGIN
		-- CALL CopyMessageHeaders();
		CALL CopyEntireMessage();
		
		DECLARE rs ROW;
		DECLARE staffId INTEGER;
		-- Step 1: Execute PASSTHRU and store result into ROW variable
		SET rs.row[] = PASSTHRU('SELECT nextval(''myschema.staff_seq'') AS staff_id');		
		-- Step 2: Create a REFERENCE to the returned rowset
		DECLARE ref REFERENCE TO rs;	
		-- Step 3: Move to the first row
		MOVE ref FIRSTCHILD;		
		-- Step 4: Extract the integer value
		SET staffId = ref.staff_id;		
		-- Step 5: Set in the output JSON
		SET OutputRoot.JSON.Data.staffId = staffId;

		RETURN TRUE;
	END;
	
	CREATE PROCEDURE CopyMessageHeaders() BEGIN
		DECLARE I INTEGER 1;
		DECLARE J INTEGER;
		SET J = CARDINALITY(InputRoot.*[]);
		WHILE I < J DO
			SET OutputRoot.*[I] = InputRoot.*[I];
			SET I = I + 1;
		END WHILE;
	END;

	CREATE PROCEDURE CopyEntireMessage() BEGIN
		SET OutputRoot = InputRoot;
	END;
END MODULE;
```

⑤ Ctrl + s でComputeノードを保存します。

⑥ サブ・フローに戻り、「InsertRecord」をダブルクリックしてマッピング・エディターを開きます。マップではデータベースへの挿入処理を行います。 

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/18.png)

⑦ 以下の画面では、そのまま「終了」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/19.png)

⑧ 下記のようにData項目同士をマッピングします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/20.png)

⑨ PostgreSQLのテーブルにデータを挿入する処理を追加します。下記のようにマッピング・エディター上部の「データベース表へ行を挿入します」ボタンをクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/22.png)

⑩ 下記のように挿入先のテーブルのスキーマに「myschema」を、テーブル名に「staff」を設定し、「OK」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/23.png)

⑪ マップのターゲットにデータベース・テーブルが表示されるとともに、変換「Insert」がマップ上に追加されます。下記のようにマップ元の「Data」と、「Insert」をマップします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/24.png)

⑫ 下記のように「Insert」をクリックし、サブ・マップを開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/25.png)

⑬ サブ・マップで下記のようにStaffメッセージとテーブルのカラムをマッピングします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/26.png)

⑭ Ctrl + sで保存します。

### 7.6.2 操作の実装（従業員情報の参照）

ここでは従業員情報の参照用API操作を実装していきます。
従業員情報の参照用の操作はパス・パラメーターとして渡される”id”をキーにしてStaffテーブルから従業員情報を参照し、JSON形式で返します。

① RESTアプリケーションの「REST API 記述」文書を開き、リソース「/staff/{id}」以下の「GET」の「サブフローの作成」リンクを開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/27.png)

サブ・フローが開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/28.png)

② 下記のようにMappingノードを1つ配置してワイヤリングを行います。Mappingノードの名称は、”selectRecord”とします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/29.png)

このMappingノードでは受信したパス・パラメーターのidをキーにStaffテーブルを参照し、従業員情報を準備します。

③ Mappingノードをダブルクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/30.png)

④ 下記の画面で「メッセージ・フロー・ノードによって呼び出されるシンプルなメッセージ・マップ」を選択し、「次へ」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/31.png)

⑤ マップ元、マップ先に「RESTDemo」→「JSON タイプ」配下の「Staff」を選択し、「終了」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/32.png)

⑥ REST APIのパラメーターはLocalEnvironmentに設定されます。今回はパス・パラメーターとして渡される従業員情報のidパラメーターにアクセスする必要があるため、マップ元にLocalEnvironmentを追加します。マップ元のメッセージ・アセンブリーを選択し、プロパティービューより「ヘッダーとフォルダー」の「Properties」リンクをクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/33.png)

⑦ LocalEnvironmentを選択し、「OK」をクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/34.png)

⑧ マップ元のメッセージ・アセンブリーから「LocalEnvironment」→「REST」→「Input」→「Parameters」を展開し、「any」を右クリックして「ユーザー定義の追加」を選択します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/35.png)

⑨ 新たなエレメントが追加されますので名前を”id”に変更します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/36.png)

以上の操作でクライアントから渡されるパラメーターへのアクセスが行えるようになりました。

⑩ データベース連携のためのロジックを実装していきます。マッピング・エディター上部にある「データベースから行を選択します」ボタンをクリックします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/38.png)

⑪ 下記の画面で「SQL where 節」に設定されている”1=1”という式を削除します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/39.png)

⑫ クエリーを作成します。テーブル名に「staff」をチェックし、下記を順番にダブルクリックすることで「SQL where 節」を生成します。生成されたクエリーを確認して「OK」をクリックします。
- 表の列 : staffnum
- 演算子 : “=”
- 列の値に使用できる入力 : LocalEnvironment → REST → Input → Parameters → choice of cast item → id

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/40.png)

⑬ マップ先（右側）一番下にあるDataに対して、Selectの結果のをマップします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/41.png)

⑭ 「Select」をダブルクリックしてサブ・マップを開きます。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/42.png)

⑮ サブ・マップでは下記のようにテーブル内容をマップ先メッセージにマップします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/43.png)

⑯ ここまでの開発物をCtrl + sで保存します。

以上でフローの実装は完了です。

[戻る](#ex7)

<a id="ex7.7"></a>

## 7.7 デプロイ

ここからは作成したREST APIアプリケーションをデプロイ、テストしていきます。

### 7.7.1 REST APIアプリケーションをデプロイするための追加の設定

IBM App Connect EnterpriseがHTTPプロトコルを処理する際、2つの方式があります。

- 統合ノード・リスナー　・・・・統合ノード単位に構成され、内部のキュー経由でデータをフローに受け渡す。

- 統合サーバー・リスナー　・・・統合サーバー単位に構成され、オンメモリーでデータをフローに受け渡す。

REST APIアプリケーションを稼働させるには統合サーバー・リスナーを利用するように構成を変更する必要があります。

① 	ACEのコマンド・コンソールを開き、下記のコマンドを実行します。

```CMD
mqsichangeproperties ACE13NODE -b httplistener -o HTTPListener -n startListener -v false
```
```CMD
mqsistop ACE13NODE
```
```CMD
mqsichangeproperties ACE13NODE -f -e ACE13SERVER -o ExecutionGroup -n soapNodesUseEmbeddedListener -v true
```
```CMD
mqsichangeproperties ACE13NODE -f -e ACE13SERVER -o ExecutionGroup -n httpNodesUseEmbeddedListener -v true
```
```CMD
mqsistart ACE13NODE
```

### 7.7.2 REST アプリケーションのデプロイ

作成したREST APIアプリケーションをデプロイしていきます。
この演習ではBARファイルを明示的に作成しない、簡易的なデプロイ手順を使います。

① 「アプリケーション開発」ビューと「Integration Explorer」ビューを最前面に表示します。

② REST APIアプリケーション「RESTDemo」を、「Integration Explorer」ビューの「統合ノード」→「ACE13NODE」→「ACE13SERVER」へドラッグ＆ドロップし、アプリケーションをデプロイします。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/44.png)

③ デプロイ成功のポップアップが表示されたら「閉じる」ボタンで閉じます。

[戻る](#ex7)

<a id="ex7.8"></a>

## 7.8 REST APIアプリケーションのテスト

REST API のテストには、CURL コマンドを使用します。
CURL を使って、Windows のコマンドラインから HTTP プロトコルで RESTDemoAPI を呼び出します。

### 7.8.1 CURLを利用したREST APIアプリケーションのテストのための準備

ここでは、RESTDemo API アプリケーションをテストするための CURL コマンドを作成します。
作成する CURL コマンドは、従業員情報の作成用と従業員情報の取得用の2種類です。

① 従業員情報の作成用CURL

```CMD
curl -v --request POST ^
  --url http://localhost:7800/StaffInfo/v1/staff/createStaff ^
  --header "accept: application/json" ^
  --header "content-type: application/json" ^
  --data "{ \"staffId\": \"6854846747508736\", \"lastChanged\": \"2017-11-21T08:39:35.659Z\", \"firstName\": \"Zachary\", \"lastName\": \"Fuchs\", \"mail\": \"mokcuj@puronno.sl\" }"
```

②　従業員情報の取得用CURL

```CMD
curl -v --request GET ^
  --url http://localhost:7800/StaffInfo/v1/staff/REPLACE_ID ^
  --header "accept: application/json"
```

### 7.8.2 テストの実行

**createStaffのテスト**

① 従業員情報の作成用CURLコマンドをコマンドプロンプトにて実行します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/45.png)

② 要求コードが「200」となっていることを確認し、レスポンスボディに返ってきたJSONデータより、サーバー側で採番されたstaffIdを確認します。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/46.png)

**getStaffData操作のテスト**

① 従業員情報の取得用CURLコマンドをコマンドプロンプトにて実行します。

> 💡 **Note:**  
> `REPLACE_ID` に入力するのは、「従業員情報の作成用 CURL」実行時に返却された staffId です。  
> 下図の例では、該当する staffId は **【 48 】** です。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/47.png)

② 要求コードが「200」となっていることを確認し、レスポンスボディに返ってきたJSONデータを確認します。データが同じであれば、テストは完了です。

![スクリーンショット](../images/演習/7-（オプション）REST_API%20アプリケーションの開発/48.png)

以上で演習は終了です。
お疲れ様でした！

[戻る](#ex7)






























---
