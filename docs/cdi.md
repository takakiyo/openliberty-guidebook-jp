## Context and Dependency Injection (CDI)

### CDIとは

CDIとは，JavaでDI：Dependency Injection（依存性注入）を実現する仕様です。

DIは，ソフトウェアエンジニアリングにおける設計パターンの一つです。このパターンの目的は、コードの再利用性、テストの容易さ、およびモジュール間の結合度の低減を実現することにあります。

CDIでは，型や限定子の情報をもとに，`@Inject`のついたインスタンス変数や初期化メソッドの引数に，CDIコンテナがオブジェクトを取得し，注入してくれます。

インスタンスを作成あるいは取得し，代入するコードを自身で記述すると，どうしても対象の詳細な情報が必要となり，それにより依存関係が生じてしまいます。DIでは，インスタンスの取得と代入をDIコンテナに委譲することにより，依存関係を少なくし，結合度をさげることができます。

``` java
@Inject
@Named("request")
private BaseBean request; // 型BaseBeanと限定子@Named("request")の情報を元に注入
```

CDIの大きな役割は，注入するオブジェクトのライフサイクルを管理し，コンテキストに応じた適切なインスタンスを呼び出してくれることです。

これを実現するため，実際の注入の際には，変数に直接オブジェクトが代入されるのではなく，Proxyクラスが作成されて代入されます。

![変数にはProxyが代入される](../images/cdi1.png)

メソッド呼び出しは，Proxyが中継して実際のオブジェクトに転送されます。

![メソッド呼び出しはProxyに中継される](../images/cdi2.png)

これにより，同じ変数経由でも異なるコンテキストであれば別のインスタンスを呼び出したり，異なる変数経由でも同じコンテキストであれば同一のインスタンスを呼び出したり，という機能が実現できています。

![Proxyがコンテキストに応じた適切なインスタンスを選択する](../images/cdi3.png)

### CDIを使ってみよう

Liberty Starterで新しく作成したプロジェクト`guide-cdi`でサンプルのコードを作成します。以下の条件でプロジェクトを作成しました。

- **Group** ：`com.demo`のまま
- **Artifact** ：`guide-cdi`
- **Build Tool** ：Maven
- **Java SE Version** ：17
- **Java EE/Jakarta EE Version** ：10.0
- **MicroProfile Version** ：None

使用するAPIは，CDIとServlet（およびHTTPSアクセス）だけですので，`src/main/liberty/config`フォルダーの`server.xml`を編集し，以下の３つのFeatureだけを有効にします。

``` xml
<!-- Enable features -->
<featureManager>
    <feature>cdi-4.0</feature>
    <feature>servlet-6.0</feature>
    <feature>transportSecurity-1.0</feature>
</featureManager>
```

CDIを使用する場合は，`beans.xml`というファイルをアプリケーションに同梱することが推奨されています。`src/main`フォルダーに`webapp`フォルダーを，そこに`WEB-INF`フォルダーを新規作成し，`src/main/webapp/WEB-INF`フォルダーに`beans.xml`というファイルを新規作成します。ファイルには，以下の内容をコピーします。

Beanとしてインジェクションされる対象を探す`bean-discovery-mode`として，推奨される`annotated`を設定しています。

``` xml
<beans xmlns="https://jakarta.ee/xml/ns/jakartaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="https://jakarta.ee/xml/ns/jakartaee https://jakarta.ee/xml/ns/jakartaee/beans_4_0.xsd"
        version="4.0" bean-discovery-mode="annotated">
</beans>
```

インジェクションされる側から使用するabstractクラスであるBaseBeanを作成します。`src/main/java/com/demo`フォルダーに`bean`フォルダーを新規作成し，そこに`BaseBean.java`というファイルを新規作成し，以下の内容をコピーします。

インスタンスが作成された時刻を覚えておいて，自身のクラス名とオブジェクトのハッシュ，作成時刻をStringで提供します。また，直列化可能（Serializable）になっています。

``` java
package com.demo.bean;

import java.io.Serializable;
import java.time.LocalDateTime;

public abstract class BaseBean implements Serializable {
    private LocalDateTime created;
    protected BaseBean() {
        created = LocalDateTime.now();
    }
    abstract String getName();
    public String getInfo() {
        return getName() + "(id=" + System.identityHashCode(this) + ") created at " + created;
    }
}
```

実際にインジェクションする3つのクラスを作成します。`src/main/java/com/demo/bean`フォルダーに`RequestScopedBean.java`，`SessionScopedBean.java`，`ApplicationScopedBean.java`というファイルを新規作成し，以下の内容をコピーします。

それぞれ，異なるスコープのBean定義アノテーションが付与され，`@Named`限定子で区別できるようになっています。

``` java
package com.demo.bean;

import jakarta.enterprise.context.RequestScoped;
import jakarta.inject.Named;

@RequestScoped
@Named("request")
public class RequestScopedBean extends BaseBean {
    @Override
    String getName() {
        return "RequestScopedBean";
    }
}
```

``` java
package com.demo.bean;

import jakarta.enterprise.context.SessionScoped;
import jakarta.inject.Named;

@SessionScoped
@Named("session")
public class SessionScopedBean extends BaseBean {
    @Override
    String getName() {
        return "SessionScopedBean";
    }
}
```

``` java
package com.demo.bean;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Named;

@ApplicationScoped
@Named("application")
public class ApplicationScopedBean extends BaseBean {
    @Override
    String getName() {
        return "ApplicationScopedBean";
    }
}
```

インジェクション対象となる2つのServletを定義します。一つ目のServletがリクエストを受け，RequestDispacherで二つ目のServletに処理をfowardし，二つ目のServletが実際の画面表示を行います。

一つ目のサーブレットです。`src/main/java/com/demo`フォルダーに`FiestServlet.java`というファイルを新規作成し，以下の内容をコピーします。

``` java
package com.demo;

import java.io.IOException;

import com.demo.bean.BaseBean;

import jakarta.inject.Inject;
import jakarta.inject.Named;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/first")
public class FiestServlet extends HttpServlet {
    @Inject
    @Named("request")
    private BaseBean request;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setAttribute("request.in.first", request.getInfo());
        req.getRequestDispatcher("/second").forward(req, resp);
    }
}
```

二つ目のサーブレットです。`src/main/java/com/demo`フォルダーに`FiestServlet.java`というファイルを新規作成し，以下の内容をコピーします。

``` java
package com.demo;

import java.io.IOException;
import java.io.PrintWriter;

import com.demo.bean.BaseBean;

import jakarta.inject.Inject;
import jakarta.inject.Named;
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/second")
public class SecoundServlet extends HttpServlet {
    @Inject
    @Named("request")
    private BaseBean request;

    @Inject
    @Named("session")
    private BaseBean session;

    @Inject
    @Named("application")
    private BaseBean application;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        writer.print(
            """
            <!DOCTYPE html>
            <html>
            <head><title>SecoundServlet</title></head>
            <body>
            <h1>Injection Test</h1>
            <ol>
            """);
        writer.println("<dt>request in FirstServlet<dd>" + req.getAttribute("request.in.first"));
        writer.println("<dt>request in SecondServlet<dd>" + request.getInfo());
        writer.println("<dt>session in SecondServlet<dd>" + session.getInfo());
        writer.println("<dt>application in SecondServlet<dd>" + application.getInfo());
        writer.print(
            """
            </ol>
            </body>
            </html>
            """);
    }
}
```


ブラウザで[http://localhost:9080/guide-cdi/first](http://localhost:9080/guide-cdi/first)にアクセスしてみましょう。

「request in FirstServlet」と「request in SecondServlet」に全く同じ内容が表示されていることがわかるかと思います。

この2つは，異なるクラスのそれぞれの変数にインジェクションされたものです。それでも，おなじリクエストを処理している間は，両者で全く同じインスタンスにアクセスできています。

`@Named("request")`の限定子によって，`@RequestScoped`のスコープを持つ`RequestScopedBean`のインスタンス（へのProxy）がインジェクションされているためです。

``` java
public class FiestServlet extends HttpServlet {
    @Inject
    @Named("request")
    private BaseBean request;
```

``` java
public class SecoundServlet extends HttpServlet {
    @Inject
    @Named("request")
    private BaseBean request;
```

また，この画面をリロードすると，そのたびに「request in FirstServlet」などに，新しく作成されたインスタンスが使用されていること分かります。

Servletのインスタンス変数は，通常はサーバーが起動している間，全てのユーザーについて有効なはずです。ですが，インジェクションされたProxyが，指定されたスコープに応じて毎回新しいインスタンスを作成し，アクセスできるようにしてくれています。

一方，リロードを繰り返しても「session in SecondServlet」に表示される内容は変わらず，オブジェクトは同じものが再使用されていることが分かります。同じクライアントからの一連のリクエストは，同一のセッションとみなされ，その間は同一のスコープとして扱われ，その間は同じオブジェクトが呼び出されています。

`@Named("session")`の限定子によって，`@SessionScoped`のスコープを持つ`SessionScopedBean`のインスタンスがインジェクションされているためです。

``` java
@Inject
@Named("session")
private BaseBean session;
```

ブラウザのメニューから「新しいInPrivateウィンドウ」や「新しいシークレットウィンドウ」などを開いて，新しいセッションで同じURLにアクセスしてみましょう。異なるセッションでは，「session in SecondServlet」に別の情報が表示され，異なるインスタンスが呼び出されていることが分かります。

別のセッションの画面でも，「application in SecondServlet」には同一の情報が表示されていることが確認できるでしょう。アプリケーション全体で1つのオブジェクトが共有されていることが分かります。

`@Named("application")`の限定子によって，`@ApplicationScoped`のスコープを持つ`ApplicationScopedBean`のインスタンスがインジェクションされているためです。

``` java
@Inject
@Named("application")
private BaseBean application;
```
