# EdgeのIEモード設定手順

ブラウザの設定について以下に示します。Windows10、Microsoft Edge（64bit）バージョン 98.0.1108.62 の画面を使用して説明しますが、Windows11及び他のエディション、Microsoft Edge（64bit）及び他のMicrosoft Edge（32bit）も操作は同じになります。

## 手順１.　Edge用グループポリシーテンプレートをインストールする

グループポリシーで設定する場合、事前にEdge向けの「**グループポリシーテンプレート**」をMicrosoftのサイトから「**ポリシーファイル**」をダウンロードする。

1. ブラウザでMicrosoftの「[ビジネス向け Microsoft Edge のダウンロード](https://www.microsoft.com/ja-jp/edge/business/download)」ページを開く

2. 64bit版Windows OSの最新版Edgeが対象なら、「**Edge for Business最新のビルドとバージョンのダウンロード**」という見出しの下にある［**Windows 64-bitのポリシー**］をクリック、　これで「**MicrosoftEdgePolicyTemplates.cab**」というCABファイルがダウンロードできる

   ![image-20220303114151453](D:\project\soumu\Edge設定手順書.assets\image-20220303114151453.png)

3. ダウンロードしたCABファイルをエクスプローラー上でダブルクリックして、格納されているZIPファイル「**MicrosoftEdgePolicyTemplates.zip**」を解凍する。

   ![image-20220303115154955](D:\project\soumu\Edge設定手順書.assets\image-20220303115154955.png)

   

4. そのZIPファイルをエクスプローラーで開き、以下のように各ファイルをWindowsフォルダ以下にコピーする。

   - windows\admx\msedge*.admx → %windir%\PolicyDefinitionsフォルダへ

     ![image-20220303115313652](D:\project\soumu\Edge設定手順書.assets\image-20220303115313652.png)

     

   - windows\admx\ja-JP\msedge*.adml → %windir%\PolicyDefinitions\ja-JPフォルダへ

     ![image-20220303115417474](D:\project\soumu\Edge設定手順書.assets\image-20220303115417474.png)

   

   ## 手順２.　グループポリシーでIEモードを有効化する

   1. グループポリシーエディターを起動

      ［**Windows**］＋［**R**］キーで「**ファイル名を指定して実行する**」ダイアログを開き、「**gpedit.msc**」と記入して実行

      ![image-20220303151733161](D:\project\soumu\Edge設定手順書.assets\image-20220303151733161.png)

   2. 左ペインで［**コンピューターの構成**］－［**管理用テンプレート**］－［**Microsoft Edge**］を選択

   3. 右ペインで［**Internet Explorer統合を構成する**］をダブルクリック

   4. 開いたダイアログで［**有効**］を選ぶ

   5. 左下欄のプルダウンリストで［**Internet Explorerモード**］を選ぶ

   6. ［**OK**］ボタンをクリック

   ![image-20220303152533524](D:\project\soumu\Edge設定手順書.assets\image-20220303152533524.png)

   

   ## 手順３.　グループポリシーで「IEモード」のサイト一覧ファイルの保存先パスの設定

   1. グループポリシーエディターを起動

      ［**Windows**］＋［**R**］キーで「**ファイル名を指定して実行する**」ダイアログを開き、「**gpedit.msc**」と記入して実行

      ![image-20220303151733161](D:\project\soumu\Edge設定手順書.assets\image-20220303151733161.png)

   2. 起動したグループポリシーエディター（gpedit.msc）の左ペインで、［**コンピューターの構成**］－［**管理用テンプレート**］－［**Microsoft Edge**］（手順１をやらないと、該当設定アイテムがない）を選択

   3. 右ペインで［**エンタープライズモードサイトリストを構成する**］をダブルクリック

   4. 開いたダイアログで［**有効**］を選ぶ

   5. 左下欄のテキストボックスにサイト一覧ファイルのパスを入力

      統一管理考えて、サーバーのルートフォルダにサイト一覧ファイル（ie-site-list.xml）をアップロードしておく。

   6. ［**OK**］ボタンをクリック

   ![image-20220303152235268](D:\project\soumu\Edge設定手順書.assets\image-20220303152235268.png)

   　

## 手順４.　「IEモード」のサイト一覧ファイルの編集

1. xmlファイルを新規作成して、下記の通りに記入して、ie-site-list.xmlというファイル名を保存する。

   ```xml
   <site-list version="205">
       <!-- File creation header -->
       <created-by>
           <tool>EnterpriseSitelistManager</tool>
           <version>10240</version>
           <date-created>20150728.135021</date-created>
       </created-by>
   	<site url="https://ais.gogats.hq.admix.go.jp/">
   		<compat-mode>Default</compat-mode>
   		<open-in allow-redirect="true">IE11</open-in>
   	</site>
   	<site url="https://cpms.gogats.hq.admix.go.jp/">
   		<compat-mode>Default</compat-mode>
   		<open-in allow-redirect="true">IE11</open-in>
   	</site>
   </site-list>
   ```

   

2. １．で作成したie-site-list.xmlファイルをWebサーバーに（https://cpms.gogats.hq.admix.go.jp/ie-site-list.xml）アップロードします。

## 手順５.　Edgeの確認

1. Edgeを開く

2. アドレスバーにURL「edge://compat/」を入力する

3. 左側欄に「エンタープライズ モード サイト リスト」アイテムがあること

4. サイト一覧ファイルの場所は「https://cpms.gogats.hq.admix.go.jp/ie-site-list.xml」であること

5. 右下のドメイン欄に下記二つURLであること。

   ```
   https://ais.gogats.hq.admix.go.jp/
   
   https://cpms.gogats.hq.admix.go.jp/
   ```

![image-20220303155249510](D:\project\soumu\Edge設定手順書.assets\image-20220303155249510.png)



