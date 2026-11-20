{% raw %}
# 監査の記録 {#recording-audits}

あなたやチームの仲間が行った、プロジェクトの依存の監査は、`audits.toml` に記録されます。なお、これらの依存も `cargo vet` を使っていれば、それぞれ独自の `audits.toml` ファイルを持っていることがあります。しかし、それらは `config.toml` で明示的にインポートしないかぎり、あなたのプロジェクトには影響しません。

## `audits.toml` {#auditstoml}

`audits.toml` にクレートを載せることは、あなたがそのクレートを調べ、指定した基準を満たしていると判断したことを意味します。

それぞれのクレートは、1 つ以上の監査エントリを持つことができ、エントリではさまざまなフィールドを使えます。`version` を指定すると、所有者がそのバージョンの全体を監査したことを意味します。`delta` を指定すると、所有者が 2 つのバージョンの差分を監査し、その変更によって該当する性質が保たれていると判断したことを意味します。

監査の途中で、基準を満た**さない**クレートが見つかった場合は、`violation` を使ってそのことも記録できます。

`audits.toml` の例は、次のようになります。
```
[criteria]

...

[[audits.bar]]
version = "1.2.3"
who = "Alice Foo <alicefoo@example.com>"
criteria = "safe-to-deploy"

[[audits.bar]]
delta = "1.2.3 -> 1.2.4"
who = "Bob Bar <bobbar@example.com>""
criteria = "safe-to-deploy"

[[audits.bar]]
version = "2.1.3"
who = "Alice Foo <alicefoo@example.com>"
criteria = "safe-to-deploy"

[[audits.bar]]
delta = "2.1.3 -> 2.1.1"
who = "Alice Foo <alicefoo@example.com>"
criteria = "safe-to-deploy"

[[audits.baz]]
version = "0.2"
who = "Alice Foo <alicefoo@example.com>"
criteria = "safe-to-run"

[[audits.foo]]
version = "0.2.1 -> 0.3.1"
who = "Bob Bar <bobbar@example.com>""
criteria = "safe-to-deploy"

[[audits.malicious_crate]]
violation = "*"
who = "Bob Bar <bobbar@example.com>""
criteria = "safe-to-run"

[[audits.partially_vulnerable_crate]]
violation = ">=2.0, <2.3"
who = "Bob Bar <bobbar@example.com>""
criteria = "safe-to-deploy"
```

それぞれのエントリには、`version`、`delta`、`violation` のうち、ちょうど 1 つを指定しなければなりません。

このファイルは、以前に記録したエントリが誤りだったと分かった場合を除いて、刈り込みを行わないことが想定されています。所有者が該当するクレートをもう使っていなくても、監査の記録は、エコシステムのほかの人たちの役に立つことがあります。

## `config.toml` の `exemptions` テーブル {#the-exemptions-table-in-configtoml}

このテーブルには、監査されていないにもかかわらず、プロジェクトが使っている依存を列挙します。構造はおおむね `audits` テーブルと同じですが、[いくつか違い](config.md#the-exemptions-table)があります。
{% endraw %}
