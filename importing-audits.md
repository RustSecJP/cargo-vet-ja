{% raw %}
# 監査のインポート {#importing-audits}

`exemptions` の一覧を最も手早く減らす方法は、`config.toml` の `imports` ディレクティブを使って、信頼するほかのプロジェクトの監査の集合を取り込むことです。このディレクティブを使うと、ほかのプロジェクトの監査の一覧を、自分のプロジェクトの一覧に仮想的に統合できます。

```
[imports.foo]
url = "https://raw.githubusercontent.com/foo-team/foo/main/supply-chain/audits.toml"

[imports.bar]
url = "https://hg.bar.org/repo/raw-file/tip/supply-chain/audits.toml"
```
`cargo vet` は、実行されるとそれぞれの URL からデータを取得し、関係するデータを取り出して、その情報を `imports.lock` に保存します。`cargo vendor` と同じように、`--locked` を渡すと取得を省略します。

この仕組みは推移的ではないことに注意してください。つまり、ほかの誰かのインポートの一覧を、直接インポートすることはできません。これは意図的な制限で、信頼関係を直接的で、把握しやすいものに保つためのものです。とはいえ、ほかのプロジェクトの `config.toml` を参考として調べ、自分の要件に合う `imports` のエントリを明示的に取り入れることは、いつでもできます。

[組み込みの基準](built-in-criteria.md)は、すべてのプロジェクトで同じ意味を持ちます。そのため、`safe-to-run` の監査をインポートすることは、同じ監査を自分の `audits.toml` に追記するのと同じ効果を持ちます。既定では、外部の監査ファイルで定義されたカスタムの基準は、独立した名前空間に置かれ、ローカルのプロジェクトでは意味を持ちません。ただし、必要に応じて、ローカルで定義した基準に[対応づける](config.md#criteria-map)ことができます。

## レジストリ {#the-registry}

見つけやすくするために、`cargo vet` は、よく知られた組織が公開している監査の集合をまとめた中央のレジストリを管理しています。この情報は、`cargo vet` の[リポジトリ](https://github.com/mozilla/cargo-vet)に、ソースコードと並べて置かれている [`registry.toml`](https://raw.githubusercontent.com/mozilla/cargo-vet/main/registry.toml) ファイルに保存されています。プルリクエストを送ることで、自分の監査の集合をレジストリに含めるよう依頼できます。

レジストリを直接調べて、インポートしたい監査の集合を探すことができます。さらに、`cargo vet` は監査を提案するときに、レジストリに載っている集合を取得し、見つかった不足を埋めるためにインポートできるエントリがあれば提示します。これについては、あとで[詳しく](performing-audits.md#suggestions-from-the-registry)説明します。
{% endraw %}
