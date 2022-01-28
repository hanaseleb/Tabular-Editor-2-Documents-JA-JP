はじめに
2021-09-21
ダニエル・オティキエル
インストール方法
リリースページから.msiファイルをダウンロードし、.msiをインストールしてください。

前提条件
特にありません。

注意事項
Tabular EditorはTabular Object Modelを使用して、Model.bimファイルや既存のデータベースとの間でメタデータのロードと保存を行います。これは、.msiインストーラーに含まれています。Analysis Services Client Librariesについては、Microsoftの公式ドキュメントを参照してください。

システム要件
オペレーティングシステムです。Windows 7、Windows 8、Windows 10、Windows Server 2016、Windows Server 2019 またはそれ以降のバージョン
.NETフレームワーク。4.6
タブラエディタでの作業
推奨されるワークフローは、通常通りSSDTを使ってテーブルとリレーションシップを設定し、あとはTabular Editorを使って作業を行うことです。つまり、以下のことです。計算列、メジャー、階層、パースペクティブ、トランスレーション、表示フォルダなど、思いつく限りの微調整を行います。

Model.bimファイルをロードするには、FileメニューのOpen > From File...オプション（CTRL+O）を選択するか、Analysis ServicesのインスタンスからOpen > From DB...オプションを選択して既存のデータベースを開きます。後者の場合は、サーバー名とオプションの認証情報の入力が求められます。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/Connect.png

これは、新しいAzure Analysis Services PaaSでも機能します。ローカルインスタンス」ドロップダウンを使用して、Power BI DesktopやVisual Studio Integrated Workspacesの実行中のインスタンスを参照して接続することができます。なお、Tabular EditorはTOMを介してPower BIモデルに変更を加えることができますが、Microsoftがサポートしているすべてのモデリング操作ではありません。その他の情報

OK」をクリックすると、サーバー上のデータベースの一覧が表示されます。

これは、Tabular Editorにモデルが読み込まれた後のUIの様子です。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/Main%20UI.png

画面の左側のツリーには、Tabular Modelのすべてのテーブルが表示されます。テーブルを展開すると、そのテーブル内のすべての列、メジャー、階層が表示フォルダごとにグループ化されて表示されます。ツリーのすぐ上にあるボタンを使用すると、表示フォルダ、隠しオブジェクト、特定のタイプのオブジェクト、名前によるオブジェクトのフィルタリングを切り替えることができます。ツリー内の任意の場所を右クリックすると、新しいメジャーの追加、オブジェクトの非表示、オブジェクトの複製、オブジェクトの削除などの一般的な操作を行うコンテキストメニューが表示されます。F2を押すと、現在選択されているオブジェクトの名前を変更できます。また、複数選択して右クリックすると、複数のオブジェクトの名前を一括して変更できます。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/BatchRename.png

メイン UI の右上には DAX エディタがあり、モデル内の任意のメジャーや計算列の DAX 式の編集に使用できます。DAX Formatter」ボタンをクリックすると、www.daxformatter.com を通じてコードが自動的にフォーマットされます。

右下のプロパティ・グリッドを使用して、フォーマット文字列や説明、トランスレーションやパースペクティブ・メンバーシップなどのオブジェクトのプロパティを調べたり設定したりすることができます。ここでは「表示フォルダ」のプロパティを設定することもできますが、ツリー内のオブジェクトをドラッグ・アンド・ドロップするだけで「表示フォルダ」を更新することができます（CTRLまたはSHIFTを使って複数のオブジェクトを選択してみてください）。

パースペクティブやトランスレーション（カルチャ）を編集するには、ツリーで「モデル」オブジェクトを選択し、プロパティグリッドで「モデルパースペクティブ」または「モデルカルチャ」プロパティを探します。小さなエリプシスボタンをクリックすると、パースペクティブやカルチャを追加／削除／編集するためのコレクションエディタが開きます。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/Edit%20Perspectives.png

変更内容をModel.bimファイルに保存するには、保存ボタンをクリックするか、CTRL+Sを押します。既存の Tabular データベースを開いていた場合は、変更内容が直接データベースに保存されます。データベースをTabular Editorに読み込んだ後に変更されたかどうかを確認するプロンプトが表示されます。CTRL+Zを押すと、いつでも変更を取り消すことができます。

モデルを別の場所にデプロイしたい場合は、「モデル」メニューで「デプロイ」を選択します。

デプロイメント
Tabular Editorにはデプロイメントウィザードが付属しており、SSDTからのデプロイと比較していくつかの利点があります。デプロイ先のサーバとデータベースを選択した後、デプロイに必要な以下のオプションがあります。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/Deployment.png

Deploy Connections "ボックスをオフにしておくと、ターゲットデータベース上のすべてのデータソースには手が加えられません。モデルに、ターゲットデータベースに存在しないデータソースを持つテーブルが1つ以上含まれていると、エラーが発生します。

同様に、「テーブルパーティションのデプロイ」を省略すると、テーブル上の既存のパーティションが変更されず、パーティション内のデータがそのまま残るようになります。

Deploy Roles」にチェックを入れると、ターゲットデータベースのロールはロードされたモデルにあるものを反映して更新されますが、「Deploy Role Members」にチェックを入れないと、各ロールのメンバーはターゲットデータベースでは変更されません。

コマンドラインの使い方
自動デプロイメントには、コマンドラインを使用することができます。GUIで利用可能なすべてのデプロイメントオプションは、コマンドラインでも利用可能です。

デプロイメントの例
TabularEditor.exe c:̫⃝_Projects\Model.bim

Tabular EditorのGUIを開き、指定されたModel.bimファイルをロードします（デプロイは行いません）。

TabularEditor.exe c:Projects\Model.bim -deploy localhost AdventureWorks

指定したModel.bimファイルをlocalhostで動作するSSASインスタンスにデプロイし、AdventureWorksデータベースを上書きまたは作成します。GUIはロードされません。

デフォルトでは、パーティション、データソース、ロールはターゲットデータベースに上書きされません。この動作は、上記のコマンドに以下のスイッチを1つ以上追加することで変更できます。

-P パーティションを上書きする
-C 接続（データソース）を上書きします。
-R ロールを上書きする
-M ロールメンバーを上書き
コマンドラインオプションの詳細については、こちらを参照してください。

注意
TabularEditor.exeはWindowsフォームアプリケーションなので、コマンドラインから実行すると、アプリケーションは別のスレッドで実行され、制御は直ちに呼び出し元に戻ります。このため、バッチジョブの一部としてデプロイメントを実行する場合、ジョブを進める前にデプロイメントが成功するのを待つ必要があり、問題が発生することがあります。このような問題が発生した場合は、start /waitを使用して、TabularEditorがジョブを終了してから呼び出し元に制御を返します。

start /wait TabularEditor.exe c:\Projects\Model.bim -deploy localhost AdventureWorks

高度なスクリプティング
Tabular Editorでは、C#を使用してロードされたモデルへの変更をスクリプト化することができます。これは、一度に多くのオブジェクトに複数の変更を適用したい場合に実用的です。アドバンスド スクリプト エディタは2つのオブジェクトにアクセスできます。

Selectedは、エクスプローラツリーで現在選択されているすべてのオブジェクトを表します。
モデルは、Tabular Object Modelツリー全体を表します。
アドバンスドスクリプトエディタには、いくつかの限定されたインテリセンス機能があり、使い始めることができます。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/AdvancedEditor%20intellisense.png

アドバンスドスクリプトの詳細なドキュメントやサンプルは、こちらをご覧ください。