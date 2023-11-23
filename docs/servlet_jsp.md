## Java EEの基本: ServletとJSP

### Servletとは

Servletは，Java EE/Jakarta EE仕様の一部として提供されるAPIで，主にWebアプリケーションのバックエンドとして利用されます。クライアント（通常はWebブラウザ）からのリクエストに応じて動的なコンテンツを生成し，レスポンスとして返す役割を果たします。

ServletはHTTPリクエストとレスポンスを直接扱うことができる機能を提供しています。昨今では，この階層を意識することなく，Webアプリケーションを簡便に作成できる多くのフレームワークが提供されています。ですが，それらのフレームワークの中にも，内部ではServlet APIを使用しているものが多くあります（Spring Boot/Spring MVCなどが代表例です）。

実際の業務アプリケーションの構築にあたっては，フレームワークに隠蔽されて，Servletの機能をつかう機会は少なくなってるかもしれません。それでも，初学者がHTTP通信を利用したWebアプリケーションでは内部でなにがおこなわれているかを学習するにあたっては，現在でも有用な手段でしょう（またフレームワークでトラブルがあった場合も，内部で使用されているServletの知識があると，解決に役立つことが多々あります）。

``` java
package com.demo;

import java.io.IOException;
import java.io.PrintWriter;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
        // リクエストの指定するロケールを取得
        String language = req.getLocale().getLanguage();
        resp.setHeader("Vary", "Accept-Language");
        resp.setHeader("Content-Language", language);
        // レスポンスをHTMLに指定してPrintWriterを取得
        resp.setContentType("text/html; charset=UTF-8");
        PrintWriter writer = resp.getWriter();
        // レスポンスを出力
        writer.println("<!DOCTYPE html>");
        writer.println("<html>");
        writer.println("<head><title>Hello Servlet</title></head>");
        writer.println("<body>" + switch(language) {
            case "fr" -> "Bonjour, ";
            case "ja" -> "こんにちは、";
            case "zh" -> "你好，";
            default -> "Hello, ";
        } + "Open Liberty!</body>");
        writer.println("</html>");
    }
}

```

#### HTTPリクエストとレスポンス

#### URLとServletのマッピング

#### GETリクエストとPOSTリクエスト

#### HttpServletRequest：リクエストの処理 

#### HttpServletResponse：レスポンスの処理

#### HttpSession：複数のリクエストにまたがるセッション

#### Servletフィルター

### JSPとは

