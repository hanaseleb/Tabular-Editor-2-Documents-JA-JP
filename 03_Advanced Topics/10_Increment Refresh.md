インクリメンタル・リフレッシュ
2021-02-15
ダニエル・オティキエル
Power BIサービスでホストされているデータセットは、1つまたは複数のテーブルにIncremental Refreshを設定することができます。Power BIデータセットでIncremental Refreshを設定または変更するには、Power BIサービスのXMLAエンドポイントを直接使用するか、以下のようにXMLAエンドポイントに接続されたTabular Editorを使用することができます。

Tabular Editorで最初からインクリメンタル・リフレッシュを設定する
ワークスペースのPower BI XMLA R/Wエンドポイントに接続し、インクリメンタル・リフレッシュを設定したいデータセットを開きます。
インクリメンタル・リフレッシュにはRangeStartとRangeEndのパラメータを作成する必要があるので（詳細）、まずはTabular Editorで2つの新しいShared Expressionsを追加してみましょう：https://user-images.githubusercontent.com/8976200/121341006-8906e900-c920-11eb-97af-ee683ff40609.png
それぞれRangeStartとRangeEndと名付け、Kindプロパティを「M」に設定し、式を以下のように設定します（データ更新を開始する際にPBIサービスによって設定されるため、実際に指定する日時の値は重要ではありません）。
#datetime(2021, 6, 9, 0, 0, 0) meta [IsParameterQuery=true, Type="DateTime", IsParameterQueryRequired=true].
https://user-images.githubusercontent.com/8976200/121342389-dc2d6b80-c921-11eb-8848-b67950e55e36.png4. 次に、インクリメンタル・リフレッシュを有効にしたいテーブルを選択します 5. テーブルの EnableRefreshPolicy プロパティを "true" に設定します。https://user-images.githubusercontent.com/8976200/121339872-3842c080-c91f-11eb-8e63-a051b34fb36f.png6. 残りのプロパティを、必要な増分リフレッシュ・ポリシーに従って設定します。SourceExpressionプロパティにM式を指定することを忘れないでください（これは、増分リフレッシュ・ポリシーによって作成されたパーティションに追加される式であり、RangeStartおよびRangeEndパラメータを使用してソースのデータをフィルタリングする必要があります）。https://user-images.githubusercontent.com/8976200/121342866-5f4ec180-c922-11eb-8a7a-cef44d3a407b.png7。モデルを保存します（Ctrl+S）。8. テーブルを右クリックして、「リフレッシュポリシーの適用」を選択します。https://user-images.githubusercontent.com/8976200/121342947-78577280-c922-11eb-82b5-a517fbe86c3e.png

以上で完成です。この時点で、Power BI サービスが、指定したポリシーに基づいて、テーブルにパーティションを自動生成したことがわかります。

https://user-images.githubusercontent.com/8976200/121343417-eef47000-c922-11eb-8731-1ac4dde916ef.png

次のステップは、パーティション内のデータを更新することです。Power BIサービスを利用することもできますが、SQL Server Management StudioでXMLA/TMSLを使ってパーティションを一括でリフレッシュすることもできますし、Tabular Editorのスクリプトを利用することもできます。

既存のリフレッシュポリシーの変更
Power BI Desktopを使って設定した既存のリフレッシュポリシーをTabular Editorで修正することもできます。この場合は、上記のステップ6-8に従うだけです。

EffectiveDateを使ったリフレッシュポリシーの適用
現在の日付を上書きしながらパーティションを生成したい場合（異なるローリングウィンドウの範囲を生成する目的で）、Tabular Editorの小さなスクリプトを使用して、EffectiveDateパラメータでリフレッシュポリシーを適用することができます。

インクリメンタル・リフレッシュ・テーブルを選択した状態で、Tabular Editorの「Advanced Scripting」ペインで、上記のステップ8の代わりに以下のスクリプトを実行します。

var effectiveDate = new DateTime(2020, 1, 1); // Todo: あなたの実効日に置き換えてください。
Selected.Table.ApplyRefreshPolicy(effectiveDate);
https://user-images.githubusercontent.com/8976200/121344362-f9633980-c923-11eb-916c-44a35cf03a36.png