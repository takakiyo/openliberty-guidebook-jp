## RESTful Webサービス: Jakarta RESTful Web Services/JAX-RS

### RESTful Webサービスとは

REST（Representational State Transfer）やRESTfulなWebサービスとは，インターネット上でのデータ交換や相互作用を行うためのアーキテクチャスタイルの一つです。

RESTはリソース指向である，といわれています。

従来のWebサービスは，特定の操作や関数を呼び出すことに重点を置いており，呼び出すエンドポイント（WebサービスならばURL）は操作（たとえば「在庫照会」）を表現しており，操作対象のリソースはパラメーターなどの引数で与えられていました。それにたいしてRESTではエンドポイントのURLは操作対象のリソースを表現しており，HTTPのメソッド（POST/GET/PUT/DELETE）でCRUD操作（Create/Read/Update/Delete）を表現します。

また，通常のRESTはステートレスな通信として実装されます。

クライアントとサーバー間の複数回のリクエスト・レスポンスの組合わせは，それぞれで独立しており，以前のリクエストからの情報を保存しません。これにより，システムはより単純かつスケーラブルになります。HTTPの持つ冪等性など性質を活用することにより，HTTPのキャッシュの仕組みを活用したり，障害時のリトライなどもやりやすくなります。

RESTでは，データ型としてJSONが多用されます。

RESTでは，サーバーはJSONやXMLなどの様々なフォーマットでクライアントにリソースのデータを提供します。また，クライアントからもJSONなどでリソースの更新などを行います。RESTにおいてのデータの表現には，特にJSONが多用されます。RESTの主なクライアントであるブラウザ上では，JavaScriptでアプリケーションが実装されます。JSONは，特別なライブラリなどを利用しなくても，JavaScriptでそのまま解釈できるからです。

Java EE/Jakarta EEでは，RESTfulなWebサービスを構築するためのAPIをJAX-RS（Java API for RESTful Web Services）仕様で提供しています。また，JSONを扱うためのAPIを，JSON-P/JSON Processing仕様や，JSON-B/JSON Binding仕様で提供しています。

>[!NOTE]
>JAX-RSという名称は，Java EE仕様で使用されていた古い名前です。Jakarta EEでの正式な仕様名は「Jakarta RESTful Web Services」となっています。この仕様名に対するアクロニム（頭文字を取った略語）は提供されていません。というか，Jakarta RESTful Web Servicesの仕様書の中でも「JAX-RS application」という用語が多用されています。なので，このガイドの中でも，仕様名の略称として旧仕様名である「JAX-RS」を使用します。

JAX-RSでは，リソースクラスやメソッドにアノテーションを付けることで，Webサービスのエンドポイントを定義します。`@GET`，`@POST`，`@PUT`，`@DELETE`などのアノテーションによって，HTTPのメソッドに応じた処理を柔軟に記述できます。また，`@Path`や`@PathParam`などのアノテーションによって，URLのパスとして表現されたリソースを容易に識別できます。

また，JAX-RSでは，さまざまリソースの表現型に容易に対応できる仕組みが提供されています。


### JAX-RSを使ってみよう

この章では，JAX-RSについて実際に開発を行いながら学習します。

Liberty Starterで新しく作成したプロジェクト`guide-rest`でサンプルのコードを作成します。以下の条件でプロジェクトを作成しました。

- **Group** ：`com.demo`のまま
- **Artifact** ：`guide-rest`
- **Build Tool** ：Maven
- **Java SE Version** ：17
- **Java EE/Jakarta EE Version** ：10.0
- **MicroProfile Version** ：None

`src/main/liberty/config`フォルダーの`server.xml`を編集し，使用するFeatureだけを有効にします。

使用するAPIは，RESTful Web Services`restfulWS-3.1`とJSON Binding`jsonb-3.0`，章の後半で使用するのでCDI`cdi-4.0`も有効にします。JSON Processing`jsonp-2.1`も使用しますが，`restfulWS-3.1`の依存関係で自動的に有効になります。あと，HTTPS通信のために`transportSecurity-1.0`を有効にします。

``` xml
<!-- Enable features -->
<featureManager>
    <feature>restfulWS-3.1</feature>
    <feature>jsonb-3.0</feature>
    <feature>cdi-4.0</feature>
    <feature>transportSecurity-1.0</feature>
</featureManager>
```

Liberty Starterで作成したアプリケーションには，`@ApplicationPath("/api")`をもつ`RestApplication`というJAX-RSのアプリケーションクラスが既に定義されていますので，リソースクラスだけ作成します。

`src/main/java/com/demo/rest`フォルダーに，`SysPropResource.java`というファイルを新規作成し，以下の内容を保存します。

``` java
package com.demo.rest;

import java.util.Properties;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.PathParam;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;
import jakarta.ws.rs.core.Response.Status;

@Path("/system")
public class SysPropResource {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Properties getAllSysProps() {
        // 全てのシステムプロパティを結果として返す
        return System.getProperties();
    }
    @GET
    @Path("/{prop}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getSysProp(@PathParam("prop") String prop) {
        Properties result = new Properties();
        if (prop.endsWith("*")) {
            // Pathが*で終わっていたら前方一致で検索し，見つかったら結果に追加する
            String prefix = prop.substring(0, prop.length()-1);
            for (String name : System.getProperties().stringPropertyNames()) {
                if (name.startsWith(prefix)) {
                    result.setProperty(name, System.getProperty(name));
                }
            }
        } else {
            // それ以外は，完全一致で存在したら結果に追加する
            if (System.getProperties().containsKey(prop))
                result.setProperty(prop, System.getProperty(prop));
        }
        return (result.size() > 0)? 
            Response.ok(result).build() 
            : Response.status(Status.NOT_FOUND).build();
    }
}
```

「LIBERTY DASHBOARD」から「guide-rest」の右クリックから「Stert」でLibertyを起動し，正常に起動したら，ブラウザで[http://localhost:9080/guide-rest/api/system](http://localhost:9080/guide-rest/api/system)にアクセスしてみましょう。

JSON形式で，全てのシステムプロパティが表示されていることを確認できるかと思います。

>[!TIP]
>JSON形式のアプリケーションを開発している場合，結果をブラウザで確認していると，内容が整形されておらず，目でみて理解することが困難です。JSONの結果確認には，jqコマンドを入手して導入しておくと便利です。curlコマンドのレスポンスでJSON出力をだし，パイプ`|`でjqコマンドに送ると，人間が読みやすい形式に整形してくれます。
>
>``` terminal
>$ curl "http://localhost:9080/guide-rest/api/system" | jq
>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
>                                 Dload  Upload   Total   Spent    Left  Speed
>100  5240  100  5240    0     0   448k      0 --:--:-- --:--:-- --:--:--  852k
>{
>  "awt.toolkit": "sun.lwawt.macosx.LWCToolkit",
>  "java.specification.version": "17",
>  "com.ibm.ws.beta.edition": "false",
>  "sun.jnu.encoding": "UTF-8",
>  "wlp.install.dir": "/Users/takakiyo/src/guide-rest/target/liberty/wlp/",
>  "wlp.workarea.dir": "workarea/",
>    ...
>```

curlコマンドを使用して，コマンドプロンプトやターミナルからも結果を確認してみましょう。

```
curl "http://localhost:9080/guide-rest/api/system"
```

```
curl "http://localhost:9080/guide-rest/api/system/java.home"
```

```
curl "http://localhost:9080/guide-rest/api/system/wlp.*"
```

### JAX-RSアプリケーションの構造

Libertyでは，JAX-RSのアプリケーションは，Servletなどと同じWebアプリケーションとしてWARファイルにパッケージされます。Mavenのソースツリー`src/main/webapp`におかれたHTMLやJavaScript，CSSや画像ファイルなどは，WARファイルにそのまま組み込まれ，クライアントからのリクエストで提供されます。Webアプリケーションは，コンテキスト・ルートを指定して，Liberty上にデプロイされます。

``` xml
<webApplication contextRoot="/guide-rest" location="guide-rest.war" />
```

JAX-RSのアプリケーションには，Javaのクラスとして，アプリケーションクラスとリソースクラスが含まれます。必要に応じてプロバイダークラスが追加されます。

### アプリケーションクラス

アプリケーションクラスは，`jakarta.ws.rs.core.Application`を継承（extends）し，アノテーション`@ApplicationPath`が付与されたクラスです。通常は，アプリケーションに一つ存在します。

``` java
package com.demo.rest;

import jakarta.ws.rs.ApplicationPath;
import jakarta.ws.rs.core.Application;

@ApplicationPath("/api")
public class RestApplication extends Application {
}
```

Webアプリケーションのコンテキスト・ルートに，`@ApplicationPath`で指定されたパスを追加したものが，JAX-RSのサービスが提供されるURLになります。たとえば，サンプルのアプリケーションでは`/guide-rest`というコンテキストルートを持つWebアプリケーションに`/api`という`@ApplicationPath`が指定されていますので，`/guide-rest/api`で始まるパスがJAX-RSアプリケーションへのリクエストとして処理されます。

`@ApplicationPath("/")`を指定すると，Webアプリケーションへのリクエストは全てJAX-RSでハンドリングされるようになります。

Webアプリケーションに，二つ以上のアプリケーションクラスを含めることもできます。その場合は，Webアプリケーションに含まれるリソースクラス（やプロバイダークラス）がどちらのJAX-RSアプリケーションに含まれるのか，一覧を返すメソッドを実装する必要があります。

``` java
@ApplicationPath("/common")
public class CommonRestApplication extends Application {
    @Override
    public Set<Class<?>> getClasses() {
        Set<Class<?>> classes = new HashSet<>();
        // アプリケーションに属するリソースクラスを登録
        classes.add(CommonUserResource.class);
        classes.add(CommonItemResource.class);
        // プロバイダクラスの登録
        classes.add(MyCustomExceptionMapper.class);
        return classes;
    }
}
```

``` java
@ApplicationPath("/admin")
public class AdminRestApplication extends Application {
    @Override
    public Set<Class<?>> getClasses() {
        Set<Class<?>> classes = new HashSet<>();
        // アプリケーションに属するリソースクラスを登録
        classes.add(AdminAuditResource.class); 
        classes.add(AdminControlResource.class);
        return classes;
    }
}
```

Webアプリケーション中に，JAX-RSのアプリケーションクラスが一つだけの場合，メソッドのオーバーライドは必要ありません。Webアプリケーション中に含まれるリソースクラスなどは，全てそのJAX-RSアプリケーションに属するとみなされます。

### リソースクラス

リソースクラスとは，Webリソースを表し、クライアントからのHTTPリクエストを処理するクラスです。

`@Path`アノテーションがついたクラスで，HTTPのメソッドを表す`@GET`，`@POST`，`@PUT`，`@DELETE`などのアノテーションがついたpublicなメソッドを持つクラスとして実装します。

>[!NOTE]
>HTTPのメソッドを表すアノテーションとしては，他に`@PATCH`，`@HEAD`，`@OPTION`がありますが，これらはほとんど使用されません。

デフォルトでは，リソースクラスのインスタンスはリクエストごとに作成されます。インスタンス変数に保存した値などは，リクエストが完了すると失われます。

インスタンス変数には，CDIによりインジェクション，およびJAX-RSランタイムによる各種パラメーターのインジェクションが可能です。たとえば，以下のようなインスタンス変数を定義すれば，リクエストのUser-Agentヘッダーを参照することができます。

``` java
@HeaderParam("User-Agent")
private String agent;
```

ただ，これらについては，後述するリソースメソッドの引数でも同様のものが利用できますので，あまり利用する機会はないかと思います。

### リソースメソッド

リソースクラスに実装され，リクエストを処理するのがリソースメソッドです。

リソースメソッドは`@GET`などのHTTPメソッドを表すアノテーションが付与されたpublicなメソッドです。以下は，サンプルのアプリケーションの`SysPropResource`に定義されているリソースメソッドです。

``` java
@GET
@Produces(MediaType.APPLICATION_JSON)
public Properties getAllSysProps() {
    // 全てのシステムプロパティを結果として返す
    return System.getProperties();
}
```

`@Path`アノテーションが付与されていない場合は，リソースクラスの`@Path`に指定されたパスへのアクセスを処理します。このメソッドでは，

- アプリケーション`guide-rest`のコンテキスト・ルート`/guide-rest`
- アプリケーションクラス`RestApplication`の`@ApplicationPath("/api")`
- リソースクラス`SysPropResource`の`@Path("/system")`

を連結した`/guide-rest/api/system`や`/guide-rest/api/system/`へのリクエストを処理します（末尾に`/`をつけたパスも同一のものとして扱われます）。このパスへのGETリクエストが，このメソッドで処理されます。

メソッドがクライアントに返すデータ型，`Context-type`ヘッダで指定されるMIME型を`@Produces`で指定します。ここでは，定数`MediaType.APPLICATION_JSON`として定義された`application/json`を応答として返しています。

応答として正常応答を表すHTTPステータス200（とボディのない正常応答である204）のみを返す場合は，応答のボディに含めるオブジェクトを直接メソッドから返すことができます。

応答のボディをどのように作成すればいいのか，この場合だとJavaの`Properties`クラスのインスタンスをどのように`application/json`に変換するのかは，後述するプロバイダークラスで制御されます。この場合は，システムでデフォルトで用意されているプロバイダーが適切に変換してくれます。

>[!NOTE]
>このオブジェクトとメディアタイプの組み合わせだと，JSON-BのMessageBodyWriterが選択されます。JSON-Bでは，Javaのコレクションクラスは，標準でJSONに変換が可能です。

サンプルのアプリケーションの`SysPropResource`に定義されている以下のメソッドは，`@Path`アノテーションのついたリソースメソッド（サブ・リソースメソッドと呼ばれることもあります）の例です。

``` java
@GET
@Path("/{prop}")
@Produces(MediaType.APPLICATION_JSON)
public Response getSysProp(@PathParam("prop") String prop) {
    Properties result = new Properties();
    if (prop.endsWith("*")) {
        // Pathが*で終わっていたら前方一致で検索し，見つかったら結果に追加する
        String prefix = prop.substring(0, prop.length()-1);
        for (String name : System.getProperties().stringPropertyNames()) {
            if (name.startsWith(prefix)) {
                result.setProperty(name, System.getProperty(name));
            }
        }
    } else {
        // それ以外は，完全一致で存在したら結果に追加する
        if (System.getProperties().containsKey(prop))
            result.setProperty(prop, System.getProperty(prop));
    }
    return (result.size() > 0)? 
        Response.ok(result).build() 
        : Response.status(Status.NOT_FOUND).build();
}
```

`@Path("/{prop}")`は，パラメーターによってテンプレート化されたURLパスの例になります。中括弧でくくられた`{名前}`の部分は，パラメーターとして名前で参照ができます。パラメーターは`/`を除くURLのパス部分の任意の文字列にマッチします。このメソッドの扱うパスは，コンテキスト・ルートや各種のパスを追加して，以下のようになります。

- `/guide-rest/api/system/{prop}`

たとえば，以下のようなパスのリクエストがあった場合，

- `/guide-rest/api/system/wlp.*`

`wlp.*`という文字列が`prop`の名前で参照できます。このパラメーターを，メソッドの引数に`@PathParam("prop")`として受け取っています。


#### リソースメソッドのアノテーションが付与された引数

メソッドの引数には，他に以下のようなアノテーションを使用して，リクエストの情報を受け取ることができます。

- `@PathParam("path")`
    - テンプレート化されたURLパスに含まれている`{path}`などのパラメーターに指定された値
- `@QueryParam("key")`
    - GETリクエストのQuery文字列（`?key=value`など）や，POSTリクエストの`application/x-www-form-urlencoded`形式のボディで送られたパラメーターの値
- `@CookieParam("name")`
    - 指定した名前のCookieに設定されている文字列
- `@HeaderParam("name")`
    - 指定した名前のヘッダーに設定されている文字列
- `@MatrixParam`
    - URLのパスにMatrix形式で埋めこまれたパラメーター

また，以下の型のメソッドの引数に`@Context`のアノテーションをつけて，該当するインスタンスを取得することも可能です。

- `jakarta.ws.rs.core.UriInfo`
    - リクエストのURIについての各種情報を参照できます
- `jakarta.ws.rs.core.HttpHeaders`
    - リクエストのヘッダーついての各種情報を参照できます
- `jakarta.ws.rs.core.Application`
    - 実行しているJAX-RSのアプリケーションクラスを参照できます
- `jakarta.ws.rs.core.Configuration`
    - 実行しているJAX-RSアプリケーションの各種設定

これらのインスタンスでえられる情報をテストするリソースクラスです。`src/main/java/com/demo/rest`フォルダーに，`ContextResource.java`というファイルを新規作成し，以下の内容を保存します。

``` java
package com.demo.rest;

import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.Application;
import jakarta.ws.rs.core.Configuration;
import jakarta.ws.rs.core.Context;
import jakarta.ws.rs.core.HttpHeaders;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.UriInfo;

@Path("/context")
public class ContextResource {
    @GET
    @Path("/uriInfo/{path}")
    @Produces(MediaType.APPLICATION_JSON)
    public Map<?, ?> getUriInfo(@Context UriInfo uriInfo) {
        HashMap<String, Object> result = new HashMap<>();
        result.put("Path", uriInfo.getPath());
        result.put("Path(not decoded)", uriInfo.getPath(false));
        result.put("Path Segments", uriInfo.getPathSegments());
        result.put("Path Segments(not decoded)", uriInfo.getPathSegments(false));
        result.put("Request URI", uriInfo.getRequestUri());
        result.put("Absolute Path", uriInfo.getAbsolutePath());
        result.put("Base URI", uriInfo.getBaseUri());
        result.put("Path Parameters", uriInfo.getPathParameters());
        result.put("Path Parameters(not decoded)", uriInfo.getPathParameters(false));
        result.put("Query Parameters", uriInfo.getQueryParameters());
        result.put("Query Parameters(not decoded)", uriInfo.getQueryParameters(false));
        result.put("Matched URIs", uriInfo.getMatchedURIs());
        result.put("Matched URIs(not decoded)", uriInfo.getMatchedURIs(false));
        result.put("Matched Resources", uriInfo.getMatchedResources());
        return result;
    }

    @GET
    @Path("/headers")
    @Produces(MediaType.APPLICATION_JSON)
    public Map<?, ?> getHttpHeaders(@Context HttpHeaders headers) {
        HashMap<String, Object> result = new HashMap<>();
        result.put("Request Headers", headers.getRequestHeaders());
        result.put("Acceptable Media Types", headers.getAcceptableMediaTypes());
        result.put("Acceptable Languages", headers.getAcceptableLanguages());
        result.put("Media Type", headers.getMediaType());
        result.put("Language", headers.getLanguage());
        result.put("Cookies", headers.getCookies());
        result.put("Date", headers.getDate());
        result.put("Length", headers.getLength());
        return result;
    }

    @GET
    @Path("/application")
    @Produces(MediaType.APPLICATION_JSON)
    public Map<?, ?> getApplication(@Context Application application) {
        HashMap<String, Object> result = new HashMap<>();
        result.put("Application Class Name", application.getClass().getName());
        result.put("Classes", classNames(application.getClasses()));
        result.put("Properties", application.getProperties());
        return result;
    }

    @GET
    @Path("/configuration")
    @Produces(MediaType.APPLICATION_JSON)
    public Map<?, ?> getConfiguration(@Context Configuration config) {
        HashMap<String, Object> result = new HashMap<>();
        result.put("Runtime Type", config.getRuntimeType());
        result.put("Properties", config.getProperties());
        result.put("Classes", classNames(config.getClasses()));
        result.put("Instances", config.getInstances());
        return result;
    }

    private static Set<String> classNames(Set<Class<?>> classes) {
        Set<String> result = new HashSet<>();
        for (Class<?> c : classes) {
            result.add(c.getName());
        }
        return result;
    }
}
```

ブラウザで，以下のようなURLにアクセスしてテストしてみてください。

- http://localhost:9080/guide-rest/api/context/uriInfo/日本語?test=hoge&test=fuga&foo=bar
- http://localhost:9080/guide-rest/api/context/headers
- http://localhost:9080/guide-rest/api/context/application
- http://localhost:9080/guide-rest/api/context/configuration

Servlet APIが有効になっている環境では，以下の型のメソッドの引数にも`@Context`のアノテーションをつけて，インスタンスを取得できます。

- `jakarta.servlet.ServletConfig`
- `jakarta.servlet.ServletContext`
- `jakarta.servlet.http.HttpServletRequest`
- `jakarta.servlet.http.HttpServletResponse`

`HttpServletRequest`が取得できますので，`HttpSession`によるセッション管理も原理的には可能です。ただ，RESTの原則であるステートレスなサービスではなくなってしまうため，使用にあたってはそのデメリットも考慮しつつ，注意深くアプリケーションを設計してください。

リソースメソッドは，アノテーションのつかない引数を一つだけとることができます。その引数には，（もし存在すれば）リクエストのボディが代入されます。送られてきたボディが，どのように指定されたJavaの型に変換されるかは，後述のプロバイダークラスで制御されます。

#### リソースメソッドからのResponseによる応答

リソースメソッドの戻り値として，`Response`を返すと，HTTPステータス200以外の応答を返したり，各種ヘッダーやCookieを付与したレスポンスを生成することができます。

`Response`はビルダーパターンを採用しています。まず`Response.ResponseBuilder`を返すstaticメソッドの何れかを呼び出します。

|メソッド|説明|
|--------|----|
|`ok()`|200応答を返すResponseBuilder|
|`ok(Object entity)`|200応答を返すResponseBuilderで，ボディとなる`entity`を設定|
|`created(URI location)`|201応答を返すResponseBuilder|
|`accepted()`|202200応答を返すResponseBuilder|
|`accepted(Object entity)`|203応答を返すResponseBuilderで，ボディとなる`entity`を設定|
|`noContent()`|204応答を返すResponseBuilder|
|`seeOther(URI location)`|303応答を返すResponseBuilderで，locationヘッダに設定するURLを指定|
|`notModified()`|304応答を返すResponseBuilder|
|`temporaryRedirect(URI location)`|307応答を返すResponseBuilder|
|`status(int status)`|任意の応答を返すResponseBuilder|
|`status(int status, String reasonPhrase)`|応答を返すResponseBuilderで，レスポンスのステータスラインの文字列を指定|
|`status(Response.Status status)`|任意の応答を返すResponseBuilder|

`Response.ResponseBuilder`には，未設定であればボディを`entity(Object entity)`でセットしたり，`header(String name, Object value)`で任意のヘダーを設定したり，`language(String language)`でContent-languageヘッダーを設定したり，`cookie(NewCookie... cookies)`でCookieを設定したりない，レスポンスに対するさまざまな加工が行えます。

最後に`build()`メソッドを実行して，`Response`のオブジェクトを生成します。

[^1]:詳細は`Response.ResponseBuilder`のJavadocを参照してください。
[https://jakarta.ee/specifications/restful-ws/3.1/apidocs/jakarta.ws.rs/jakarta/ws/rs/core/response.responsebuilder](https://jakarta.ee/specifications/restful-ws/3.1/apidocs/jakarta.ws.rs/jakarta/ws/rs/core/response.responsebuilder)

### プロバイダークラス

プロバイダークラスは，JAX-RSランタイムの拡張ポイントであり，RESTfulサービスのさまざまな側面をカスタマイズするために使用されます。

プロバイダークラスには，以下の4種類があります。

- Javaのオブジェクトを変換して応答ストリームに書き込むMessageBodyWriter
- 要求ストリームを処理してJavaのオブジェクトに変換するMessageBodyReader
- 発生した例外を応答にマッピングするExceptionMapper
- コンテキスト情報を提供するContextResolver

`MessageBodyWriter`は，特定のJavaのクラスを，指定されたメディアタイプで応答に書き込むためのプロバイダーです。`MessageBodyWriter`は，開発者が独自に作成してJAX-RSアプリケーションに組み込むこともできますが，デフォルトの`MessageBodyWriter`もいくつか用意されています。

メディアタイプとして`MediaType.APPLICATION_JSON`が指定されたとき，リソースメソッドから以下の型が返されたときには，何もせずにそのまま（UTF-8のエンコーディングで）クライアントに出力する組み込みの`MessageBodyWriter`が使用されます。これらの型には，整形済みのJSONが格納されていることが期待されています。

- `String`，`byte[]`
- `java.io.InputStream`，`java.io.Reader`，`java.io.File`
- `javax.ws.rs.core.StreamingOutput`，`javax.activation.DataSouce`

これ以外のオブジェクトが返された場合，JSON-B/JSON Bindingが利用できる環境では，それを利用した`MessageBodyWriter`が選択されます。そのため，通常は，JSON-BでJavaのオブジェクトがどのようにJSONに変換されるかを理解し，必要に応じてJSON-Bの仕組みで変換をカスタマイズすることが必要となります。

また，MessageBodyReaderについても，以下の型については，そのまま（UTF-8のエンコーディングで）クライアントから送信されたボディを受け取ることができます。クライアントが`MediaType.APPLICATION_JSON`でリクエストのボディを送信してきて，リソースメソッドで下記以外のJavaのオブジェクト型を引数にボディを受け取った時には，同様にJSON-Bの`MessageBodyReader`が利用されます。

- `String`，`byte[]`
- `java.io.InputStream`，`java.io.Reader`，`java.io.File`
- `javax.activation.DataSouce`

>[!NOTE]
>XMLも同様です。`MediaType.APPLICATION_XML`でレスポンスを送信するとき，`application/xml`のMIMEタイプのリクエストのボディを受信するときも，上記の`String`や`byte[]`等の型を使用したときには，そのままで送受信されます。これらは整形済みのXMLが格納されることが想定されています。その他の型が指定されたときには，JAX-Bが利用できる環境では，それが利用されます。


### JSONとは

ここで，RESTfulなWebサービスでよく使用されるJSONというデータ形式について，あらためて説明します。

JSON（JavaScript Object Notation）は，JavaScriptにおけるオブジェクトの表記法をベースとした軽量なデータ記述言語です。単純で明快な構文をもち，あらゆるプログラミング言語で容易に扱うことができます。そのため，汎用のデータ交換フォーマットや構成ファイルとして多用されています。

``` json
 {
     "firstName": "John", "lastName": "Smith", "age": 25,
     "address" : {
         "streetAddress": "21 2nd Street",
         "city": "New York",
         "state": "NY",
         "postalCode": "10021"
     },
     "phoneNumber": [
         { "type": "home", "number": "212 555-1234" },
         { "type": "fax", "number": "646 555-4567" }
     ],
     "deleted": false
 }
```

JSONの構文は，配列とオブジェクトの2つのデータ構造，それらを含んだ7つのvalueタイプだけから構成されています。

配列は`[`と`]`でくくられ，`,`で区切って並べられたvalueの列です。

``` json
[ 2, 3, 5, 7, 11, 13, 17, 19, 23 ]
```

オブジェクトは`{`と`}`でくくられ，`,`で区切って並べられた`文字列:value`の列です。

``` json
{ "firstName": "John", "lastName": "Smith", "age": 25 }
```

valueタイプは以下の7つが定義されています。

- 配列
- オブジェクト
- 文字列（`"`でくくられた文字のあつまり）
- 数値（10進数のみで，16進数表記などはありません）
- `true`
- `false`
- `null`

自由フォーマットで，valueや`[`，`]`，`{`，`}`，`,`の前後には，自由に改行や空白，タブを入れられます。エンコーディングはUTF-8が指定されています。

あと仕様で決まっているのは数値や文字列の表現くらいで，これで仕様の全部，という非常に単純なデータ形式です。XMLの仕様書が，6章の本編仕様，8章の名前空間仕様，スキーマ関係が各5章で3部，という膨大な内容なのと比較すると，極めてシンプルです。

>[!NOTE]
>JSONの仕様には，コメントの記法すらありません。これでは構成ファイルなどに用いる際に非常に困るので，各種の拡張や記法が考案されています。JSONを拡張したJSON5では，通常のJavaScriptのように`/*`と`*/`でくくられたコメントや，`//`から改行までのコメントが許されています。VS Codeの`settings.json`でも，`//`から改行までのコメントが使用できます。

JavaにおいてJSONをあつかうAPIは，XMLをあつかうAPIと同じような形式で提供されています。以下にその関係を示します。

|APIの種類|JSON|XML|
|---|----|---|
|Processing<br>(Object Model)|Jakarta JSON Processing<br>`jakarta.json`|Java API for XML Processing (JAXP)<br>`javax.xml`|
|Processing<br>(Streaming Model)|Jakarta JSON Processing<br>`jakarta.json.stream`|Streaming API for XML (StAX)<br>`javax.xml.stream`|
|Binding|Jakarta JSON Binding<br>`jakarta.json.bind`|Jakarta XML Binding<br>`jakarta.xml.bind`|



### JSON ProcessingによるJSONの生成

JSON-Pは，JSONを直接読み書きするためのAPIです。

JSON-Pには，Object ModelのAPIとStreaming ModelのAPIの二種類があります。

Object Modeでは，提供されるオブジェクトを組み立てて，それをJSONとして出力することができます。また逆にJSONを読み込んで，Object Modeのオブジェクトのツリーを得ることができます。

### JSON BindingによるJSONの生成

### リソースクラスのライフサイクル

### @ContextによるServlet APIとの連携

