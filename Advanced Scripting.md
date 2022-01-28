高度なスクリプティング
ここでは、Tabular Editorの高度なスクリプティング機能についてご紹介します。このドキュメントの情報は変更されることがあります。また、便利なスクリプトスニペットの記事では、Tabular Editorのスクリプト機能でできることの実例をいくつか紹介しています。

高度なスクリプティングとは？
Tabular EditorのUIの目標は、Tabularモデルを構築する際によく必要とされるほとんどのタスクを簡単に実行できるようにすることです。例えば、複数のメジャーの表示フォルダを一度に変更するには、エクスプローラー・ツリーでオブジェクトを選択し、ドラッグ・アンド・ドロップするだけです。エクスプローラツリーの右クリックコンテキストメニューでは、パースペクティブにオブジェクトを追加/削除したり、複数のオブジェクトの名前を変更したりするなど、これらのタスクの多くを便利に実行できます。

しかし、他の多くの一般的なワークフローのタスクは、UIでは簡単に実行できないかもしれません。このため、Tabular EditorにはAdvanced Scriptingが導入されており、上級ユーザがC#の構文を使ってスクリプトを書き、ロードされたTabular Modelのオブジェクトをより直接的に操作できるようになっています。

オブジェクト
スクリプトAPIは、ModelとSelectedという2つのトップレベルオブジェクトにアクセスできます。前者には、Tabular Modelのすべてのオブジェクトを操作するためのメソッドとプロパティが含まれており、後者には、エクスプローラツリーで現在選択されているオブジェクトのみが公開されます。

Modelオブジェクトは、Microsoft.AnalysisServices.Tabular.Modelクラスのラッパーであり、そのプロパティのサブセットを公開しています。また、トランスレーション、パースペクティブ、オブジェクトコレクションの操作を容易にするために、いくつかのメソッドとプロパティが追加されています。同様のことが、Table、Measure、Columnなどの子孫オブジェクトにも当てはまり、これらはすべて対応するラッパー・オブジェクトを持っています。タブラー・エディターのラッパー・ライブラリーのオブジェクト、プロパティ、メソッドの完全なリストは付録Aをご覧ください。

このラッパーを使って作業することの主な利点は、すべての変更がTabular Editor UIから元に戻せることです。スクリプトを実行した後にCTRL+Zを押すだけで、スクリプトによって行われたすべての変更がすぐに取り消されるのがわかります。さらに、ラッパーは、多くの一般的なタスクを簡単なワンライナーに変える便利なメソッドを提供しています。以下、いくつかの例をご紹介します。ここでは、Tabular Editorsのスクリプト機能のこれらの側面については説明しませんので、読者はすでにC#とLINQにある程度精通していることを前提としています。C#やLINQに精通していないユーザーでも、以下の例に従うことができるはずです。

オブジェクトのプロパティの設定
特定のオブジェクトのプロパティを変更したい場合、当然ながら最も簡単な方法はUIから直接行うことです。しかし、例として、同じことをスクリプトで実現する方法を見てみましょう。

例えば、「FactInternetSales」テーブルの「Sales Amount」メジャーのフォーマット文字列を変更したいとします。エクスプローラ・ツリーでメジャーを見つけたら、それをスクリプト・エディタにドラッグします。Tabular Editor は、Tabular オブジェクト・モデルでこの特定のメジャーを表す以下のコードを生成します。

Model.Tables["FactInternetSales"].Measures["Sales Amount"].
右端のブラケットの後にドット (.) を追加すると、オートコンプリートメニューが表示され、この特定のメジャーに存在するプロパティやメソッドが表示されます。メニューから「FormatString」を選択するか、最初の数文字を入力してTabキーを押します。次に、等号の後に「0.0%」と入力します。また、このメジャーの表示フォルダを変更してみましょう。最終的なコードは以下のようになります。

Model.Tables["FactInternetSales"].Measures["Sales Amount"].FormatString = "0.0%";
Model.Tables["FactInternetSales"].Measures["Sales Amount"].DisplayFolder = "New Folder";
注：各行の最後にセミコロン（;）を入れることを忘れないでください。これはC#の要求事項です。これを忘れてしまうと、スクリプトを実行するときに構文エラーが表示されてしまいます。

F5キーを押すか、スクリプトエディタの上にある「Play」ボタンを押して、スクリプトを実行します。すぐに、変更されたディスプレイ・フォルダを反映して、エクスプローラ・ツリー内でメジャーが移動するのが確認できます。メジャーをプロパティ・グリッドで確認すると、Format String プロパティも変更されていることがわかります。

複数のオブジェクトを一度に操作する
オブジェクト・モデルの多くのオブジェクトは、実際には複数のオブジェクトのコレクションです。例えば、各 Table オブジェクトは Measures コレクションを持っています。ラッパーでは、これらのコレクションに対して一連の便利なプロパティやメソッドを公開しており、複数のオブジェクトに対して同じプロパティを一度に簡単に設定することができます。この点については、以下で詳しく説明します。さらに、すべての標準的なLINQ拡張メソッドを使用して、コレクションのオブジェクトをフィルタリングおよび参照することができます。

以下に、最も一般的に使用されるLINQ拡張メソッドの例をいくつか示します。

Collection.First([predicate]) オプションの[predicate]条件を満たす、コレクション内の最初のオブジェクトを返します。
Collection.Any([predicate]) 任意のオブジェクトがコレクションに含まれている場合、trueを返します（オプションで[predicate]の条件を満たします）。
Collection.Where(predicate) 述語の条件でフィルタリングされたオリジナルのコレクションを返します。
Collection.Select(map) コレクション内の各オブジェクトを、指定されたマップに従って別のオブジェクトに投影します。
Collection.ForEach(action) コレクション内の各要素に対して指定されたアクションを実行します。
上記の例では、predicateは、1つのオブジェクトを入力として受け取り、ブール値を出力として返すラムダ式です。例えば、Collectionがメジャーのコレクションである場合、典型的な述語は次のようになります。

m => m.Name.Contains("Reseller")

この述語は、メジャーの Name に文字列 "Reseller" が含まれている場合にのみ true を返します。より高度なロジックが必要な場合は、式を中括弧で囲み、return キーワードを使用します。

.Where(obj => {)
    if(obj is Column) {
        falseを返します。
    }
    return obj.Name.Contains("test");
})
上の例に戻ると、mapは入力として単一のオブジェクトを受け取り、出力として任意の単一のオブジェクトを返すラムダ式です。actionは入力として単一のオブジェクトを受け取りますが、値を返さないラムダ式です。

他にどのようなLINQメソッドがあるかは、アドバンスド・スクリプト・エディタのインテリセンス機能を使用するか、LINQ-to-Objectsのドキュメントを参照してください。

Modelオブジェクトの操作
現在ロードされているTabular Modelの任意のオブジェクトを素早く参照するには、エクスプローラツリーからAdvanced Scriptingエディタにオブジェクトをドラッグ＆ドロップします。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/DragDropTOM.gif

モデルとその子孫オブジェクトに存在するプロパティの概要については、TOMドキュメントを参照してください。また、ラッパー・オブジェクトが公開するプロパティとメソッドの完全なリストについては、付録Aを参照してください。

選択されたオブジェクトの操作
Tabular Modelの任意のオブジェクトを明示的に参照できることは、一部のワークフローには最適ですが、エクスプローラツリーからオブジェクトを選択して、選択されたオブジェクトに対してのみスクリプトを実行したい場合もあります。そんなときにはSelectedオブジェクトが便利です。

Selectedオブジェクトには、現在何が選択されているかを簡単に識別するためのさまざまなプロパティが用意されており、特定のタイプのオブジェクトに選択を限定することもできます。フォルダーを表示してブラウズしているときに、エクスプローラーのツリーで1つ以上のフォルダーが選択されていると、その子アイテムもすべて選択されているとみなされます。単一選択の場合は、アクセスしたいオブジェクトの種類を表す単数形の名前を使用します。例えば、以下のようになります。

Selected.Hierarchy

は、ツリー内で現在選択されている階層を指します。複数の選択項目を扱う場合は、複数形の名前を使用してください。

Selected.Hierarchies

単数形のオブジェクトに存在するすべてのプロパティは、いくつかの例外を除いて、複数形にも存在します。つまり、前述のLINQ拡張メソッドを使わずに、たった1行のコードで、一度に複数のオブジェクトに対してこれらのプロパティの値を設定することができるのです。例えば、現在選択されているすべてのメジャーを「Test」という名前の新しい表示フォルダに移動させたいとします。

Selected.Measures.DisplayFolder = "Test";

ツリーでメジャーが現在選択されていない場合、上記のコードは何も行わず、エラーも発生しません。そうでない場合は、選択されたすべてのメジャーの DisplayFolder プロパティが "Test" に設定されます (Selected オブジェクトには選択されたフォルダ内のオブジェクトも含まれるため、フォルダ内に存在するメジャーも含まれます)。Measures ではなく Measure という単数形を使用すると、現在の選択対象にちょうど 1 つのメジャーが含まれていない限り、エ ラーが発生します。

複数のオブジェクトの Name プロパティを一度に設定することはできませんが、いくつかのオプションが用意されています。ある文字列のすべての出現箇所を別の文字列に置き換えたいだけの場合は、次のように提供されている「Rename」メソッドを使用できます。

Selected.Measures
        .Rename("Amount", "Value");
これにより、現在選択されているすべてのメジャーの名前で、「Amount」という単語が「Value」という単語に置き換えられます。また、上述の LINQ ForEach() メソッドを使用して、より高度なロジックを組み込むこともできます。

選択された.メジャー
        .ForEach(m => if(m.Name.Contains("Reseller")) m.Name += " DEPRECATED") を使用します。
この例では、選択されたすべてのメジャーの名前に「Reseller」という単語が含まれている場合、その名前に「DEPRECATED」というテキストが追加されます。別の方法として、LINQ 拡張メソッド Where() を使用して、ForEach() 操作を適用する前にコレクションをフィルタリングすることもできますが、この場合もまったく同じ結果になります。

Selected.Measures
        .Where(m => m.Name.Contains("Reseller"))
        .ForEach(m => m.Name += " DEPRECATED");
ヘルパーメソッド
スクリプトのデバッグを容易にするために、Tabular Editorには特別なヘルパーメソッドのセットが用意されています。内部的には、これらは[ScriptMethod]属性で装飾された静的メソッドです。この属性により、スクリプトは名前空間やクラス名を指定することなく、直接メソッドを呼び出すことができます。プラグインも同様に、[ScriptMethod]属性を使ってパブリックなスタティック・メソッドをスクリプト用に公開することができます。

2.7.4時点で、Tabular Editorは以下のスクリプトメソッドを提供しています。これらの中には、拡張メソッドとして呼び出されるものもあることに注意してください。例えば、object.Output();とOutput(object);は同等です。

Output(object); - スクリプトの実行を停止し、指定されたオブジェクトに関する情報を表示します。スクリプトがコマンドライン実行の一部として実行されている場合は、オブジェクトの文字列表現をコンソールに書き込みます。
SaveFile(filePath, content); - テキストデータをファイルに保存する便利な方法です。
ReadFile(filePath); - ファイルからテキストデータを読み込むのに便利です。
ExportProperties(objects, properties); - 複数のオブジェクトからプロパティのセットをTSV文字列としてエクスポートする便利な方法です。
ImportProperties(tsvData); - TSV文字列から複数のオブジェクトにプロパティをロードする便利な方法です。
CustomAction(name); - 名前を指定してカスタムアクションを起動します。
CustomAction(objects, name); - 指定されたオブジェクトにカスタムアクションを起動します。
ConvertDax(dax, useSemicolons); - DAX式をUS/UKと非US/UKのロケール間で変換します。useSemicolonsがtrue（デフォルト）の場合、dax文字列はネイティブのUS/UK形式から非US/UK形式に変換されます。つまり、カンマ（リストの区切り文字）はセミコロンに、ピリオド（小数点の区切り文字）はカンマに変換されます。useSemicolonsがfalseに設定されている場合は、その逆になります。
FormatDax(string dax); - www.daxformatter.com を使用して DAX 式をフォーマットします (非推奨、使用しないでください！)
FormatDax(IEnumerable<IDaxDependantObject> objects, bool shortFormat, bool? skipSpace) - 指定されたコレクション内の全てのオブジェクトに対して DAX 式をフォーマットします。
FormatDax(IDaxDependantObject obj) - スクリプトの実行が完了したときや、CallDaxFormatterメソッドが呼ばれたときに、DAX式のフォーマットのためにオブジェクトをキューに入れます。
CallDaxFormatter(bool shortFormat, bool? skipSpace) - これまでにキューに入れられたオブジェクトのすべてのDAX式をフォーマットします。
Info(string); - コンソールに情報メッセージを書き込みます(スクリプトがコマンドライン実行の一部として実行された場合のみ)。
Warning(string); - コンソールに警告メッセージを書き込みます (スクリプトがコマンドライン実行の一部として実行された場合のみ)。
Error(string); - コンソールにエラーメッセージを書き込みます（スクリプトがコマンドライン実行の一部として実行された場合のみ）。
スクリプトのデバッグ
前述のとおり、Output(object); メソッドを使用すると、スクリプトの実行を一時停止し、渡されたオブジェクトに関する情報を含むダイアログボックスを開くことができます。また、このメソッドを拡張メソッドとして使用し、object.Output();のように起動することもできます。ダイアログが閉じられると、スクリプトの実行が再開されます。

ダイアログは、出力されるオブジェクトの種類に応じて、4つの異なる方法のいずれかで表示されます。

オブジェクトの.ToString()メソッドを呼び出すことで、単数形のオブジェクト(文字列、整数、DateTimesなど、TabularNamedObjectから派生したオブジェクトを除く)は、シンプルなメッセージダイアログとして表示されます。
https://user-images.githubusercontent.com/8976200/29941982-9917d0cc-8e94-11e7-9e78-24aaf11fd311.png

単一のTabularNamedObject(Tabular Editorで利用可能なテーブル、メジャー、その他のTOM NamedMetadataObjectなど)は、ツリーエクスプローラでオブジェクトが選択されたときと同様に、プロパティグリッドに表示されます。オブジェクトのプロパティはグリッドで編集することができますが、スクリプト実行の後の時点でエラーが発生した場合、「エラー時のロールバック」が有効になっていれば、編集は自動的に取り消されることに注意してください。
https://user-images.githubusercontent.com/8976200/29941852-2acc9846-8e94-11e7-9380-f84fef26a78c.png

オブジェクトの任意の IEnumerable (TabularNamedObjects を除く) は、リストに表示され、各リスト項目には IEnumerable 内のオブジェクトの .ToString() 値とタイプが表示されます。
https://user-images.githubusercontent.com/8976200/29942113-02dad928-8e95-11e7-9c04-5bb87b396f3f.png

TabularNamedObjectsのIEnumerableを使用すると、ダイアログは左にオブジェクトのリストを、右にプロパティグリッドを表示します。プロパティグリッドには、リストで選択されているオブジェクトが入力され、単一のTabularNamedObjectが出力されているときと同様に、プロパティを編集することができます。
https://user-images.githubusercontent.com/8976200/29942190-498cbb5c-8e95-11e7-8455-32750767cf13.png

左下の「Don't show more outputs」チェックボックスにチェックを入れると、それ以降の.Output()の呼び出しでスクリプトが停止するのを防ぐことができます。

.NETリファレンス
Tabular Editorバージョン2.8.6では、複雑なスクリプトの作成が非常に簡単になりました。新しいプリプロセッサのおかげで、通常のC#ソースコードのように、usingキーワードを使ってクラス名などを短縮できるようになりました。さらに、Azure Functionsで使用される.csxスクリプトと同様に、#r "<アセンブリ名またはDLLパス>"という構文を使用して、外部アセンブリをインクルードすることができます。

例えば、以下のスクリプトは期待通りに動作するようになりました。

// アセンブリの参照は、ファイルの一番上にする必要があります。
#r "System.IO.Compression"

// 使用するキーワードは、他のステートメントの前になければなりません。
using System.IO.Compression;
using System.IO.Compression;;

var xyz = 123;

// Usingステートメントは、想定された通りに動作します。
using(var data = new MemoryStream())
using(var zip = new ZipArchive(data, ZipArchiveMode.Create)) 
{
   // ...
}
デフォルトでは、Tabular Editorは以下のusingキーワードを適用して（スクリプトで指定されていなくても）、一般的な作業を容易にします。

using System;
using System.Linq;
using System; using System.Linq; using System.Collections.Generic.Json
using Newtonsoft.Json;
using TabularEditor.TOMWrapper;
using TabularEditor.TOMWrapper.Utils;
using TabularEditor.UI;
また、以下の.NET Frameworkのアセンブリがデフォルトでロードされます。

システム.Dll
システム.コア.Dll
システム.データ.Dll
System.Dll System.Core.Dll System.Data.Dll System.Windows.Forms.Dll
Microsoft.Csharp.Dll
Newtonsoft.Json.Dll
TomWrapper.Dll
TabularEditor.Exe
Microsoft.AnalysisServices.Tabular.Dll
Roslynでのコンパイル
Visual Studio 2017で導入された新しいRoslynコンパイラーを使用してスクリプトをコンパイルしたい場合は、Tabular Editorバージョン2.12.2から、「ファイル」>「環境設定」>「一般」で設定できます。これにより、文字列補間などのより新しいC#言語の機能を使用することができます。コンパイラの実行ファイル（dsc.exe）が格納されているディレクトリのパスを指定し、コンパイラのオプションとして言語バージョンを指定するだけです。

https://user-images.githubusercontent.com/8976200/92464140-0902f580-f1cd-11ea-998a-b6ecce57b399.png

Visual Studio 2017
典型的なVisual Studio 2017 Enterpriseのインストールでは、Roslynコンパイラーはここにあります。

c:Program Files (x86)‾‾Microsoft Visual Studio‾‾2017‾‾Enterprise‾‾MSBuild‾‾15.0‾‾Bin‾Roslyn
これには、C# 6.0の言語機能がデフォルトで含まれています。

https://user-images.githubusercontent.com/8976200/92464584-a52cfc80-f1cd-11ea-9b66-3b47ac36f6c6.png

Visual Studio 2019 の場合
一般的な Visual Studio 2019 Community のインストールでは、Roslyn コンパイラーはここにあります。

c:Program Files (x86)‾Microsoft Visual Studio‾2019‾Community‾MSBuild‾Current‾Bin‾Roslyn
VS2019 に同梱されているコンパイラーは、C# 8.0 の言語機能をサポートしており、コンパイラーオプションに以下を指定することで有効になります。

-ランゲージバージョン:8.0