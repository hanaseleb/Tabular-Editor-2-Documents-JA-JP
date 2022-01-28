# Useful Script Snippets (便利なスクリプトスニペット)

ここでは、Tabular Editorの高度なスクリプト機能を使い始めるための小さなスクリプトスニペットのコレクションを紹介します。これらのスクリプトの多くは、カスタムアクションとして保存しておくと、コンテキストメニューから簡単に再利用できるようになります。

他のスクリプトを試してみたい場合や、自分のスクリプトを投稿したい場合は、Tabular Editor Scriptsのリポジトリにアクセスしてください。

## 列からのメジャーの作成

```C#
// 現在選択されているすべての列に対して SUM メジャーを作成し、その列を非表示にします。
foreach(var c in Selected.Columns)
{
    var newMeasure = c.Table.AddMeasure(
        "Sum of " + c.Name, // Name
        "SUM(" + c.DaxObjectFullName + ")", // DAX式
        c.DisplayFolder // 表示フォルダ
    );
    
    // 新規メジャーにフォーマット文字列を設定します。
    newMeasure.FormatString = "0.00";

    // ドキュメントを作成します。
    newMeasure.Description = "This measure is the sum of column " + c.DaxObjectFullName

    // ベースの列を非表示にします。
    c.IsHidden = true;
}
```

このスニペットでは、`<Table>.AddMeasure(<name>, <expression>, <displayFolder>)` 関数を使用して、テーブルに新しいメジャーを作成しています。DaxObjectFullName プロパティを使用して、DAX 式で使用する列の完全修飾名である 'TableName'[ColumnName] を取得します。

## タイム・インテリジェンス・メジャーの生成

まず、個々のタイム・インテリジェンス集約のカスタム・アクションを作成します。たとえば、以下のようになります。

```dax
// 選択されたすべてのメジャーに対してTOTALYTDメジャーを作成します。
foreach(var m in Selected.Measures) {.
    m.Table.AddMeasure(
        m.Name + " YTD", // Name
        "TOTALYTD(" + m.DaxObjectName + ", 'Date'[Date])", // DAX式
        m.DisplayFolder // 表示フォルダ
    );
}
```

ここでは、`DaxObjectName`プロパティを使用して、DAX式で使用する非限定的な参照を生成しています。 これは、これがメジャーです：`[MeasureName]`。 これを、メジャーに適用される「Time Intelligence\Create YTD measure」というカスタムアクションとして保存します。MTD、LY、その他必要に応じて同様のアクションを作成します。次に、以下を新しいアクションとして作成します。

```DAX
// すべてのTime Intelligenceカスタムアクションを呼び出します。
CustomAction(@"Time Intelligence\Create YTD measure");
CustomAction(@"Time Intelligence\Create MTD measure");
CustomAction(@"Time Intelligence\Create LY measure");
```

これは、別のアクションの中から1つ（または複数）のカスタムアクションを実行する方法を示しています（循環参照に注意してください - Tabular Editorがクラッシュします）。これを新しいカスタムアクション`Time Intelligence\All of the above`として保存すれば、ワンクリックですべてのTime Intelligenceメジャーを生成する簡単な方法ができます。

<https://user-images.githubusercontent.com/8976200/36632257-5565c8ca-197c-11e8-8498-82667b6e1049.png>

もちろん、以下のようにすべてのタイムインテリジェンスの計算を1つのスクリプトにまとめることもできます。

```dax
var dateColumn = "'Date'[Date]";

// 選択されたすべてのメジャーに対して、タイム・インテリジェンス・メジャーを作成します。
foreach(var m in Selected.Measures) { 。
    // 累計時間。
    m.Table.AddMeasure(
        m.Name + " YTD", // 名前
        "TOTALYTD(" + m.DaxObjectName + ", " + dateColumn + ")", // DAX式
        m.DisplayFolder // 表示フォルダ
    );

    // 前年度の様子
    m.Table.AddMeasure(
        m.Name + " PY", // 名前
        "CALCULATE(" + m.DaxObjectName + ", SAMEPERIODLASTYEAR(" + dateColumn + "))", // DAX式
        m.DisplayFolder // 表示フォルダ
    );    
    
    // 前年同期比
    m.Table.AddMeasure(
        m.Name + " YoY", // 名前
        m.DaxObjectName + " - [" + m.Name + " PY]", // DAX式
        m.DisplayFolder // 表示フォルダ
    );
    
    // 前年同期比 %:
    m.Table.AddMeasure(
        m.Name + " YoY%", // 名前
        "DIVIDE(" + m.DaxObjectName + ", [" + m.Name + " YoY])", // DAX式
        m.DisplayFolder // 表示フォルダ
    ).FormatString = "0.0 %"; // フォーマット文字列をパーセンテージとして設定
    
    // 四半期ごとの推移
    m.Table.AddMeasure(
        m.Name + " QTD", // 名前
        "TOTALQTD(" + m.DaxObjectName + ", " + dateColumn + ")", // DAX式
        m.DisplayFolder // 表示フォルダ
    );
    
    // Month-to-dateを表示します。
    m.Table.AddMeasure(
        m.Name + " MTD", // 名前
        "TOTALMTD(" + m.DaxObjectName + ", " + dateColumn + ")", // DAX式
        m.DisplayFolder // 表示フォルダ
    );
}
```

## 追加プロパティの設定

新しく作成されたメジャーに追加のプロパティを設定したい場合は、上記のスクリプトを次のように変更できます。

```dax
// 選択されたすべてのメジャーに対して TOTALYTD メジャーを作成します。
foreach(var m in Selected.Measures) { 以下はその例です。
    var newMeasure = m.Table.AddMeasure(
        m.Name + " YTD", // 名前
        "TOTALYTD(" + m.DaxObjectName + ", 'Date'[Date])", // DAX式
        m.DisplayFolder // 表示フォルダ
    );
    newMeasure.FormatString = m.FormatString; // 元のメジャーからフォーマット文字列をコピーします。
    foreach(var c in Model.Cultures) {.
        newMeasure.TranslatedNames[c] = m.TranslatedNames[c] + " YTD"; // すべての文化の翻訳された名前をコピーします。
        newMeasure.TranslatedDisplayFolders[c] = m.TranslatedDisplayFolders[c]; // 翻訳された表示フォルダをコピーする
    }
}
```

## パースペクティブの処理

メジャー、列、階層、テーブルはすべて `InPerspective` プロパティを公開しています。このプロパティは、モデル内のすべてのパースペクティブについて、指定されたオブジェクトがそのパースペクティブのメンバーであるかどうかを示すTrue/Falseの値を保持します。つまり、たとえば以下のようになります。

```dax
foreach(var measure in Selected.Measures)
{
    measure.InPerspective["Inventory"] = true;
    measure.InPerspective["Reseller Operation"] = false。
}
```

上記のスクリプトでは、選択されたすべてのメジャーが「インベントリ」パースペクティブに表示され、"Reseller Operation" パースペクティブでは非表示になります。

`InPerspective`プロパティは、個々のパースペクティブのメンバーシップの取得/設定に加えて、以下のメソッドもサポートしています。

`<<object>>.InPerspective.None()` - すべてのパースペクティブからオブジェクトを削除します。
`<<object>>.InPerspective.All()` - すべてのパースペクティブにオブジェクトを含める。
`<<object>>.CopyFrom(string[] perspectives)` - 指定されたすべてのパースペクティブ（パースペクティブの名前を含む文字列の配列）にオブジェクトを含みます。
`<<object>>.CopyFrom(perspectiveIndexer perspectives)` - 別の`InPerspective`プロパティからパースペクティブのインクルードをコピーします。
後者は、パースペクティブメンバシップをあるオブジェクトから別のオブジェクトコピーするために使用できます。たとえば、ベース・メジャー[Reseller Total Sales] があり、現在選択されているすべてのメジャーが、このベース・メジャーと同じパースペクティブとして表示されるようにしたいとします。以下のスクリプトで実現できます。

```C#
var baseMeasure = Model.Tables["Reseller Sales"].Measures["Reseller Total Sales"];

foreach(var measure in Selected.Measures)
{
    /*「baseMeasure」があるパースペクティブから「measure」を非表示にしたい場合は、以下の行をアンコメントしてください。
       を非表示にしたい場合は、以下の行をアンコメントしてください。*/
    // measure.InPerspective.None();

    measure.InPerspective.CopyFrom(baseMeasure.InPerspective);
}
```

この手法は、コードから新しいオブジェクトを生成する場合にも使用できます。たとえば、自動生成された `Time Intelligence` メジャーが、そのベース・メジャーと同じパースペクティブでのみ表示されるようにするには、前のセクションのスクリプトを以下のように拡張します。

```C#
// 選択されたすべてのメジャーに対して TOTALYTD メジャーを作成します。
foreach(var m in Selected.Measures) { 以下のように拡張できます。
    var newMeasure = m.Table.AddMeasure(
        m.Name + " YTD", // 名前
        "TOTALYTD(" + m.DaxObjectName + ", 'Date'[Date])", // DAX式
        m.DisplayFolder // 表示フォルダ
    );
    newMeasure.InPerspective.CopyFrom(m.InPerspective); // ベースメジャーからパースペクティブを適用する
}
```

オブジェクトのプロパティをファイルにエクスポートする
ワークフローによっては、Excel を使用して複数のオブジェクト プロパティを一括して編集すると便利な場合があります。以下のスニペットを使用して、標準的なプロパティのセットを .TSV ファイルにエクスポートし、後からインポートすることができます (以下を参照)。

// 現在選択されているオブジェクトのプロパティをエクスポートします。
var tsv = ExportProperties(Selected);
SaveFile("Exported Properties 1.tsv", tsv);
できあがった .TSV ファイルを Excel で開くと、次のようになります。<https://user-images.githubusercontent.com/8976200/36632472-e8e96ef6-197e-11e8-8285-6816b09ad036.pngThe> 最初の列 (Object) の内容は、オブジェクトへの参照です。この列の内容を変更すると、その後のプロパティのインポートが正しく行われない場合があります。オブジェクトの名前を変更する場合は、2列目（Name）の値のみを変更してください。

デフォルトでは、ファイルはTabularEditor.exeがあるのと同じフォルダに保存されます。デフォルトでは、以下のプロパティのみがエクスポートされます（エクスポートされたオブジェクトの種類に応じて、該当するものがあります）。

名前
説明
ソースカラム
式
形式文字列
データタイプ
異なるプロパティをエクスポートするには、ExportProperties の 2 番目の引数として、エクスポートするプロパティ名をコンマで区切ったリストを指定します。

// 現在選択されているテーブルのすべてのメジャーの名前と詳細行の式をエクスポートします。
var tsv = ExportProperties(Selected.Table.Measures, "Name,DetailRowsExpression");
SaveFile("Exported Properties 2.tsv", tsv) を実行します。
利用可能なプロパティ名は、TOM APIのドキュメントで確認できます。これらは、キャメルケースを使ってスペースを取り除いたTabular Editorのプロパティグリッドに表示される名前とほとんど同じです（いくつかの例外がありますが、例えば、「Hidden」プロパティはTOM APIではIsHiddenと呼ばれます）。

プロパティをインポートするには、次のスニペットを使用します。

// 指定されたファイルのプロパティをインポートして適用します。
var tsv = ReadFile("Exported Properties 1.tsv");
ImportProperties(tsv)。
インデックス付きプロパティのエクスポート
Tabular Editor 2.11.0では、ExportPropertiesおよびImportPropertiesメソッドはインデックス付きのプロパティをサポートしています。インデックス付きのプロパティとは、プロパティ名に加えてキーを取るプロパティです。たとえば、myMeasure.TranslatedNamesがあります。このプロパティは、myMeasure の名前の翻訳として適用されたすべての文字列のコレクションを表します。C#では、myMeasure.TranslatedNames["da-DK"]というインデックス演算子を使って、特定の文化の翻訳されたキャプションにアクセスできます。

要するに、Tabularモデル内のオブジェクトのすべての翻訳、パースペクティブ情報、アノテーション、拡張プロパティ、行レベルおよびオブジェクトレベルのセキュリティ情報をエクスポートできるようになりました。

たとえば、次のスクリプトは、すべてのモデルメジャーと、それぞれがどのパースペクティブで表示されるかについての情報のTSVファイルを生成します。

var tsv = ExportProperties(Model.AllMeasures, "Name,InPerspective");
SaveFile(@"c:Project\MeasurePerspectives.tsv", tsv);
TSVファイルをExcelで開くと、以下のようになります。

<https://user-images.githubusercontent.com/8976200/85208532-956dec80-b331-11ea-8568-32dbd4cc5516.png>

上の例のように、Excelで変更して保存し、ImportPropertiesを使って更新された値をTabular Editorにロードして戻すことができます。

特定のパースペクティブだけをリストアップしたい場合は、ExportPropertiesの呼び出しの第2引数でそれらを指定することができます。

var tsv = ExportProperties(Model.AllMeasures, "Name,InPerspective[Inventory]");
SaveFile(@"c:‾‾Project\MeasurePerspectiveInventory.tsv", tsv);
同様に、翻訳やアノテーションなどについても。例えば、テーブル、列、階層、レベル、メジャーに適用されたすべてのデンマーク語翻訳を確認したい場合

// オブジェクトのリストを作成します。
var objects = new List<TabularNamedObject>();
objects.AddRange(Model.Tables);
objects.AddRange(Model.AllColumns);
objects.AddRange(Model.AllHierarchies);
objects.AddRange(Model.AllLevels);
objects.AddRange(Model.AllMeasures);

var tsv = ExportProperties(objects, "Name,TranslatedNames[da-DK],TranslatedDescriptions[da-DK],TranslatedDisplayFolders[da-DK]");
SaveFile(@"c:Project\ObjectTranslations.tsv", tsv);
ドキュメントの作成
上記のExportPropertiesメソッドは、モデルの全部または一部を文書化したい場合にも使用できます。次のスニペットは、Tabular Model のすべての可視メジャーまたはカラムからプロパティのセットを抽出し、TSV ファイルとして保存します。

// 表示されているすべての列とメジャーのリストを作成します。
var objects = Model.AllMeasures.Where(m => !m.IsHidden && !m.Table.IsHidden).Cast<ITabularNamedObject>()
      .Concat(Model.AllColumns.Where(c => !c.IsHidden && !c.Table.IsHidden));

// TSV形式（タビュレーターで区切られた）でプロパティを取得します。
var tsv = ExportProperties(objects, "Name,ObjectType,Parent,Description,FormatString,DataType,Expression");

// (オプション) 画面への出力 (その後、Excel にコピーペーストできます)。
// tsv.Output();

// ...または、TSVをファイルに保存します。
SaveFile("documentation.tsv", tsv) を実行します。
ファイルからのメジャーの生成
上記のプロパティのエクスポート/インポートのテクニックは、モデル内の既存オブジェクトのオブジェクト・プロパティを一括して編集したい場合に便利です。しかし、まだ存在していないメジャーのリストをインポートしたい場合はどうすればよいでしょうか。

例えば、既存の Tabular モデルにインポートしたいメジャーの名前、説明、DAX 式を含む TSV (tab-separated values) ファイルがあるとします。以下のスクリプトを使用すると、ファイルを読み込んで行と列に分割し、メジャーを生成することができます。また、このスクリプトは各メジャーに特別なアノテーションを割り当て、同じスクリプトを使用して以前に作成されたメジャーを削除できるようにします。

var targetTable = Model.Tables["Program"]; // メジャーを格納するテーブルの名前
var measureMetadata = ReadFile(@"c:001Test\MyMeasures.tsv"); // c:001Test\MyMeasures.tsv は、ヘッダ行と 3 つの列を持つタブ区切りのファイルです。Name、Description、Expression の 3 つの列があります。

// ターゲット・テーブルから、値が "1" の "AUTOGEN" アノテーションを持つすべてのメジャーを削除します。
foreach(var m in targetTable.Measures.Where(m => m.GetAnnotation("AUTOGEN") == "1").ToList())
{
    m.Delete();
}

// CR文字とLF文字で行に分割します。
var tsvRows = measureMetadata.Split(new[] {'\r','\n'},StringSplitOptions.RemoveEmptyEntries);

// すべての行をループしますが、最初の行はスキップします。
foreach(var row in tsvRows.Skip(1))
{
    var tsvColumns = row.Split('˶ˆ꒳ˆ˵'); // ファイルがタブをカラムセパレータとして使用していると仮定する
    var name = tsvColumns[0]; // 1 列目にメジャー名が含まれます。
    var description = tsvColumns[1]; // 2 番目の列にはメジャーの説明が含まれます。
    var expression = tsvColumns[2]; // 3 列目にメジャーの式が含まれます。

    // これは、モデルに同じ名前のメジャーが既に含まれていないことを前提としています (含まれている場合、新しいメジャーには数字の接尾辞が付きます)。
    var measure = targetTable.AddMeasure(name);
    measure.Description = 説明。
    measure.Expression = expression;
    measure.SetAnnotation("AUTOGEN", "1"); // メジャーに特別な注釈を設定して、次回のスクリプト実行時に見つけて削除できるようにします。
}
この処理を自動化する必要がある場合は、上記のスクリプトをファイルに保存し、Tabular Editor CLIを以下のように使用します。

start /wait TabularEditor.exe "<path to bim file>" -S "<path to script file>" -B "<path to modified bim file>"
のようにします。

start /wait TabularEditor.exe "c:ЯeaLAdventureWorks\Model.bim" -S "c:ЯeaLAutogenMeasures.cs" -B "c:ЯeaLAdventureWorks˶Build˶Model.bim"
...または、すでにデプロイされたデータベースに対してスクリプトを実行する場合は、以下のようになります。

start /wait TabularEditor.exe "localhost" "AdventureWorks" -S "c:\\AutogenMeasures.cs" -D "localhost" "AdventureWorks" -O
パーティションソースのメタデータからデータカラムを作成する
注：バージョン2.7.2以降をお使いの方は、新機能の「テーブルのインポート...」をぜひお試しください。

OLE DB プロバイダーのデータソースに基づいて Query パーティションを使用しているテーブルの場合、以下のスニペットを実行することで、そのテーブルのカラムメタデータを自動的に更新することができます。

Model.Tables["Reseller Sales"].RefreshDataColumns();
これは、モデルに新しいテーブルを追加するときに便利で、テーブル上のすべてのデータカラムを手動で作成する必要がなくなります。上記のスニペットは、「Reseller Sales」テーブルのパーティション・ソースの既存の接続文字列を使って、パーティション・ソースにローカルでアクセスできることを前提としています。上記のスニペットは、パーティション・クエリからスキーマを抽出し、ソース・クエリの各カラムのデータ・カラムをテーブルに追加します。

この操作に別の接続文字列を指定する必要がある場合は、スニペットの中でそれを行うこともできます。

var source = Model.DataSources["DWH"] as ProviderDataSource;
var source = Model.DataSources["DWH"] as ProviderDataSource; var oldConnectionString = source.ConnectionString;
source.ConnectionString = "..."; // メタデータの更新に使用する接続文字列を入力する
Model.Tables["Reseller Sales"].RefreshDataColumns();
source.ConnectionString = oldConnectionString;
これは、「Reseller Sales」テーブルのパーティションが、「DWH」という名前のプロバイダ・データソースを使用していることを想定しています。

DAX式のフォーマット
詳細については、FormatDaxを参照してください。

// Tabular Editor バージョン 2.13.0 以降で動作します。
Selected.Measures.FormatDax()です。
別の構文です。

// Tabular Editorのバージョン2.13.0以降で動作します。
foreach(var m in Selected.Measures)
    m.FormatDax();
テーブルのソースカラムのリストを生成する
次のスクリプトは、現在選択されているテーブルのソースカラムのリストをきれいにフォーマットして出力します。これは、SELECT * を使用するパーティション・クエリを明示的な列で置き換える場合に便利です。

string.Join(",r\n",
    選択されたテーブルのデータカラム
        .OrderBy(c => c.SourceColumn)
        .Select(c => "[" + c.SourceColumn + "]")
    ).Output()となります。
リレーションシップの自動作成
チーム内で特定の命名規則を一貫して使用している場合は、スクリプトがさらに強力なものになることがすぐにわかるでしょう。

次のスクリプトを 1 つ以上のファクト・テーブルで実行すると、列名に基づいて、関連するすべてのディメンジョン・テーブルへのリ レーションシップが自動的に作成されます。このスクリプトは、名前パターン xxxyyyKey を持つファクト・テーブル列を検索します。xxx はロールプレイ用のオプションの修飾語で、yyy はディメンジョン・テーブル名です。ディメンション・テーブルにはyyyKeyという名前の列が存在し、ファクト・テーブルの列と同じデータ型である必要があります。例えば、"ProductKey "という名前の列は、Productテーブルの "ProductKey "列と関連します。Key」の代わりに使用する別の列名サフィックスを指定することもできます。

ファクト・テーブルとディメンジョン・テーブルの間に既にリレーションシップが存在する場合、スクリプトは非アクティブとして新しいリレーションシップを作成します。

var keySuffix = "Key";

// 現在選択されているすべてのテーブル(ファクト・テーブルと仮定)をループします。
foreach(var fact in Selected.Tables)
{
    // 現在のテーブル上のすべての SK カラムをループします。
    foreach(var factColumn in fact.Columns.Where(c => c.Name.EndsWith(keySuffix)))
    {
        // 現在のSK列に対応するディメンション・テーブルを検索します。
        var dim = Model.Tables.FirstOrDefault(t => factColumn.Name.EndsWith(t.Name + keySuffix));
        if(dim != null)
        {
            // ディメンション・テーブルのキー・カラムを見つけます。
            var dimColumn = dim.Columns.FirstOrDefault(c => factColumn.Name.EndsWith(c.Name));
            if(dimColumn != null)
            {
                // 2つの列の間に既に関係が存在するかどうかをチェックします。
                if(!Model.Relationships.Any(r => r.FromColumn == factColumn && r.ToColumn == dimColumn))
                {
                    // 2つのテーブルの間に既に関係が存在する場合、新しい関係は非アクティブとして作成されます。
                    var makeInactive = Model.Relations.Any(r => r.FromTable == fact && r.ToTable == dim);

                    // 新しいリレーションシップを追加します。
                    var rel = Model.AddRelationship();
                    rel.FromColumn = factColumn;
                    rel.ToColumn = dimColumn;
                    factColumn.IsHidden = true;
                    if(makeInactive) rel.IsActive = false;
                }
            }
        }
    }
}
DumpFilters メジャーの作成
この記事に触発されて、現在選択されているテーブルに [DumpFilters] メジャーを作成するスクリプトを紹介します。

var dax = "VAR MaxFilters = 3 RETURN ";
var dumpFilterDax = @"IF (
    isfiltered ( {0} ),
    VAR ___f = FILTERS ( {0} )
VAR___r = COUNTROWS ( ___f )
VAR___t = TOPN ( MaxFilters, ___f, {0} )
VAR___d = CONCATENATEX ( ___t, {0}, "", "" )
VAR___x = ""{0} = "" & ___d
        & IF(___r > MaxFilters, "", ... ["" & ___r & "" items selected]") & "" ""
    RETURN ___x & UNICHAR(13) & UNICHAR(10)
)";

// モデルのすべての列をループして、完全なDAX式を構築する。
bool first = true;
foreach(var column in Model.AllColumns)
{
    if(!first) dax += " & ";
    dax += string.Format(dumpFilterDax, column.DaxObjectFullName);
    if(first) first = false;
}

// 現在選択されているテーブルにメジャーを追加します。
Selected.Table.AddMeasure("DumpFilters", dax);
キャメルケースから適切なケースへ
リレーションデータベースのカラムやテーブルの一般的な命名方法は、キャメルケースです。つまり、名前にはスペースが含まれず、個々の単語は大文字で始まります。Tabularモデルでは、隠されていないテーブルやカラムがビジネスユーザに見えるので、「より美しい」命名法を使用することが望ましい場合があります。次のスクリプトは、CamelCasedの名前をProper Caseに変換します。大文字の連続はそのまま維持されます（アクロニム）。たとえば、このスクリプトは以下のように変換します。

CustomerWorkZipcode」→「お客様の仕事用郵便番号
CustomerAccountID → お客様のアカウントID
NSASecurityID → NSAセキュリティID
このスクリプトをカスタムアクションとして保存し、すべてのオブジェクトタイプに適用することを強くお勧めします（リレーションシップ、KPI、テーブル権限、翻訳を除く、これらのオブジェクトには編集可能な「名前」プロパティがないため）。

foreach(var obj in Selected.OfType<ITabularNamedObject>()) { 。
    var oldName = obj.Name;
    var newName = new System.Text.StringBuilder();
    for(int i = 0; i < oldName.Length; i++) {。
        // 最初の文字は常に大文字でなければなりません。
        if(i == 0) newName.Append(Char.ToUpper(oldName[i]));

        // 大文字2文字の後に小文字が続く場合は、最初の文字の後にスペースを挿入する必要があります。
        // 最初の文字の後にスペースを挿入する必要があります。
        else if(i + 2 < oldName.Length && char.IsLower(oldName[i + 2]) && char.IsUpper(oldName[i + 1]) && char.IsUpper(oldName[i]))
        {
            newName.Append(oldName[i]);
            newName.Append(" ");
        }

        // 小文字の後に大文字が続くその他のシーケンスは、最初の文字の後にスペースを入れる必要があります。
        // 最初の文字の後にスペースを入れる必要があります。
        else if(i + 1 < oldName.Length && char.IsLower(oldName[i]) && char.IsUpper(oldName[i+1]))
        {
            newName.Append(oldName[i]);
            newName.Append(" ");
        }
        その他
        {
            newName.Append(oldName[i]);
        }
    }
    obj.Name = newName.ToString();
}
テーブルとメジャー間の依存関係のエクスポート
大規模で複雑なモデルがあり、基礎となるデータの変更によってどのメジャーが影響を受ける可能性があるかを知りたいとします。

以下のスクリプトは、モデルのすべてのメジャーをループし、各メジャーについて、そのメジャーが直接および間接的に依存しているテーブルのリストを出力します。このリストは、タブで区切られたファイルとして出力されます。

string tsv = "Measure\DependsOnTable"; // TSV ファイルのヘッダ行

// すべてのメジャーをループします。
foreach(var m in Model.AllMeasures) {。

    // このメジャーが参照するすべてのオブジェクトのリストを取得します (直接および他のメジャーを介して間接的に)。
    var allReferences = m.DependsOn.Deep();

    // 前述の参照リストをテーブル参照のみにフィルタリングします。列の参照については、各列が属するテーブルを取得しましょう。
    // 各カラムが属するテーブルを取得します。最後に、異なるテーブルのみを保持します。
    var allTableReferences = allReferences.OfType<Table>()
        .Concat(allReferences.OfType<Column>().Select(c => c.Table)).Distinct();

    // 各テーブルの参照ごとにTSVの行を出力します。
    foreach(var t in allTableReferences)
        tsv += string.Format("\\n{0}\t{1}", m.Name, t.Name);
}

tsv.Output();
// SaveFile("c:‾‾MyProjects‾‾SSAS‾‾MeasureTableDependencies.tsv", tsv); // 出力をファイルに保存する場合は、この行のコメントを外してください。
集約の設定（Power BIデータセットのみ）
Tabular Editor 2.11.3より、列にAlternateOfプロパティを設定できるようになり、モデルに集約テーブルを定義できるようになりました。この機能は、Power BIサービスのXMLAエンドポイントを通じて、Power BIデータセット（互換性レベル1460以上）に対して有効です。

カラムの範囲を選択し、以下のスクリプトを実行して、カラムのAlternateOfプロパティを開始します。

foreach(var col in Selected.Columns) col.AddAlternateOf();
列を1つずつ確認しながら、基本となる列にマッピングし、それに応じて要約を設定します（Sum/Min/Max/GroupBy）。また、この処理を自動化したい場合で、集約テーブルのカラムが基本テーブルのカラムと同じ名前の場合は、以下のスクリプトを使用すると、カラムのマッピングを行ってくれます。

// ツリーで2つのテーブルを選択する（ctrl+クリック）。集約テーブルは、列数が最も少ないものとします。
// このスクリプトは、アグリゲーションテーブルのすべてのカラムにAlternateOfプロパティを設定します。アグリゲーションテーブルのカラムは
// このスクリプトが動作するためには、ベーステーブルのカラムと同じ名前を持つ必要があります。
var aggTable = Selected.Tables.OrderBy(t => t.Columns.Count).First();
var baseTable = Selected.Tables.OrderByDescending(t => t.Columns.Count).First();

foreach(var col in aggTable.Columns)
{
    var summarization = SummarizationType // カラムがデータタイプ decimal/double を使用していない限り、スクリプトはサマリタイプを "Group By" に設定します。
    var summarization = SummarizationType.GroupBy;
    if(col.DataType == DataType.Double || col.DataType == DataType.Decimal)
        summarization = SummarizationType.Sum;

    col.AddAlternateOf(baseTable.Columns[col.Name], summarization);
}
このスクリプトを実行すると、AlternateOf プロパティがアッグテーブルのすべての列に割り当てられていることがわかります (以下のスクリーンショットを参照)。集約を行うには、ベーステーブルのパーティションでDirectQueryを使用する必要があることに注意してください。

<https://user-images.githubusercontent.com/8976200/85851134-6ed70800-b7ae-11ea-82eb-37fcaa2ca9c4.png>

分析サービスへの問い合わせ
バージョン 2.12.1 以降、Tabular Editor にはモデルに対して DAX クエリを実行したり DAX 式を評価したりするためのヘルパーメソッドがいくつか用意されています。これらのメソッドは、「ファイル > 開く > DBから...」オプションを使用した場合や、Tabular EditorのPower BI外部ツール統合を使用した場合など、モデルのメタデータがAnalysis Servicesのインスタンスから直接読み込まれた場合にのみ動作します。

以下の方法があります。

メソッドの説明
void ExecuteCommand(string tmsl) このメソッドは、指定された TMSL スクリプトを、接続された Analysis Services のインスタンスに渡します。これは、AS インスタンス上のテーブルのデータを更新したい場合に便利です。なお、このメソッドを使用してモデルのメタデータを変更すると、ローカルのモデルのメタデータがASインスタンス上のメタデータと同期しなくなり、次にモデルのメタデータを保存しようとするとバージョンの競合警告が表示される可能性があります。
IDataReader ExecuteReader(string dax) 接続されているASデータベースに対して指定されたDAXクエリを実行し、結果としてAmoDataReaderオブジェクトを返します。一度に複数のデータリーダーを開くことはできないことに注意してください。リーダを明示的に閉じるか処分するのを忘れた場合、Tabular Editorは自動的にそれらを閉じます。
DataSet ExecuteDax(string dax) 接続されているASデータベースに対して指定されたDAXクエリを実行し、クエリから返されたデータを含むDataSetオブジェクトを返します。非常に大きなデータテーブルを返すと、メモリ不足やその他の安定性に関するエラーが発生する可能性があるため、お勧めできません。
Object EvaluateDax(string dax) 接続されているASデータベースに対して指定されたDAX式を実行し、その結果を表すオブジェクトを返します。DAX式がスカラーの場合は、関連するタイプのオブジェクトが返されます（string、long、decimal、double、DateTime）。DAX 式がテーブル値の場合は、DataTable が返されます。
これらのメソッドはModel.Databaseオブジェクトにスコープされていますが、プレフィックスなしで直接実行することもできます。

Darren Gosbell氏は、ExecuteDaxメソッドを使用してデータ駆動型のメジャーを生成する興味深い使用例をここで紹介しています。

もう一つの方法は、テーブルを更新するための再利用可能なスクリプトを作成することです。例えば、再計算を行うには次のようにします。

var type = "calculate";
var database = Model.Database.Name;
var table = Selected.Table.Name;;
var tmsl = "{ ˶ˆ꒳ˆ˵": {type\ \type\」：「%type%\」、「objects\」：「オブジェクト」。[ { ˶‾᷄덴‾᷅˵ } ] } }"
    .Replace("%type%", type)
    .Replace("%db%", database)
    .Replace("%table%", table);

ExecuteCommand(tmsl)を実行します。
クエリ結果の視覚化
Outputヘルパー・メソッドを使用して、EvaluateDaxから返されたDAX式の結果を直接視覚化することもできます。

EvaluateDax("1 + 2").Output(); // 整数の場合
EvaluateDax("Hello from AS\").Output(); // 文字列の場合
EvaluateDax("{ (1, 2, 3) }").Output(); // 表
<https://user-images.githubusercontent.com/8976200/91638299-bbd59580-ea0e-11ea-882b-55bff73c30fb.png>

...または、現在選択されているメジャーの値を返したい場合は、以下のようになります。

EvaluateDax(Selected.Measure.DaxObjectFullName).Output();
<https://user-images.githubusercontent.com/8976200/91638367-6f3e8a00-ea0f-11ea-90cd-7d2e4cff6e31.png>

また、複数のメジャーを一度に選択して評価することができる、より高度な例を紹介します。

var dax = "ROW(" + string.Join(",", Selected.Measures.Select(m => "\" + m.Name + "\", " + m.DaxObjectFullName).ToArray()) + ")";
EvaluateDax(dax).Output()を実行します。
<https://user-images.githubusercontent.com/8976200/91638356-546c1580-ea0f-11ea-8302-3e40829e00dd.png>

上級者向けには、SUMMARIZECOLUMNS またはその他の DAX 関数を使用して、選択したメジャーを列でスライスして視覚化することもできます。

var dax = "SUMMARIZECOLUMNS('Product'[Color], " + string.Join(",", Selected.Measures.Select(m => "˶" + m.Name + "˶", " + m.DaxObjectFullName).ToArray())。+ ")";
EvaluateDax(dax).Output()を実行します。
<https://user-images.githubusercontent.com/8976200/91638389-9b5a0b00-ea0f-11ea-819f-d3eee3ddfa71.png>

これらのスクリプトをカスタムアクションとして保存するには、スクリプトエディタのすぐ上にある「+」アイコンをクリックしてください。このようにして、再利用が容易なDAXクエリのコレクションを手に入れ、タブラーエディタのコンテキストメニューから直接実行したり、表示したりすることができます。

<https://user-images.githubusercontent.com/8976200/91638790-305e0380-ea12-11ea-9d84-313f4388496f.png>

データのエクスポート
以下のスクリプトを使用して、DAX クエリを評価し、その結果をファイルにストリームすることができます（このスクリプトではタブ区切りのファイル形式を使用しています）。

using System.IO;

// このスクリプトは、DAX クエリを評価し、タブ区切りのフォーマットを使用して結果をファイルに書き込みます。

var dax = "EVALUATE 'Customer'";
var file = @"c:temp\file.csv";
var columnSeparator = "˶ˆ꒳ˆ˵";

using(var daxReader = ExecuteReader(dax))
using(var fileWriter = new StreamWriter(file))
{
    // カラムのヘッダーを書きます。
    fileWriter.WriteLine(string.Join(columnSeparator, Enumerable.Range(0, daxReader.FieldCount - 1).Select(f => daxReader.GetName(f)))).);

    while(daxReader.Read())
    {
        var rowValues = new object[daxReader.FieldCount];
        daxReader.GetValues(rowValues);
        var row = string.Join(columnSeparator, rowValues.Select(v => v == null ? "" : v.ToString())を入力します。）
        fileWriter.WriteLine(row);
    }
}
これらのメソッドの他の面白い使い方を思いついたら、コミュニティスクリプトのリポジトリで共有することをご検討ください。ありがとうございました。

Power Queryサーバーとデータベース名の置換
SQL ServerベースのデータソースからデータをインポートするPower BIデータセットには、以下のようなM式が含まれていることがよくあります。Tabular Editorには、残念ながらこのような式を「解析」する仕組みはありませんが、元の値を知らずにこの式のサーバー名とデータベース名を何か別のものに置き換えたい場合は、値が二重引用符で囲まれていることを利用することができます。

では
    Source = Sql.Databases("devsql.database.windows.net"),
    AdventureWorksDW2017 = Source{[Name="AdventureWorks"]}[Data],
    dbo_DimProduct = AdventureWorksDW2017{[Schema="dbo",Item="DimProduct"]}[Data] とします。
で
    dbo_DimProduct
次のスクリプトは、最初に出現する二重引用符で囲まれた値をサーバー名に、二番目に出現する二重引用符で囲まれた値をデータベース名に置き換えます。どちらの置換値も環境変数から読み込まれます。

// このスクリプトを使用して、すべてのパワークエリパーティションのサーバー名とデータベース名を置き換えます。
// すべてのパワークエリパーティションのサーバ名とデータベース名を、環境変数で指定されたものに置き換えるために使用されます。
// 変数で指定されたものに置き換えます。
var サーバー = "٩(*´ ꒳ `*)۶" + Environment.GetEnvironmentVariable("SQLServerName") + "˶ˆ꒳ˆ˵";
var データベース = "\"" + Environment.GetEnvironmentVariable("SQLDatabaseName") + "\"";

// この関数は、M式から引用符で囲まれたすべての値を抽出し、抽出された値を含む文字列のリストを返します。
// ただし，引用符の前にハッシュタグ (#) が付いているものは無視します．
// ただし、ハッシュタグ(#)が引用符の前にある場合は無視します。
var split = new Func<string, List<string>>(m => {)
    var result = new List<string>();
    var i = 0;
    foreach(var s in m.Split('"')) { 。
        if(s.EndsWith("#") && i % 2 == 0) i = -2;
        if(i >= 0 && i % 2 == 1) result.Add(s);
        i++;
    }
    return result;
});
var GetServer = new Func<string, string>(m => split(m)[0]); // サーバー名は通常、最初に遭遇する文字列です。
var GetDatabase = new Func<string, string>(m => split(m)[1]); // データベース名は通常2番目に遭遇する文字列です。

// モデル上のすべてのパーティションをループし、パーティションのサーバー名とデータベース名を
// 環境変数で指定されたものに置き換えます。
foreach(var p in Model.AllPartitions.OfType<MPartition>())
{
    var oldServer = "٩(*´ ꒳ `*)۶" + GetServer(p.Expression) + "\"";
    var oldDatabase = "˶ˆ꒳ˆ˵" + "˶ˆ꒳ˆ˵" + GetDatabase(p.Expression) + "˶ˆ꒳ˆ˵";
    p.Expression = p.Expression.Replace(oldServer, server).Replace(oldDatabase, database);
}
Power QueryのデータソースやパーティションをLegacyに置き換える
Power BIベースのモデルで、SQL Serverベースのデータソースに対するパーティションにPower Query（M）の式を使用している場合、残念ながらTabular Editorのデータインポートウィザードを使用したり、スキーマチェック（インポートした列とデータソースの列を比較すること）を実行したりすることができません。

この問題を解決するには、モデル上で以下のスクリプトを実行して、パワー・クエリ・パーティションを対応するネイティブSQLクエリ・パーティションに置き換え、モデル上にレガシー（プロバイダ）データソースを作成して、Tabular Editorのデータ・インポート・ウィザードで動作させることができます。

このスクリプトには2つのバージョンがあります。最初のバージョンは、作成されたレガシーデータソース用のMSOLEDBSQLプロバイダと、ハードコードされた認証情報を使用します。これはローカル開発に便利です。2つ目のバージョンは、Azure DevOps上でMicrosoftがホストするビルドエージェントで利用可能なSQLNCLIプロバイダを使用し、環境変数から資格情報やサーバー/データベース名を読み取るため、Azure Pipeliensでの統合に便利なスクリプトとなっています。

MSOLEDBSQLバージョンでは、Mパーティションから接続情報を読み取り、Azure ADを介してユーザー名とパスワードの入力を促します。

# r "Microsoft.VisualBasic"

// このスクリプトは、このモデルのすべての Power Query パーティションを
// 提供された接続文字列を使用してレガシーパーティションをINTERACTIVE
// AAD 認証を行います。このスクリプトでは、すべての Power Query パーティションが
// 同じ SQL Server ベースのデータソースからデータを読み込みます。

// 以下の情報を提供してください。
var authMode = "ActiveDirectoryInteractive";
var userId = Microsoft.VisualBasic.Interaction.InputBox("Type your AAD user name", "User name", "name@domain.com", 0, 0);
if(userId == "") return;
var password = ""; // ActiveDirectoryInteractive認証を使用している場合は空欄にします。

// この関数は、M式から引用符で囲まれた値をすべて抽出し、抽出された値を持つ文字列のリストを返します。
// ただし、引用符の前にハッシュタグ(#)が付いているものは無視します。
// ただし、ハッシュタグ(#)が引用符の前にある場合は無視します。
var split = new Func<string, List<string>>(m => {)
    var result = new List<string>();
    var i = 0;
    foreach(var s in m.Split('"')) { 。
        if(s.EndsWith("#") && i % 2 == 0) i = -2;
        if(i >= 0 && i % 2 == 1) result.Add(s);
        i++;
    }
    return result;
});
var GetServer = new Func<string, string>[m => split(m](0)); // サーバー名は通常、最初に遭遇する文字列です。
var GetDatabase = new Func<string, string>[m => split(m](1)); // データベース名は通常2番目に遭遇する文字列です。
var GetSchema = new Func<string, string>[m => split(m](2)); // スキーマ名は通常、3番目に遭遇する文字列です。
var GetTable = new Func<string, string>[m => split(m](3)); // テーブル名は通常、4番目に遭遇する文字列です。

var server = GetServer(Model.AllPartitions.OfType<MPartition>().First().Expression);
var database = GetDatabase(Model.AllPartitions.OfType<MPartition>().First().Expression);

// レガシーデータソースをモデルに追加します。
var ds = Model.AddDataSource("AzureSQL");
ds.Provider = "System.Data.OleDb";
ds.ConnectionString = string.Format(
    "Provider=MSOLEDBSQL;Data Source={0};Initial Catalog={1};Authentication={2};User ID={3};Password={4}",
    サーバーを使用します。
    データベースを使用します。
    authMode,
    userId,
    password)を使用します。

// すべてのテーブルから Power Query パーティションを削除し、単一の Legacy パーティションに置き換えます。
foreach(var t in Model.Tables.Where(t => t.Partitions.OfType<MPartition>().Any()))
{
    var mPartitions = t.Partitions.OfType<MPartition>();
    if(!mPartitions.Any())続きます。
    var schema = GetSchema(mPartitions.First().Expression);
    var table = GetTable(mPartitions.First().Expression);
    t.AddPartition(t.Name, string.Format("SELECT * FROM [{0}].[{1}]", schema, table));
    foreach(var p in mPartitions.ToList()) p.Delete();
}
環境変数から接続情報を読み取る SQLNCLI バージョン。

// このスクリプトは、このモデルのすべての Power Query パーティションをレガシーパーティションに置き換えます。
// このスクリプトは、このモデル上のすべての Power Query パーティションをレガシーパーティションに置き換え、 SQL サーバー名、データベース名、ユーザー名、パスワードを対応する環境変数から読み取ります。
// とパスワードを対応する環境変数から読み込みます。このスクリプトでは
// すべての Power Query パーティションが同じ SQL Server ベースのデータソースからデータを読み込むことを前提としています。
// データソースからデータを読み込むと仮定しています。

var server = Environment.GetEnvironmentVariable("SQLServerName");
var database = Environment.GetEnvironmentVariable("SQLDatabaseName");
var userId = Environment.GetEnvironmentVariable("SQLUserName");
var password = Environment.GetEnvironmentVariable("SQLUserPassword");

// この関数は、M 式からすべての引用符で囲まれた値を抽出し、抽出された値を含む文字列のリストを返します。
// 抽出された値を（順に）含む文字列のリストを返しますが、引用符の前にハッシュタグ（#）がある引用値は無視します。
// ただし、ハッシュタグ(#)が引用符の前にある場合は無視します。
var split = new Func<string, List<string>>(m => {)
    var result = new List<string>();
    var i = 0;
    foreach(var s in m.Split('"')) { 。
        if(s.EndsWith("#") && i % 2 == 0) i = -2;
        if(i >= 0 && i % 2 == 1) result.Add(s);
        i++;
    }
    return result;
});
var GetServer = new Func<string, string>[m => split(m](0)); // サーバー名は通常、最初に遭遇する文字列です。
var GetDatabase = new Func<string, string>[m => split(m](1)); // データベース名は通常2番目に遭遇する文字列です。
var GetSchema = new Func<string, string>[m => split(m](2)); // スキーマ名は通常、3番目に遭遇する文字列です。
var GetTable = new Func<string, string>[m => split(m](3)); // テーブル名は通常、4 番目に遭遇する文字列です。

// レガシーデータソースをモデルに追加します。
var ds = Model.AddDataSource("AzureSQL");
ds.Provider = "System.Data.SqlClient";
ds.ConnectionString = string.Format(
    "Server={0};Initial Catalog={1};Persist Security Info=False;User ID={2};Password={3}",
    サーバーを
    データベースの
    userId,
    password)を使用します。

// すべてのテーブルから Power Query パーティションを削除し、単一の Legacy パーティションに置き換えます。
foreach(var t in Model.Tables.Where(t => t.Partitions.OfType<MPartition>().Any()))
{
    var mPartitions = t.Partitions.OfType<MPartition>();
    if(!mPartitions.Any())続きます。
    var schema = GetSchema(mPartitions.First().Expression);
    var table = GetTable(mPartitions.First().Expression);
    t.AddPartition(t.Name, string.Format("SELECT * FROM [{0}].[{1}]", schema, table));
    foreach(var p in mPartitions.ToList()) p.Delete();
}
