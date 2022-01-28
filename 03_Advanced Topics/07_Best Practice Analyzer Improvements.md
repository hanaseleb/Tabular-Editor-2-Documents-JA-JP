ベストプラクティスアナライザの改善
Tabular Editor 2.8.1では、Best Practice Analyzerが大幅に改良されました。

まず最初に気づくのは、Tabular EditorのメインUIでベストプラクティス問題の数が直接レポートされるようになったことです。

https://user-images.githubusercontent.com/8976200/53631987-baee5880-3c0b-11e9-9d66-e906cccce2be.png

モデルに変更が加えられると、Best Practice Analyzer はバックグラウンドで問題をスキャンします。この機能は [ファイル] > [環境設定] で無効にすることができます。

このリンクをクリック（または F10 を押す）すると、新しくなった「Best Practice Analyzer」の UI が表示されます。

https://user-images.githubusercontent.com/8976200/53631947-9eeab700-3c0b-11e9-9217-5739d4de2f88.png

以前のバージョンでBest Practice Analyzerを使用したことがある場合、まず最初に気づくのは、UIのデザインが完全に変更され、画面上での占有面積が少なくなったことです。これにより、ウィンドウをデスクトップの片側にドッキングさせ、メインウィンドウをもう片側に置いたまま、両方を同時に操作することができます。

ベストプラクティスアナライザ」ウィンドウは、モデル上のすべての有効なルールと、各ルールに違反しているオブジェクトを継続的にリストアップします。リスト内の任意の場所を右クリックするか、ウィンドウ上部のツールバーボタンを使用すると、以下のアクションを実行できます。

ルールの管理...: これにより、後述する「ルールの管理」UIが開きます。このUIは、メインUIの「Tools > Manage BPA Rules...」メニューからもアクセスできます。
Go to object... (オブジェクトへ移動) このオプションを選択するか、リスト内のオブジェクトをダブルクリックすると、メインUIの同じオブジェクトに移動します。
アイテム/アイテムを無視する。リスト内の1つまたは複数のオブジェクトを選択してこのオプションを選択すると、Best Practice Analyzerが今後そのオブジェクトを無視することを示す注釈が、選択したオブジェクトに適用されます。誤ってオブジェクトを無視してしまった場合は、画面上部の「Show ignored」ボタンを押してください。これにより、以前に無視されていたオブジェクトの無視を解除することができます。
Ignore rule: リストで1つまたは複数のルールを選択した場合、このオプションは、選択したルールを常に無視することを示す注釈をモデルレベルで表示します。繰り返しになりますが、「無視を表示」ボタンを切り替えることで、ルールの無視を解除することもできます。
修正スクリプトを生成する。簡単に修正できるルール（オブジェクトの1つのプロパティを設定するだけで問題が解決できることを意味する）は、このオプションが有効になります。このオプションをクリックすると、C#スクリプトがクリップボードにコピーされます。このスクリプトは、後でTabular EditorのAdvanced Scriptingエリアに貼り付けて、修正を適用するために実行する前に確認することができます。
修正プログラムを適用します。このオプションは、前述のように簡単に修正できるルールにも使用できます。スクリプトをクリップボードにコピーするのではなく、すぐに実行されます。
ベストプラクティスルールの管理
モデルに適用するルールを追加、削除、修正する必要がある場合は、新しいUIが用意されています。Best Practice Analyzerウィンドウの左上のボタンをクリックするか、メインウィンドウの「Tools > Manage BPA Rules...」メニューを使用することで表示されます。

https://user-images.githubusercontent.com/8976200/53632990-2f29fb80-3c0e-11e9-82fe-ee9c921662c7.png

このUIには2つのリストがあります。一番上のリストには、現在読み込まれているルールのコレクションが表示されます。このリストでコレクションを選択すると、そのコレクション内で定義されているすべてのルールが下のリストに表示されます。デフォルトでは、3つのルールコレクションが表示されます。

現在のモデル内のルール。名前が示すように、これは現在のモデル内で定義されたルールのコレクションです。ルールの定義は、Modelオブジェクトのアノテーションとして保存されます。
ローカルユーザーのためのルール。これは、%AppData%...Local\TabularEditor\BPARules.jsonファイルに格納されているルールです。これらのルールは、現在ログインしているWindowsユーザーがTabular Editorで読み込んだすべてのモデルに適用されます。
ローカルマシン上のルール。これらのルールは %ProgramData%%TabularEditor\BPARules.json ファイルに保存されます。これらのルールは、現在のマシンでTabular Editorに読み込まれているすべてのモデルに適用されます。
同じルール（ID）が複数のコレクションにある場合、優先順位は上から下の順になります。つまり、モデル内で定義されたルールは、ローカルマシンで定義された同じIDのルールよりも優先されます。これにより、モデル固有の規約を考慮するなど、既存のルールを上書きすることができます。

リストの一番上には、「（Effective rules）」という特別なコレクションがあります。このコレクションを選択すると、現在読み込まれているモデルに実際に適用されるルールのリストが表示され、前述のように同一のIDを持つルールの優先順位が尊重されます。下部のリストには、ルールがどのコレクションに属しているかが表示されます。また、同様のIDを持つルールがより優先度の高いコレクションに存在する場合、ルールの名前が抹消されることにも気づくでしょう。

https://user-images.githubusercontent.com/8976200/53633831-74e7c380-3c10-11e9-925e-1419987f5a17.png

コレクションの追加
Tabular Editor 2.8.1の新機能として、他のソースのルールをモデルに含めることができます。例えば、ネットワーク共有上にルールファイルがある場合、そのファイルを現在のモデルのルールコレクションとして含めることができるようになりました。そのファイルの場所に書き込み権限があれば、そのファイルからルールを追加/修正/削除することもできます。このようにして追加されたルール・コレクションは、モデル内で定義されたルールよりも優先されます。このようなコレクションを複数追加した場合は、それらを上下に移動させて相互の優先順位をコントロールすることができます。

新しいルール・コレクションをモデルに追加するには、「追加...」ボタンをクリックします。これにより、以下のオプションが提供されます。

https://user-images.githubusercontent.com/8976200/53634211-7cf43300-3c11-11e9-8fed-7df113264a6f.png

Create new Rule File: これは、指定された場所に新しい空の.jsonファイルを作成し、そこにルールを追加することができます。ファイルを選択する際、相対ファイルパスを使用するオプションがあることに注意してください。これは、ルールファイルを現在のモデルと同じコードリポジトリに保存したい場合に便利です。ただし、相対的なルール ファイルの参照は、モデルがディスクからロードされた場合にのみ機能しますので、ご注意ください（Analysis Services のインスタンスからモデルをロードする際には作業ディレクトリが存在しないため）。
ローカル ルール ファイルを含める：モデルに含めるルールを含む.jsonファイルがすでにある場合、このオプションを使用します。ここでも、相対ファイルパスを使用するオプションがあります。これは、ファイルがモデルのメタデータの近くに配置されている場合に有効です。ファイルがネットワーク共有に置かれている場合（一般的には、現在読み込まれているモデル・メタデータが存在する場所とは異なるドライブに置かれている場合）、絶対パスを使用してのみインクルードすることができます。
URLからルールファイルを含める。このオプションでは、有効なルール定義（json）を返すHTTP/HTTPS URLを指定することができます。これは、オンラインソースからルールを含める場合に便利です。たとえば、BestPracticeRules GitHubサイトの標準BPAルールなどがあります。オンラインソースから追加されたルールコレクションは読み取り専用になることに注意してください。
コレクション内のルールの修正
画面の下部では、現在選択されているコレクション内のルールの追加、編集、クローン、および削除を行うことができます。ただし、コレクションが保存されている場所への書き込みアクセス権がある場合に限ります。また、「Move to...」ボタンを押すと、選択したルールを別のコレクションに移動またはコピーすることができ、複数のルールのコレクションを簡単に管理することができます。ルール定義を編集するUIはTabular Editorの旧バージョンから変更されていませんので、その使い方については旧Best Practice Analyzerの記事を参照してください。

ルール説明のプレースホルダー
以前のバージョンと比較して少し改善された点は、Best Practice Ruleの説明に以下のプレースホルダー値を使用できるようになったことです。これにより、Best Practice UI のツールチップとして表示される説明をよりカスタマイズできるようになりました。

object% は、現在のオブジェクトへの完全修飾された DAX 参照（該当する場合）を返します。
objectname% は、現在のオブジェクトの名前のみを返します。
objecttype% は、現在のオブジェクトのタイプを返します。