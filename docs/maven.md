## Open LibertyのMavenプロジェクト構成

Libertyで実行するアプリケーションは，MavenやGradleを利用してビルドすることが推奨されています。この章では，Libertyと連携するMavenのプロジェクトについて解説します。

### Mavenとは

Mavenは，広く使われているOSSのJavaのビルドツールです。

Apache License 2.0で公開されており，誰もが無料で使用，配布することが可能です。Apache Licenseはオープンソースコミュニティにおいて広く採用されているライセンスで，ユーザーに対して広範な自由と柔軟性を提供しており，商用プロジェクトでの使用も可能です。

Mavenは標準化されたビルドライフサイクルを提供し，一貫したプロジェクト構造を促進します。Mavenのプロジェクト構造はデファクトスタンダードのようになっていて，最近はEclipse IDEも標準のプロジェクト形式として採用しています。

「Hello, Open Liberty」でも簡単に説明しましたが，あらためてMavenのプロジェクトの全体像を解説します。

- `src`：ソースファイルが置かれるディレクトリ
    - `main`：アプリケーションのソース。
        - `java`：アプリケーションのJavaソースコードが格納されます。
        - `resources`：リソース（プロパティファイル、設定ファイルなど）が格納されます。`*.java`をコンパイルした`*.class`ファイルと同じディレクトリにコピーされます。
        - `webapp`：WARファイル（Webアプリケーション）をビルドする場合に用意します。WARファイルにパッケージされるWebコンテンツや構成が格納されます。
    - `test`：テスト実行時にのみ利用されるソース。最終的な成果物にはパッケージされません。
        - `java`：テスト用のJavaソースコードが格納されます。
        - `resources`：テスト用のリソースが格納されます。
- `target`：ビルドによって生成されたファイルがおかれるディレクトリ。中間ファイルや，最終的な成果物がおかれる。
- `pom.xml`：Mavenのプロジェクト構成ファイル。ビルドに必要な情報を記述する。

Mavenは，依存関係を自動的に解決する機能をもっています。必要な依存関係を構成しておけば，プロジェクトのビルドに必要なライブラリ（依存関係）を自動的にダウンロードして管理します。Mavenが利用するセントラルリポジトリには，多くのOSSや商用製品のモジュールが，ビルド用に登録されています。

また，Mavenはプラグインベースのアーキテクチャをとっています。様々な追加のタスクを実行するためのプラグインを利用することができ，拡張性を提供しています。プラグイン自身も，大部分がセントラルリポジトリに登録されており，構成ファイルに記述するだけで使用できるようになっています。

> [!NOTE]
>Gradleは，Mavenにならんで人気のあるビルドツールです。
>
>Gradleも，Mavenと同様のプロジェクト構造を持ち，また同じようにセントラルリポジトリからのダウンロードで依存関係を自動的に解決してくれます。
>
>大きな違いとして，構成ファイルの記述形式があります。Mavenは，拡張子からも分かるように，XMLファイルでビルドの構成を記述します。一方でGradleは，GroovyまたはKotlinベースのDSL（ドメイン固有言語）を使用します。Gradleの構成はより表現力があり，柔軟であるといわれています。
>
>また，インクリメンタルビルドとキャッシュを標準でサポートしており，ビルド時間を大幅に短縮することができます。

### LibertyのMaven Plugin

Mavenにたいしては，Libertyのプラグインが提供されており，無償で利用できます。

プラグイン，`liberty-maven-plugin`を組み込むことによって，アプリケーションのソースだけでなく，それを実行するLibertyの構成ファイルもソースの一部として管理できるようになります。また，サーバー環境のダウンロード・導入・構成がビルドの過程で自動的に実行されます。さらに，その環境，Libertyのランタイムと構成とアプリケーションをまとめて導入可能なアーカイブファイルにパッケージすることができます。

MavenのLibertyプラグインを活用することにより，アプリケーションサーバーは事前に実働環境に導入しておくものではなくなります。従来のアプリケーションサーバーで行っていた以下の手順が，

- アプリケーションサーバーの導入
- Fixpackの適用
- サーバーの作成，構成
- ビルドしたアプリケーションのデプロイ

以下の手順に変わります。

- ビルドしたアーカイブファイルの展開

サーバーの構成変更やバージョン更新は，すべてソースの修正となり，環境の再作成にかわります。

Libertyのプラグインを組み込んだ最小限のMavenのプロジェクトファイル，`pom.xml`の例は以下のようになります。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>maven-sample</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <properties>
        <maven.compiler.target>17</maven.compiler.target>
        <maven.compiler.source>17</maven.compiler.source>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.eclipse.microprofile</groupId>
            <artifactId>microprofile</artifactId>
            <version>5.0</version>
            <type>pom</type>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>maven-sample</finalName>
        <plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
            <plugin>
                <groupId>io.openliberty.tools</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>3.10</version>
            </plugin>
        </plugins>
    </build>
</project>
```
ここでは，Java SE 17とMicroProfile 5.0に準拠したアプリケーションを実行するプロジェクトになっています。

準拠するJava SEのバージョンを`<properties>`で設定します。

``` xml
<properties>
    <maven.compiler.target>17</maven.compiler.target>
    <maven.compiler.source>17</maven.compiler.source>
</properties>
```

アプリケーションが利用するAPIを，`<dependencies>`で記述します。ここでは，MicroProfile 5.0を依存関係に指定しています。

``` xml
<dependency>
    <groupId>org.eclipse.microprofile</groupId>
    <artifactId>microprofile</artifactId>
    <version>5.0</version>
    <type>pom</type>
    <scope>provided</scope>
</dependency>
```

Jakarta EE 10を使用する場合は，以下のような依存関係を指定します。

``` xml
<dependency>
    <groupId>jakarta.platform</groupId>
    <artifactId>jakarta.jakartaee-api</artifactId>
    <version>10.0.0</version>
    <scope>provided</scope>
</dependency>
```

Java EE 8を使用する場合は，以下のような依存関係を指定します。

``` xml
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
</dependency>
```

Libertyのプラグインを`<build>`の`<plugins>`に`<plugin>`として組み込みます。ここでは直接組み込んでいますが，`<pluginManagement>`経由で組み込んだり，`<profile>`として組み込んだりすることもできます。

``` xml
<plugin>
    <groupId>io.openliberty.tools</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>3.10</version>
</plugin>
```

### Libertyのプラグインのプロジェクト構造

通常のMavenプロジェクトに対して，`src/main/liberty/config`ディレクトリが追加されます。

こちらには，`server.xml`ファイルを置いておく必要があります。オプションとして`server.env`，`jvm.options`，`bootstrap.properties`などの構成ファイル，`configDropins`，`variables`などの追加ディレクトリもおくことができます。これらのファイルやディレクトリは，すべて作成されるLibertyの`${server.config.dir}`に自動的にコピーされます。

このディレクトリにおかれたものは，最終的な成果物であるLiberty環境をパッケージしたアーカイブにも格納されます。

### Libertyのプラグインで使用可能になるゴール

`liberty-maven-plugin`を組み込むことで使用可能になる主なゴールについて説明します。

以下のゴールは，Libertyのサーバー環境を作成します。

|ゴール|説明|
|------|----|
|`liberty:install-server`|導入に必要なアーカイブをセントラルリポジトリからダウンロードし，指定された場所に展開する|
|`liberty:create`|サーバーを作成する。事前に暗黙的に`liberty:install-server`を呼び出す。|
|`liberty:install-feature`|サーバーに追加導入が必要なFeatureがあれば，導入する。|
|`liberty:deploy`|アプリケーションと構成をサーバーにデプロイする。事前に`compile`および`liberty:create`を実行済みである必要がある。|

以下のゴールは，開発を行うためにLiberty環境を構築し，起動します。

|ゴール|説明|
|------|----|
|`liberty:dev`|Libertyを開発者モードで起動する。事前に暗黙的に`liberty:create`，`liberty:install-feature`，`compile`および`liberty:deploy`を呼び出す。ソースコードに変更があれば，自動的に検知して`compile`と`liberty:deploy`を再実行する。`Ctrl`+`C`を押すか，`liberty:stop`を実行するまで，Mavenのビルドは終了しない。|
|`liberty:devc`|Libertyをコンテナ上で起動する。プロジェクトは`Dockerfile`もしくは`Containerfile`を含んでいる必要がある。また実行する環境にdockerコマンドもしくはpodmanコマンドが導入されている必要がある。|

以下のゴールは，`server`コマンドの同名のアクションを呼び出します。

|ゴール|説明|
|------|----|
|`liberty:run`|サーバーをフォアグランドで起動する。事前に暗黙的に`liberty:create`，`liberty:install-feature`，`compile`および`liberty:deploy`を呼び出す。`Ctrl`+`C`を押すか，`liberty:stop`を実行するまで，Mavenのビルドは終了しない。|
|`liberty:start`|サーバーをバックグランドで起動する。事前に暗黙的に`liberty:create`，`liberty:install-feature`，`compile`および`liberty:deploy`を呼び出す。Libertyが起動すると，Mavenのビルドは終了する。|
|`liberty:stop`|稼働しているサーバーを停止する。事前に暗黙的に`liberty:install-server`を呼び出す。|
|`liberty:status`|サーバーが稼働しているか確認する。事前に暗黙的に`liberty:install-server`を呼び出す。|
|`liberty:package`|サーバー環境をパッケージする。事前に暗黙的に`liberty:install-server`を呼び出す。|
|`liberty:dump`|サーバー環境の問題判別のための資料を作成する。|

開発を行う際には，`liberty:dev`ゴールを利用して，サーバーでのアプリケーションのテスト実行を行います。

### Libertyのプラグインの構成

`liberty-maven-plugin`の構成を行うには，`<plugin>`の子要素`<configuration>`の下に，各種の設定を記述します。

``` xml
<plugin>
    <groupId>io.openliberty.tools</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>3.10</version>
    <configuration>
        <!-- ここに各種設定を記述する -->
        <serverName>guideServer</serverName>
    </configuration>
</plugin>
``` 

`<serverName>`は，サーバー名をデフォルトの`defaultServer`から変更するときに使用します。

#### ビルドに使用するLibertyの種類とバージョンの構成

`<runtimeArtifact>`は，Liberty環境を構築するためのパッケージ，およびバージョンを構成します。

Open Libertyの特定のバージョン，たとえば23.0.0.9を使用するには，`<configuration>`の下に以下のように記述します。

``` xml
<runtimeArtifact>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-kernel</artifactId>
    <version>23.0.0.9</version>
</runtimeArtifact>
<skipInstallFeature>true</skipInstallFeature>
```

WebSphere Libertyを使用する場合は`<configuration>`の下に以下のように記述します。

``` xml
<runtimeArtifact>
    <groupId>com.ibm.websphere.appserver.runtime</groupId>
    <artifactId>wlp-kernel</artifactId>
    <version>23.0.0.9</version>
</runtimeArtifact>
<skipInstallFeature>true</skipInstallFeature>
```

> [!NOTE]
>Open Libertyのランタイムとしては，全てのFeatureを含んだ`openliberty-runtime`と，カーネルだけでFeatureを全く含まない`openliberty-kernel`の2種類が提供されています。従来は，これらにはトレードオフがありました。
>
>`openliberty-kernel`は，ダウンロードサイズも最小限で，`liberty:install-server`ゴールも素早く完了するが，`liberty:dev`ゴールで毎回実行される`liberty:install-feature`ゴールに時間がかかります。
>
>`openliberty-runtime`は，ダウンロードされるサイズが大きく，`liberty:install-server`ゴールに時間がかかるが，`liberty:dev`ゴールで毎回実行される`liberty:install-feature`ゴールは素早く完了します。
>
>Libertyプラグインのバージョン3.9から，`liberty:dev`で必要なFeatureが導入済みであれば`liberty:install-feature`の実行をスキップする設定`<skipInstallFeature>`が追加されました。これを`true`に設定しておけば，ダウンロードするサイズを最小限にしながら，`liberty:dev`の繰り返しが素早く行えるようになります。

#### Liberty環境へのファイルの追加

Libertyの環境に，依存関係で定義されたJARファイルを追加で導入したい場合は，`<copyDependencies>`を使用します。たとえば，JDBC Driverの定義で使用する，ドライバーの実装ファイルが入ったJARファイルなどです。

たとえば，Db2にアクセスするためのドライバーが，`<dependencies>`で以下のように定義されているとします。

``` xml
<dependency>
    <groupId>com.ibm.db2</groupId>
    <artifactId>jcc</artifactId>
    <version>11.5.8.0</version>
</dependency>
```

この依存関係で提供されているJARファイルをLiberty環境にコピーするには，`<configuration>`の下に以下のように記述します。`<location>`は，`${server.config.dir}`からの相対パスになります。`<copyDependencies>`の下には，`<dependencyGroup>`を複数書くこともできます。

``` xml
<copyDependencies>
    <dependencyGroup>
        <location>resources/db2</location>
        <dependency>
            <groupId>com.ibm.db2</groupId>
            <artifactId>jcc</artifactId>
        </dependency>
    </dependencyGroup>
</copyDependencies>
```

これをつかってJDBC Driverを定義するには，`server.xml`に以下のように記述します。

``` xml
<jdbcDriver id="db2jdbcDriver">
    <library>
        <fileset dir="${server.config.dir}/resources/db2" includes="*.jar" />
    </library>
</jdbcDriver>
```

#### pom.xmlによるjvm.optionsの設定

Libertyを起動するJVMの引数を指定する`jvm.options`は，ファイルを`src/main/liberty/config`に置いておくこともできますが，プロジェクト構成ファイル`pom.xml`の`<properties>`からも生成することができます。`<liberty.jvm.〜>`の名前を持つプロパティが存在すれば，それらに設定された値をマージして，動的に`jvm.options`が作成して利用されます。

JVMの設定は，開発者が`liberty:dev`でテストをしている段階と，実働環境で実行される段階では，多くの設定が異なるでしょう。Windows環境でJava 21を利用している場合など，

``` xml
<profiles>
    <profile>
        <id>for-dev</id><!-- 開発環境のためのためのJVM構成 -->
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <liberty.jvm.maxHeap>-Xmx512m</liberty.jvm.maxHeap>
            <!-- WindowsでJava 21を使用している時の文字化け対策 -->
            <liberty.jvm.stdout>-Dstdout.encoding=UTF-8</liberty.jvm.stdout>
            <liberty.jvm.stderr>-Dstderr.encoding=UTF-8</liberty.jvm.stderr>
        </properties>
    </profile>
    <profile>
        <id>package</id><!-- デプロイ用のパッケージ作成のためのJVM構成 -->
        <properties>
            <liberty.jvm.maxHeap>-Xmx4096m</liberty.jvm.maxHeap>
            <liberty.jvm.verbosegc>-verbose:gc</liberty.jvm.verbosegc>
            <liberty.jvm.verbosegclog>-Xverbosegclog:logs/verbosegc.%Y%m%d.%H%M%S.log</liberty.jvm.verbosegclog>
        </properties>
    </profile>
</profiles>
```

Mavenで利用するローカルのテスト環境の`etc`ディレクトリにファイルを置きたいときには，たとえば`src/main/liberty/etc`ディレクトリを作成し，そこに`server.env`ファイルなどを置き，`<configuration>`の下に以下のように記述します。

``` xml
<libertySettingsFolder>src/main/liberty/etc</libertySettingsFolder>
```

`etc`ディレクトリに置かれたファイルは，必要に応じてサーバーの構成に組み込まれますが，サーバーをパッケージしたときには，その中には含まれません。

### Libertyの導入可能パッケージ作成のための構成



