FormatDaxの非推奨
FormatDaxメソッド（Tabular Editorで利用可能なヘルパーメソッドの1つ）は、Tabular Editor 2.13.0のリリースで非推奨となりました。

非推奨となった理由は、https://www.daxformatter.com/ のウェブサービスで、複数のリクエストが連続して発生して負荷が高くなり、問題が発生していたためです。これは、FormatDaxメソッドがスクリプトの中で呼び出されるたびにWebリクエストを実行するためで、多くの人が以下のようなスクリプトを使用していました。

これはやめてください

foreach(var m in Model.AllMeasures)
{
    // こんなことはしないでください
    m.Expression = FormatDax(m.Expression);
}
小節数が数十個の小規模なモデルでは問題ありませんが、www.daxformatter.com のトラフィックを見ると、上記のようなスクリプトが数千個の小節を持つ複数のモデルで、しかも1日に数回も実行されていることがわかります。

この問題に対処するため、Tabular Editor 2.13.0では、上記の構文でFormatDaxが連続して3回以上呼び出されると、警告が表示されます。さらに、その後の呼び出しは、各呼び出しの間に5秒の遅延を設けてスロットルされます。

代替構文
Tabular Editor 2.13.0では、FormatDaxを呼び出す2つの異なる方法が導入されています。上記のスクリプトは、以下のいずれかに書き換えることができます。

foreach(var m in Model.AllMeasures)
{
    m.FormatDax();
}
...あるいは単純に...。

Model.AllMeasures.FormatDax()です。
これらの方法は、すべての www.daxformatter.com 呼び出しを1つのリクエストにまとめます。お好みでグローバルメソッドの構文を使用することもできます。

foreach(var m in Model.AllMeasures)
{
    FormatDax(m);
}
...あるいは、単に...:

FormatDax(Model.AllMeasures)です。
詳細
技術的には、FormatDaxは2つのオーバーロードされた拡張メソッドとして実装されています。

void FormatDax(this IDaxDependantObject obj)
void FormatDax(this IEnumerable<IDaxDependantObject> objects, bool shortFormat = false, bool? skipSpaceAfterFunctionName = null)
上記のオーバーロード#1は、スクリプトの実行が完了したとき、または新しいvoid CallDaxFormatter()メソッドの呼び出しが行われたときに、提供されたオブジェクトをフォーマットのためにキューに入れます。オーバーロード#2は、列挙可能なオブジェクトで提供されたすべてのオブジェクトのすべてのDAX式をフォーマットする単一のWebリクエストで直ちにwww.daxformatter.com を呼び出します。これらのメソッドのいずれかを必要に応じて使用することができます。

新しいメソッドは、文字列の引数を取らないことに注意してください。このメソッドは、提供されたオブジェクトのすべての DAX プロパティを考慮してフォーマットを行います (メジャーの場合は Expression および DetailRowsExpression プロパティ、KPI の場合は StatusExpression、TargetExpression および TrendExpression など)。