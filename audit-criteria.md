{% raw %}
# 監査基準 {#audit-criteria}

コードの監査に取りかかる前に、監査で何を確かめるのかを決めておく必要があります。これを表すのが「監査基準」です。監査基準は、何を確認すべきかを人間が読める形で書いた説明に対応する、単なるラベルです。

`cargo vet` には、[safe-to-run](built-in-criteria.md#safe-to-run) と [safe-to-deploy](built-in-criteria.md#safe-to-deploy) という 2 つの組み込みの基準があらかじめ用意されています。これらは、追加の設定なしで使えます。

## カスタムの基準 {#custom-criteria}

`audits.toml` で、任意のカスタムの基準を指定することもできます。たとえば次のとおりです。

```
[criteria.crypto-reviewed]
description = '''
The cryptographic code in this crate has been reviewed for correctness by a
member of a designated set of cryptography experts within the project.
'''
```

すべての機能については、[こちら](config.md#the-criteria-table)に説明があります。

[集約した監査](multiple-repositories.md)を使っている場合、それぞれの基準の `description` は、すべてのリポジトリで**完全に同一**でなければなりません。そうでないと、集約は失敗します。

## 複数の基準の集合 {#multiple-sets-of-criteria}

複数の基準の集合を使い分けたくなる理由は、いくつもあります。
* **一部のクレートに追加の確認を課す:** たとえば、`crypto-reviewed` という基準を定義し、アプリケーションが依存する暗号アルゴリズムを実装したクレートの監査では、その基準を必須にできます。

* **一部のクレートについて監査の要件を緩める:** たとえば、本番環境で外部にさらされないクレートは、`safe-to-deploy` ではなく `safe-to-run` で十分だと判断できます。こうしたクレートは、悪意のある入力の扱いについて監査する必要がないからです。

    ```
    [policy.mycrate]
    criteria = ["safe-to-deploy"]
    dependency-criteria = { non-exposed-crate = ["safe-to-run"] }
    ```

* **共有しやすくする:** あるプロジェクトが問題 A と B について監査したいと考え、別のプロジェクトが B と C について監査したいと考えている場合、A、B、C それぞれに別々の基準の集合を定義しておけば、2 つのプロジェクトは作業の一部を共有できます。

別々の基準の集合は、好きなだけ定義して使えます。

### 基準の集合の例: Google の `ub-risk-N` {#example-criteria-set-googles-ub-risk-n}

Google の Rust クレートの監査では、含意の連鎖をなす 7 つの監査基準が定義されています。`ub-risk-0` から `ub-risk-4` までと、unsafe Rust の専門家 2 人が監査を行ったことを示す `ub-risk-1-thorough` および `ub-risk-2-thorough` です。この基準の集合を使いたいプロジェクトの多くは、ポリシーの基準として `ub-risk-2` を指定し、`ub-risk-3` のクレートについてはクレートごとにポリシーの例外を指定するとよいでしょう。

この基準の集合の注目すべき特徴は、組織として使用を認められないと判断したクレートについても、監査を記録できること（`ub-risk-4`）です。これは、将来そのクレートを見直すときに、問題が修正されたかどうかを追跡するのに役立ちます。

これらの基準は、`registry.toml` の `[registry.google]` にあるリンク、および <https://github.com/google/rust-crate-audits/blob/main/auditing_standards.md> で確認できます。
{% endraw %}
