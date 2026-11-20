{% raw %}
# 複数のリポジトリ {#multiple-repositories}

ここまでの説明は、プロジェクトが 1 つのリポジトリにあることを前提にしていました。しかし、組織が複数のリポジトリにまたがってコードを管理していることはよくあります。一見すると、これは、監査の記録を 1 か所に集めるべきか、分散させるべきかという板挟みを生みます。すべてを 1 か所に置けば、利用するのは楽になりますが、作るのは面倒になります。あるリポジトリでパッケージを更新するたびに、開発者が別のリポジトリに新しい監査を記録しなければならなくなることがあるからです。

`cargo vet aggregate` サブコマンドは、この板挟みを解消します。このコマンド自体は、監査ファイルの URL の一覧を受け取り、1 つに統合したファイルを作るだけです[^1]。
推奨する作業の流れは次のとおりです。
1. 統合した監査を置くための専用のリポジトリを作ります（[例](https://github.com/mozilla/supply-chain)）。
2. このリポジトリに `sources.list` というファイルを追加します。このファイルには、各プロジェクトの監査ファイルの URL を、そのまま並べて書きます。
3. そのリポジトリに、`cargo vet aggregate
   sources.list > audits.toml` を実行し、結果が変わっていればコミットする定期的なタスクを作ります[^2]。
4. 集約した監査ファイルを、個々のリポジトリの `imports` テーブルに追加します。

この方法は、プロジェクト内の作業の流れを効率よくするだけではありません。ほかの人たちが、さまざまな元のリポジトリの細かな事情を把握しなくても、監査の集合の全体を簡単にインポートできるようにもなります。

[^1]: 新しいファイルのエントリには、元の場所を指す `aggregated-from` フィールドが追加されます。

[^2]: GitHub では、次の内容を `.github/workflows/aggregate.yml` に追加することで実現できます。
```yml
name: CI
on:
  schedule:
    # 5 分ごと（GitHub が許可している最大の頻度）
    - cron:  '*/5 * * * *'

permissions:
  contents: write

jobs:
  aggregate:
    name: Aggregate Dependencies
    runs-on: ubuntu-latest
    env:
      CARGO_VET_VERSION: X.Y.Z
    steps:
    - uses: actions/checkout@master
    - name: Install Rust
      run: rustup update stable && rustup default stable
    - uses: actions/cache@v2
      with:
        path: ${{ runner.tool_cache }}/cargo-vet
        key: cargo-vet-bin-${{ env.CARGO_VET_VERSION }}
    - name: Add the tool cache directory to the search path
      run: echo "${{ runner.tool_cache }}/cargo-vet/bin" >> $GITHUB_PATH
    - name: Ensure that the tool cache is populated with the cargo-vet binary
      run: cargo install --root ${{ runner.tool_cache }}/cargo-vet --version ${{ env.CARGO_VET_VERSION }} cargo-vet
    - name: Invoke cargo-vet aggregate
      run: cargo vet aggregate --output-file audits.toml sources.list
    - name: Commit changes (if any)
      run: |
        git config --global user.name "cargo-vet[bot]"
        git config --global user.email "cargo-vet-aggregate@invalid"
        git add audits.toml
        git commit -m "Aggregate new audits" || true
    - name: Push changes (if any)
      run: git push origin main
```
{% endraw %}
