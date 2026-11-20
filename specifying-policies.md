{% raw %}
# ポリシーの指定 {#specifying-policies}

既定では、`cargo vet` は、すべての最上位のクレートの推移的な依存すべてを、すべてのプラットフォームについて、次の基準に照らして確認します。
* 通常の依存: `safe-to-deploy`
* dev-dependencies: `safe-to-run`
* build-dependencies[^1]: `safe-to-deploy`

状況によっては、要件をより正確に書き表すことで、作業の量を減らせることがあります。たとえば、ワークスペースに本番用の製品と社内用のツールの両方が含まれていて、後者の依存は `safe-to-run` であれば十分だと判断する場合です。

既定の動作で問題がなければ、何も指定する必要はありません。上のようなポリシーを書き表したい場合は、[config.toml](config.md#the-policy-table) で指定できます。

## 脚注 {#footnotes}

[^1]: 厳密に言えば、build-dependencies そのものは `safe-to-run` であり、それらがビルドにもたらすもの（たとえば生成されたコード）は safe-to-deploy であることが望まれます。
cargo-vet は、この微妙な違いを明示的に扱うために別々の基準を導入するのではなく、`safe-to-deploy` の[定義](built-in-criteria.md#safe-to-deploy)の中にまとめて含めています。
実際には、あるクレートがビルド時に動くものなのか、実行時に動くものなのかは、たいていはっきりしています。そのため、こうすることで、正確さをあまり損なわずに、物事をより単純で直感的に保てます。
{% endraw %}
