## Hello, Open Liberty: 最初のプログラム

### Starterによるプロジェクトの作成

Libertyのアプリケーション開発は，基本的にLibertyのプラグインを組み込んだMavenプロジェクトでおこないます。

開発のためのMavenプロジェクトの作成は，OpenLiberty.ioにあるStarterを利用するのが便利です。「Get started with Open Liberty」（[https://openliberty.io/start/](https://openliberty.io/start/)）をひらきます。

![Create a starter application](../images/starter.png)

「Group」と「Artifact」にプロジェクトの情報を入力します。このガイドブックの例では，「Group」はデフォルトの`com.demo`のまま，「Artifact」は`guide-app`を使用します。

「Build Tool」はMavenを使用しますが，Gradleを選択することもできます。MavenではなくGradleの方が使い慣れている場合は，こちらを選択するとGradleのプロジェクトが作成できます。このガイドブックでは，Mavenを利用します。

つづいてアプリケーションで使用する各種仕様のバージョンを指定できます。このガイドブックでは，「Java SE」が17，「Java EE/Jakarta EE」が10，「MicroProfile」は使用しないのでNoneを選択します。

「Generate Project」をおして，プロジェクトの雛形ファイルが格納されたZIPファイルをダウンロードします。



