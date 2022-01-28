Excelからテーブルをインポートする
2021-11-10
ダニエル・オティキエル
Excelワークシートをテーブルとして表形式モデルに追加する必要がある場合、Tabular Editor 2.xとExcel ODBCドライバで可能です。

前提条件
タブラー・エディター2.xは32ビットのアプリケーションですが、多くの人は通常64ビット版のOfficeをインストールしています（64ビット版のExcel ODBCドライバーも含まれています）。残念ながら、Tabular Editor 2.xは64ビットのドライバを使用することができず、32ビットのドライバをダウンロードしてインストールしようとすると、すでに64ビット版のOfficeがインストールされている場合はエラーが発生します。しかし、この回避策を使えば、64ビット版Officeの隣に32ビット版Excel ODBCドライバをインストールすることができます。

32ビット版のドライバーをこちらからダウンロードしてください。https://www.microsoft.com/en-us/download/details.aspx?id=54920
AccessDatabaseEngine.exeファイルを解凍します。
中にはaceredist.msiファイルが入っています。このファイルは、コマンドラインで/passiveスイッチを使って実行します。
aceredist.msi /passive
ODBC Data Sources (32-bit)の設定（Windowsのスタートボタンから "ODBC "を検索し、以下のスクリーンショットのように "32/64 bit "と表示されているはずです）でインストールを確認します：.../images/excel-odbc-32-64.png
ODBCデータソースの設定
上述のように32ビットODBC Excelドライバがインストールされていることを確認した後、Tabular Editor 2.xでExcelファイルからテーブルを追加するには、以下の手順が必要です。

Tabular Editorでモデルを右クリックし、"Import tables... "を選択し、"Next "をクリックする。
接続のプロパティ」ダイアログで、「変更...」をクリックします。Microsoft ODBC Data Source "を選択し、"OK "をクリックする。
"Use connection string "を選択し、"Build... "を押す。Excel Files」を選択し、「OK」を押す.../images/odbc-connection-properties-excel.png
テーブルを読み込みたいExcelファイルを選択し、「OK」を押す。すると、以下のような接続文字列が生成されるはずです。
Dsn=Excel Files;dbq=C:Users#DanielOtykier#Documents#A Beer Dataset Calculation.xlsx;defaultdir=C:Users#DanielOtykier#Documents;driverid=1046;maxbuffersize=2048;pagetimeout=5
OK」をクリックすると、Tabular EditorはExcelファイルのワークシートとデータ領域のリストを表示します。残念ながら、テーブルのインポートウィザードは、無効なSQL文を生成するため、現在データをプレビューすることができません：.../images/import-tables-excel.png
しかし、インポートしたいテーブルにチェックマークを付けることはできます。エラーメッセージは無視して、「Import」ボタンを押してください。
新しく追加されたテーブルで、パーティションの位置を確認し、SQLを修正して空のブラケットとワークシート名の前のドットを削除します。変更を適用する（F5キーを押す）.../images/fix-partition-expressions-excel.png
次に、パーティションを右クリックして「Refresh Table Metadata...」を選択します。Tabular EditorがODBCドライバを通じてExcelファイルから列のメタデータを読み取るようになります：.../images/refresh-metadata-excel.png
(オプション）テーブルへのデータの更新にODBCを使用したくない場合は、パーティションを交換して、同じワークシートデータを読み込むMベースの式を使用する必要があります。これを行うには、テーブルに新しい Power Query パーティションを追加します（「パーティション」を右クリックし、「新しいパーティション（Power Query）」を選択します）。レガシーパーティションを削除します。次に、新しいパーティションのM式を以下のように設定します。
レット
    Source = Excel.Workbook(File.Contents("<excel file path>"), null, true),
    Customer_Sheet = Source{[Item="<sheet name>",Kind="Sheet"]}[Data],
    #「プロモートされたヘッダー」 = Table.PromoteHeaders(Customer_Sheet, [PromoteAllScalars=true])
で
    #"プロモートヘッダ"
<excelファイルのパス>と<シート名>のプレースホルダーを実際の値に置き換えます。

まとめ
ExcelファイルからテーブルをインポートすることはTabular Editor 2.xで可能ですが、上記のようにODBC Excelドライバを使用する必要があり、プロセスに複雑さが加わります。