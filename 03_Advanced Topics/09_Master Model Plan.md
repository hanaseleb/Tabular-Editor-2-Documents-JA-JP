マスターモデルパターン
組織内に複数のTabularモデルがあり、機能的にもかなり重複していることは珍しくありません。開発チームにとって、これらのモデルを共有機能に合わせて最新の状態に保つことは苦痛になります。この記事では、これらのモデルをすべて1つの「マスター」モデルにまとめ、それを部分的に複数の異なるサブセットモデルに展開することに意味があるような状況に適した別のアプローチを見ていきます。タブラー・エディターは、特別な方法でパースペクティブを使用することにより、このアプローチを可能にします（パースペクティブは通常の方法で動作します）。

免責事項：この手法は動作しますが、Microsoftではサポートされておらず、かなりの学習、スクリプト、ハッキングが必要になります。あなたのチームにとってこの手法が正しいかどうかは、ご自身で判断してください。

簡単にするために、AdventureWorksのサンプルモデルを考えてみましょう。

https://user-images.githubusercontent.com/8976200/43959290-895c1c96-9cae-11e8-8112-008f54cb400a.png

何らかの理由で、インターネット販売に関するものは1つのモデルに、再販業者の販売に関するものは別のモデルに展開する必要があるとします。これは、セキュリティ上の理由、パフォーマンス上の理由、スケーラビリティ上の理由、あるいは、チームが複数の外部顧客にサービスを提供しており、各顧客が共有機能と特定機能の両方を含む独自のモデルのコピーを必要としている場合も考えられます。

ここで紹介する手法は、異なるバージョンごとに1つの開発ブランチを維持する代わりに、デプロイ時にモデルをどのように分割すべきかを示すメタデータを使用して、1つのモデルだけを維持することができます。

(パースペクティブの使用方法
アイデアは非常にシンプルです。まず、デプロイするターゲットモデルの数に対応する数の新しいパースペクティブをモデルに追加します。これらのパースペクティブには必ず一貫した方法で接頭辞を付けて、ユーザー指向のパースペクティブとは区別します。

https://user-images.githubusercontent.com/8976200/43960154-6b637042-9cb1-11e8-906b-6671bbb9558e.png

ここでは、パースペクティブ名のプレフィックスとして$記号を使用しています。後で、これらのパースペクティブがどのようにモデルから取り除かれるかを見てみましょう、そうすればエンド・ユーザーはこれらを見ることはありません。これらはモデルの開発者のみが使用します。

さて、個々のモデルに必要なすべてのオブジェクトをこれらのパースペクティブに追加するだけです。モデルに必要なオブジェクトが含まれていることを確認するには、Tabular EditorのPerspectiveドロップダウンを使用します。ここでは、すべての依存関係がパースペクティブに含まれていることを確認するために使用できる便利なスクリプトを紹介します。

// 現在のパースペクティブですべての階層を調べます。
foreach(var h in Model.AllHierarchies.Where(h => h.InPerspective[Selected.Perspective]))
{
    // 階層レベルで使用されるカラムがパースペクティブに含まれていることを確認します。
    foreach(var level in h.Levels) { 以下のようになります。
        level.Column.InPerspective[Selected.Perspective] = true;
    }
}

// 現在のパースペクティブ内のすべてのメジャーおよび列をループします。
foreach(var obj in Model.AllMeasures.Cast<ITabularPerspectiveObject>()
    .Concat(Model.AllColumns).Where(m => m.InPerspective[Selected.Perspective])
    .OfType<IDaxDependantObject>().ToList())
{
    // 現在のオブジェクトが依存しているすべてのオブジェクトをループします。
    foreach(var dep in obj.DependsOn.Deep())
    {
        // カラム、メジャー、およびテーブルの依存関係を含めます。
        var columnDep = dep as Column; if(columnDep != null) columnDep.InPerspective[Selected.Perspective] = true;
        var measureDep = dep as Measure; if(measureDep != null) measureDep.InPerspective[Selected.Perspective] = true; となります。
        var tableDep = dep as Table; if(tableDep != null) tableDep.InPerspective[Selected.Perspective] = true; となります。
    }    
}

// 現在のパースペクティブでSortByColumnを持つすべての列を調べます。
foreach(var c in Model.AllColumns.Where(c => c.InPerspective[Selected.Perspective] && c.SortByColumn != null))
{
    c.SortByColumn.InPerspective[Selected.Perspective] = true;   
}
説明します。まず、このスクリプトは、現在のパースペクティブ（画面上部のドロップダウンで現在選択されているパースペクティブ）のすべての階層をループします。そのような階層ごとに、階層レベルとして使用されるすべての列がパースペクティブに表示されるようにします。次に、スクリプトは現在のパースペクティブのすべての列とメジャーをループします。これらのオブジェクトのそれぞれについて、メジャー、列、またはテーブルの参照の形で、すべての DAX 依存関係もパースペクティブに含まれます。DISTINCTCOUNT('Customer'[CustomerId])のような式は、'Customer'テーブルのすべての列がパースペクティブに含まれることになりますのでご注意ください。最後に、このスクリプトでは、「Sort By」列として使用されているすべての列がパースペクティブに含まれるようにします。

このスクリプトをモデルレベルのカスタムアクションとして保存しておくと、今後も簡単に呼び出すことができます。

ところで、パースペクティブのコピーを作成したい場合は、すでにUIで行うことができます。エクスプローラツリーの「パースペクティブ」ノードをクリックし、プロパティグリッドの楕円ボタンをクリックしてください。

https://user-images.githubusercontent.com/8976200/44028910-c7ffab80-9efb-11e8-813a-5b0f5c137bab.png

すると、パースペクティブの作成や削除、既存のパースペクティブのクローン作成ができるダイアログが表示されます。

https://user-images.githubusercontent.com/8976200/44028953-f13c91ca-9efb-11e8-936a-1f0e1d4eb93f.png

これを補足するために、パースペクティブからすべての不可視オブジェクトや未使用オブジェクトを削除するスクリプトを以下に示しますので、ちょっとした整理が必要な場合にご利用ください。

// 現在のパースペクティブのすべての列をループします。
foreach(var c in Model.AllColumns.Where(c => c.InPerspective[Selected.Perspective])) { 。
    if(
        // カラムが隠されている場合（または親テーブルが隠されている場合）。
        (c.IsHidden || c.Table.IsHidden) 

        // そして、どのリレーションでも使用されていません。
        && !c.UsedInRelationships.Any()
        
        // そして、パースペクティブ内の他の列の SortByColumn として使用されていません。
        && !c.UsedInSortBy.Any(sb => !sb.IsHidden && sb.InPerspective[Selected.Perspective])
        
        // そして、パースペクティブ内のどの階層でも使用されていない。
        && !c.UsedInHierarchies.Any(h => h.InPerspective[Selected.Perspective])
        
        // そして、パースペクティブ内の他の可視オブジェクトの DAX 式で参照されていません。
        && !c.ReferencedBy.Deep().OfType<ITabularPerspectiveObject>()
            .Any(obj => obj.InPerspective[Selected.Perspective] && !(obj as IHideableObject).IsHidden)
            
        // そして、どのロールからも参照されていません。
        && !c.ReferencedBy.Roles.Any() )
    {
        // 上記のすべてが満たされていれば、現在のパースペクティブからカラムを削除することができます。
        c.InPerspective[Selected.Perspective] = false; 
    }
}

// 現在のパースペクティブのすべてのメジャーをループします。
foreach(var m in Model.AllMeasures.Where(m => m.InPerspective[Selected.Perspective])) { 。
    if(
        // メジャーが非表示の場合 (または親テーブルが非表示の場合)。
        (m.IsHidden || m.Table.IsHidden) 

        // そして、パースペクティブ内の他の可視オブジェクトの DAX 式で参照されていません。
        && !m.ReferencedBy.Deep().OfType<ITabularPerspectiveObject>()
            .Any(obj => obj.InPerspective[Selected.Perspective] && !(obj as IHideableObject).IsHidden)
    )
    {
        // 上記のすべてが満たされていれば、現在のパースペクティブからカラムを削除できます。
        m.InPerspective[Selected.Perspective] = false となります。
    }
}
説明します。このスクリプトはまず、現在選択されているパースペクティブのすべての列をループします。次のすべてが真である場合にのみ、パースペクティブから列を削除します。

列が非表示である（または、列が存在するテーブルが非表示である）。
その列がどのようなリレーションシップにも参加していない。
その列が、パースペクティブ内の他の可視列の SortByColumn として使用されていない。
カラムは、パースペクティブ内のどの階層でもレベルとして使用されていない。
列は、パースペクティブ内の他の可視オブジェクトの DAX 式において、直接的または間接的に参照されていない。
列が行レベルのフィルタ式で使用されていない。
メジャーについても同じことを行いますが、以下の条件を満たすメジャーのみを削除するように簡略化します。

メジャーが非表示である (または、メジャーが存在するテーブルが非表示である)
メジャーが、パースペクティブ内の他の可視オブジェクトの DAX 式で直接または間接的に参照されていない。
モデルを作成している開発者のチームであれば、Tabular Editorsの「Save to Folder」機能をGitなどのソース・コントロール環境と一緒に使用しているはずです。ファイル」→「環境設定」→「フォルダに保存」で、「パースペクティブをオブジェクトごとにシリアライズする」オプションを必ずチェックして、パースペクティブの定義で大量のマージコンフリクトが発生しないようにしてください。

https://user-images.githubusercontent.com/8976200/44029969-935e0efe-9eff-11e8-93de-c1223f7ebe7f.png

より詳細なコントロールの追加
もうお分かりだと思いますが、事前に設定した開発者用のパースペクティブごとに、スクリプトを使用して1つのバージョンのモデルを作成する予定です。このスクリプトは、特定の開発者視点に含まれていないすべてのオブジェクトをモデルから削除します。しかし、その前に処理しなければならない状況がいくつかあります。

パースペクティブ以外のオブジェクトを制御する
パースペクティブ、データソース、ロールなどの一部のオブジェクトは、パースペクティブ自体には含まれず、除外もされませんが、それらがどのモデルバージョンに属するべきかを指定する方法が必要な場合があります。このような場合には、アノテーションを使用します。Adventure Worksモデルに戻って、「在庫」と「インターネット操作」のパースペクティブを「$InternetModel」と「$ManagementModel」に表示し、「再販業者の操作」を「$ResellerModel」と「$ManagementModel」に表示したいとします。

そこで、オリジナルの3つのパースペクティブのそれぞれに「DevPerspectives」という新しいアノテーションを追加し、開発者のパースペクティブの名前をカンマ区切りの文字列で供給してみましょう。

https://user-images.githubusercontent.com/8976200/44032304-01bdcc70-9f07-11e8-9b28-db0912ea1ade.png

新しいユーザー・パースペクティブをモデルに追加するときは、同じアノテーションを追加し、そのユーザー・パースペクティブを含めたい開発者のパースペクティブの名前を指定することを忘れないでください。後日、モデルの最終バージョンをスクリプトで作成する際に、これらのアノテーションの情報を使用して、必要なパースペクティブを含めることになります。これと同じことを、データソースやロールに対しても行うことができます。

オブジェクトのメタデータを制御する
また、同じメジャーであっても、モデル・バージョンごとに表現やフォーマット文字列が若干異なる場合があります。この場合も、アノテーションを使用して開発者ごとにメタデータを提供し、最終的なモデルをスクリプトアウトする際にメタデータを適用することができます。

すべてのオブジェクトのプロパティをテキストに直列化する最も簡単な方法は、おそらくExportPropertiesスクリプト関数でしょう。ここでは、アノテーションとして保存したいプロパティを直接指定してみましょう。次のようなスクリプトを作成します。

foreach(var m in Selected.Measures) {. 
    m.SetAnnotation(Selected.Perspective.Name + "_Expression", m.Expression);
    m.SetAnnotation(Selected.Perspective.Name + "_FormatString", m.FormatString);
    m.SetAnnotation(Selected.Perspective.Name + "_Description", m.Description);
}
そして、これを「メタデータを注釈として保存」という名前のカスタムアクションとして保存します。

https://user-images.githubusercontent.com/8976200/44033695-7a754482-9f0b-11e8-937b-0bc0987ce7cb.png

同様に、次のスクリプトを「Load Metadata from Annotations」というカスタムアクションとして保存します。

foreach(Measure m in Selected.Measures) {... 
    var expr = m.GetAnnotation(Selected.Perspective.Name + "_Expression"); if(expr == null) continue;
    m.Expression = expr;
    m.FormatString = m.GetAnnotation(Selected.Perspective.Name + "_FormatString");
    m.Description = m.GetAnnotation(Selected.Perspective.Name + "_Description");
}
これは、開発者の視点ごとに、異なるバージョンを維持したいプロパティごとに1つのアノテーションを作成するというものです。スクリプトに表示されている以外のプロパティ（Expression、FormatString、Description）を個別に管理する必要がある場合は、スクリプトに追加するだけです。他のオブジェクト・タイプでも同じことができますが、メジャーや計算列、パーティション (モデル・バージョンごとに異なるクエリ式を維持する場合など) 以外では、あまり意味がないでしょう。

新しいカスタム・アクションを使用して、モデル・バージョン固有の変更を開発者のパースペクティブに適用します（または、手作業でアノテーションを追加します）。例えば、Adventure Works のサンプルでは、[Day Count] メジャーが $ResellerModel パースペクティブで異なる式を持つようにしたいので、メジャーに変更を適用し、ドロップダウンで「$ResellerModel」パースペクティブを選択した状態で、「メタデータを注釈として保存」アクションを呼び出します。

https://user-images.githubusercontent.com/8976200/44033944-3104e414-9f0c-11e8-9f06-396bf85a0e4f.png

上のスクリーンショットでは、開発者のパースペクティブごとに3つのアノテーションがあります。しかし、実際には、プロパティが本来の値と異なる開発者のパースペクティブに対してのみ、これらのアノテーションを作成する必要があります。

パーティション・クエリの変更
同様の手法を用いて、異なるバージョン間でパーティション・クエリに変更を加えることができます。例えば、バージョンによってパーティション・クエリのSQL WHERE基準を変えたい場合があります。まず、テーブル・オブジェクトに一連の新しいアノテーションを作成し、バージョンごとにパーティションで使用したい基本的なSQLクエリを指定してみましょう。例えば、3つのバージョンのうち2つのバージョンで、Productテーブルに含まれるレコードを制限したいとします。

https://user-images.githubusercontent.com/8976200/44736562-69221580-aaa4-11e8-82ee-88388015d30d.png

複数のパーティションを持つテーブルでは、WHERE基準を「プレースホルダー」を使って指定しますが、これは後で置き換えられます。

https://user-images.githubusercontent.com/8976200/44737015-b3f05d00-aaa5-11e8-9bad-cadd5b4dae35.png

各パーティション内でプレースホルダーの値を定義します（注意：UIでパーティションアノテーションを編集するには、Tabular Editor v.2.7.3以降を使用する必要があります）。

https://user-images.githubusercontent.com/8976200/44737199-2a8d5a80-aaa6-11e8-8813-8189b593da98.png

ダイナミック・パーティショニングのシナリオでは、新しいパーティションを作成する際に使用するスクリプトに、これらのアノテーションを含めることを忘れないでください。次のセクションでは、デプロイ時にこれらのプレースホルダー値を適用する方法を説明します。

異なるバージョンのデプロイ
最後に、モデルを3つの異なるバージョンとしてデプロイする準備ができました。残念ながら、Tabular EditorのDeployment Wizard UIでは、作成したパースペクティブやアノテーションに基づいてモデルを分割することができないので、モデルを特定のバージョンに切り詰める追加のスクリプトを作成する必要があります。このスクリプトは、コマンドラインのデプロイメントの一部として実行することができ、デプロイメントプロセス全体をコマンドファイルやPowerShell実行ファイルにパッケージしたり、ビルドや自動デプロイメントプロセスに統合したりすることができます。

必要なスクリプトは以下のようなものです。開発者の視点ごとに1つのスクリプトを作成することを考えています。スクリプトをテキストファイルとして保存し、ResellerModel.csのような名前にします。

var version = "`$`ResellerModel"; // TODO: これを開発者視点の名前に置き換える

// パースペクティブに含まれていないテーブル、メジャー、カラム、階層を削除します。
foreach(var t in Model.Tables.ToList()) { 。
    if(!t.InPerspective[version]) t.Delete();
    else {
        foreach(var m in t.Measures.ToList()) if(!m.InPerspective[version]) m.Delete();   
        foreach(var c in t.Columns.ToList()) if(!c.InPerspective[version]) c.Delete();
        foreach(var h in t.Hierarchies.ToList()) if(!h.InPerspective[version]) h.Delete();
    }
}

// アノテーションに基づくユーザーの視点と、すべての開発者の視点を削除します。
foreach(var p in Model.Perspectives.ToList()) { 。
    if(p.Name.StartsWith("`$`")) p.Delete();

    // "DevPerspectives" アノテーションを持たない他のすべてのパースペクティブを維持し、 // アノテーションを持つパースペクティブを削除します。
    // アノテーションを持つパースペクティブは削除します（アノテーションに <version> が指定されていない場合）。
    if(p.GetAnnotation("DevPerspectives") != null && !p.GetAnnotation("DevPerspectives").Contains(version)) 
        p.Delete();
}

// アノテーションに基づいてデータソースを削除します。
foreach(var ds in Model.DataSources.ToList()) {.
    if(ds.GetAnnotation("DevPerspectives") == null) continue;
    if(!ds.GetAnnotation("DevPerspectives").Contains(version)) ds.Delete();
}

// アノテーションに基づいてロールを削除します。
foreach(var r in Model.Roles.ToList()) {.
    if(r.GetAnnotation("DevPerspectives") == null) continue;
    if(!r.GetAnnotation("DevPerspectives").Contains(version)) r.Delete();
}

// アノテーションに基づいてメジャーを修正します。
foreach(Measure m in Model.AllMeasures) { 。
    var expr = m.GetAnnotation(version + "_Expression"); if(expr == null) continue;
    m.Expression = expr;
    m.FormatString = m.GetAnnotation(version + "_FormatString");
    m.Description = m.GetAnnotation(version + "_Description");    
}

// アノテーションに応じて、パーティションクエリを設定します。
foreach(Table t in Model.Tables) { ...
    var queryWithPlaceholders = t.GetAnnotation(version + "_PartitionQuery"); if(queryWithPlaceholders == null) continue;
    
    // このテーブルにあるすべてのパーティションをループします。
    foreach(Partition p in t.Partitions) {...
        
        var finalQuery = queryWithPlaceholders;

        // すべてのプレースホルダーの値を置き換えます。
        foreach(var placeholder in p.Annotations.Keys) {
            finalQuery = finalQuery.Replace("%" + placeholder + "%", p.GetAnnotation(placeholder));
        

        p.Query = finalQuery;
    }
}

// TODO: 必要に応じて、アノテーションに基づいて他のオブジェクトを変更する...
説明します。まず、スクリプトの 1 行目で定義されたパースペクティブの一部ではない、すべてのテーブル、列、メジャー、および階層を削除します。次に、前述のように「DevPerspectives」アノテーションを適用した可能性のある追加オブジェクトを、すべての開発者パースペクティブ自体とともに削除します。その後、アノテーションに基づいて、メジャー式、フォーマット文字列、説明に変更があれば、それを適用します。最後に、アノテーションで定義されているパーティション・クエリを適用し、プレースホルダの値をアノテーションで定義された値に置き換えます（該当する場合）。

必要であれば、特定のモデルの変更をこのスクリプトに直接追加することもできますが、この演習の目的は、Tabular Editorから複数のモデルを直接管理する方法です。上記のスクリプトは、どのバージョンをデプロイするかに関わらず同じです（もちろん、1行目を除きます）。

最後に、Model.bimファイルの読み込み、スクリプトの実行、変更したモデルのデプロイを、以下のコマンドライン構文を使って一度に行うことができます。

start /wait /d "c:‾Program Files (x86)‾Tabular Editor" TabularEditor.exe Model.bim -S ResellerModel.cs -D localhost AdventureWorksReseller -O -R
インターネット版やマネジメント版を展開する場合も同様に、対応するスクリプトを用意する必要があります。

start /wait /d "c:‾Program Files (x86)‾Tabular Editor" TabularEditor.exe Model.bim -S InternetModel.cs -D localhost AdventureWorksInternet -O -R
start /wait /d "c:‾Program Files (x86)‾Tabular Editor" TabularEditor.exe Model.bim -S ManagementModel.cs -D localhost AdventureWorksManagement -O -R
これは、Model.bimファイル（「フォルダに保存」機能を使用している場合はDatabase.jsonファイル）のディレクトリ内でコマンドラインを実行していることを想定しています。Sスイッチは、供給されたスクリプトをモデルに適用するようTabular Editorに指示し、-Dスイッチはデプロイメントを実行します。Oスイッチは同じ名前の既存のデータベースを上書きすることができ、-Rスイッチは対象のデータベースのロールも上書きしたいことを示します。

マスターモデルの処理
専用の処理サーバーがあり、大量のデータが個々のモデル間で重複している場合、データを分割する前に、まずマスターモデルに処理することに意味がある場合があります。こうすることで、同じデータを何度も個別のモデルに処理することを避けることができます。ただし、このセクションで示したように、バージョン間でパーティション・クエリが変更されたテーブルを処理していないことが前提です。このためのレシピを以下に示します。

(オプション - メタデータの変更があった場合) マスターモデルを処理サーバーにデプロイする
マスター・モデルで必要な処理を行う（バージョン固有のパーティション・クエリを持つテーブルは処理しない）。
マスター・モデルを各個別モデルに同期し、同期後に上記のコマンドを使用して個別モデルをストリップダウンし、必要に応じてProcessRecalcを実行します。
(オプション）個々のモデル上で、バージョン固有のパーティション・クエリを持つテーブルを処理します。
ヒントとコツ
カスタムアノテーションを頻繁に使用するようになると、特定のアノテーションを持つすべてのオブジェクトをリストアップしたい場合があります。そんなときに便利なのが、Filter-boxのDynamic LINQ expressionsです。

まず、「$InternetModel_Expression」という名前のアノテーションを追加したオブジェクトをすべて探したいとします。フィルタのテキストボックスに次のように入力し、ENTERを押します。

:GetAnnotation("`$`InternetModel_Expression")<>null
また、「_Expression」という単語で終わるアノテーションを持つすべてのオブジェクトを検索したい場合は、次のようにします。

:GetAnnotations().Any(EndsWith("_Expression"))
これらの関数は大文字と小文字を区別するので、アノテーションが小文字で書かれている場合は、上記のフィルタでは検出されませんのでご注意ください。

また、アノテーションが特定の値を持つオブジェクトを検索することもできます。

:GetAnnotation(`$`InternetModel_Description).Contains("TODO")
まとめ
ここで紹介したテクニックは、カレンダーテーブルやその他の共通のディメンションなど、多くの共有機能を持つ似たようなモデルを多数管理する際に非常に役立ちます。使用したスクリプトは、Tabular Editorのカスタムアクションとしてきれいに再利用でき、実際のデプロイメントはさまざまな方法で自動化できます。