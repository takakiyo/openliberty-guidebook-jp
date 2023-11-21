## Hello, Open Liberty: 最初のプログラム

### Starterによるプロジェクトの作成

Libertyのアプリケーション開発は，基本的にLibertyのプラグインを組み込んだMavenプロジェクトでおこないます。

開発のためのMavenプロジェクトの作成は，OpenLiberty.ioにあるStarterを利用するのが便利です。「Get started with Open Liberty」（[https://openliberty.io/start/](https://openliberty.io/start/)）をひらきます。

![Create a starter application](../images/starter.png)

「Group」と「Artifact」にプロジェクトの情報を入力します。このガイドブックの例では，「Group」はデフォルトの`com.demo`のまま，「Artifact」は`guide-app`を使用します。

「Build Tool」はMavenを使用しますが，Gradleを選択することもできます。MavenではなくGradleの方が使い慣れている場合は，こちらを選択するとGradleのプロジェクトが作成できます。このガイドブックでは，Mavenを利用します。

つづいてアプリケーションで使用する各種仕様のバージョンを指定できます。このガイドブックでは，「Java SE」が17，「Java EE/Jakarta EE」が10，「MicroProfile」は使用しないのでNoneを選択します。

「Generate Project」をおして，プロジェクトの雛形ファイルが格納されたZIPファイルをダウンロードします。

ダウンロードしたZIPファイルを適当なフォルダーに展開します。ここでは，自身のホームディレクトリに`src`というフォルダーを作成し，その中の`guide-app`にファイルを展開しています

![展開したプロジェクト](../images/firststep1.png)

### VS Codeによるプロジェクトの確認

このフォルダーをVS Codeで開きます。「ファイル」「フォルダーを開く...」で上記のファイルを選択して開きます。

最初に開いたときには，フォルダーのファイルを信頼するかのダイアログボックスが表示されます。信頼できない場所から入手したプログラムのコードは，実行するとPCに害を与える可能性があるために警告が出ています。ここでは「はい、作成者を信頼します」をクリックします。

![制限モードの確認](../images/firststep2.png)

左のアクティビティ・バーで「エクスプローラー」をナビゲートしてプロジェクト内のファイルを確認します。

{% note %}

`pom.xml`を開いたときに，追加で「Dependency Analytics」拡張を導入するかたずねる通知がでることがあります。

システムにMavenが導入済みであれば，プロジェクトを扱う上で有益な拡張なので「Install」をおして導入しておきましょう。時間がたって通知が消えてしまったら，右下の🔔のボタンをクリックすると再度参照できます。

![Dependency Analytics導入の確認](../images/firststep3.png)

Mavenが導入されていなければDependency Analyticsは正常に動作しませんので入れなくてもかまいません。

{% endnote %}

「エクスプローラー」のカラムを開くと，以下のようなフォルダー構成がみえます。

![Libertyプロジェクトのファイル構成](../images/firststep5.png)

簡単にプロジェクトの内容を説明します。

- `.mvn`：Maven Wrapperの実行に必要なファイルが格納されています
- `src/main`：プロジェクトのソースで，実際のアプリケーションのソース
    - `java`：Javaのソースコード
    - `liberty`：Libertyの構成ファイル
- `target`：ビルドによって生成されたファイルが置かれるフォルダー
- `.docerignore`：Dockerイメージをビルドする際に，特定のファイルやディレクトリを無視するよう指示するファイル
- `.gitignore`：Gitに対して特定のファイルやディレクトリを追跡しないように指示するファイル
- `Dockerfile`：Dockerイメージをビルドするための構成ファイル
- `mvnw`：Maven Wrapperの実行ファイル
- `mvnw.cmd`：Maven Wrapperの実行ファイル（Windows用）
- `pom.xml`：Mavenプロジェクトの構成ファイル
- `README.txt`：プロジェクトの説明ファイル

Maven Wrapperは，最小限のファイルだけをプロジェクトに含み，Maven本体に必要なファイルをMavenセントラルレポジトリからダウンロードして実行するためのコマンドです。`pom.xml`をみて，必要なバージョンのMavenが自動的に選択されます。

プロジェクトの構成ファイル`pom.xml`をみるとLibertyのプラグイン`liberty-maven-plugin`が組み込まれていることが分かります。

``` xml
<build>
    <finalName>guide-app</finalName>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
            </plugin>
            <plugin>
                <groupId>io.openliberty.tools</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>3.9</version>
            </plugin>
        </plugins>
    </pluginManagement>
    <plugins>
        <plugin>
            <groupId>io.openliberty.tools</groupId>
            <artifactId>liberty-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

`<pluginManagement>`でプラグインの定義を行い，その下の`<plugins>`で実際のプラグインの組み込みを行っています。プラグインに追加の構成を行う場合には，プラグインの定義をしている`<pluginManagement>`の方に記述します。

``` xml
<build>
    <finalName>guide-app</finalName>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
            </plugin>
            <plugin>
                <groupId>io.openliberty.tools</groupId>
                <artifactId>liberty-maven-plugin</artifactId>
                <version>3.9</version>
                <configuration>
                    <!-- 追加の構成はここに記述する -->
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
    <plugins>
        <plugin>
            <groupId>io.openliberty.tools</groupId>
            <artifactId>liberty-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### Open Libertyを実行してみる

左のアクティビティ・バーで「エクスプローラー」を選んだ画面で，画面右下に「LIBERTY DASHBOARD」というカラムが追加されています。このカラムの＞をクリックして∨の状態にすると「guide-app」というアプリケーションが認識されています。

この「guide-app」で右クリックして「Start」を選びます。

![LIBERTY DASHBOARDでStartを選択](../images/firststep6.png)

MavenがWrapper経由で起動されてビルドが行われます。最初に実行したときには，大量のファイルがセントラルレポジトリーからダウンロードされます（2回目以降はキャッシュした内容を使用するので，すぐに開発が開始できます）。

Libertyのサーバーが構成されて起動すると，初回はファイアウォールの警告が出ることがあります。「アクセスを許可する」を選択してください。

![ファイアウォールの警告](../images/firststep7.png)


{% note %}

ターミナルが文字化けしたら


{% endnote %}

