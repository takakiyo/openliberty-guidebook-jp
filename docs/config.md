## Open Libertyの構成ファイル 



### server.xmlの基本

Libertyの構成を行う際の中心となるファイルで，XML形式で記述します。server.xmlは`${server.config.dir}`ディレクトリにあるファイルが使用されます。

#### デフォルトベースの構成

server.xmlに記述する構成要素には，基本的に全てデフォルトの値が決まっています。ファイルにはデフォルトから変更するものだけを記述します。このデフォルト値は，バージョンが上がっても変更されないようになっていますので，記述をしなくても安全です。環境に合わせてどのように変えたかを明確に管理するためにも，あえて利用するデフォルト値を構成に記述する，というのは避けましょう。

デフォルトの値は，Web上のWebSphere Liberty，Open Libertyのオンラインマニュアルでも確認できますが，Liberty Developer Toolsなどの開発ツールでも参照することが可能です。

> aside{構成ファイルのSchemaの生成}
> 
> server.xmlの構成項目やその説明，デフォルトの値が記載されたXSDファイル(XML Schema)を動的に生成することができます。Libertyの構成ファイルを操作するツールを自作する際などに，利用します。binディレクトリにある`schemaGen`コマンドを利用します。
>
> ```
> schemaGen /path/to/schema.xsd
> ```
> 
> 以下のようにオプションでロケールを指定すると，日本語で説明が記述されたXSDファイルが生成されます。
>
> ```
> schemaGen  --locale=ja_JP /path/to/schema.xsd
> ```



#### 変数の使用

#### 時刻の記述方法

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

