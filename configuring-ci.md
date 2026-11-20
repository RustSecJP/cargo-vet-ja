{% raw %}
# CI の設定 {#configuring-ci}

プロジェクトのセットアップの最後の手順として、プロジェクトの継続的インテグレーションの仕組みの一部として検証が実行されるようにしておきましょう。

プロジェクトが GitHub でホストされている場合は、`.github/workflows` にある新規または既存の `.yml` ファイルに、次の内容を追加することで実現できます（`X.Y.Z` は使いたいバージョンに置き換えてください）。

```yml
name: CI
on: [push, pull_request]
jobs:
  cargo-vet:
    name: Vet Dependencies
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
    - name: Invoke cargo-vet
      run: cargo vet --locked
```

これにより、PR 経由でも直接のプッシュでも、リポジトリに加えられるすべての変更について、依存の集合が完全に検証済みであることが保証されます。ツールキャッシュまわりの追加の処理は、cargo-vet のバイナリを毎回一からコンパイルするのではなく、GitHub にその複製を保持させるためのものです。これにより、結果が数分ではなく数秒で表示されるようになります。
{% endraw %}
