## RESTful Webサービス: Jakarta RESTful Web Services/JAX-RS

### RESTful Webサービスとは



JAX-RSという名称は，Java EE仕様で使用されていた古い名前です。Jakartaでの正式な仕様名は「Jakarta RESTful Web Services」となっています。が，この仕様名に対するアクロニム（頭文字を取った略語）は提供されていません。というか，Jakarta RESTful Web Servicesの仕様書の中でも「JAX-RS application」という用語が多用されています。なので，このガイドの中でも，仕様名の略称として旧仕様名である「JAX-RS」を使用します。

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
    @Path("/")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getAllSysProps() {
        // 全てのシステムプロパティを結果として返す
        return Response.ok(System.getProperties()).build();
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

### アプリケーションクラス

### リソースクラス

### リソースメソッド

### MessageBodyWriter

### JSON ProcessingによるJSONの生成

### JSON BindingによるJSONの生成

### リソースクラスのライフサイクル

### @ContextによるServlet APIとの連携

