# Microsoft Power Platform
- Power Apps
  Power Apps は、すべてのデバイスで実行する Web およびモバイル アプリケーションの作成を実現します。
- Power Automate
  ユーザーがアプリケーションとサービスの間に自動化されたワークフローを作成するのに役立ちます。 コミュニケーション、データの収集、意思決定の承認など、繰り返される業務プロセスの自動化を支援します。
- Power BI
  **データを分析する**ためのインサイトを提供するビジネス分析サービスです。
  **レポートとダッシュボード**を構成するデータの視覚化を通じてそれらの分析情報を共有できます。
- Power Virtual Agents
  強力なチャットボットを簡単に作成できます。

# データ コネクタ
コネクタは、データ ソースからアプリケーションまたはワークフローへのブリッジと見なすことができます。

## データ ソース
データ ソースの 2 つのタイプは、表形式と関数ベースです。

### 表形式のデータ
 例として、Microsoft Dataverse、SharePoint、および SQL Server があります。

### 関数ベースのデータ
関数は、データのテーブルを返すために使用できますが、メールの送信、アクセス許可の更新、またはカレンダー イベントの作成など、より広範囲のアクションを提供します。 例には、Office 365 Users、Project Online、Azure Blob Storage が含まれます。

## コネクタ
コネクタ は、アプリ、ワークフロー、またはダッシュボードにデータ ソースを接続するものです。
- 標準
- プレミアム

コネクタを使用すると、データ ソースと Power Platform の間の入出力が可能になります。

たとえば、Customer Service などの Dynamics 365 アプリを使用すると、特定の顧客タイプが追加されたときに**ユーザーに通知**するように Power Automate を設定できます。 また、SharePoint **ドキュメント ライブラリ**を使用して、Power Apps に取り込むファイルを格納し、これらを管理および配布することもできます。 また、Microsoft では、Azure サービスへのコネクタを提供し、**画像内のテキストの読み取り**などのタスクや、**画像内の顔認識**などの認知サービスを実行するための高度な **AI 技術**をもたらします。

すべての Microsoft Power Platform ビジネス ソリューションは、Teams などの Microsoft 365 アプリで使用および実装できます。**これにより、ユーザーは Teams 内で Power Apps を使用したり、Teams 内のアクションやイベントから Power Automate を実行したりできます。**

### トリガーとアクション
データ ソースを設定してコネクタを構成したら、トリガーまたはアクションの 2 つのタイプの操作を使用できます。
- トリガー
   Power Automate でのみ使用され、開始するフローを要求します。 トリガーには時間を基準とするものにできます。たとえば、毎日午前 8 時で始まるフローや、テーブルに新しい行を作成したり、メールを受信するなどのアクションに基づいたトリガーを実行することもできます。 ワークフローをいつ実行するかを指定するには、必ずトリガーを作成する必要があります。
- アクション
  Power Automate および Power Apps で使用されます。 アクションはユーザーまたはトリガーから要求され、データ ソースとの対話を何らかの関数によって行うことができます。 たとえば、アクションとしてワークフローまたはアプリにメールを送信したり、データ ソースに新しい行を書き込んだりすることができます。

## カスタム コネクタ
Microsoft Power Platform には 200 以上のコネクタが用意されていますが、カスタム コネクタを作成することもできます。 一般に利用可能な API や、Azure などのクラウド プロバイダーでホストしているカスタム API を呼び出すことによって、アプリを拡張することができます。

### カスタム コネクタの作成
カスタム コネクタを作成するには 3 種類の方法があります。

https://docs.microsoft.com/ja-jp/learn/modules/introduction-power-platform/3-data-connectors

- 空のカスタム コネクタの使用

- OpenAPI 定義から

- Postman コレクションから

# データ損失防止、コンプライアンス、プライバシー、およびアクセシビリティ
https://docs.microsoft.com/ja-jp/learn/modules/introduction-power-platform/3a-data-loss-prevention-compliance-privacy-accessibility

## データ損失防止ポリシー
データ損失防止 (DLP) ポリシーを作成すると、ユーザーが誤って組織データを露出してしまうことを防止するためのガートレールとしてポリシーを機能させることができます。

## データ保護
顧客と Microsoft データセンターの間で確立される接続は暗号化され、すべてのパブリック エンドポイントは業界標準の TLS を使用して保護されます。
現時点では、サーバーのエンドポイントにアクセスするには TLS 1.2 (またはそれ以上) が必要です。

# アクセシビリティ
https://docs.microsoft.com/ja-jp/powerapps/maker/canvas-apps/accessibility-checker

# まとめ
Microsoft Power Platform は、分析、活動、自動化を支援することで、ビジネスに付加価値を与えます。 Power Apps でのカスタム アプリの構築、Power Automate で収集したデータに基づくプロセスの自動化、および Power BI で収集したデータの分析を行います。

## Microsoft Power Platform は、3 つの重要なアクション
1. データからインサイトを得て
2. ユーザーが構築するアプリを介してインテリジェントな業務プロセスを推進 (Act) し
3. ビジネス プロセスを自動化 (Automate) します。

## Power BI では、
内部および外部のリソースから取得したデータを利用して、統合型プラットフォームでデータの分析と視覚化ができます。

## Power Apps では、
埋め込み、またはスタンドアロンとに関わらず、あらゆるデバイスで動作する Web およびモバイルのカスタム アプリを構築し、展開できます。

## コネクタは、
データ ソースからアプリケーションやワークフローに情報を送信することができるブリッジです。

## power Automate では、
簡単なシナリオから高度なシナリオまで自動化ワークフローを作成できます。

# リソース

## Power BI

Power BI https://powerbi.microsoft.com/

Power BI顧客ショーケース https://powerbi.microsoft.com/customer-showcase/

## Power Apps 

Power Apps https://powerapps.microsoft.com/

Power Apps リソース  https://powerapps.microsoft.com/blog/microsoft-powerapps-learning-resources/

## Power Automate

Power Automate  https://flow.microsoft.com/

Power Automate のドキュメント https://docs.microsoft.com/ja-jp/flow/

## コネクタの詳細

コネクタ参照 https://docs.microsoft.com/ja-jp/connectors/

Power Appsのキャンバス アプリ コネクタの概要 https://docs.microsoft.com/ja-jp/powerapps/maker/canvas-apps/connections-list

## カスタム コネクタの基本

空のカスタム コネクタの使用 https://docs.microsoft.com/ja-jp/connectors/custom-connectors/define-blank

OpenAPI 定義から https://docs.microsoft.com/ja-jp/connectors/custom-connectors/define-openapi-definition

Postman コレクションから https://docs.microsoft.com/ja-jp/connectors/custom-connectors/define-postman-collection

PowerApps のキャンパス アプリでカスタム コネクタを使用する
https://docs.microsoft.com/ja-jp/learn/modules/use-custom-connectors-in-powerapps-canvas-app/

## Power Virtual Agents に関する入門情報

Power Virtual Agents https://powervirtualagents.microsoft.com/