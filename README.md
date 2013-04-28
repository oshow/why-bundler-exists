原文→ http://gembundler.com/v1.3/rationale.html


理屈はいいからオススメのワークフローを知りたい、という人はページ下部の要約へどうぞ。

目次
----

  * Bundler の存在意義
  * アプリケーションが Bundler を使うようにする
  * 依存 gem をグループ分けする
  * コードをバージョン管理する
  * アプリケーションを他の開発者と共有する
  * 依存関係の更新
  * Gemfile を変更せずに gem をアップデートする
  * アプリケーションのデプロイ
  * FAQ: なぜ、`=` を使った依存関係の指定だけではダメなのか？
  * FAQ: git submodule ではダメなのか？
  * FAQ: `--without` で指定したグループの gem も Bundler がダウンロードするのは何故？
  * FAQ: インストール時にフラグをつける必要がある C 拡張があるのだが。
  * 要約
   * Bundler の基本的なワークフロー
  * 注釈

Bundler の存在意義
----

Bundler が作られた理由は、たくさんの開発環境・ステージング環境・本番環境でコードを簡単に共有できるようにするためだ。アプリケーションや gem を共有したいだけなら、GitHub に置いておきそれを色んな所から clone する、といったやり方もあるだろう。Bundler が違うのは、アプリケーションが問題なく起動し実行し続けられるように依存関係を満たしてくれることだ。

そのためにはまず、`Gemfile` と呼ばれるファイルに依存関係を宣言する必要がある。`Gemfile` はアプリケーションのルートディレクトリに置き、中身は以下のようになる。

```ruby
source 'http://rubygems.org'

gem 'rails', '3.0.0.rc'
gem 'rack-cache'
gem 'nokogiri', '~> 1.4.2'
```

この `Gemfile` で述べていることはそれほど多くない。まず、`Gemfile` 内で宣言している gem を探す先として `http://rubygems.org` を指定している。source は複数指定することも可能で、その場合は書いた順番に source を見に行く。

次に、いくつかの依存関係を宣言している。

* `rails` の バージョン `3.0.0.rc`
* `rack-cache` のどれかのバージョン
* `nokogiri` のバージョン `1.4.2` 以上、`1.5.0` 未満

依存関係が宣言できたら、Bundler に取りに行ってもらおう。

    $ bundle install    # bundle だけでもOK。bundle install のショートカット版だ

訳注：`bundle --path vendor/bundle` のように、`bundle install` 用のオプションも受け取ってくれる模様。

Bundler は `rubygems.org` に (他に宣言した source があれば、そこにも) 接続し、`Gemfile` で指定した gem が必要とする gem のリストを得る。`Gemfile` に書いた gem は依存する gem を持っており (また、その依存先の gem もさらに依存する gem を持っている場合がある) 、上記で実行した `bundle install` では多くの gem がインストールされることになる。

    $ bundle install
    Fetching source index for http://gemcutter.org/
    Using rake (0.8.7)
    Using abstract (1.0.0)
    Installing activesupport (3.0.0.rc)
    Using builder (2.1.2)
    Using i18n (0.4.1)
    Installing activemodel (3.0.0.rc)
    Using erubis (2.6.6)
    Using rack (1.2.1)
    Installing rack-mount (0.6.9)
    Using rack-test (0.5.4)
    Using tzinfo (0.3.22)
    Installing actionpack (3.0.0.rc)
    Using mime-types (1.16)
    Using polyglot (0.3.1)
    Using treetop (1.4.8)
    Using mail (2.2.5)
    Installing actionmailer (3.0.0.rc)
    Using arel (0.4.0)
    Installing activerecord (3.0.0.rc)
    Installing activeresource (3.0.0.rc)
    Using bundler (1.0.0.rc.3)
    Installing nokogiri (1.4.3.1) with native extensions
    Installing rack-cache (0.5.2)
    Installing thor (0.14.0)
    Installing railties (3.0.0.rc)
    Installing rails (3.0.0.rc)
    Your bundle is complete! Use `bundle show [gemname]` to see where a bundled gem is installed.

必要な gem が既にインストールされていたら、Bundler はそれを使う。そして gem をインストールしたのち、Bundler は gem の名前とバージョンを `Gemfile.lock` に書き込む。

アプリケーションが Bundler を使うようにする
----

Bundler の役割は、`Gemfile` に書いた gem (と、それが依存する gem) を Ruby が見つけられるようにすることだ。Rails3 アプリケーションならば、何もしなくとも Bundler を利用するようなコードになっている。Rails2.3 ならば、[設定方法](http://gembundler.com/v1.3/rails23.html)があるのでそれを読んでほしい。

その他のアプリケーション (例えば Sinatra 等) では、gem を require する前に Bundler の設定を書く必要がある。アプリケーションで最初に読み込むファイル ( Sinatra ならば、`require 'sinatra'` を書くファイル) に、以下のコードを書こう。

```ruby
require 'rubygems'
require 'bundler/setup'
```

訳注：Ruby1.9 以降なら require 'rubygems' は不要。

これにより `Gemfile` が自動的に認識され、そこに書いた gem が Ruby から利用可能になる (詳細を言ってしまうと、gem をロードパスにおいている)。これは例えるなら、`require 'rubygems'` の能力を拡張したと考えることもできる。

これで必要な gem を require するコードが書けるようになった。例えば `require 'sinatra'` を実行できる。たくさんの依存関係を持っている場合は「`Gemfile` に書いた gem を全部 require したいなぁ」と思うかもしれない。そういう時は、`require 'bundler/setup'` の直後に以下のコードを書く。

```ruby
Bundler.require(:default)
```

例で取り上げている Gemfile では、この行は以下を書いたのと同じになる。

```ruby
require 'rails'
require 'rack-cache'
require 'nokogiri'
```

鋭い読者ならば `rack-cache` の場合は `require 'rack-cache'` ではなく `require 'rack/cache'` が正しいのでは、と気づいただろう。`require 'rack/cache'` をするように Bundler に伝えるには、Gemfile を修正する。

```ruby
source 'http://rubygems.org'

gem 'rails', '3.0.0.rc'
gem 'rack-cache', :require => 'rack/cache'
gem 'nokogiri', '~> 1.4.2'
```

`Gemfile` が小さいなら、`Bundler.require` を使わずに手動で gem を require するのをお勧めする(特に、`:require` 指定を入れなきゃいけないような時は)。`Gemfile` が大きい場合は、require を大量に書くのを避けるために `Bundler.require` を使おう。

依存 gem をグループ分けする
----

「一部、特定の環境下でのみ使いたい gem がある」というパターンはないだろうか。例えば開発の初期において、開発環境では SQLite を使い、デプロイには `mysql2` や `pg` を使いたいという状況だ。この例では、開発マシンには MySQL や Postgres がインストールされておらず、Bundler にはそれらに関する gem の利用をスキップして欲しい。

そのためには、依存関係をグループ分けしよう。

```ruby
source 'http://rubygems.org'

gem 'rails', '3.2.2'
gem 'rack-cache', :require => 'rack/cache'
gem 'nokogiri', '~> 1.4.2'

group :development do
  gem 'sqlite3'
end

group :production do
  gem 'pg'
end
```

そして、開発環境では `production` グループをスキップするように Bundler に命令する。

    $ bundle install --without production

`--without production` を使ってインストールした、ということを Bundler は覚えておいてくれる。興味がある人向けに説明すると、`APP_ROOT/.bundle/config` にそのフラグが格納されている。Bundler がそこに記録した設定は `bundle config` を実行すると見ることができ、またこのコマンドでは `~/.bundle/config` に格納されたグローバルな設定や環境変数にセットされた設定も出力される。Bundler の設定についての詳細は、[Advanced Usage: Configuring Bundler](http://gembundler.com/v1.0/configuring.html) に載っている。

訳注：リンク切れ。どこにあるんだろう。

この状態では、コード中で `require 'bundler/setup'` する時、Bundler は --without で指定したグループを無視する。 また、次に `bundle install` を実行する時、Bundler は「最後の `bundle install` には `--without production` を付けていた」という事を覚えているので、`bundle install` に何も付けなくてももう一度そのフラグを付けてくれる。

`Bundler.require` の引数に指定することで、特定のグループを選んで require することも出来る。`:default` グループは、何のグループにも入っていない gem を意味する。`Bundler.require(:default, :development)` の様にすれば、`:default` グループと `:development` グループの両方を `require` してくれる。

Ralis が生成したアプリケーションは、デフォルトでは `application.rb` 内で `Bundler.require(:default, Rails.env)` を呼ぶようになっている。これは Rails environment に合わせて `Gemfile` 内のグループを選ぶ仕組みだ。Rails environment とは無関係のグループを require したいなら、`Bundler.require` の引数にそれを追加すればいい。

忘れないで欲しいのは、`Bundler.require` でグループを指定するのではなく、通常の `require` を使うことも出来るということだ。ある1つの gem だけ require をやめたい時もあるし、毎起動時に必要としない gem もあるだろう。

コードをバージョン管理する
----

開発を重ねたあとは、アプリケーションのコードと一緒に `Gemfile` と `Gemfile.lock` をチェックインしよう。そうすれば、最後にアプリケーションが正しく動いてた時に使っていた gem のバージョンがリポジトリに記録されることになる。例では `Gemfile` に3つの gem しか (場合によってバージョンが変化しうる指定方法で) 列挙されていないが、それらの gem が暗黙的に依存するものを含めれば、非常に多くの gem に依存していることは覚えておこう。

これは重要なことだ。`Gemfile.lock` はあなたが書いたコードとサードパーティのコードの両方合わせてうまく動いていた時の状態を一つのパッケージにしてくれる。`Gemfile` でサードパーティのコードのバージョンを厳密に指定したとしても、同じ保証は得られない。なぜならば、そのサードパーティの gem 自体の依存関係の宣言には、バージョンの範囲が指定されていることが多いからだ。

次に同じマシンで `bundle install` を実行したとき、Bundler は依存する gem が既に存在することを確認し、インストールプロセスをスキップする。

`.bundle` ディレクトリや、その中のファイルはチェックインしてはいけない。これらのファイルは各マシンにより特有の内容になる。各環境での `bundle install` 時のオプションの違いにもよるだろう。

`bundle pack` を実行すれば、必要とされる gem (git リポジトリを直接指定した gem は除く) が `vendor/cache` ディレクトリにダウンロードされる。必要な gem がそのディレクトリに存在していて、リポジトリにチェックインされていれば、インターネット(あるいは Rubygems サーバ)に接続しなくても Bundler が実行できる。リポジトリのサイズが増加してしまうので、これは必須のステップというわけではない。

アプリケーションを他の開発者と共有する
----

共同開発者(あるいは、自分の別マシン)がコードをチェックアウトする段階の話をしよう。この時、最後に開発していたマシンで使われていたサードパーティの gem のその時のバージョン(`Gemfile.lock` に書かれているものだ)を持ってくることが可能だ。共同開発者あるいは自分の別マシン(とにかく、先程 `bundle install` したのとは別マシン)で `bundle install` を実行すると Bundler は `Gemfile.lock` を見て、依存関係を解決する処理をスキップする。代わりに、元のマシンで使われているものと同じ gem をインストールする。

言い換えると、どのバージョンの依存 gem をインストールすべきかで悩む必要はないということだ。先ほどの例では、`rack-cache` 自体の依存関係には `rack >= 0.4` と書かれているが、`rack 1.2.1` を使えば動くことはわかっている。Rack 開発陣が `rack 1.2.2` をリリースしたとしても、Bundler は `1.2.1` を常にインストールしてくれる。これによりアプリケーション開発者の労力が大きく削減されるだろう。どのマシンでもサードパーティのコードは全て同一になっているのだから。

依存関係の更新
----

これで話は終わりではない。ある依存 gem をバージョンアップしたいと思うことが出てくるはずだ。例えば、`rails` を `3.0.0` final にアップデートしたいとしよう。重要なのは、一つの依存 gem だけをアップデートしたいのであって、依存関係の解決をやり直して全ての gem を最新版にしたいわけじゃないという事だ。引き続き使っている例では、依存 gem が3つだけだが、このケースでは全てをアップデートさせると複雑な事態を引き起こしかねない。

この件を説明するために、以下を考えてみよう。`rails 3.0.0.rc` は `actionpack 3.0.0.rc` に依存しており、その actionpack 3.0.0.rc は `rack ~> 1.2.1` という依存関係を持っている(これは `1.2.1` 以上 `1.3.0` 未満を意味する)。`rack-cache` は `rack >= 0.4` という依存関係を持つ。さらに仮定として、`rails 3.0.0` final も `rack ~> 1.2.1` という依存関係を持ち、`rails 3.0.0` がリリースされてからどこかの時点で Rack 開発陣は `rack 1.2.2` をリリースしたとする。

Rails をアップデートするために単純に全ての gem をアップデートすると、`rails 3.0.0` と `rack-cache` の要求を満たす `rack 1.2.2` がインストールされる。しかし、`rack-cache` は(なんらかの理由で) `rack 1.2.2` とは互換性がないかも知れない。アップデートして良いかを `rack-cache` に明確に尋ねたわけではないのだ。もっとも、`rack` を `1.2.1` から `1.2.2` にアップデートしても何も壊れることは多分ないだろうが、似たシナリオでバージョンがもっと大きく飛ぶような場合は問題が起こるかも知れない(それについてはページ下の [1] でスペースを取って検討している)。

この問題を避けるために、Bundler は「他の gem から依存されている gem はアップデートしない」という策を取る。今の例で言えば、`rack-cache` が `rack` に依存しているので、Bundler は `rack` をアップデートしない。これにより、`rails` をアップデートすることで不用意に `rack-cache` を壊してしまうことがなくなる。`rails 3.0.0` が依存する `actionpack 3.0.0` は `rack 1.2.1` でも依存関係を満たすので、Bundler は問題なしと判断する。これで、`rack-cache` が仮に `rack 1.2.2` と非互換だったとしても罠にハマらずに済む。

元々の例では依存関係に `rails 3.0.0.rc` と書いていた。これを `rails 3.0.0` にアップデートしたいなら、単に `Gemfile` の内容を `gem 'rails', '3.0.0'` にしてから以下を実行する。

    $ bundle install

上で説明した通り、`bundle install` コマンドは常に保守的なアップデートを心がけるので、`Gemfile` で明示していない gem のアップデートは拒否される。つまり、`Gemfile` で `rack-cache` に関していじっていないなら、Bundler は 「`rack-cache` 自身とその依存する gem (`rack`)」を変更できない一つの塊として扱う。もし `rails 3.0.0` が `rack-cache` と一緒には使えない(依存関係に矛盾がある)と分かれば、Bundler は依存関係のスナップショット(`Gemfile.lock`)とアップデートした `Gemfile` の間にコンフリクトがあることを報告する。

`Gemfile` をアップデートしたが既に依存関係が全て満たされていた場合は、次にアプリケーションを起動した時に Bundler は `Gemfile.lock` を静かに書き換える。例えば `Gemfile` に `mysql` を追加し、それが既にシステムにインストールされていたとする。その場合は `bundle install` をせずにアプリケーションを起動でき、Bundler は「最後にうまくいっていたもの」として `Gemfile.lock` にスナップショットを残す。

これは、最低限必要な gem(データベースドライバ、`wirble`、`ruby-debug` 等)が追加／更新される時に便利だ。そういったものは、たくさんの依存 gem を持つもの(`rails`)やたくさんの gem から依存されているもの(`rack`)をアップデートした時に失敗しやすい。もし静かなアップデートが失敗するなら、アプリケーションの起動も失敗し、Bundler は `bundle install` を実行せよと出力してくる。

Gemfile を変更せずに gem をアップデートする
----

場合によっては、Gemfile を変更せずに依存 gem をアップデートしたいこともあるだろう。例えば `rack-cache` の最新版を使いたいといった場合だ。`Gemfile` には `rack-cache` のバージョンを明示していないので、`rack-cache` を定期的に最新版にしたいつもりだ。そうであるなら、`bundle update` コマンドを使おう。

    $ bundle update rack-cache

上記コマンドで、`rack-cache` の依存関係は `Gemfile` が許す限り新しいものにアップデートされる(この例では、実際に最新版が使用される)。それ以外の依存 gem は変更されない。

だがしかし、必要に応じて他の gem がアップデートされることもある。例えば、最新版の `rack-cache` には `rack >= 1.2.2` という依存関係が指定されているとしたら、`rack` をアップデートしてくれと言わなくても Bundler は `rack` を `1.2.2` へアップデートする。もし他の gem から依存されている gem をアップデートせざるを得ない事になったら、それはアップデートが完了した後に知らされる。

Gemfile に載っている全ての gem を可能な限り最新版にしたい場合は、以下を実行する。

    $ bundle update

これは `Gemfile.lock` を無視して、依存関係の解決を全て最初からやり直す。これをする場合は、`git reset --hard` 等の元に戻す手段や、コードの動作を確認するテストスイートを準備しておこう。依存関係の解決を最初から全部やり直すことは予期しない結果になり得る。特に、前回のフルアップデートから多くのサードパーティ製パッケージの新しいバージョンがリリースされているような時には。

アプリケーションのデプロイ
----

`bundle install` を実行する時、デフォルトでは「システムの gem」へインストールされる。つまり、`bundle install` した gem は `gem list` でリストアップされるようになる。この場合、たくさんのアプリケーションを開発しているなら gem が共通で扱われるので再度ダウンロードしたりインストールしなくて済む。これは開発中は良いのだが、デプロイでは問題になることがある。

訳注：開発中の手元の環境でも、システムの gem に入れるのは嬉しくないという考え方もある。bundle install 時にプロジェクトのディレクトリに gem をインストールしたいなら、bundle install --path vendor/bundle のようにする。--path には vendor/bundle ディレクトリが指定されることが多い(慣習)。このディレクトリはリポジトリにチェックインしてはいけない。

デプロイ時にはシステムの gem 置き場へのアクセス権がないユーザでやらなければならない時もある。仮にそれが出来るユーザ(あるいは `sudo` 等)にてデプロイしたとしても、アプリケーションを起動するユーザがアクセス権を持たないかもしれない。例えば Passenger は `nobody` ユーザで Ruby のサブプロセスを生成する。`nobody` は通常のユーザよりも権限がいくらか低い。デプロイ環境の「隔離か統合か」というトレードオフは、隔離の方向へ傾きがちだ(サードパーティの依存関係が変更された時に `bundle install` によりデプロイ時間が長くかかるとしてもだ)。

このため、Bundler にはデプロイ環境のための一連のベストプラクティスが `--deployment` フラグとして備わっている。これらのプラクティスは Bundler を開発している時にもらった多くのフィードバックや、デプロイ時の Bundler のベストな設定が分からずにバグとして報告されてきたものがベースとなっている。`--deployment` フラグは以下の動作を追加する。

* gem をシステムにインストールする代わりに、アプリケーション内の `vendor/bundle` ディレクトリにインストールする。Bundler はこの場所にインストールしたことを覚えておいてくれて、(`Bundler.setup` や `Bundler.require` が)そこを参照するようになる。

* システムに既にインストールされている gem があっても、それを使わない。

* `bundle pack` を実行して `vendor/cache` へダウンロードしており、それをチェックインしていた場合(つまりデプロイ先でもこのディレクトリがチェックアウトされ、存在している場合)で、かつ git リポジトリを指定した gem がない場合は、gem インストール時にインターネット接続をしない。

* Bundler は `Gemfile.lock` を必須のものとして要求する。なければ動作しない。

* `Gemfile.lock` が `Gemfile` より古くても、Gemfile.lock を静かにアップデートしない。

Capistrano を使っているなら、`vendor/bundle` から `shared/vendor_bundle` へシンボリックリンクを張るのを忘れずに。こうすると Bundler が各デプロイ同士で gem のインストールを共有でき(gem に何も変更がなければデプロイが高速化できる)、かつ他のアプリケーションから隔離されているという利点は守られる。

デフォルトで gem を入れるディレクトリは `vendor/bundle` であり、そこに gem をインストールすることはデプロイプロセスの一部だ。そしてアプリケーションをチェックアウトしたユーザが、サードパーティーのコードもインストールすることになる。つまり、Passenger(あるいは Unicorn)からアプリケーションが見えるなら、その依存 gem まで見えることを意味する。

`--deployment` フラグは `Gemfile.lock` が最新であることを要求する。(開発環境やステージング環境で)テストが完了して、プロダクションに置くべきコードになっているかを確実にするためだ。 また `bundle check` を使うことで、アプリケーションをデプロイする前に `Gemfile.lock` が最新かどうかを確かめることが出来る。最後に Gemfile をいじって bundle install をしてから、アプリケーションの起動(あるいはテストの実行)が成功していれば、Gemfile.lock は常に最新と言える。

FAQ: なぜ、`=` を使った依存関係の指定だけではダメなのか？
----

Q: 依存 gem のバージョン指定の範囲を決めなければならないのはわかったが、`Gemfile` に書く依存関係を全て `=` で指定してしまうのではダメなのか？　そうしたら `Gemfile.lock` は不要な気がするが。

A: あなたが使う gem の多くはそれ自身が依存関係を持っており、それらが `=` で指定されていることは少ない。また、それらのバージョン指定を厳しくすることはあまり賢明ではないだろう。`Gemfile.lock` は、アプリケーションが必要とする gem の特定のバージョンを指定させてくれるものであり、アプリケーションが最後にうまく動いていた時に使っていたサードパーティのコードのバージョンを記録してくれている。

`Gemfile` にゆるめのバージョン指定(`nokogiri ~> 1.4.2` のような)を使うことで、`bundle update nokogiri` を実行した時に `nokogiri` 「だけを」アップデートし、しかも `~> 1.4.2` という範囲に入る限りで最新版を入れるという操作が可能になる。また、わざわざバージョンを調べなくても「nokogiri の最新版を使いたい」という欲求を満たすこともできるし(`gem 'nokogiri'` を `Gemfile` に書く)、その場合でもサードパーティのコードは全て同じバージョンを確実に使うようにするという利点も損なわれていない。

FAQ: git submodule ではダメなのか？
----

Q: Bundler のやり方で gem を管理しなければならない理由がわからない。必要な gem を取得してそれを git submodule 等で固定し、それぞれの submodule をロードパスに置くのではダメなのか？

A: 残念ながら、その方法では依存関係の解決を全て手でやらなければならない。しかも、依存する gem が依存する gem に関してもだ。またそれが成功したとしても、特定の gem をアップデートしたいと思っただけでまたやり直さなければならない。例えば `rails` をアップデートしたいとしたら、`rails` の依存 gem が依存する gem(`rack`, `erubis`, `i18n`, `tzinfo`等) を自分で探し、それぞれの依存関係を満たすような 新しいバージョンを調べないといけない。

はっきり言ってこれはコンピュータに解かせるべき問題であり、開発者たるあなたが時間を費やすべきことではない。

もっと言うと、手で依存関係を解決する過程でミスをしてしまっても、依存関係同士の衝突に関してのフィードバックが何もないはずだ。その結果は、事前にわからないランタイムエラーだ。例えば submodule で間違ったバージョンの `rack` を固定してしまった場合に、Rails や他の gem がそのバージョンの rack には存在しないメソッドを呼んだとしたら、実行時エラーを起こしてしまう。

結論： 一見そっちの方が簡単に見えるのだが、間違いなく大幅に複雑化する。

FAQ: `--without` で指定したグループの gem も Bundler がダウンロードするのは何故？
----

Q: `bundle install --without production` を実行したのだが、Bundler は `:production` グループの gem もダウンロードしているようだ。なぜ？

A: Bundler の `Gemfile.lock` は `Gemfile` に書かれた全ての依存 gem の特定のバージョンを含んでいなければならない。あなたがどんなオプションを渡したとしてもだ。でなければ、本番環境にアプリケーションをデプロイするだけで依存関係が変化してしまい、Bundler の利点がなくなってしまう。もはや本番環境のアプリケーションが開発時やテスト時に使っていたのと同じ gem を使っているかどうかは保証できない。さらに言えば、production グループに依存 gem を追加すると、そのアプリケーションはデプロイ不可能になってしまうかもしれない。

例えば、`rack =1.1` という依存指定の、本番環境のみで使う gem (`rack-debugging` としよう) があると想像してみて欲しい。`bundle install --without production` を実行した時に production グループが評価されないとすると、そのアプリケーションをデプロイした時に「`rack-debugging` が `rails`( `rack ~> 1.2.1` という依存関係を持つ `actionpack` に依存している) と衝突しています」というエラーが返ってくるだけだ。

別の例を挙げよう。`Gemfile` にただ `gem 'rack'` とだけ書いているシンプルな Rack アプリケーションを考えてみる。そして再び、`:production` グループに `rack-debugging` を書いたとしよう。`bundle install --without production` した時に `:production` グループが評価されないとすると、開発環境では `rack 1.2.1` が使われ、デプロイ時にはテストで使っていたバージョンの Rack と `rack-debugging` が衝突を起こしてしまう。

上記の例とは対照的に、実際に指定する環境と関係なく全てのグループの gem を評価することで、`rack-debugger` の依存関係を認識できる。つまり、`gem 'rack'` と書いた依存宣言を満たすように `rack 1.1` をインストールできる。

まとめると、特定の環境でどれを使いたいかという意図とは関係なく Gemfile にある全ての依存関係を常に評価することで、別の環境で別のグループの集合に切り替えた時に良くない驚きに遭遇するのを避けられる。そしてダウンロードだけをする(インストールはしない)ことで、本番環境だけ(あるいは開発環境だけ)で使う gem の難しい*インストール*プロセスが実現可能かどうかを心配しなくてよくなる。

FAQ: インストール時にフラグをつける必要がある C 拡張があるのだが。
----

Q: `mysql` をはじめとする C 拡張 の gem があり、コンパイルしてインストールするためにフラグをつける必要がある。Bundler ではこれらの gem のインストールプロセスのどこでフラグを渡せばよいのか？

A: まず第一に、`mysql` gem を置き換える `mysql2` gem ではこの問題は存在しない。モダンな C 拡張は通常、必要なヘッダを正しく見つけてくれる。

それでも C 拡張にフラグを渡さなければならないとしたら、`bundle config` コマンドをこう使う。

    $ bundle config build.mysql --with-mysql-config=/usr/local/mysql/bin/mysql_config

Bundler はこの設定を `~/.bundle/config` に保存し、同じユーザが `bundle install` を実行する時はこの設定を使ってくれる。よって、ある gem に必要なビルドオプションを一旦指定すれば、あとはもう心配をせずに済む。

要約
----

Bundler の基本的なワークフロー
----

Rails アプリケーションを作り始める時、`Gemfile` は既についてくる。他の種類のアプリケーション(Sinatra のような)を作る時は、以下を実行する。

    $ bundle init

`bundle init` コマンドはシンプルな `Gemfile` を作成してくれる。

次に、アプリケーションが依存する gem を Gemfile に追加していく。使いたい gem のバージョンについて考えていることがあるなら、そこに適切なバージョン指定を含めるようにする。

```ruby
source 'http://rubygems.org'

gem 'sinatra', '~> 0.9.0'
gem 'rack-cache'
gem 'rack-bug'
```

gem がまだシステムにインストールされていないなら、以下を実行する。

    $ bundle install

gem のバージョン要求を更新したいなら、まず Gemfile を編集する。

```ruby
source 'http://rubygems.org'

gem 'sinatra', '~> 1.0.0'
gem 'rack-cache'
gem 'rack-bug'
```

それから以下を実行。

    $ bundle install

もし `bundle install` が `Gemfile` と `Gemfile.lock` の間に衝突があると報告してきたら、次のようにする。

    $ bundle update sinatra

これは Sinatra gem と、その依存 gem だけをアップデートしてくれる。

`Gemfile` にある gem を全て可能な限り最新にしたいなら、以下を実行する。

    $ bundle update

`Gemfile.lock` が変更されたときはいつでも、リポジトリにそれをチェックインしよう。アプリケーションがうまく動いていたときに使っていたサードパーティコードのバージョンを、履歴に含めるんだ。

ステージングや本番サーバにコードをデプロイする時は、まずテストを走らせ(あるいはローカルの開発サーバで走らせる)、リポジトリに `Gemfile.lock` を間違いなくチェックインさせておこう。それからリモートサーバ上で、以下を実行する。

    $ bundle install --deployment

注釈
----

[1] 例えば、`rails 3.0.0` が `rack 2.0` に依存しているとする。この場合でも `rack-cache` の要求する `rack >= 1.0` を満たしてしまっている。無論、`rack-cache` のバージョン指定方法がアホなだけだ、と言うことだって出来る。だが、プロジェクトが依存するバージョンをどうするかでジレンマが起こるのはよくある話で、こういった状況は現実に(広範囲で)存在している。依存関係を厳しくしすぎる(`rack =1.2.1`)とそのプロジェクトを他のプロジェクトで使うのが難しくなる。依存関係を緩くしすぎる(`rack >= 1.0`)と新しい Rack がリリースされた時に自分のコードが壊れてしまうかもしれない。`rack ~> 1.2.1` のような依存指定を使いつつ、[Semantic Versioning](http://semver.org/) に従うことでほぼこの問題を解決できるが、それは漏れがあっては意味の無い話だ。Rubygems は10万個以上あるのだから、この仮定を置くことは現実には難しい。




