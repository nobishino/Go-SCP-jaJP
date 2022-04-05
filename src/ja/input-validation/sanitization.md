Sanitization
============

Sanitization refers to the process of removing or replacing submitted data.
When dealing with data, after the proper validation checks have been made,
sanitization is an additional step that is usually taken to strengthen data
safety.

The most common uses of sanitization are as follows:

サニタイズ
============

サニタイズとは、送信されたデータを部分的に削除したり、置き換えたりする処理のことです。適切なバリデーションチェックが行われた後にデータの安全性を強化するために行われる追加的なステップです。

サニタイズの代表的な用途は以下の通りです。

## 比較演算子 `<` をエンティティーに変換する

ネイティブパッケージ `html` には、サニタイズに利用できる関数が 2 つあります。
HTML テキストをエスケープするためのものと、HTML をアンエスケープするためのものです。
関数 `EscapeString()` は、文字列を受け取って、エスケープ処理した文字列を返します。例えば文字列中の`<`が`&lt;`にエスケープされます。

**NOTE**: この関数は次の5文字のみをエスケープします。`<`、`>`、`&`、`'`、`"`

その他の文字は、手動でエンコードするか、サードパーティライブラリを使う必要があります。
逆に、`UnescapeString()`という関数はエンティティを文字に変換します。

## Strip all tags

Although the `html/template` package has a `stripTags()` function, it's
unexported. Since no other native package has a function to strip all tags, the
alternatives are to use a third-party library, or to copy the whole function
along with its private classes and functions.

## すべてのタグを取り除く

`html/template` パッケージには `stripTags()` 関数がありますが、これは
プライベート関数なためエクスポートできません。他のネイティブパッケージにはすべてのタグを除去する関数がないのでサードパーティのライブラリを使うか、`stripTags()` 関連するプライベートなクラスや関数を一緒にをコピーする必要があります。

これを実現するためのサードパーティライブラリには、以下のようなものがあります。

* https://github.com/kennygrant/sanitize
* https://github.com/maxwells/sanitize
* https://github.com/microcosm-cc/bluemonday

## Remove line breaks, tabs and extra white space

The `text/template` and the `html/template` include a way to remove whitespaces
from the template, by using a minus sign `-` inside the action's delimiter.

Executing the template with source

## 改行、タブ、余分な空白を削除する

`text/template` と `html/template` を用いて、
アクションの区切り文字にマイナス記号 `-` を使用することでテンプレートから空白を削除できます。

```
{{- 23}} < {{45 -}}
```

は以下に変換されます

```
23<45
```

**NOTE**: If the minus `-` sign is not placed immediately after the opening
action delimiter ``{{`` or before the closing action delimiter ``}}``, the
minus sign `-` will be applied to the value

**NOTE**: マイナス記号 `-` がオープニングデリミタ`{{`の直後、またはクロージングデリミタ `}}` の直後に置かれていない場合は通常のマイナス記号 `-` として扱われます。

```
{{ -3 }}
```

は以下に変換されます。

```
-3
```

## URLリクエストパス

`net/http` パッケージには、以下のような HTTP リクエストのマルチプレクサがあります。
`ServeMux` です。これは、送られてくるリクエストをパターンマッチすることで、要求されたURLに最も近いハンドラを呼び出します。
さらに URLをサニタイズし、`.`や`..`要素を含むリクエスト、またはスラッシュを繰り返すようなURLパターンを、簡潔な同等のURLに変換してリダイレクトします。

簡単なMuxの例を示します。

```go
func main() {
  mux := http.NewServeMux()

  rh := http.RedirectHandler("http://yourDomain.org", 307)
  mux.Handle("/login", rh)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```

**NOTE**: `ServMux`は`CONNECT`リクエストに対してURLリクエストのパスを[変更しないこと][2]に注意してください。リクエストメソッドを制限しない場合 [パストラバーサル脆弱性][3] にさらされる可能性があるということです。

以下のサードパーティパッケージは、ネイティブの HTTP リクエストマルチプレクサの代替となるものです。
継続的にテストされ、メンテナンスされているパッケージを選択してください。


* [Gorilla Toolkit - MUX][1]

[1]: http://www.gorillatoolkit.org/pkg/mux
[2]: https://golang.org/pkg/net/http/#ServeMux.Handler
[3]: https://ilyaglotov.com/blog/servemux-and-path-traversal