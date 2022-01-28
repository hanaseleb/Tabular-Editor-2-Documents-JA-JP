コマンドライン
2021-08-26
ダニエル・オティキエル
Tabular Editorは、コマンドラインから実行してさまざまなタスクを実行することができます。これは、自動化されたビルドやデプロイメントのシナリオなどで役に立つでしょう。

注：TabularEditor.exeはWinFormsアプリケーションなので、Windowsのコマンドプロンプトから直接実行すると、スレッドがすぐにプロンプトに戻ってしまいます。このため、コマンドスクリプトなどで問題が発生することがあります。TabularEditor.exeがコマンドラインのタスクを完了するのを待つには、常に次の方法で実行してください： start /wait TabularEditor ...

Tabular Editorで使用できるコマンドラインオプションを表示するには、次のコマンドを実行します。

Windowsのコマンドラインです。

start /wait TabularEditor.exe /?
PowerShellです。

$p = Start-Process -filePath TabularEditor.exe -Wait -NoNewWindow -PassThru -ArgumentList "/?"
出力します。

タブラー・エディター2.16.0（ビルド2.16.7781.40242）の場合
--------------------------------
使用方法

TABULAREDITOR ( ファイル | サーバー データベース ) [-S script1 [script2] [...]]...
    [-SC] [-A [rules] | -AX rules] [(-B | -F) output [id]] [-V | -G] [-T resultsfile].
    [-D [サーバーデータベース] [-L ユーザーパス] [-O [-C [plch1 value1 [plch2 value2 [...]]]]
        [-p [-y]] [-r [-m]]]となります。
        [-X xmla_script]] [-W] [-E]] を参照してください。

file 読み込む Model.bim ファイルまたは database.json モデルフォルダのフルパスです。
server モデルの読み込み元となる Server_instance 名または接続文字列
database ロードするモデルのデータベース ID
-S / -SCRIPT 読み込み後のモデルに対して、指定したスクリプトを実行します。
  scriptN 実行する C# スクリプトまたはインラインスクリプトを含む 1 つまたは複数のファイルのフルパス。
                      スクリプトを実行します。
-SC / -SCHEMACHECK テーブルスキーマの変更を検出するために、すべてのプロバイダーデータソースへの接続を試みます。
                    の変更を検出します。出力は...
                      不一致のデータタイプとマッピングされていないソースカラムに対する警告
                      マッピングされていないモデルカラムのエラー
-A / -ANALYZE ベストプラクティス・アナライザーを実行し、結果をコンソールに出力します。
  rules 分析対象となる追加の BPA ルールのファイルまたは URL のパスを指定します。指定した場合
                      指定された場合、モデルはローカル・ユーザ/ローカル・マシン・ルールに対して分析されません。
                      しかし、モデル内で定義されたルールは適用されます。
-AX / -ANALYZEX -A / -ANALYZEと同じですが、モデルのアノテーションで指定されたルールは除外されます。
-B / -BIM / -BUILD モデルを（オプションのスクリプト実行後に）Model.bimファイルとして保存します。
  output 保存先となる Model.bim ファイルのフルパスです。
  id 保存時に Database オブジェクトに割り当てる任意の ID/名前です。
-F / -FOLDER (オプションのスクリプト実行後)モデルをフォルダー構造として保存します。
  output 保存先のフォルダのフルパス。フォルダが存在しない場合は作成されます。
  id 保存時に Database オブジェクトに割り当てる任意の ID/名前です。
-V / -VSTS Visual Studio Team Servicesのログコマンドを出力します。
-G / -GITHUB GitHub Actions ワークフロー コマンドを出力します。
-T / -TRX 実行内容の詳細を記載したVSTEST（trx）ファイルを出力します。
  resultsfile VSTEST XML ファイルのファイル名です。
-D / -DEPLOY コマンドライン・デプロイメント
                      追加のパラメーターが指定されていない場合、このスイッチはモデルのメタデータをソース（ファイルまたはデータベー ス）に保存します。
                      ソース（ファイルまたはデータベース）に戻します。
  server 配置先のサーバー名、または Analysis Services への接続文字列。
  database 配置するデータベースの ID (作成/上書き) を指定します。
  -L / -LOGIN サーバーへの接続時に統合セキュリティを無効にします。指定します。
    user ユーザー名 (サーバーの管理者権限を持つユーザーである必要があります)
    pass パスワード
  -O / -OVERWRITE 既存のデータベースのデプロイ（上書き）を許可します。
    -C / -CONNECTIONS モデル内の既存のデータソースのデプロイ（上書き）を許可します。C スイッチの後には
                        C スイッチの後には、（オプションで）任意の数のプレースホルダーと値のペアを指定できます。これを行うと
                        モデル内のすべてのデータ・ソースの接続文字列で、指定されたプレースホルダー（plch1、plch2、...）の出現をすべて置き換えます。
                        モデル内のすべてのデータ・ソースの接続文字列で、指定されたプレースホルダー（plch1、plch2、...）を、指定された値
                        (value1, value2, ...)に置き換えられます。
    -P / -PARTITIONS モデル内の既存のテーブルパーティションを展開（上書き）します。
      -Y / -SKIPPOLICY インクリメンタル・リフレッシュ・ポリシーが定義されているパーティションを上書きしません。
    -R / -ROLES ロールをデプロイします。
      -M / -MEMBERS ロールのメンバーを配置します。
  -X / -XMLA デプロイしません。代わりにXMLA/TMSLスクリプトを生成して、後でデプロイします。
    xmla_script 新しい XMLA/TMSL スクリプト出力のファイル名です。
  -W / -WARN 未処理のオブジェクトに関する情報を警告として出力します。
  -E / -ERR メタデータのデプロイ/更新後に Analysis Services が何らかのエラー メッセージを返した場合、0 以外の終了コードを返します。
                      メタデータの展開/更新後に、Analysis Services が何らかのエラーメッセージを返した場合、0 以外の終了コードを返します。
Azure Analysis Servicesへの接続
コマンドのサーバー名の代わりに、任意の有効なSSAS接続文字列を使用できます。次のコマンドは、Azure Analysis Servicesからモデルをロードし、Model.bimファイルとしてローカルに保存します。

Windows コマンドライン。

start /wait TabularEditor.exe "Provider=MSOLAP;Data Source=asazure://northeurope.asazure.windows.net/MyAASServer;User ID=xxxx;Password=xxxx;Persist Security Info=True;Impersonation Level=Impersonate" MyModelDB -B "C:001Projects\FromAzure\Model.bim"
PowerShell

$p = Start-Process -filePath TabularEditor.exe -Wait -NoNewWindow -PassThru `
       -ArgumentList "`"Provider=MSOLAP;Data Source=asazure://northeurope.asazure.windows.net/MyAASServer;User ID=xxxx;Password=xxxx;Persist Security Info=True;Impersonation Level=Impersonate`" MyModelDB -B C:\\FromAzure\Model.bim"
Azure Active Directory認証ではなく、サービスプリンシパル（アプリケーションIDとキー）を使って接続したい場合は、次のような接続文字列を使うことができます。

Provider=MSOLAP;Data Source=asazure://northeurope.asazure.windows.net/MyAASServer;User ID=app:<APPLICATION ID>@<TENANT ID>;Password=<APPLICATION KEY>;Persist Security Info=True;Impersonation Level=Impersonate
スクリプト変更の自動化
Tabular Editor内でスクリプトを作成し、デプロイメントの前にModel.bimファイルにこのスクリプトを適用したい場合は、コマンドラインオプションの「-S」（Script）を使用できます。

Windowsのコマンドライン。

start /wait TabularEditor.exe "C:˶‾᷄᷄MyModel˶‾᷅˵" -S "C:˶‾᷅˵MyModel˶‾᷅˵" -D localhost˶‾᷅˵ MyModel
PowerShell

$p = Start-Process -filePath TabularEditor.exe -Wait -NoNewWindow -PassThru `
       -ArgumentList "`"C:Projects\MyModel\Model.bim`" -S `"C:Projects\MyModel\MyScript.cs`" -D `"localhost\tabular`" `"MyModel`""
このコマンドは、Model.bimファイルをTabular Editorで読み込み、指定されたスクリプトを適用して、変更されたモデルを新しいデータベース「MyModel」として「localhost\tabular」サーバーにデプロイします。サーバー上にある同名の既存のデータベースを上書きしたい場合は、「-O」（Overwrite）スイッチを使用します。

D"(Deploy)スイッチの代わりに"-B"(Build)スイッチを使用すると、変更したモデルを直接サーバーにデプロイするのではなく、新しい Model.bim ファイルとして出力することができます。これは、他のデプロイメントツールを使ってモデルをデプロイしたい場合や、デプロイ前にVisual StudioやTabular Editorでモデルを検査したい場合に便利です。また、自動化されたビルドシナリオでは、デプロイする前に変更されたモデルをリリースの成果物として保存する場合にも便利です。

デプロイ時の接続文字列の変更
以下のような接続文字列を持つデータソースを含むモデルがあるとします。

Provider=SQLOLEDB.1;Data Source=sqldwdev;Persist Security Info=False;Integrated Security=SSPI;Initial Catalog=DW
デプロイ時には、この文字列を変更してUATまたは本番データベースを指すようにします。これを行う最も良い方法は、まず接続文字列全体をプレースホルダー値に変更するスクリプトを使用し、次に-Cスイッチを使用してプレースホルダーと実際の接続文字列を入れ替えることです。

以下のスクリプトを "ClearConnectionStrings.cs "などのファイルに記述します。

// モデル内のすべてのプロバイダ（レガシー）データソースの接続文字列を、 // プレースホルダーに基づいて置き換えます。
// データソースの名前に基づいたプレースホルダーで置き換えます。例えば、データソースの名前が
例えば、データソースの名前が // "SQLDW "であれば、このスクリプトを実行した後の接続文字列は "SQLDW" になります。

foreach(var ds in Model.DataSources.OfType<ProviderDataSource>())
    ds.ConnectionString = ds.Name;
Tabular Editorにスクリプトの実行を指示し、次のコマンドを使ってプレースホルダの交換を行います。

Windowsのコマンドラインです。

start /wait TabularEditor.exe "Model.bim" -S "ClearConnectionStrings.cs" -D localhost\tabular MyModel -C "SQLDW" "Provider=SQLOLEDB.1;Data Source=sqldwprod;Persist Security Info=False;Integrated Security=SSPI;Initial Catalog=DW"
パワーシェルで

$p = Start-Process -filePath TabularEditor.exe -Wait -NoNewWindow -PassThru `.
       -ArgumentList "Model.bim -S ClearConnectionStrings.cs -D localhost\tabular MyModel -C SQLDW `"Provider=SQLOLEDB.1;Data Source=sqldwprod;Persist Security Info=False;Integrated Security=SSPI;Initial Catalog=DW`""
上記のコマンドを実行すると、Model.bimファイルが「localhost\tabular」SSASインスタンス上の新しいSSASデータベース「MyModel」としてデプロイされます。デプロイする前に、スクリプトを使用して、プロバイダ（レガシー）データソースのすべての接続文字列を、プレースホルダーとして使用するデータソースの名前に置き換えます。SQLDW」というデータソースが1つしかないと仮定すると、-Cスイッチは接続文字列を更新し、「SQLDW」を指定された文字列全体で置き換えます。

この手法は、同じモデルを複数の環境にデプロイし、それらの環境で異なる（同一の）ソースからのデータを処理したい場合に便利です。Azure DevOps（下記参照）を使用する場合は、使用する実際の接続文字列をコマンドにハードコーディングするのではなく、変数に格納することを検討してください。

Azure DevOpsとの統合
Azure DevOpsパイプライン内でTabular Editor CLIを使用したい場合は、スクリプトで実行されるTabularEditor.exeコマンドに「-V」スイッチを使用する必要があります。このスイッチにより、Tabular EditorはAzure DevOpsが読める形式でログ・コマンドを出力します。これにより、Azure DevOpsがエラーなどに適切に対応できるようになります。

コマンドラインでデプロイメントを実行すると、未処理のオブジェクトに関する情報がプロンプトに出力されます。自動化されたデプロイメントシナリオでは、新しい列の追加、計算テーブルのDAX式の変更など、オブジェクトが未処理になる状況にビルドエージェントが反応したい場合があります。このような場合、前述の「-V」スイッチに加えて「-W」スイッチを使用することで、この情報を警告として出力することができます。そうすることで、デプロイが完了した後、Azure DevOpsに「SucceededWithIssues」というステータスを返すようになります。また、デプロイが成功した後、サーバーからDAXエラーが報告された場合に、デプロイメントに「Failed」ステータスを返したい場合は、「-E」スイッチを使用することもできます。

Azure DevOpsパイプラインのコマンドラインタスク内でTabularEditor.exeを実行する場合、start /waitは必要ありません。これは、タスクによって生成されたすべてのスレッドが終了するまで、コマンドラインタスクが完了しないためです。言い換えると、start /waitを使用する必要があるのは、TabularEditor.exeの呼び出しの後に追加のコマンドがある場合のみで、この場合は必ずstart /B /waitを使用してください。TabularEditor.exeからの出力を正しくパイプラインログに戻すためには、/Bスイッチが必要です。

TabularEditor.exe "C:Projects\My Model\Model.bim" -D ssasserver databasename -O -C -P -V -E -W
複数のコマンドを使うこともできます。

start /B /wait TabularEditor.exe "C:\Projects\Finance\Model.bim" -D ssasserver Finance -O -C -P -V -E -W
start /B /wait TabularEditor.exe "C:Press\Sales\Model.bim" -D ssasserver Sales -O -C -P -V -E -W
このようなビルドをAzure DevOpsで行うと下図のようになります。

https://user-images.githubusercontent.com/8976200/27128146-bc044356-50fd-11e7-9a67-b893fc48ea50.png

何らかの理由でデプロイメントが失敗すると、「-W」スイッチを使用しているかどうかにかかわらず、Tabular Editorは「Failed」ステータスをAzure DevOpsに返します。

Azure DevOpsとTabular Editorの詳細については、このブログシリーズ（特に第3章以降）をご覧ください。

Azure DevOpsのPowerShellタスク
コマンドライン・タスクの代わりにPowerShellタスクを使用したい場合は、上で示したようにStart-Processコマンドレットを使用してTabularEditor.exeを実行する必要があります。さらに、PowerShellスクリプトのexitパラメータとしてプロセスの終了コードを渡すようにして、Tabular Editorで発生したエラーがPowerShellタスクの失敗につながるようにします。

$p = Start-Process -filePath TabularEditor.exe -Wait -NoNewWindow -PassThru `
       -引数リスト "`"C:Projects\My Model\Model.bim`" -D ssasserver databasename -O -C -P -V -E -W"
exit $p.ExitCode
ベストプラクティスアナライザーの実行
A」スイッチを使用すると、Tabular Editorがモデルをスキャンして、ローカルマシン（%AppData%...Local\TabularEditor\BPARules.jsonファイル）で定義されたベストプラクティスルール、またはモデル自体のアノテーションに違反しているすべてのオブジェクトを探すことができます。また、「-A」スイッチの後に、ベストプラクティスルールを含む.jsonファイルのパスを指定すると、そのファイルに定義されたルールを使ってモデルをスキャンします。違反しているオブジェクトは、コンソールに出力されます。

また、「-V」スイッチを使用している場合は、各ルールの深刻度レベルによって、ルール違反がビルドパイプラインにどのように報告されるかが決まります。

深刻度 = 1 は、情報提供のみです。
重大度=2の場合、警告が表示されます。
深刻度が3以上の場合はERRORとなります。
データソースのスキーマチェックを行う
バージョン 2.8 では、-SC (-SCHEMACHECK) スイッチを使ってテーブルソースのクエリを検証することができます。これは、モデルへの変更は行われませんが、スキーマの違いがコンソールに報告されることを除けば、Refresh Table Metadata UIの起動と同じです。変更されたデータ型やソースに追加された列は、警告として報告されます。ソースにないカラムはエラーとして報告されます。SC（-SCHEMACHECK）と-S（-SCRIPT）の両方のスイッチが指定されている場合、スキーマ チェックはスクリプトが正常に実行された後に実行され、スキーマ チェックが実行される前にデータ ソースのプロパティを変更することができます。

また、スキーマチェックで特定の方法で処理したい場合は、テーブルやカラムに注釈を付けることができます。詳細はこちらをご覧ください。

コマンドライン出力と終了コード
コマンドラインには、使用したスイッチや実行中に発生したイベントに応じて、さまざまな詳細情報が表示されます。終了コードはバージョン2.7.4で導入されました。

レベル コマンド メッセージの明確化
Error (Any) Invalid argument syntax 無効な引数がTabular Editor CLIに提供されました。
エラー (Any) ファイルが見つかりません: ...	
Error (Any) Error loading file: ... ファイルが壊れているか、有効なTOMメタデータがJSON形式で含まれていません。
Error (Any) Error loading model: ... 提供されたAnalysis Servicesインスタンスに接続できません、データベースが見つかりません、データベースのメタデータが破損しています、またはデータベースの互換性レベルがサポートされていません。
エラー -SCRIPT 指定されたスクリプト ファイルが見つかりません。	
エラー -SCRIPT スクリプトのコンパイル エラーです。	スクリプトに無効な C# 構文が含まれています。詳細は次の行に出力されます。
エラー -SCRIPT スクリプト実行エラー: ... スクリプト実行時に処理されない例外が発生しました。
情報 -SCRIPT スクリプトの行番号: ... スクリプト内での Info(string) または Output(string) メソッドの使用。
Warning -SCRIPT Script warning: ... スクリプト内の Warning(string)メソッドの使用。
Error -SCRIPT スクリプトのエラー: ... スクリプト内の Error(string) メソッドの使用。
エラー -FOLDER, -BIM -FOLDER と -BIM の引数は、相互に排他的です。	Tabular Editorは、現在ロードされているモデルをフォルダ構造と.bimファイルに1回の実行で保存することができません。
エラー -ANALYZE ルールファイルが見つかりません: ...	
エラー -ANALYZE Invalid rulefile: ... 指定されたBPAルールファイルが壊れているか、有効なJSONが含まれていません。
情報 -ANALYZE ... ルールに違反しています ...	重大度レベル1以下のルールに対するBest Practice Analyzerの結果です。
警告 -ANALYZE ... ルールに違反しています ...	重大度レベル2のルールに対するBest Practice Analyzerの結果です。
エラー -ANALYZE ... ルールに違反しています ...	重大度レベル3以上のルールに対するベストプラクティス・アナライザーの結果です。
エラー -DEPLOY 展開に失敗しました! ...	Analysis Service インスタンスから直接返された障害理由（例：データベースが見つかりません、データベースのオーバーライドが許可されていません、など）。
情報 -DEPLOY 未処理のオブジェクト: ... デプロイ成功後に「NoData」または「CalculationNeeded」の状態になっているオブジェクト。これらを Level=Warning として扱うには -W スイッチを使用します。
Warning -DEPLOY "Ready "状態ではないオブジェクト： ... デプロイが成功した後、"DependencyError"、"EvaluationError"、"SemanticError "のいずれかの状態になったオブジェクト。W スイッチを使用した場合は、状態が "NoData" または "CalculationNeeded" のオブジェクトも含まれます。
警告 -DEPLOY X:...でエラーが発生しました。	正常に展開された後に、無効な DAX を含むオブジェクト (メジャー、計算列、計算テーブル、ロール) が表示されます。これらを Level=Error として扱うには、-E スイッチを使用します。
Error」レベルの出力のいずれかに遭遇した場合、Tabular EditorはExit Code = 1を返します。それ以外は0です。