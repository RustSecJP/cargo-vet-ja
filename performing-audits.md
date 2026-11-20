{% raw %}
# 監査の実施 {#performing-audits}

人間の注意力は貴重な資源です。そこで `cargo vet` は、その注意力をできるだけ効率よく使うための機能をいくつか備えています。

## 依存の変更を管理する {#managing-dependency-changes}

`cargo update` を実行すると、たいてい新しいクレートや、既存のクレートの新しいバージョンが取り込まれます。その結果、`cargo vet` が失敗することがあります。このような場合、`cargo vet` は該当するクレートを特定し、どのように監査すればよいかを勧めます。

```
$ cargo update
  ....

$ cargo vet
  Vetting Failed!

  3 unvetted dependencies:
      bar:1.5 missing ["safe-to-deploy"]
      baz:1.3 missing ["safe-to-deploy"]
      foo:1.2.1 missing ["safe-to-deploy"]

  recommended audits for safe-to-deploy:
      cargo vet diff foo 1.2 1.2.1  (10 lines)
      cargo vet diff bar 2.1.1 1.5  (253 lines)
      cargo vet inspect baz 1.3     (2033 lines)

  estimated audit backlog: 2296 lines

  Use |cargo vet certify| to record the audits.
```

あるクレートのほかのバージョンがすでに検証されている場合、レビューの方法は複数あることに注意してください。一から行う方法と、すでに監査済みの 1 つ以上のバージョンを基準にして行う方法です。このような場合、`cargo vet` は取りうる方法をすべて計算し、最も小さいものを選びます。

もちろん、検証されていない依存を監査する代わりに、その 1 つ以上を `exemptions` の一覧に追加することもできます。状況によっては、これが手っ取り早いこともあります。ただし、これを頻繁に行うと、このツールがもたらす価値が損なわれます。

## クレートを調べる {#inspecting-crates}

行いたい監査が決まったら、次の手順は、調べる対象となる成果物を用意することです。これは、思うほど簡単ではありません。プロジェクトが GitHub のような場所でホストされていても、リポジトリ内のコードが、crates.io に提出された中身と一致している保証はないからです。しかも、crates.io のパッケージは、手作業では簡単にダウンロードできません。

これを簡単にするために、`cargo vet inspect` サブコマンドは、[Sourcegraph](https://about.sourcegraph.com/) でホストされている、そのクレートのまさにそのバージョンへのリンクを示します。

監査が終わったら、`cargo vet certify` を使って、エントリを `audits.toml` に追加できます。

```
$ cargo vet inspect baz 1.3
You are about to inspect version 1.3 of 'baz', likely to certify it for "safe-to-deploy", which means:

   ...

You can inspect the crate here: https://sourcegraph.com/crates/baz@v1.3

(press ENTER to open in your browser, or re-run with --mode=local)

$ cargo vet certify baz 1.3

  I, Alice, certify that I have audited version 1.3 of baz in accordance with
  the following criteria:

  ...

 (type "yes" to certify): yes

  Recorded full audit of baz version 1.3
```

`--mode=local` フラグを使って、`inspect` にクレートのソースコードをダウンロードさせ、それを調べるための入れ子のシェルに入ることもできます。

同じように、`cargo vet diff` は、2 つのバージョンの差分を表示する [Sourcegraph](https://about.sourcegraph.com/) のリンクを示します。

```
$ cargo vet diff foo 1.2 1.2.1

You are about to diff versions 1.2 and 1.2.1 of 'foo', likely to certify it for "safe-to-deploy", which means:

   ...

You can inspect the diff here: https://sourcegraph.com/crates/foo/-/compare/v1.2...v1.2.1

$ cargo vet certify foo 1.2 1.2.1

  I, Alice, certify that I have audited the changes between versions 1.2 and
  1.2.1 of baz in accordance with the following criteria:

  ...

  (type "yes" to certify): yes

  Recorded relative audit between foo versions 1.2 and 1.2.1
```

`--mode=local` フラグを使って、`diff` に 2 つのクレートをダウンロードさせ、両者の git 互換の差分を表示させることもできます。

## `exemptions` テーブルを減らす {#shrinking-the-exemptions-table}

プロジェクトが `cargo vet` に合格していても、`exemptions` にエントリが残っていれば、脆弱なままになっているおそれがあります。そのため、このテーブルを減らすことは、取り組む価値のある作業です。

悪意のあるクレートは、どれであってもプログラムを危険にさらしうるものです。しかし、すべてのクレートの検証に同じだけの手間がかかるわけではありません。クレートによって大きさは異なりますし、同じクレートの異なるバージョンは、たいていよく似ています。これを生かすために、`cargo vet suggest` は、`exemptions` のエントリの数を減らし、ひいては攻撃対象領域を減らすために行える、最も手間の少ない監査を見積もります。

より正確に言うと、`cargo vet suggest` は、免除されている依存それぞれについて、レビューが必要になる行数を計算し、順に並べて表示します。これは、`exemptions` を空にして `cargo vet` を実行し直したときに得られるのと同じ情報です。

## レジストリからの提案 {#suggestions-from-the-registry}

`cargo vet` は、検証に失敗したあとでも、`cargo vet suggest` の実行中でも、監査を提案するときに、[レジストリ](importing-audits.md#the-registry)の内容もあわせて取得し、利用できる集合の中に、不足の一部または全部を埋める監査が含まれていないかを確かめます。含まれていれば、それらを列挙します。開発者は、監査をすべて自分で行う代わりに、それらをインポートすることを検討できます。

```
$ cargo vet suggest
  recommended audits for safe-to-deploy:
      cargo vet inspect baz 1.3   (used by mycrate)  (2033 lines)
        NOTE: cargo vet import mozilla would reduce this to a 17-line diff
      cargo vet inspect quxx 2.0  (used by baz)      (1000 lines)
        NOTE: cargo vet import mozilla would eliminate this

  estimated audit backlog: 3033 lines

  Use |cargo vet certify| to record the audits.
```
{% endraw %}
