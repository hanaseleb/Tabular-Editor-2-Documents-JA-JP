高度なオブジェクトフィルタリング
この記事では、複雑なモデルを操作する際に非常に便利な機能である、Tabular Editorの「フィルタ」テキストボックスの使い方を説明します。

フィルタリングモード
2.7.4以降、Tabular Editorでは、階層内のオブジェクトにフィルタを適用する方法や、検索結果の表示方法を決めることができるようになりました。これは、Filterボタンの隣にある右端の3つのツールバーボタンで制御します。

https://user-images.githubusercontent.com/8976200/46567931-08a4b480-c93d-11e8-96fd-e197e87a0587.png

https://user-images.githubusercontent.com/8976200/46567944-44d81500-c93d-11e8-91e2-d9822078dba7.pngHierarchical by parent: 検索は親オブジェクト、つまりテーブルやディスプレイフォルダ（これらが有効な場合）に適用されます。親アイテムが検索条件に一致した場合、すべての子アイテムが表示されます。
https://user-images.githubusercontent.com/8976200/46567940-2ffb8180-c93d-11e8-9fba-84fbb79b6bb3.pngHierarchical by children: 検索は、子オブジェクト (メジャー、列、階層など) に適用されます。親オブジェクトは、検索条件に一致する子オブジェクトが1つ以上ある場合にのみ表示されます。
https://user-images.githubusercontent.com/8976200/46567941-37bb2600-c93d-11e8-9c02-86502f41bce8.pngFlat: 検索はすべてのオブジェクトに適用され、結果はフラットリストで表示されます。子アイテムを含むオブジェクトは、これらを階層的に表示します。
簡単な検索
Filterのテキストボックスに何かを入力して[Enter]を押すと、オブジェクト名の大文字小文字を区別しないシンプルな検索ができます。たとえば、"親別 "のフィルタリングモードで、"sales "と入力すると、次のような結果が得られます。

https://user-images.githubusercontent.com/8976200/46568002-5f5ebe00-c93e-11e8-997b-7f89dfd92076.png

いずれかのテーブルを展開すると、そのテーブルのすべてのメジャー、列、階層、パーティションが表示されます。フィルタリング・モードを「子で」に変更すると、以下のような結果になります。

https://user-images.githubusercontent.com/8976200/46568016-9f25a580-c93e-11e8-9bc2-c0a16a890256.png

Employee」テーブルがリストに表示されていることに注目してください。これは、「sales」という単語を含むいくつかの子項目(この場合は列)を持っているからです。

ワイルドカード検索
フィルタ」テキストボックスに文字列を入力する際、ワイルドカード「?」は任意の1文字を表し、「*」は任意の連続した文字（0個以上）を表します。たとえば、*sales* と入力すると、上の図とまったく同じ結果が得られますが、sales* と入力すると、名前が "sales" で始まるオブジェクトのみが表示されます（これも大文字と小文字を区別しません）。

親でsales*を検索する。

https://user-images.githubusercontent.com/8976200/46568043-19eec080-c93f-11e8-8d81-2a6214bfa572.png

sales*を子で検索すると

https://user-images.githubusercontent.com/8976200/46568117-f9733600-c93f-11e8-96ab-f87769b8097c.png

sales*のフラット検索（情報欄を[Ctrl]+[F1]で切り替えると、各オブジェクトの詳細情報が表示されます）。

https://user-images.githubusercontent.com/8976200/46568118-042dcb00-c940-11e8-82d1-516207450559.png

ワイルドカードは文字列のどこにでも入れることができ、必要な数だけ含めることができます。これだけでは複雑ではないという方は、この先を読んでみてください...

ダイナミックLINQ検索
Dynamic LINQを使用してオブジェクトを検索することもできます。これはBest Practice Analyzerのルールを作成するときに行うのと同じです。フィルタボックスでDynamic LINQモードを有効にするには、検索文字列の前に : (コロン)を付けるだけです。例えば、名前が「Key」で終わるすべてのオブジェクトを表示するには、次のように記述します（大文字と小文字を区別します）。

:Name.EndsWith("Key")
...と入力し、[Enter]キーを押します。フラット」フィルタリングモードでは、結果は次のようになります。

https://user-images.githubusercontent.com/8976200/46568130-33dcd300-c940-11e8-903c-193e1acde0ad.png

Dynamic LINQで大文字と小文字を区別せずに検索するには、次のように入力文字列を変換することができます。

:Name.ToUpper().EndsWith("KEY")
または、次のようにStringComparison引数を指定します。

:Name.EndsWith("Key", StringComparison.IvariantCultureIgnoreCase)
オブジェクトの名前の中を検索することに制限はありません。ダイナミックLINQの検索文字列は、オブジェクトの任意のプロパティ（サブプロパティも含む）を評価するために、好きなだけ複雑にすることができます。例えば、"TODO "という単語を含む式を持つすべてのオブジェクトを検索したい場合、次のような検索フィルターを使用します。

:Expression.ToUpper().Contains("TODO")
別の例として、次のようにすると、モデル内で他から参照されていない隠れたメジャーがすべて表示されます。

:ObjectType="Measure" and (IsHidden or Table.IsHidden) and ReferencedBy.Count=0
正規表現を使用することもできます。以下は、名前に「Number」または「Amount」という単語が含まれるすべての列を検索します。

:ObjectType="Column" and RegEx.IsMatch(Name,"(Number)|(Amount)")
表示オプション（ツリーの直上にあるツールバーボタン）は、「親別」および「子別」のフィルタリングモードを使用する際の結果に影響を与える可能性があることに注意してください。例えば、上記のLINQフィルタは列のみを返しますが、表示オプションが「列を表示しない」に設定されている場合、何も表示されません。