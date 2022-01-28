カスタムアクション
注意
この機能は、多次元モデルで利用可能なカスタムアクション機能とは無関係ですのでご注意ください。

Selectedオブジェクトを使用して便利なスクリプトを作成し、エクスプローラツリーの異なるオブジェクトでスクリプトを複数回実行できるようにしたいとします。スクリプトを実行するたびに「再生」ボタンを押すのではなく、Tabular Editorではカスタムアクションとして保存することができます。

https://user-images.githubusercontent.com/8976200/33581673-0db35ed0-d952-11e7-90cd-e3164e198865.png

カスタムアクションを保存すると、エクスプローラツリーの右クリックコンテキストメニューから直接利用できるようになり、ツリーで選択されたオブジェクトに対して非常に簡単にスクリプトを呼び出すことができるようになります。カスタムアクションは必要な数だけ作成できます。コンテキストメニュー内にサブメニュー構造を作成するには、名前にバックスラッシュ（\）を使用します。

https://raw.githubusercontent.com/otykier/TabularEditor/master/Documentation/InvokeCustomAction.png

カスタムアクションは、%AppData%Local%TabularEditor 内の CustomActions.json ファイルに保存されます。上記の例では、このファイルの内容は以下のようになります。

{
  "Actions": [
    {
      "Name": "Custom Formatting\\Number with 1 decimal" (カスタムフォーマット)
      "Enabled" (有効) "Enabled": "true",
      "Execute": "Selected.Measures.ForEach(m => m.FormatString = ˶‾᷄ -̫ ‾᷅˵);",
      "Tooltip": "FormatString "プロパティを "0.0\"に設定します。
      "ValidContexts": "Measure, Column"
    }
  ]
}
ご覧のとおり、NameとTooltipは、アクションの保存時に指定されたものから値を取得しています。Execute は、アクションが呼び出されたときに実行される実際のスクリプトです。CustomActions.jsonファイルに構文エラーがあると、Tabular Editorはすべてのカスタムアクションの読み込みを完全にスキップするので、カスタムアクションとして保存する前に、Advanced Scriptingエディタでスクリプトを正常に実行できることを確認してください。

ValidContextsプロパティには、アクションが利用可能なオブジェクトタイプのリストが格納されています。ツリーでオブジェクトを選択する際、ValidContexts プロパティに記載されているタイプと異なるオブジェクトが選択されていると、コンテキストメニューからアクションが非表示になります。

アクションの有効性の制御
コンテキスト メニューからアクションを呼び出すタイミングをさらに細かく制御する必要がある場合は、Enabled プロパティに、指定された選択範囲でアクションを使用できるかどうかを示すブール値を返さなければならないカスタム式を設定することができます。デフォルトでは、Enabledプロパティの値は「true」で、有効なコンテキスト内では常にアクションが有効であることを意味しています。Selected.MeasureやSelected.Tableなど、Selectedオブジェクトに単数形のオブジェクト参照を使用する場合は、現在の選択範囲にそのタイプのオブジェクトが1つも含まれていない場合にエラーが発生するので、この点に注意してください。このような場合には、Enabled プロパティを使用して、必要なタイプのオブジェクトが 1 つだけ選択されていることを確認することをお勧めします。

{
  "Actions": [
    {
      "Name": "Reset measure name" (メジャー名のリセット)
      "Enabled": "Selected.Measures.Count == 1" です。
      "Execute": "Selected.Measure.Name == \"New Measure\"",
      "ValidContexts": "Measure"
    }
  ]
}
これにより、ツリーで正確に 1 つのメジャーが選択されていない限り、コンテキスト・メニュー項目が無効になります。

カスタム・アクションの再利用
リリース 2.7 では、新しいスクリプト・メソッド CustomAction(...) が導入され、以前に保存したカスタム・アクションを呼び出すことができます。このメソッドは、独立したメソッド (Output(...)と同様) として使用することも、任意のオブジェクト・セットの拡張メソッドとして使用することもできます。

// 現在の選択範囲に対して「私のカスタムアクション」を実行します。
CustomAction("My custom action");                

// モデル内のすべてのテーブルに対して "My custom action "を実行します。
CustomAction(Model.Tables, "My custom action");

// 現在の選択範囲にある、名前が "Sum" で始まるすべてのメジャーに対して "My custom action" を実行します。
Selected.Measures.Where(m => m.Name.StartsWith("Sum")).CustomAction("My custom action");
カスタムアクションの完全な名前は、コンテキストメニューのフォルダ名も含めて指定する必要があることに注意してください。

指定した名前のアクションが見つからない場合は、スクリプトの実行時にエラーが発生します。