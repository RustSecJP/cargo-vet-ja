{% raw %}
# 設定 {#configuration}

この節では、`cargo vet` が使うさまざまな設定ファイルの構造と意味を説明します。

## 置き場所 {#location}

既定では、`cargo vet` のデータは `Cargo.lock` と同じ場所にある `supply-chain`
ディレクトリに置かれます。この場所は、Cargo.toml の `[package.metadata.vet]`
ディレクティブで変更できます。仮想ルートを持つワークスペースを使っている場合は、
`[workspace.metadata.vet]` でも変更できます。

既定の設定は、次の内容と同じです。

```toml
[package.metadata.vet]
store = { path = './supply-chain' }
```

## `audits.toml` {#auditstoml}

このファイルには、プロジェクトのメンバーが行った監査と、監査基準の説明が入ります。
このファイルの情報は、ほかのプロジェクトからインポートできます。

### `criteria` テーブル {#the-criteria-table}

このテーブルでは、独自の基準をいくつでも定義できます。各エントリには、
次のようなフィールドを指定できます。

#### `description` {#description}

基準の簡潔な説明です。このフィールド（または `description-url`）は必須です。

#### `description-url` {#description-url}

`description` の代わりに使えるフィールドで、基準の文章が置かれている、
誰でもアクセスできる URL を指定します。複数のリポジトリで基準の説明を共有したい
ときに便利です。

#### `implies` {#implies}

このエントリに包含されるほかの基準を、文字列または配列で指定します（省略可能）。
このエントリの基準で認定された監査エントリは、ここに挙げた基準でも暗黙のうちに
認定されたものとして扱われます。

たとえば、[組み込みの基準](built-in-criteria.md)を独自の基準として書くと、
次のようになります。

```
[criteria.safe-to-run]
description = '...'

[criteria.safe-to-deploy]
description = '...'
implies = 'safe-to-run'
```

### `audits` テーブル {#the-audits-table}

このテーブルには、クレート名ごとに監査エントリが入ります。1 つのクレートに対して
複数の監査（異なるバージョン、差分監査など）があることが多いので、監査エントリは
テーブルの配列、つまり `[[audits.foo]]` の形で書きます。

監査エントリの各キーの意味は、[こちら](audit-entries.md)で説明しています。

### `trusted` テーブル {#the-trusted-table}

このテーブルには、クレート名ごとに信頼する公開者のエントリが入ります。1 つのクレートに
複数の公開者がいることがあるので、信頼エントリはテーブルの配列、つまり
`[[trusted.foo]]` の形で書きます。

信頼エントリの各キーの意味は、[こちら](trusted-entries.md)で説明しています。

## `config.toml` {#configtoml}

このファイルには、そのプロジェクトに固有の設定情報が入ります。このファイルは、
ほかのプロジェクトからインポートできません。

### `default-criteria` {#default-criteria}

このトップレベルのキーは、`cargo vet certify` が監査を記録するときに使う既定の基準を
指定します。指定しない場合の既定値は `safe-to-deploy` です。

### `cargo-vet` テーブル {#the-cargo-vet-table}

このテーブルには、ストアの作成に使われた cargo-vet のバージョンを追跡するための
メタデータが入ります。将来は、そのほかの全体的な設定を指定するためにも使われる
可能性があります。

### `imports` テーブル {#the-imports-table}

このテーブルには、このプロジェクトにインポートする外部の監査セットを列挙します。
キーは利用者が自由に付ける呼び名なので、エントリは `[imports.foo]` の形で書きます。

#### `url` {#url}

リモートの `audits.toml` を取得するための HTTPS の URL を指定します。この
フィールドは必須です。

#### `criteria-map` {#criteria-map}

インポートした監査セットの基準を、ローカルの基準に対応づけるテーブルです。
インポートされた各監査の基準は、ピア側の `implies` の関係を考慮したうえで
この対応表を通され、インポート時にローカルの基準の集合へ変換されます。

```
[imports.peer.criteria-map]
peer-criteria = "local-criteria"
their-super-audited = ["safe-to-deploy", "audited"]
```

特に指定しないかぎり、ピアの `safe-to-run` と `safe-to-deploy` の基準は、
ローカルの `safe-to-run` と `safe-to-deploy` の基準に暗黙のうちに対応づけられます。
この動作は、基準の対応表で `safe-to-run` や `safe-to-deploy` の対応先を指定すれば
上書きできます。

```
[imports.peer.criteria-map]
safe-to-run = []
safe-to-deploy = "safe-to-run"
```

そのほかの、対応づけられていない基準は、インポート時に捨てられます。

#### `exclude` {#exclude}

このインポート元から監査エントリをインポートしないクレートのリストです。
あるクレートが適切かどうかについて意見が食い違ったときに、最後の手段として
使えます。

### `policy` テーブル {#the-policy-table}

このテーブルでは、`cargo vet` がさまざまな依存に対して強制する監査の要件を、
プロジェクトごとに設定できます。指定しない場合、トップレベルではないクレートは
ポリシーの属性のほとんどを親から継承し、トップレベルのクレートには以下で説明する
既定値が適用されます。

ここでいう「トップレベル」とは、基本的には逆依存（自分に依存しているクレート）を
持たないクレートのことです。ただし、dev-dependencies を評価するときは例外で、
ワークスペースのすべてのメンバーがルートとみなされます。

このテーブルのキーには、クレート名（この場合、ポリシーはそのクレートの
_すべての_ バージョンに適用されます）か、`"CRATE:VERSION"` という形式の文字列を
使えます（バージョン文字列にはピリオドが含まれるので、TOML ではたいてい引用符で
囲む必要があります）。バージョンを指定する場合は、依存グラフに含まれている
クレートのバージョンだけを指定できます。

#### `criteria` {#criteria}

このクレートと、その依存のサブツリーに対して強制する基準を、文字列または文字列の
配列で指定します。

これを指定できるのは、ファーストパーティのクレートだけです。サードパーティの
クレートに対する要件は、継承または `dependency-criteria` を通じて適用してください。

トップレベルのクレートでは、既定値は `safe-to-deploy` です。

#### `dev-criteria` {#dev-criteria}

上と同じですが、dev-dependencies に適用されます。

トップレベルのクレートでは、既定値は `safe-to-run` です。

#### `dependency-criteria` {#dependency-criteria}

上記の値を、依存ごとに上書きできます。

```
[policy.foo]
dependency-criteria = { bar = [] }
notes = "bar is only used to implement a foo feature we never plan to enable."
```

`criteria` や `dev-criteria` と違って、`dependency-criteria` はサードパーティの
クレートにも直接適用できます（上の例では、`foo` と `bar` のどちらもサードパーティで
かまいません）。サードパーティのクレートに `criteria` を指定できないのは、
ある 1 つのサードパーティのクレートが、プロジェクトの依存グラフの中で、互いに関係の
ない複数の場所から使われることがよくあるためです。つまり上の例では、`foo` から
使われるかぎりにおいて `bar` を監査の対象から外したいのであって、ほかの場所に
現れた場合まで外したいとはかぎらない、ということです。

`dependency-criteria` を持つサードパーティのクレートは、ポリシーのテーブルの中で
特定のバージョンと結びつけなければなりません（上記の、ポリシーのテーブルのキーの
説明を参照してください）。さらに、あるクレートに `dependency-criteria` が指定されて
いて、そのクレートのいずれかのバージョンがサードパーティのクレートとしてグラフに
存在する場合は、そのクレートのすべてのバージョンを、ポリシーのテーブルのキーで
明示的に指定しなければなりません。

既定値は空集合で、継承はされません。

#### `audit-as-crates-io` {#audit-as-crates-io}

このクレート名を持つファーストパーティのパッケージに対して、crates.io から取得した
場合と同じように監査を強制するかどうかを指定します。詳しくは
[ファーストパーティのコード](first-party-code.md)を参照してください。

#### `notes` {#notes}

判断の理由や、そのほかの関連する情報を記録するための、自由形式の文字列です。

### `exemptions` テーブル {#the-exemptions-table}

このテーブルには、必要な監査が欠けているにもかかわらず使われているクレートを
列挙します。`audits.toml` の `audits` テーブルと似た構造ですが、各エントリで
使えるフィールドはより少なくなっています。

#### `version` {#version}

免除するバージョンを、正確に指定します。

#### `criteria` {#criteria-1}

免除の対象となる基準を指定します。

#### `notes` {#notes-1}

判断の理由や、そのほかの関連する情報を記録するための、自由形式の文字列です。

#### `suggest` {#suggest}

このエントリを `cargo vet suggest` の提案に出してよいかどうかを示す真偽値です。

既定値は true です。これは、何らかの理由で当面は対応するつもりのない提案を
表示させないようにするためのものです。

## `imports.lock` {#importslock}

このファイルは `cargo vet` が自動的に生成するもので、その形式は実装の詳細として
扱ってください。
{% endraw %}
