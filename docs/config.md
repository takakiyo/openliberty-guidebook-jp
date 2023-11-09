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

環境変数`VARIABLE_SOURCE_DIRS`は変数を定義するファイルを格納するディレクトリを指定します。複数のディレクトリを指定するには，Windows環境では`;`で，そのほかの環境では`:`で区切って指定します。この環境変数が設定されていなかった場合には，デフォルトで`${server.config.dir}/variables`ディレクトリが使用されます。

これらのディレクトリにファイルを置いた場合，ファイル名が変数名として利用されます。値はファイルに記述された内容です。たとえば`httpPort`というファイルに`8080`という内容が保存されていた場合，`${httpPort}`という変数は`8080`になります。

ファイル名が`.properties`の拡張子を持っている場合には，一つのファイルに複数の変数定義を記述することができます。例えば以下のように記述します。

``` properties
httpPort=9080
httpsPort=9443
```

Libertyの起動コマンドでも`--`につづけて変数を指定することができます。たとえば`start`アクションでサーバーを起動する際に，以下のように変数を指定したとします。

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

#### 他のファイルの読み込み

#### 設定のマージ

#### server.xmlの動的更新

### server.xmlの構成例

#### Featureの構成

#### アプリケーションの構成

#### HTTPエンドポイントの構成

#### データベースの構成

### server.xml以外の構成ファイル

#### server.env

#### jvm.options

#### bootstrap.propreties

