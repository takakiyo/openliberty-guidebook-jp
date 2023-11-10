## Open Libertyの構成ファイル 



### server.xmlの基本

Libertyの構成を行う際の中心となるファイルで，XML形式で記述します。server.xmlは`${server.config.dir}`ディレクトリにあるファイルが使用されます。

#### デフォルトベースの構成

server.xmlに記述する構成要素には，基本的に全てデフォルトの値が決まっています。ファイルにはデフォルトから変更するものだけを記述します。これにより非構成ファイルを常に簡潔に記述することができます。

このデフォルト値はバージョンが上がっても変更されないようになっていますので，記述をしなくても安全です。構成する内容を最小限に保つためにも，また環境に合わせてどのように構成を変えたかを明確に管理するためにも，利用するデフォルト値をあえて構成に記述する，というのは避けましょう。

デフォルトの値は，Web上のWebSphere Liberty，Open Libertyのオンラインマニュアルでも確認できますが，Liberty Developer Toolsなどの開発ツールでも参照することが可能です。

{% note %}

**Note:** server.xmlの構成項目やその説明，デフォルトの値が記載されたXSDファイル(XML Schema)を動的に生成することができます。Libertyの構成ファイルを操作するツールを自作する際などに，利用します。生成にはLibertyの`schemaGen`コマンドを利用します。

```
schemaGen /path/to/schema.xsd
```

以下のようにオプションでロケールを指定すると，日本語で説明が記述されたXSDファイルが生成されます。

```
schemaGen  --locale=ja_JP /path/to/schema.xsd
```

{% endnote %}


#### 変数の使用

`server.xml`およびそこから読み込まれるXMLファイルでは，`${変数名}`で変数の値を参照することができます。変数としては以下のものが利用可能です。下のものほど優先され，同じ変数名で定義された値を上書きできます。

- 構成ファイルの`<variable>`要素の`defaultValue`属性で定義された値
- 環境変数
- `bootstrap.properties`ファイルで定義された値
- Javaシステムプロパティ（`server.config.dir`などのLibertyのディレクトリをしめす変数もシステムプロパティとして定義されています）
- `${server.config.dir}/variables`ディレクトリ，もしくは環境変数`VARIABLE_SOURCE_DIRS`で指定されたディレクトリにおかれたファイルで定義された値
- 構成ファイルの`<variable>`要素の`value`属性で定義された値
- Libertyの起動コマンドラインで指定された値

環境変数を参照する際には，指定された名前の環境変数だけでなく，英数字以外をアンダースコア`_`に変換した文字列，さらに英字を大文字に変換した文字列も参照されます。つまり`${my.env.var1}`という変数を参照する場合，以下の環境変数が順に検索され，最初に見つかったものが利用されます。

- `my.env.var1`
- `my_env_var1`
- `MY_ENV_VAR1`

環境変数`VARIABLE_SOURCE_DIRS`は変数を定義するファイルを格納するディレクトリを指定します。複数のディレクトリを指定するときには，Windows環境では`;`で，そのほかの環境では`:`で区切って指定します。この環境変数が設定されていなかった場合には，デフォルトで`${server.config.dir}/variables`ディレクトリが使用されます。

これらのディレクトリにファイルを置いた場合，ファイル名が変数名として利用されます。値はファイルに記述された内容です。たとえば`httpPort`というファイルに`8080`という内容が保存されていた場合，`${httpPort}`という変数は`8080`になります。

ファイル名が`.properties`の拡張子を持っている場合には，一つのファイルに複数の変数定義を記述することができます。例えば以下のように記述します。

``` properties
httpPort=9080
httpsPort=9443
```

Libertyの起動時のコマンドラインでも`--`につづけて変数を指定することができます。たとえば`start`アクションでサーバーを起動する際に，以下のように変数を指定したとします。

```
server start myserver -- -httpPort=10080
```

この場合，`${httpPort}`という変数は`10080`になります。

独自の変数を使用する場合は，構成ファイル内に`defaultValue`属性でデフォルトの値を定義し，環境変数や起動パラメーターで値を上書きするようにして使用します。たとえばテスト環境と本番環境で接続するデータベースのIPアドレスが異なる場合，テスト環境のアドレスを以下のように指定しておきます。

``` xml
<variable name="db.server.address" defaultValue="192.168.1.101" />
<dataSource jndiName="jdbc/db2DataSource" id="db2DataSource">
    <jdbcDriver libraryRef="db2jdbc_jar" />
    <properties.db2.jcc databaseName="appdatabase" portNumber="8200"
        serverName="${db.server.address}" />
</dataSource>
```

本番環境で実行する際には，環境変数`DB_SERVER_ADDRESS`に`192.168.2.201`のようなアドレスを記入してLibertyを起動すれば，`defaultValue`属性に記述された内容を上書きすることができます。

数値が定義された変数は，`+`や`-`，`*`などを使用した計算が可能です。

``` xml
<variable name="one" value="1" />
<variable name="two" value="${one+1}" />
<variable name="three" value="${one+two}" />
<variable name="six" value="${two*three}" />
<variable name="five" value="${six-one}" />
```

#### 時刻の記述方法

時間を設定する属性（attribute）には，単位をつけて記載します。

- `d` 日
- `h` 時間
- `m` 分
- `s` 秒
- `ms` ミリ秒

単位は複数を併記することができます。たとえば`1m30s`は1分30秒，つまり90秒を意味します。

#### 構成ファイル中でのパスワードの記述

Libertyの構成ファイルにパスワードを記述する際には，いくつかの方法で難読化が可能です。デフォルトではXORによる難読化が利用され，秘密鍵を使ったAES暗号化もサポートされています。ただ，AESを使った暗号化でも完全な安全性は得られません。Liberty自身がパスワードを使用するために，必要な情報を組み合わせてパスワードを復元する必要があるためです。Libertyの全ての構成ファイルにアクセスができる場合，原理的には記述されたパスワードを解読することが可能です。

難読化にはLibertyの`securityUtility`コマンドを利用します。

```
securityUtility encode MY_PASSWORD
```

生成された`{xor}EgYADx4MDAgQDRs=`のような文字列をパスワードに指定します。

``` xml
<containerAuthData user="db2inst1" password="{xor}EgYADx4MDAgQDRs=" />
```

難読化されたパスワードは，変数経由で提供することも可能です。

``` xml
<variable name="db2password" defaultValue="{xor}EgYADx4MDAgQDRs=" />
   ...
   <containerAuthData user="db2inst1" password="${db2password}" />
```

{% note %}

難読化のアルゴリズムにAESを指定することもできます。

```
securityUtility encode　--encoding=aes  MY_PASSWORD
```

AESの変換の際の鍵をデフォルトから変更する場合は，以下のように鍵を指定して難読化します。

```
securityUtility encode　--encoding=aes --key=MY_AES_KEY MY_PASSWORD
```

鍵をデフォルトから変更した場合は，Libertyがパスワードを解読できるように変数`wlp.password.encryption.key`で鍵を指定します。

``` xml
<variable name="wlp.password.encryption.key" value="MY_AES_KEY" />
```

このファイルはアクセスを制限できる場所に配置し，後述する`<include>`で構成ファイルに取り込むようにします。
{% endnote %}


#### 他のファイルの読み込み

`<include>`要素で`server.xml`から他のファイルを読み込むことができます。読み込み先のファイルの内容が`<include>`の場所に展開されます。

`location`属性に読み込みファイルを相対パスで記述すると，元のファイルからの相対パス，${server.config.dir}からの相対パスが検索されます。

``` xml
<include location="sessiondb.xml" />
```

変数などを使用した絶対パスでの記述がおすすめです。

``` xml
<include location="${shared.config.dir}/sessiondb.xml" />
```

読み込むファイルが存在しないとエラーとなります。存在するときにだけ読み込む場合には`optional`属性を設定します。

``` xml
<include location="${shared.config.dir}/sessiondb.xml" optional="true" />
```

`location`属性にディレクトリを指定すると，そのディレクトリにある全てのXMLファイルが読み込まれます。サブレディレクトリにあるXMLファイルは読み込まれません。

``` xml
<include location="${shared.config.dir}/commons/" />
```

`location`属性にはURLも指定することができます。

``` xml
<include location="https://server.example.com/config/sessiondb.xml" />
```

以下のディレクトリにXMLファイルが存在していると，自動的に読み込まれて`server.xml`の構成に追加されます。

- `${server.config.dir}/configDropins/defaults/` にあるXMLファイル
  - server.xmlより先に読み込まれる
  - 同じ設定項目がある場合には，`server.xml`が優先される
- `${server.config.dir}/configDropins/overrides/` にあるXMLファイル
  - server.xmlより後に読み込まれる
  - 同じ設定項目がある場合には，構成は上書きされる

#### 構成のマージ

構成ファイルは`<include>`や`configDropins`などにより複数のファイルに分けて構成できるため，同じ構成が複数のファイルに記述されることがあります。このような状況では，構成は一定のルールに従ってマージされます。

Libertyの構成には，何らかの管理オブジェクトを生成するFactory構成と，サーバー上の唯一の管理オブジェクトを設定するSingleton構成があります。'<webApplication>'や`<httpEndpoint>`，`<keyStore>`，`<dataSource>`，`<library>`などがFactory構成です。`<logging>`や`<quickStartSecurity>`，`<applicationManager>`などがSingleton構成です。

Factory構成は，複数回記述された場合もそれぞれが有効になります。下記の例ではWebアプリケーションは二つ，Liberty上に定義されることになります。

``` xml
<webApplication location="myapp.war" />
<webApplication location="myapp2.war" />
```

既存のFactory構成を上書きするには，`id`属性を指定して同一のオブジェクトである事を明示化します。

``` xml
<webApplication id="myapp" location="myapp.war" contextRoot="/myawesomeapp" />
<webApplication id="myapp" location="myapp2.war" />
```

同じIDをもつFactory構成はマージされ，上記の例は以下の構成と同じ意味になります。同じ属性である`location`はあとから記述されたものが有効になります。

``` xml
<webApplication id="myapp" location="myapp2.war" contextRoot="/myawesomeapp" />
```

Singleton構成は，`id`属性の記述がなくても自動的にマージされます。

``` xml
<logging traceSpecification="*=debug" value="ENHANCED" />
<logging traceSpecification="openjpa.jdbc.SQL=all" />
```

上記の例は以下の構成と同じ意味になります。

``` xml
<logging traceSpecification="openjpa.jdbc.SQL=all" value="ENHANCED" />
```

Singleton構成が子要素を持つ場合，それらはマージされます。

``` xml
<featureManager>
    <feature>servlet-4.0</feature>
</featureManager>
<featureManager>
    <feature>restConnector-2.0</feature>
</featureManager>
```

上記の例は以下の構成と同じ意味になります。

``` xml
<featureManager>
    <feature>servlet-4.0</feature>
    <feature>restConnector-2.0</feature>
</featureManager>
```

#### server.xmlの動的更新

Libertyの起動中に構成ファイルを変更して保存した場合，変更は即座に反映されます。`server.xml`だけでなく，そこから`<include>`されているファイルや`configDropins`にあるファイルも，変更は検知され動的に反映されます。

この挙動は開発時には非常に便利なのですが，本番環境で変更を動的に反映させたくない場合は，以下の構成を追加します。これによりMBean経由でトリガーされない限り（もしくはLibertyを再起動しない限り）変更は自動で反映されなくなります。

``` xml
<config updateTrigger="mbean"/>
```

またアプリケーションの変更も動的に反映されます。これを抑止するには以下の構成を追加します。

``` xml
<applicationMonitor updateTrigger="mbean" />
```

### server.xmlの構成例

#### Featureの構成

#### アプリケーションの構成

#### HTTPエンドポイントの構成

#### データベースの構成

### server.xml以外の構成ファイル

#### server.env

#### jvm.options

#### bootstrap.propreties

