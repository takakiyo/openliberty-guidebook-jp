## Java EEの基本: ServletとJSP

この章では，ServletとJSPについて実際に開発を行いながら学習します。

アプリケーションは前の章で利用した`guide-app`を使用します。

`guide-app`で利用しているLibertyの構成ファイルでは，Jakarta EE 10のAPIを提供するFeatureが全て有効になっています。Libertyでは，利用する機能，Featureだけを有効にすることで実行環境を軽量にたもてるという特徴を持っています。より快適に開発を行うために，必要なAPIだけを有効にします。

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

`guide-app`にサーブレットを実装します。

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
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
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

指定する言語によって，ことなる挨拶が返ってきていることが確認できるかと思います。

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
    - メッセージの1行目です。リクエストラインとも呼ばれます。
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

Servlet APIは，このリクエストおよびレスポンスのメッセージを受信し解析し，レスポンスを生成して送信するための多くの機能を提供しています。

#### URLとServletのマッピング

ServletやJSPなどのWebアプリケーションはWARファイルにパッケージされて，サーバーで実行されます。Webアプリケーションをサーバーに登録する際には，アプリケーションを呼び出すコンテキスト・ルート（Context Root）を指定します。

今回使用している`guide-app`は，サーバーの構成ファイル`server.xml`で以下のように登録されています。

``` xml
<webApplication contextRoot="/guide-app" location="guide-app.war" />
```

コンテキストルートはURLの構造において，ドメイン名に続く最初のパスセグメントとして利用されます。Servletを呼び出したときのURL，`http://localhost:9080/guide-app/hello`のパスセグメントは`/guide-app/hello`です。このうち`/guide-app`がWebアプリケーションを指定するコンテキスト・ルートとして使用されます。アプリケーション内のパスは`/hello`になります。

Webアプリケーション内のマッピングは，Javaコード内の`@WebServlet`アノテーションで指定されています。これにより，`/hello`のパスでこのServletが呼び出されるようにマップされます。

``` java
@WebServlet("/hello")
```

`@WebServlet`アノテーションに複数の情報を記述する場合は，マッピング情報は`urlPatterns`で記述します。

``` java
@WebServlet(urlPatterns = "/hello", loadOnStartup = 100)
```

マッピングには`/*`で終わる文字列を指定できます。その場合はパス一致として扱われ，そのディレクトリ以下の全ての呼出しにマッチし，指定されたサーブレットが呼び出されます。

``` java
@WebServlet("/admin/*")
```

また`*.`で始まる文字列で拡張子の指定も可能です。以下のような指定では，`.do`でおわる任意のパスセグメントとマッチし，サーブレットが呼び出されます。

``` java
@WebServlet("*.do")
```


#### GETリクエストとPOSTリクエスト

#### HttpServletRequest：リクエストの処理 

#### HttpServletResponse：レスポンスの処理

#### HttpSession：複数のリクエストにまたがるセッション

#### Servletフィルター

### JSPとは

