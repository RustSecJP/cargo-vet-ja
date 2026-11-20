{% raw %}
## 公開者を信頼する {#trusting-publishers}

`cargo vet` は、監査に加えて、あるクレートの特定の公開者によるリリースを信頼することにも対応しています。

### 動機 {#motivation}

`cargo vet` の中心的な目的は、あなたが使うそれぞれのクレートの中身に信頼を割り当てることです。このツールが監査を軸にしているのは、エコシステムにあるクレートの出どころが非常にさまざまだからです。すべての依存が信頼できる提供元によって**開発された**ことを求めるのは、たいてい現実的ではありません。そこで、次善の策として、すべてが信頼できる提供元によって**監査された**ことを確かめるのです。

しかし、開発者そのものを信頼している場合もあります。`cargo vet` では、そうしたクレートについて追加の監査の記録を求める代わりに、あるクレートの開発者が、求める基準を満たすコードを常にリリースすると信頼している、と宣言できます。

### 仕組み {#mechanics}

信頼する公開者は、`cargo vet trust` で追加できます。エントリには信頼の有効期限が必要で、これにより、その判断が定期的に見直されるようになります。

信頼関係は、`audits.toml` の `trusted` セクションに記録されます。
```
[[trusted.baz]]
criteria = "safe-to-deploy"
user-id = 5555 // Alice Jones
start = ...
end = ...
notes = "Alice is an excellent developer and super-trustworthy."
```

### 提案 {#suggestions}

あなたの監査の集合、またはインポートしている監査の集合の中に、ある公開者についての信頼のエントリがすでにある場合、`cargo vet suggest` は、同じ公開者による、まだ監査されていない新しいクレートについて、信頼のエントリを追加することを検討するよう提案します。

```
$ cargo vet suggest
  recommended audits for safe-to-deploy:
      cargo vet inspect baz 1.3   (used by mycrate)  (2033 lines)
        NOTE: mozilla trusts Alice Jones (ajones) - consider cargo vet trust baz or cargo vet trust --all ajones
```

信頼のエントリは、本質的には経験則にすぎません。信頼する公開者に確認を取るわけではなく、その公開者がすべてのコードを自分で書いたり、レビューしたりしているとは限りません。そのため、クレートを信頼する前に、リスクを評価し、場合によっては開発とリリースの進め方について調べておくことが重要です。
{% endraw %}
