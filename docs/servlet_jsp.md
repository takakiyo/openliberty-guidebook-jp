## Java EEの基本: ServletとJSP

この章では，ServletとJSPについて実際に開発を行いながら学習します。

アプリケーションは前の章で利用した`guide-app`を使用します。

`guide-app`で利用しているLibertyの構成ファイルでは，Jakarta EE 10のAPIを提供するFeatureが全て有効になっています。Libertyは，利用する機能，Featureだけを有効にすることで実行環境を軽量にたもてるという特長を持っています。より快適に開発を行うために，必要なAPIだけを有効にします。

`src/main/liberty/config`ディレクトリにある`server.xml`を開きます。

`<featureManager>`の設定では，Jakarta EE 10の全てのAPIを使用できるようにする`jakartaee-10.0`のFeatureが有効にされています。

``` xml
<!-- Enable features -->
<featureManager>
    <feature>jakartaee-10.0</feature>
    
</featureManager>
```

これを以下のように書き換えて保存します。

``` xml
<!-- Enable features -->
<featureManager>
    <feature>servlet-6.0</feature>
    <feature>pages-3.1</feature>
</featureManager>
```

これでLibertyの使用するメモリが少なくなり，起動時間もわずかに短くなります。

### Servletとは

Servletは，Java EE/Jakarta EE仕様の一部として提供されるAPIで，主にWebアプリケーションのバックエンドとして利用されます。クライアント（通常はWebブラウザ）からのリクエストに応じて動的なコンテンツを生成し，レスポンスとして返す役割を果たします。

ServletはHTTPリクエストとレスポンスを直接扱うことができる機能を提供しています。昨今では，この階層を意識することなく，Webアプリケーションを簡便に作成できる多くのフレームワークが提供されています。ですが，それらのフレームワークの中にも，内部ではServlet APIを使用しているものが多くあります（Spring Boot/Spring MVCなどが代表例です）。

実際の業務アプリケーションの構築にあたっては，フレームワークに隠蔽されて，Servletの機能をつかう機会は少なくなってるかもしれません。それでも，初学者がHTTP通信を利用したWebアプリケーションでは内部でなにがおこなわれているかを学習するにあたっては，現在でも有用な手段でしょう（またフレームワークでトラブルがあった場合も，内部で使用されているServletの知識があると，解決に役立つことが多々あります）。

### サンプルServletの実装

`guide-app`にServletを実装します。

`src/main/java/com/demo`フォルダーに`HelloServlet.java`というファイルを作成します。VS Codeのエクスプローラーで`java/com/demo/rest`の`demo`の部分を右クリックし「新しいファイル...」をクリックしてファイル名を入力します。

作成したファイルに以下の内容を記述して保存します。Servletは，`HttpServlet`クラスを継承（extends）したクラスとして作成します。

``` java
package com.demo;

import java.io.IOException;
import java.io.PrintWriter;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
    {
        // リクエストの指定する言語を取得
        String language = req.getLocale().getLanguage();
        String greeting = switch(language) {
            case "fr" -> "Bonjour, ";
            case "ja" -> "こんにちは、";
            case "zh" -> "你好，";
            default -> "Hello, ";
        };
        // レスポンスのヘッダを設定
        resp.setHeader("Vary", "Accept-Language");
        resp.setHeader("Content-Language", language);
        // レスポンスをHTMLに指定してPrintWriterを取得
        resp.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        // レスポンスを出力
        writer.println("<!DOCTYPE html>");
        writer.println("<html>");
        writer.println("<head><title>Hello Servlet</title></head>");
        writer.println("<body>" + greeting + "Open Liberty!</body>");
        writer.println("</html>");
    }
}
```

![HelloServlet.javaを作成](../images/servlet_jsp1.png)

「LIBERTY DASHBOARD」から「Stert」でLibertyを起動し，正常に起動したら，ブラウザで[http://localhost:9080/guide-app/hello](http://localhost:9080/guide-app/hello)にアクセスしてみましょう。

ほとんどの環境で以下のように表示されたかと思います。

![HelloServletの実行結果](../images/servlet_jsp2.png)

コマンドプロンプトから`curl`コマンドを実行して，ヘッダをつけてLiberty上のServletにアクセスしてみます。

``` terminal
> curl -H "Accept-Language: fr"  http://localhost:9080/guide-app/hello
<!DOCTYPE html>
<html>
<head><title>Hello Servlet</title></head>
<body>Bonjour, Open Liberty!</body>
</html>

> curl -H "Accept-Language: zh-TW"  http://localhost:9080/guide-app/hello
<!DOCTYPE html>
<html>
<head><title>Hello Servlet</title></head>
<body>你好，Open Liberty!</body>
</html>
```

指定する言語によって，異なる挨拶が返ってきていることが確認できるかと思います。

#### HTTPリクエストとレスポンス

上記のServletをブラウザで表示したときに，裏でなにが起こっていたのかを確認します。

多くのブラウザでは，上記のServletを表示している画面で`F12`キーをおすと，開発者ツールが起動します。たとえばWindowsのEdgeブラウザでは，以下のようなメッセージが表示された後，「DevToolを開く」をクリックすると開発者ツールが利用できます。

![開発者ツールの利用](../images/servlet_jsp3.png)

開発者ツールの利用方法は使用しているブラウザによって異なるため，ここでは詳細には説明しません。開発者ツールを利用すると，下記の画面のようにサーバーとブラウザの通信内容を詳細に観察することができます。

![開発者ツールによるネットワーク通信の調査](../images/servlet_jsp4.png)

先ほどのServletが実行された際に，クライアントであるブラウザからサーバーへは，以下のようなリクエストが送信されています。

```
GET /guide-app/hello HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: ja,en;q=0.9,en-GB;q=0.8,en-US;q=0.7
Cache-Control: max-age=0
Connection: keep-alive
Cookie: JSESSIONID=0000nDDVQqpo3c0u79LD11wR-8i:845f014d-75a1-40d8-8a0c-0285824f7233
Host: localhost:9080
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36 Edg/119.0.0.0
sec-ch-ua: "Microsoft Edge";v="119", "Chromium";v="119", "Not?A_Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"

```

HTTPのリクエスト・メッセージは以下のような構造になっています。

- **スタートライン**
    - メッセージの1行目です。**リクエストライン**とも呼ばれます。
    - リクエストのメソッド（`GET`や`POST`など），ターゲット（上記の例では`/guide-app/hello`）と，HTTPバージョンから構成されています。
- **ヘッダー**
    - `ヘッダー名: 内容`の形式で，情報が行ごとに並んでいます。
    - リクエストのコンテキストやサーバーへの追加指示を提供します。
- **空行**
    - ヘッダーとボディの間にある空の行。
- **ボディ**（オプショナル）
    - `POST`や`PUT`リクエストで使用されます。

サーバーからブラウザへは，以下のようなレスポンスが送信されています（ここではchunked encodingによる転送時の追加文字は削除しています）。

```
HTTP/1.1 200 OK
Vary: Accept-Language
Content-Language: ja
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Date: Thu, 23 Nov 2023 05:47:06 GMT

<!DOCTYPE html>
<html>
<head><title>Hello Servlet</title></head>
<body>こんにちは、Open Liberty!</body>
</html>
```

レスポンス・メッセージは，同様に以下のような構造になっています。

- **スタートライン**
    - メッセージの1行目です。ステータストラインとも呼ばれます。
    - HTTPバージョン，ステータスコード（リクエストに対する結果を示す3桁の数値コード），ステータステキストから構成されています。
- **ヘッダー**
    - `ヘッダー名: 内容`の形式で，情報が行ごとに並んでいます。
    - レスポンスに関するメタデータや，クライアントへの指示を提供します。
- **空行**
    - ヘッダーとボディの間にある空の行。
- **ボディ**
    - リクエストに対する実際のデータまたは内容。
    - HTML文書，画像，JSONデータなど，リクエストに応じた形式でデータが含まれます。

Servlet APIは，このリクエストのメッセージを受信し解析し，レスポンスのメッセージを生成して送信するための多くの機能を提供しています。

#### URLとServletのマッピング

ServletやJSPなどのWebアプリケーションはWARファイルにパッケージされて，サーバーで実行されます。Webアプリケーションをサーバーに登録する際には，アプリケーションを呼び出すコンテキスト・ルート（Context Root）を指定します。

今回使用している`guide-app`は，サーバーの構成ファイル`server.xml`で以下のように登録されています。

``` xml
<webApplication contextRoot="/guide-app" location="guide-app.war" />
```

コンテキストルートはURLの構造において，ドメイン名に続く最初のパスセグメントとして利用されます。Servletを呼び出したときのURL，`http://localhost:9080/guide-app/hello`のパスセグメントは`/guide-app/hello`です。このうち`/guide-app`がWebアプリケーションを指定するコンテキスト・ルートとして使用されます。アプリケーション内のパスは`/hello`になります。

Webアプリケーション内のパスのマッピングは，Javaコード内の`@WebServlet`アノテーションで指定されています。これにより，`/hello`のパスでこのServletが呼び出されるようにマップされます。このように，特定のパスを明記したマッピングを完全一致といいます。

``` java
@WebServlet("/hello")
```

`@WebServlet`アノテーションに複数の情報を記述する場合は，マッピング情報は`urlPatterns`で記述します。

``` java
@WebServlet(urlPatterns = "/hello", loadOnStartup = 100)
```

マッピングには`/*`で終わる文字列を指定できます。その場合はパス一致として扱われ，そのディレクトリ以下の全ての呼出しにマッチし，指定されたServletが呼び出されます。このマッピングを前方一致といいます。

``` java
@WebServlet("/admin/*")
```

また`*.`で始まる文字列で拡張子の指定も可能です。以下のような指定では，`.do`でおわる任意のパスセグメントとマッチし，Servletが呼び出されます。このマッピングを拡張子ルールといいます。

``` java
@WebServlet("*.do")
```

リクエストされたパスが複数のマッピング設定に一致した場合，以下の順位で上のものほど優先されます。

- 完全一致
- 拡張子ルール一致
- 前方一致（複数にマッチした場合，より長いマッピングが使用される）
- 拡張子JSPなど，実行環境で設定されているルール

いずれにも該当しないパスが要求された場合，`webapp`フォルダーの中のコンテンツから一致するものが検索され，見つかればそのファイルがそのまま送信されます。

#### GETリクエストとPOSTリクエスト

HTTPリクエストにおけるメソッドにはいくつもの種類がありますが，Servlet仕様で対応しているメソッドとしては，以下のようなものがあります。

- **GET**
    - もっとも多用されるメソッドです。ブラウザにURLを入力してページを表示したり，`<a href="〜">`によるリンクをたどったりしたときに使用されます。
    - リソースを取得するために使用されます。サーバー側のデータを変更せず，読み取り専用の操作に最適です。
    - データ操作CRUDのReadに該当します。
- **POST**
    - データをサーバーに送信するために使用されます（例: フォームデータの送信）。
    - 新しいリソースを作成するために使用されることが多いです。
    - データ操作CRUDのCreate/Updateに該当します。
- **PUT**
    - 既存のリソースを更新または置き換えるために使用されます。
    - データ操作CRUDのUpdateに該当します。
- **DELETE**
    - 指定されたリソースを削除するために使用されます。
    - データ操作CRUDのDeleteに該当します。
- **HEAD**
    - GETと同様ですが，レスポンスにはボディが含まれません。
    - リソースのメタデータ（例: ヘッダー情報）のみを取得するために使用されます。
- **OPTION**
    - 対象のリソースで利用可能な通信オプションを調べるために使用されます。
- **TRACE**
    - クライアントからサーバーへのリクエストメッセージをサーバーに「エコー」させることによって，そのリクエストがネットワーク上でどのように処理されているかをトレースします。

一般的にWebアプリケーションで使用されるのはGETとPOSTメソッドです。

{% note %}

HTTPのメソッドで重要な概念として冪等性と安全性があります。

**冪等性（Idempotency）** とは，同じリクエストを一度だけ実行しても二度以上実行しても，その結果が初回と同じであることを保証します。つまり，リクエストの繰り返しが追加の副作用を引き起こさないことを意味します。

GET，HEAD，PUT，DELETEは冪等なメソッドです。たとえば，同じリソースに対するDELETEリクエストを何度実行しても，最初の成功したリクエスト以降は何も変化しません。

POSTは冪等ではありません。同じPOSTリクエストを繰り返すと，新しいリソースが何度も作成される可能性があります。

**安全性（Safety）** とは，サーバー上のリソースの状態を変更しないことを保証します。これらのメソッドは，データの取得や読み取りに使用され，データの作成，更新，削除などの書き込み操作を行いません。

GET，HEADなどは安全なメソッドです。これらは情報の取得のみを目的としており，サーバー側のリソースの状態を変更しません。

POST，PUT，DELETEなどは安全ではありません。これらはリソースの状態を変更するために使用されます。

ブラウザやHTTPリクエストを中継，キャッシュする中間プログラムは，これらのメソッドの特性に応じた動作をします。たとえば，ブラウザでGETで表示したページでリロードを実行すると，何の警告もなく再度画面が表示されます（GETメソッドは安全なので，繰り返し要求を送ってもサーバーの状態を変更する心配がないからです）。ですが，ブラウザでPOSTで表示したページでリロードを試みると，安全ではないリクエストを再度実行してもいいのか，確認の画面が表示されます。

Webアプリケーションを実装するさいには，これらのHTTPメソッドに応じたアプリケーションの挙動の原則を守ることが重要です。

ただ，あくまで原則です。GETでサーバー側の状態を一切変更していけない，というわけではありません。安全な変更ならば構いません。たとえば，何回アクセスしたかを表示するアクセスカウンターなどは，表示するたびに数字が増えますが，何回でも安全にリクエストを送信することができますので，GETでリクエストを処理しても構いません。

{% endnote %}

`HttpServlet`クラスには，これらの7つのHTTPメソッドを処理するためのメソッドが定義されており，実際のServletではこれらのメソッドをオーバーライド（Override）して，処理内容を記述します。`GET`メソッドのリクエストを処理するのは`doGet`，`POST`メソッドを処理するのは`doPost`，というようなメソッドが定義されています。

メソッドのシグネーチャー（引数と戻り値，throwするException）は以下のようになっています。リクエストの情報を取得するための`HttpServletRequest`のインスタンス，レスポンスを返すための`HttpServletResponse`のインスタンスが引数として渡されてきます。

``` java
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException
```

処理を実行するメソッドについて，Servletの定義の中でオーバーライドします。

``` java
@Override
public void doGet(HttpServletRequest req, HttpServletResponse resp)
    throws ServletException, IOException {
    // 処理内容
}
```

対応しないメソッドについてはオーバーライドする必要はありません。オーバーライドしていないメソッドをもつリクエストが送信された場合は，`405 Method Not Allowed`のステータスコード・メッセージのレスポンスが返されます。

#### HttpServletRequest：リクエストの処理

##### HttpServletRequestから得られる情報

Servletに実行時に引数として渡される`HttpServletRequest`には，多くのGetterメソッドが定義されていて，リクエストに関するさまざまな情報を得られます。

サンプルのServletを作成して確認してみましょう。プロジェクトの`src/main/java/com/demo`フォルダーに`SnoopServlet.java`というファイルを作成し，以下の内容を記述して保存します。

``` java
package com.demo;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;
import java.util.Locale;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/snoop/*")
public class SnoopServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) 
        throws IOException, ServletException
    {
        execute(req, resp);
    }
    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws IOException, ServletException
    {
        execute(req, resp);
    }
    private void execute(HttpServletRequest req, HttpServletResponse resp)
        throws IOException, ServletException
    {
        resp.setContentType("text/plain; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        // リクエスト・ラインに関する情報
        writer.println("Request Method      : " + req.getMethod());
        writer.println("Request URI         : " + req.getRequestURI());
        writer.println("Request Protocol    : " + req.getProtocol());
        writer.println("Context Path        : " + req.getContextPath());
        writer.println("Servlet Path        : " + req.getServletPath());
        writer.println("Path Info           : " + req.getPathInfo());
        writer.println("Path Translated     : " + req.getPathTranslated());
        writer.println("Query String        : " + req.getQueryString());
    }
}
```

このサーブレットでは，リクエストラインから得られる情報をサーバーが解釈した結果を`HttpServletRequest`から取得して表示しています。`curl`コマンドなどからも呼び出しやすいように，応答のContent-typeは`text/plain`にしてあります。URLのマッピングは`@WebServlet("/snoop/*")`と前方一致で設定されていますので`/snoop`で始まる任意のパスに一致します。

コマンドラインから，以下のようなURLで呼び出して，Path Infoなどがどのように変化するか確認してください。

- `curl http://localhost:9080/guide-app/snoop`
- `curl http://localhost:9080/guide-app/snoop/`
- `curl http://localhost:9080/guide-app/snoop/test`

また，以下のような呼び出しでQuery Stringがかわります。

- `curl "http://localhost:9080/guide-app/snoop?foo=bar&hoge=fuga"`

このSnoopServletについて，さらに追加の情報を取得するように拡張したものがこちらになります。

``` java
package com.demo;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;
import java.util.Locale;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/snoop/*")
public class SnoopServlet extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) 
        throws IOException, ServletException
    {
        execute(req, resp);
    }
    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp) 
        throws IOException, ServletException 
    {
        execute(req, resp);
    }
    private void execute(HttpServletRequest req, HttpServletResponse resp) 
        throws IOException, ServletException 
    {
        resp.setContentType("text/plain; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        // リクエスト・ラインに関する情報
        writer.println("Request Method      : " + req.getMethod());
        writer.println("Request URI         : " + req.getRequestURI());
        writer.println("Request Protocol    : " + req.getProtocol());
        writer.println("Context Path        : " + req.getContextPath());
        writer.println("Servlet Path        : " + req.getServletPath());
        writer.println("Path Info           : " + req.getPathInfo());
        writer.println("Path Translated     : " + req.getPathTranslated());
        writer.println("Query String        : " + req.getQueryString());
        // ヘッダーから得られる情報
        writer.println("Content Length      : " + req.getContentLength());
        writer.println("Content Type        : " + req.getContentType());
        writer.println("Character Encoding  : " + req.getCharacterEncoding());
        writer.println("Locale              : " + req.getLocale());
        writer.print("Locales             : ");
        Enumeration<Locale> e = req.getLocales();
        while (e.hasMoreElements()) {
            writer.print(e.nextElement() + " ");
        }
        writer.println();
        // ネットワーク接続についての情報
        writer.println("Local Name          : " + req.getLocalName());
        writer.println("Local Port          : " + req.getLocalPort());
        writer.println("Server Name         : " + req.getServerName());
        writer.println("Server Port         : " + req.getServerPort());
        writer.println("Remote Address      : " + req.getRemoteAddr());
        writer.println("Remote Host         : " + req.getRemoteHost());
        writer.println("Remote Port         : " + req.getRemotePort());
        writer.println("Request ID          : " + req.getRequestId());
        writer.println("Secure Connection   : " + req.isSecure());
        // 認証についての情報
        writer.println("Authorization Scheme: " + req.getAuthType());
        writer.println("Remote User         : " + req.getRemoteUser());
    }
}
```

呼び出しているメソッドの正確な意味については，Javadocを参照してください。

- HttpServletRequest (Jakarta Servlet API documentation)
    - [https://jakarta.ee/specifications/servlet/6.0/apidocs/jakarta.servlet/jakarta/servlet/http/httpservletrequest](https://jakarta.ee/specifications/servlet/6.0/apidocs/jakarta.servlet/jakarta/servlet/http/httpservletrequest)

##### リクエスト・ヘッダの取得

`HttpServletRequest`からは，ヘッダーについての情報を直接取得することもできます。

たとえば`User-Agent`というヘッダーを`HttpServletRequest`のインスタンス`req`から取得するには，以下のようなコードを利用します。

``` java
String userAgent = req.getHeader("User-Agent");
```

受け取ったヘッダーの名前の一覧は，`getHeaderNames()`で取得できます。これをつかって，受け取ったヘッダーを全て列挙するサーブレットがこちらになります。

``` java
package com.demo;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/headers")
public class HederServlet  extends HttpServlet {
    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) 
        throws IOException, ServletException 
    {
        execute(req, resp);
    }
    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws IOException, ServletException 
    {
        execute(req, resp);
    }
    private void execute(HttpServletRequest req, HttpServletResponse resp)
        throws IOException, ServletException 
    {
        resp.setContentType("text/plain; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        Enumeration<String> names = req.getHeaderNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            String value = req.getHeader(name);
            writer.println(name + ": " + value);
        }
    }
}
```

##### パラメーターの読み取り

``` html
```

##### 属性（attribute）の読み書き

`HttpServletRequest`には，属性（attribute）の保存が可能です。

`HttpServletRequest`の属性は，Webアプリケーション開発において，リクエスト処理の間にデータを保存し，異なるコンポーネント間で共有するために使用されます。サーブレットやJSPなどのWebコンポーネント間で情報を伝達するのに便利な手段です。

後述するように，ServletとJSPの処理を連携させ，ビジネスロジックの処理をServletで実行し，結果をJSPで表示する，というような使い方ができます。このとき，ServletからJSPに情報を連携する手段として，`HttpServletRequest`の属性が使用されます。

また，認証フィルターがユーザーの認証情報を属性に設定し，その後の処理でこの情報に基づいてアクセス制御を行うこともできます。

`HttpServletRequest`のインスタンス`req`に属性を保存するには`setAttribute`メソッドを使用します。

``` java
req.setAttribute("com.demo.message", message);
```

属性を読み出すには，`getAttribute`メソッドを使用します。

``` java
Massage message = (Message)req.getAttribute("com.demo.message");
```

保存した属性が有効なのは，リクエストの処理が開始されてから処理が完了し，レスポンスを返すまでです。リクエストの処理が終了すると，内容は失われます。

#### HttpSession：複数のリクエストにまたがるセッション

#### HttpServletResponse：レスポンスの処理

#### RequestDispatcher：リクエストのディスパッチ

#### Servletとマルチスレッド，スコープ

#### Servletフィルター

### JSPとは

