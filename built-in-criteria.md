{% raw %}
# 組み込みの基準 {#built-in-criteria}

基準は好きなように定義できますが、`cargo vet` には、よく使われる 2 つの監査基準が
最初から用意されています。これらの基準は、プロジェクトをまたいで自動的に
対応づけられます。

## safe-to-run {#safe-to-run}

```
This crate can be compiled, run, and tested on a local workstation or in
controlled automation without surprising consequences, such as:
* Reading or writing data from sensitive or unrelated parts of the filesystem.
* Installing software or reconfiguring the device.
* Connecting to untrusted network endpoints.
* Misuse of system resources (e.g. cryptocurrency mining).
```

日本語訳（参考）:

> このクレートは、ローカルのワークステーションや管理された自動化環境で、
> コンパイル・実行・テストをしても、次のような予期しない結果を招きません。
> * ファイルシステムのうち、機密性の高い部分や無関係な部分のデータを読み書きする。
> * ソフトウェアをインストールしたり、デバイスの設定を変更したりする。
> * 信頼できないネットワークのエンドポイントに接続する。
> * システムの資源を不正に使用する（例: 暗号通貨の採掘）。

## safe-to-deploy {#safe-to-deploy}

```
This crate will not introduce a serious security vulnerability to production
software exposed to untrusted input.

Auditors are not required to perform a full logic review of the entire crate.
Rather, they must review enough to fully reason about the behavior of all unsafe
blocks and usage of powerful imports. For any reasonable usage of the crate in
real-world software, an attacker must not be able to manipulate the runtime
behavior of these sections in an exploitable or surprising way.

Ideally, all unsafe code is fully sound, and ambient capabilities (e.g.
filesystem access) are hardened against manipulation and consistent with the
advertised behavior of the crate. However, some discretion is permitted. In such
cases, the nature of the discretion should be recorded in the `notes` field of
the audit record.

For crates which generate deployed code (e.g. build dependencies or procedural
macros), reasonable usage of the crate should output code which meets the above
criteria.
```

日本語訳（参考）:

> このクレートは、信頼できない入力にさらされる本番環境のソフトウェアに、
> 深刻なセキュリティ上の脆弱性を持ち込みません。
>
> 監査者は、クレート全体のロジックを完全にレビューする必要はありません。
> その代わりに、すべての unsafe ブロックの挙動と、強力なインポートの使われ方について
> 十分に筋道を立てて判断できるだけのレビューをしなければなりません。実際の
> ソフトウェアでこのクレートを妥当な形で使うかぎり、攻撃者がこれらの部分の
> 実行時の挙動を、悪用できる形や予期しない形で操作できてはなりません。
>
> 理想的には、すべての unsafe コードが完全に健全であり、環境から暗黙に得られる
> 権限（例: ファイルシステムへのアクセス）が操作に対して堅牢で、クレートがうたって
> いる挙動と一致しています。ただし、ある程度の裁量は認められます。その場合は、
> 裁量の内容を監査記録の `notes` フィールドに記録してください。
>
> デプロイされるコードを生成するクレート（例: ビルド依存や手続き型マクロ）に
> ついては、そのクレートを妥当な形で使ったときに、上記の基準を満たすコードが
> 出力されなければなりません。

これは `safe-to-run` を含意します。
{% endraw %}
