## Open Libertyのディレクトリ構成

デフォルト状態のLibertyのディレクトリ構成は以下のようになります。

- `wlp`：製品ルート
    - `bin`：Libertyの管理スクリプト
    - `clients`：Libertyクライアント、シン・クライアントの管理スクリプト
    - `dev`：開発者用のライブラリや文章などのリソース
    - `lib`：プラットフォーム・ランタイム
    - `templates`：構成のテンプレート
    - `etc`：環境依存の構成（デフォルトでは存在しない）
    - `usr`：ユーザーディレクトリ
        - `extension`：ユーザー開発Feature（デフォルトでは存在しない）
        - `shared`：共有ファイルディレクトリ（デフォルトでは存在しない）
            - `apps`：共有アプリケーション
            - `config`：共有構成ファイル
            - `resource`：共有リソース
        - `servers/サーバー名`：サーバー構成ディレクトリ
            - `apps`：構成して導入するためのアプリケーション
            - `dropins`：構成なしで実行できるアプリケーション・ドロップイン
            - `logs`：サーバー・ログ・ディレクトリ
            - `workarea`：サーバー・ワークエリア
        - `clients/クライアント名`：クライアント構成ディレクトリ

これらのうち，利用者がファイルを追加したり編集したりできるディレクトリは`etc`と`usr`のみです。これ以外の場所にファイルを置いたり，既存のファイルを編集したりはしないでください。

この二つのディレクトリの目的の違いは可搬性の有無です。`usr`ディレクトリには，原則として環境に依存しない，可搬性のあるものを格納します。このディレクトリ以下は，サーバーを導入・実行が可能なアーカイブファイルにパッケージしたときに，その中に格納されます。`etc`ディレクトリには，その環境の全サーバーに影響のある構成ファイルがおかれ，環境固有の情報を設定します。このディレクトリは，パッケージしたときにアーカイブには含まれません。

### 組み込み変数によるディレクトリの参照

一部のディレクトリは，`server.xml`などの構成ファイルから参照可能な組み込み変数が事前に定義されています。

これらの変数は，Libertyを別の場所に導入したり，環境変数などでディレクトリの場所を変えたとしても，それに合わせてディレクトリの場所を参照できるようになっています。これらの変数を活用することで，構成ファイルを導入環境に依存しない可搬性のある状態に保つことができます。

|ディレクトリ|デフォルトの場所|変数名|
|----------|-------------|------|
|製品ルート|`wlp`|`${wlp.install.dir}`|
|プラットフォーム・ランタイム|`wlp/lib`|`${wlp.lib.dir}`|
|ユーザーディレクトリ|`wlp/usr`|`${wlp.user.dir}`|
|ユーザー開発Feature|`wlp/usr/extension`|`${usr.extension.dir}`|
|共有アプリケーション|`wlp/usr/shared/apps`|`${shared.app.dir}`|
|共有構成ファイル|`wlp/usr/shared/config`|`${shared.config.dir}`|
|共有リソース|`wlp/usr/shared/resource`|`${shared.resource.dir}`|
|サーバー構成ディレクトリ|`wlp/usr/servers/サーバー名`|`${server.config.dir}`|
|サーバー・アウトプット・ディレクトリ|`wlp/usr/servers/サーバー名`|`${server.output.dir}`|
|サーバー・ログ・ディレクトリ|`wlp/usr/servers/サーバー名/logs`|`${server.logs.dir}`|

また，実行中のサーバー名が変数`${wlp.server.name}`で参照可能です。

### 各ディレクトリの役割

#### Libertyの管理スクリプト

`wlp/bin`ディレクトリです。Libertyを管理するためのコマンドが格納されています。コマンドはシェルスクリプトないしバッチファイルとして実装されています。一番重要なコマンドは`server`コマンドで，サーバーの作成や起動・停止などのほとんどの作業をおこなうことができます。

#### プラットフォーム・ランタイム

`wlp/lib`ディレクトリです。Libertyを実行するためのJARファイルや，様々な構成情報のファイルが格納されています。最小限のFeatureだけを導入していればサイズは小さく，全てのFeatureを導入するとサイズが最大になります。

#### 環境依存の構成

`wlp/etc`ディレクトリです。デフォルトでは存在しないので，必要に応じて作成します。導入されている環境に固有の情報を設定します。

全てのサーバーに有効になる`server.env`，`jvm.options`や，`featureUtility`コマンドのプロパティファイルなどを格納します。ユーザーディレクトリをデフォルトの場所から変更する場合は，`server.env`に環境変数`WLP_USER_DIR`でディレクトリの場所を指定します。

Libertyの実行環境は，容易に異なる環境へのコピーが可能で，そのために環境をパッケージする機能をもっていますが，`etc`ディレクトリに格納する情報は可搬性がない情報なので，パッケージ対象となりません。

#### ユーザーディレクトリ

`wlp/usr`ディレクトリです。`server create`コマンドなどで構成作成したときに作成されます。ユーザーの構成情報や実行するアプリケーション，実行に必要となるリソースファイルなどは，全てユーザーディレクトリ以下に格納します。

#### サーバー構成ディレクトリ

`wlp/usr/servers/サーバー名`ディレクトリです。サーバーの構成情報を格納します。

`server.xml`や`server.env`，`jvm.options`，`bootstrap.properties`などの構成ファイルが置かれます。また`configDropins`および`variables`サブディレクトリも構成に使われます。

環境をパッケージする際には，サブディレクトリも含めてパッケージ対象となります。ただし，`logs`ディレクトリおよび`workarea`ディレクトリ以下のファイル，JavaダンプやHeapダンプなどのファイルはパッケージされません。

#### アプリケーション

`wlp/usr/servers/サーバー名/apps`ディレクトリです。通常はこのディレクトリに実行するアプリケーションを配置します。

このディレクトリに置いただけでは有効にならず，`server.xml`などでアプリケーションの構成を行う必要があります。アプリケーションの動作や実行時のバインドなどについて詳細に設定することが可能です。

#### アプリケーション・ドロップイン

`wlp/usr/servers/サーバー名/dropins`ディレクトリです。

このディレクトリに置かれたアプリケーションは，構成することなく，サーバーで実行することができます。ただし，様々な項目についてデフォルトでの動作となり，細かい設定はできません。

#### サーバー出力ディレクトリ

通常はサーバー構成ディレクトリと同じ，`wlp/usr/servers/サーバー名`ディレクトリです。サーバーからの出力がおかれるディレクトリです。

`logs`ディレクトリが作成されてログが出力され，動的に生成されるファイルが置かれます。また，`workarea`ディレクトリが作成されて一時ファイルやキャッシュが保存されます。この二つのディレクトリは，パッケージの対象となりません。

また，サーバー実行時のカレントディレクトリも，デフォルトではこのディレクトリになります。カレントディレクトリに作成されるダンプなどは，このディレクトリに作成されます。

サーバー構成ディレクトリをRead Onlyで利用したい場合や，出力を別のディレクトリに分けたい場合には，環境変数`WLP_OUTPUT_DIR`で別の場所に変更することができます。たとえば`server.env`で以下のように設定すると，

``` properties
WLP_OUTPUT_DIR=/output/wlp
```

`/output/wlp/サーバー名`が新しいサーバー出力ディレクトリになります。


### Libertyを管理するコマンド

Libertyのコマンドは，基本的にアクションを指定して実行します。

```
server アクション [オプション]
```

アクションに`help`を指定すると，利用方法が表示されます。

```
server help
```

また`help`に続けてアクション名を指定すると，アクションごとの詳細なヘルプを表示できます。

```
server help アクション
```

> [!NOTE]
>例外は`schemaGen`コマンドと`serverSchemaGen`コマンドです。これらはアクションを指定せずに直接実行します。また，`jaxws`などのサブディレクトリにあるコマンドも，異なる書式を持ちます。これらはいずれも，引数をつけずにコマンド名だけで実行するとヘルプが表示されます。

#### serverコマンド

Libertyの基本的な作成・起動・停止などの操作を行うコマンドです。サーバー名を指定して実行するアクションでは，サーバー名を省略すると`defaultServer`が使用されます。

主なアクションについて詳細に解説します。

##### server createアクション

サーバー構成を作成します。

```
server create サーバー名 [オプション]
```

サーバー名として使用できるのは英数の大文字小文字（`A-Z`，`a-z`）および数値（`0-9`），アンダースコア（`_`），ハイフン（`-`），プラス記号（`+`），ピリオド（`.`）です。ハイフンとピリオドは最初の文字として使用することはできません。

オプションとして`--template=テンプレート名`が使用できます。テンプレート名は，`wlp/templates/servers`ディレクトリにあるディレクトリ名から選択します。

サーバー構成を削除するには，`wlp/usr/servers`にあるサーバー構成ディレクトリをまるごと削除します。

##### server runアクション

サーバーをフォアグランドで実行します。

```
server run サーバー名 [オプション]
```

サーバーから標準出力，標準エラー出力への出力はコンソールに直接出力されます。サーバーが停止するまで，コマンドラインには復帰しません。サーバーを停止するには，コンソールでCtrl+Cを入力するか，別のコンソールから`server stop`コマンドを実行します。

オプションとして`--`に続けて`--keyName=value`の形のオプションを指定できます。指定した値は構成ファイルで`${keyName}`という変数名で参照できます。

##### server startアクション

サーバーをバックグランドで実行します。

```
server start サーバー名 [オプション]
```

サーバーから標準出力，標準エラー出力への出力はサーバー・ログ・ディレクトリの`console.log`に記録されます。サーバーの起動が終了すると，すぐにコマンドラインに復帰します。サーバーを停止するには，`server stop`コマンドを実行します。

オプションとして`--clean`を指定すると，キャッシュされた情報を利用せずに起動します。workareaの下の情報が破損して正常に起動できなくなった時に指定します。また，ランタイムのバージョンを上げたときにも，初回は`--clean`をつけないと正常に起動しないことがあります。

`run`アクションと同様のオプションも指定できます。

##### server stopアクション

実行されているサーバーを停止します。

```
server stop サーバー名 [オプション]
```

サーバーでサーブレットなど実行中の処理があると，その処理が完了するまで待機します。待機時間はデフォルトで30秒です。`--force`オプションを指定すると，実行中の処理を中断して直ちに停止処理が行われます。また`--timeout=1m30s`のようなオプションを指定すると，処理が完了するまで最大で1分30秒待機するようになります。

##### server statusアクション

指定されたサーバーが実行中かを調べます。

```
server status サーバー名
```

##### server packageアクション

Libertyのランタイム，構成，リソース，アプリケーションをアーカイブファイルにパッケージし，他の場所に保存，配布，デプロイできるようにします。

```
server package サーバー名 [オプション]
```

パッケージを行う際には，サーバーは停止している必要があります。

`--archive=ファイル名`で保存先のアーカイブファイルを指定します。拡張子は`.zip`，`.tar`，`.jar`が指定できます。

`--include=usr`を指定すると，Libertyランタイムは保存せず，ユーザーディレクトリ以下の構成，リソース，アプリケーションを保存します。デフォルトでは`wlp/usr`以下が，`wlp/etc/server.env`で環境変数`WLP_USER_DIR`が設定されていればそのディレクトリが，アーカイブにパッケージされます。

`--include=all`では，Libertyランタイムの全て，およびユーザーディレクトリ以下の構成，リソース，アプリケーションを保存します。

`--include=minify`では，サーバーで定義されたFeatureを実行するための最低限のファイルで構成されたLibertyランタイムと，構成，リソース，アプリケーションを保存します。展開サイズが最小限となるLibertyの実行環境を作成することができます。

`--include=all,runnable`もしくは`--include=minify,runnable`を指定すると，アーカイブファイルにJARファイルを指定したときに，実行可能JARファイルにするために必要なファイルが追加されます。こうして作成したアーカイブファイルは，`java -jar ファイル名.jar`コマンドで直接実行することができ，Libertyサーバーが起動します。

`--include=minify`を指定した場合，特定のOSでのみ実行可能なランタイムを作成することで，さらにサイズを小さくすることができます。オプション`--os=OS名`を指定します。OS名としては，`Linux`，`Win32`，`AIX`，`z/OS`，`MacOSX`などが指定できます。

##### server dumpアクション

サーバーの問題判別に使用するデータを収集してアーカイブファイルに保存します。

```
server dump サーバー名 [オプション]
```

アーカイブファイルには，サーバー構成，ログ，workareaに展開されたアプリケーションの詳細が含まれます。サーバーが稼働中に実行すれば，追加で以下の情報も保存できます。

- サーバー内の各 OSGi バンドルの状態
- サーバー内の各 OSGi バンドルの配置情報
- Service Component Runtime (SCR) 環境によって管理されるコンポーネントのリスト
- SCRの各コンポーネントの詳細情報
- 各OSGiバンドルの構成管理データ
- 登録されている OSGi サービスに関する情報
- Java 仮想マシン (JVM)，ヒープサイズ，オペレーティングシステム，スレッド情報，ネットワークステータスなどの実行時環境設定

オプションに`--include=診断オプション,診断オプション`を指定すると，追加の情報を取得してファイルに保存できます。診断情報に，

- `heap`を指定すると，ヒープダンプが取得されます。
- `system`を指定すると，システムダンプが取得されます。
- `thread`を指定すると，スレッドダンプが取得されます。IBM J9VM/OpenJ9 VMを使用している環境ではjavacoreとよばれる詳細な問題判別情報を取得します。

オプション`--archive=ファイル名.zip`で出力先のアーカイブファイルを指定します。

#### productInfoコマンド

導入されているLibertyの種類とバージョンを表示します。製品版のWebSphere Libertyが導入されている場合は，そのエディションも表示されます。

```
productInfo version
```

導入されているFeatureの一覧を表示します。

```
productInfo featureInfo
```

導入環境を確認します。保存されているチェックサムと実際のファイルを比較し，導入環境が破損していないかを確認します。

```
productInfo validate
```

#### featureUtilityコマンド

ネットワーク上のリポジトリーから，追加のFeatureをダウンロードして導入します。リポジトリーは，Mavenのセントラルリポジトリーが利用されます。

特定のFeatureを導入するには，以下のように実行します。Feature名は，スペースで区切って複数を指定することもできます。Featureが依存関係にあるFeatureが欠落していれば，それらも合わせて導入されます。

```
featureUtility installFeature Feature名
```

サーバーの構成ファイルで使用するように定義されているFeatureを導入するには，以下のように実行します。足りないものがあれば追加導入されます。

```
featureUtility installServerFeatures サーバー名
```

リポジトリーにあるFeatureを検索するには，以下のように実行します。たとえばキーワードに`servlet`を指定して，提供されているサーブレット仕様のバージョン一覧を検索することができます。

```
featureUtility find キーワード
```

ネットワーク接続にProxyの構成などが必要な場合，`featureUtility.properties`ファイルを作成する必要があります。手順については，以下のコマンドを実行し，指示に従ってください。

```
featureUtility viewSettings
```

> [!NOTE]
>製品版のWebSphere Libertyには，リポジトリーとしてIBMのサイトを利用する`installUtility`も同梱されています。`install`アクションで，Feature名やサーバー名を指定してFeatureを導入することができます。
>
>```
>installUtility install Feature名
>```
>
>```
>installUtility install サーバー名
>```
>
>`find`アクションや`viewSettings`も同じように使用できます。
>
>また，`installUtility`固有の機能として，導入済みのFeatureを削除することが可能です。この機能は，Open Libertyではまだ利用できません。
>
>```
>installUtility uninstall Feature名
>```

#### securityUtilityコマンド

Libertyのセキュリティに関わるいくつかの機能が提供されます。

##### securityUtility encodeアクション

Libertyの構成ファイルにパスワードを記述するために難読化した文字列を生成します。デフォルトではXORエンコードで難読化します。

```
securityUtility encode パスワード文字列
```

エンコード方式としてAESを使うこともできます。AESの秘密鍵はデフォルトのものが使用されます。

```
securityUtility encode --encoding=aes パスワード文字列
```

AESの秘密鍵をデフォルトから変える場合は，秘密鍵の文字列を`--key`オプションで指定します。

```
securityUtility encode --encoding=aes --key=鍵文字列 パスワード文字列
```

このパスワードを構成で使用する場合は，秘密鍵を変数`wlp.password.encryption.key`で指定する必要があります。

``` xml
<variable name="wlp.password.encryption.key" value="鍵文字列" />
```

Libertyが復号する必要がないパスワードで指定できる，安全な一方向ハッシュで変換したパスワードは，以下で生成できます。この方法で生成したパスワードは`<quickStartSecurity>`要素の`password`属性で使用できます。

```
securityUtility encode --encoding=hash パスワード文字列
```

##### securityUtility createSSLCertificateアクション

Libertyで使用する自己署名証明書をふくんだキーストアを作成します。

Libertyでは，設定されたキーストアが存在しなければ自動的に作成されます。ですが，何らかの理由で作成されるキーストアの構成をデフォルトから変更したい場合は，createSSLCertificateアクションであらかじめ作成しておくことができます。

```
securityUtility createSSLCertificate -server=サーバー名 --password=パスワード文字列 [オプション]
```

オプションとしては，`--validity=日数`で有効期限を変更したりできます。カスタマイズできる項目については，コマンドを実行してヘルプを表示してください。

```
securityUtility help createSSLCertificate
```

##### securityUtility createLTPAKeysアクション

IBM製品間でSSO（Single Sign On）を実現するため，LTPAを使用するためのキーファイルを作成します。詳細については，セキュリティの章で解説します。

```
securityUtility createLTPAKeys -server=サーバー名 --password=パスワード文字列 [オプション]
```

##### securityUtility tlsProfilerアクション

SSL/TLSによる疎通確認と，接続で使用できる暗号化スイートを表示します。

```
securityUtility tlsProfiler --verbose --host=接続先ホスト名 --port=ポート番号
```

通常の実行では正常に接続できた暗号化スイートのみを表示し，`--verbose`を指定すると失敗したものも表示します。失敗したときのエラーメッセージなどは表示されませんので，簡易的な疎通確認にのみ使用できます。

