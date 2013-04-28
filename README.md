
http://gembundler.com/v1.3/rationale.html



If you just want to know our recommended workflow, and don't care about the rationale, feel free to jump to the summary below.

理屈はいいからオススメのワークフローを知りたい、という人はページ下部の要約へどうぞ。


Bundler's Purpose and Rationale
----

Bundler の存在意義
----

We designed bundler to make it easy to share your code across a number of development, staging and production machines. Of course, you know how to share your own application or gem: stick it on GitHub and clone it where you need it. Bundler makes it easy to make sure that your application has the dependencies it needs to start up and run without errors.

Bundler が作られた理由は、たくさんの開発環境・ステージング環境・本番環境でコードを簡単に共有できるようにするためだ。アプリケーションや gem を共有したいだけなら、GitHub に置いておきそれを色んな所から clone する、といったやり方もあるだろう。Bundler が違うのは、アプリケーションが問題なく起動し実行し続けられるように依存関係を満たしてくれることだ。

First, you declare these dependencies in a file at the root of your application, called Gemfile. It looks something like this:

そのためにはまず、`Gemfile` と呼ばれるファイルに依存関係を宣言する必要がある。`Gemfile` はアプリケーションのルートディレクトリに置き、中身は以下のようになる。

    source 'http://rubygems.org'

    gem 'rails', '3.0.0.rc'
    gem 'rack-cache'
    gem 'nokogiri', '~> 1.4.2'

This Gemfile says a few things. First, it says that bundler should look for gems declared in the Gemfile at http://rubygems.org. You can declare multiple Rubygems sources, and bundler will look for gems in the order you declared the sources.

この `Gemfile` で述べていることはそれほど多くない。まず、`Gemfile` 内で宣言している gem を探す先として `http://rubygems.org` を指定している。source は複数指定することも可能で、その場合は書いた順番に source を見に行く。

Next, you declare a few dependencies:

次に、いくつかの依存関係を宣言している。

* on version 3.0.0.rc of rails
* on any version of rack-cache
* on a version of nokogiri that is >= 1.4.2 but < 1.5.0

* `rails` の バージョン `3.0.0.rc`
* `rack-cache` のどれかのバージョン
* `nokogiri` のバージョン `1.4.2` 以上、`1.5.0` 未満

After declaring your first set of dependencies, you tell bundler to go get them:

依存関係が宣言できたら、Bundler に取りに行ってもらおう。

    $ bundle install    # bundle is a shortcut for bundle install

    $ bundle install    # bundle だけでもOK。bundle install のショートカット版だ

訳注：`bundle --path vendor/bundle` のように、`bundle install` 用のオプションも受け取ってくれる模様。

Bundler will connect to rubygems.org (and any other sources that you declared), and find a list of all of the required gems that meet the requirements you specified. Because all of the gems in your Gemfile have dependencies of their own (and some of those have their own dependencies), running bundle install on the Gemfile above will install quite a few gems.

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

If any of the needed gems are already installed, Bundler will use them. After installing any needed gems to your system, bundler writes a snapshot of all of the gems and versions that it installed to Gemfile.lock.

必要な gem が既にインストールされていたら、Bundler はそれを使う。そして gem をインストールしたのち、Bundler は gem の名前とバージョンを `Gemfile.lock` に書き込む。

Setting Up Your Application to Use Bundler
----

アプリケーションが Bundler を使うようにする
----

Bundler makes sure that Ruby can find all of the gems in the Gemfile (and all of their dependencies). If your app is a Rails 3 app, your default application already has the code necessary to invoke bundler. If it is a Rails 2.3 app, please see Setting up Bundler in Rails 2.3.

Bundler の役割は、`Gemfile` に書いた gem (と、それが依存する gem) を Ruby が見つけられるようにすることだ。Rails3 アプリケーションならば、何もしなくとも Bundler を利用するようなコードになっている。Rails2.3 ならば、[設定方法](http://gembundler.com/v1.3/rails23.html)があるのでそれを読んでほしい。

For another kind of application (such as a Sinatra application), you will need to set up bundler before trying to require any gems. At the top of the first file that your application loads (for Sinatra, the file that calls require 'sinatra'), put the following code:

その他のアプリケーション (例えば Sinatra 等) では、gem を require する前に Bundler の設定を書く必要がある。アプリケーションで最初に読み込むファイル ( Sinatra ならば、`require 'sinatra'` を書くファイル) に、以下のコードを書こう。

    require 'rubygems'
    require 'bundler/setup'

訳注：Ruby1.9 以降なら require 'rubygems' は不要。

This will automatically discover your Gemfile, and make all of the gems in your Gemfile available to Ruby (in technical terms, it puts the gems "on the load path"). You can think of it as an adding some extra powers to require 'rubygems'.

これにより `Gemfile` が自動的に認識され、そこに書いた gem が Ruby から利用可能になる (詳細を言ってしまうと、gem をロードパスにおいている)。これは例えるなら、`require 'rubygems'` の能力を拡張したと考えることもできる。

Now that your code is available to Ruby, you can require the gems that you need. For instance, you can require 'sinatra'. If you have a lot of dependencies, you might want to say "require all of the gems in my Gemfile". To do this, put the following code immediately following require 'bundler/setup':

これで必要な gem を require するコードが書けるようになった。例えば `require 'sinatra'` を実行できる。たくさんの依存関係を持っている場合は「`Gemfile` に書いた gem を全部 require したいなぁ」と思うかもしれない。そういう時は、`require 'bundler/setup'` の直後に以下のコードを書く。

    Bundler.require(:default)

For our example Gemfile, this line is exactly equivalent to:

例で取り上げている Gemfile では、この行は以下を書いたのと同じになる。

    require 'rails'
    require 'rack-cache'
    require 'nokogiri'

Astute readers will notice that the correct way to require the rack-cache gem is require 'rack/cache', not require 'rack-cache'. To tell bundler to use require 'rack/cache', update your Gemfile:

鋭い読者ならば `rack-cache` の場合は `require 'rack-cache'` ではなく `require 'rack/cache'` が正しいのでは、と気づいただろう。`require 'rack/cache'` をするように Bundler に伝えるには、Gemfile を修正する。

    source 'http://rubygems.org'

    gem 'rails', '3.0.0.rc'
    gem 'rack-cache', :require => 'rack/cache'
    gem 'nokogiri', '~> 1.4.2'

For such a small Gemfile, we'd advise you to skip Bundler.require and just require the gems by hand (especially given the need to put in a :require directive in the Gemfile). For much larger Gemfiles, using Bundler.require allows you to skip repeating a large stack of requirements.

`Gemfile` が小さいなら、`Bundler.require` を使わずに手動で gem を require するのをお勧めする(特に、`:require` 指定を入れなきゃいけないような時は)。`Gemfile` が大きい場合は、require を大量に書くのを避けるために `Bundler.require` を使おう。

Grouping Your Dependencies
----

依存 gem をグループ分けする
----

You'll sometimes have groups of gems that only make sense in particular environments. For instance, you might develop your app (at an early stage) using SQLite, but deploy it using mysql2 or pg. In this example, you might not have MySQL or Postgres installed on your development machine, and want bundler to skip it.

「一部、特定の環境下でのみ使いたい gem がある」というパターンはないだろうか。例えば開発の初期において、開発環境では SQLite を使い、デプロイには `mysql2` や `pg` を使いたいという状況だ。この例では、開発マシンには MySQL や Postgres がインストールされておらず、Bundler にはそれらに関する gem の利用をスキップして欲しい。

To do this, you can group your dependencies:

そのためには、依存関係をグループ分けしよう。

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

Now, in development, you can instruct bundler to skip the production group:

そして、開発環境では `production` グループをスキップするように Bundler に命令する。

    $ bundle install --without production

Bundler will remember that you installed the gems using --without production. For curious readers, bundler stores the flag in APP_ROOT/.bundle/config. You can see all of the settings that bundler saved there by running bundle config, which will also print out global settings (stored in ~/.bundle/config), and settings set via environment variables. For more information on configuring bundler, please see [Advanced Usage: Configuring Bundler](http://gembundler.com/v1.0/configuring.html).

`--without production` を使ってインストールした、ということを Bundler は覚えておいてくれる。興味がある人向けに説明すると、`APP_ROOT/.bundle/config` にそのフラグが格納されている。Bundler がそこに記録した設定は `bundle config` を実行すると見ることができ、またこのコマンドでは `~/.bundle/config` に格納されたグローバルな設定や環境変数にセットされた設定も出力される。Bundler の設定についての詳細は、[Advanced Usage: Configuring Bundler](http://gembundler.com/v1.0/configuring.html) に載っている。

訳注：リンク切れ。どこにあるんだろう。

If you run bundle install later, without any flag, bundler will remember that you last called bundle install --without production, and use that flag again. When you require 'bundler/setup', bundler will ignore gems in these groups.

この状態では、コード中で `require 'bundler/setup'` する時、Bundler は --without で指定したグループを無視する。 また、次に `bundle install` を実行する時、Bundler は「最後の `bundle install` には `--without production` を付けていた」という事を覚えているので、`bundle install` に何も付けなくてももう一度そのフラグを付けてくれる。

You can also specify which groups to automatically require through the parameters to Bundler.require. The :default group includes all gems not listed under any group. If you call Bundler.require(:default, :development), bundler will require all the gems in the :default group, as well as the gems in the :development group.

`Bundler.require` の引数に指定することで、特定のグループを選んで require することも出来る。`:default` グループは、何のグループにも入っていない gem を意味する。`Bundler.require(:default, :development)` の様にすれば、`:default` グループと `:development` グループの両方を `require` してくれる。

By default, a Rails generated app calls Bundler.require(:default, Rails.env) in your application.rb, which links the groups in your Gemfile to the Rails environment. If you use other groups (not linked to a Rails environment), you can add them to the call to Bundler.require, if you want them to be automatically required.

Ralis が生成したアプリケーションは、デフォルトでは `application.rb` 内で `Bundler.require(:default, Rails.env)` を呼ぶようになっている。これは Rails environment に合わせて `Gemfile` 内のグループを選ぶ仕組みだ。Rails environment とは無関係のグループを require したいなら、`Bundler.require` の引数にそれを追加すればいい。

Remember that you can always leave groups of gems out of Bundler.require, and then require them manually using Ruby's require at the appropriate place in your app. You might do this because requiring a certain gem takes some time, and you don't need it every time you boot your application.

忘れないで欲しいのは、`Bundler.require` でグループを指定するのではなく、通常の `require` を使うことも出来るということだ。ある1つの gem だけ require をやめたい時もあるし、毎起動時に必要としない gem もあるだろう。

Checking Your Code into Version Control
----

コードをバージョン管理する
----

After developing your application for a while, check in the application together with the Gemfile and Gemfile.lock snapshot. Now, your repository has a record of the exact versions of all of the gems that you used the last time you know for sure that the application worked. Keep in mind that while your Gemfile lists only three gems (with varying degrees of version strictness), your application depends on dozens of gems, once you take into consideration all of the implicit requirements of the gems you depend on.

開発を重ねたあとは、アプリケーションのコードと一緒に `Gemfile` と `Gemfile.lock` をチェックインしよう。そうすれば、最後にアプリケーションが正しく動いてた時に使っていた gem のバージョンがリポジトリに記録されることになる。例では `Gemfile` に3つの gem しか (場合によってバージョンが変化しうる指定方法で) 列挙されていないが、それらの gem が暗黙的に依存するものを含めれば、非常に多くの gem に依存していることは覚えておこう。

This is important: the Gemfile.lock makes your application a single package of both your own code and the third-party code it ran the last time you know for sure that everything worked. Specifying exact versions of the third-party code you depend on in your Gemfile would not provide the same guarantee, because gems usually declare a range of versions for their dependencies.

これは重要なことだ。`Gemfile.lock` はあなたが書いたコードとサードパーティのコードの両方合わせてうまく動いていた時の状態を一つのパッケージにしてくれる。`Gemfile` でサードパーティのコードのバージョンを厳密に指定したとしても、同じ保証は得られない。なぜならば、そのサードパーティの gem 自体の依存関係の宣言には、バージョンの範囲が指定されていることが多いからだ。

The next time you run bundle install on the same machine, bundler will see that it already has all of the dependencies you need, and skip the installation process.

次に同じマシンで `bundle install` を実行したとき、Bundler は依存する gem が既に存在することを確認し、インストールプロセスをスキップする。

Do not check in the .bundle directory, or any of the files inside it. Those files are specific to each particular machine, and are used to persist installation options between runs of the bundle install command.

`.bundle` ディレクトリや、その中のファイルはチェックインしてはいけない。これらのファイルは各マシンにより特有の内容になる。各環境での `bundle install` 時のオプションの違いにもよるだろう。

If you have run bundle pack, the gems (although not the git gems) required by your bundle will be downloaded into vendor/cache. Bundler can run without connecting to the internet (or the Rubygems server) if all the gems you need are present in that folder and checked in to your source control. This is an optional step, and not recommended, due to the increase in size of your source control repository.

`bundle pack` を実行すれば、必要とされる gem (git リポジトリを直接指定した gem は除く) が `vendor/cache` ディレクトリにダウンロードされる。必要な gem がそのディレクトリに存在していて、リポジトリにチェックインされていれば、インターネット(あるいは Rubygems サーバ)に接続しなくても Bundler が実行できる。リポジトリのサイズが増加してしまうので、これは必須のステップというわけではない。

Sharing Your Application With Other Developers
----

アプリケーションを他の開発者と共有する
----

When your co-developers (or you on another machine) check out your code, it will come with the exact versions of all the third-party code your application used on the machine that you last developed on (in the Gemfile.lock). When **they** run bundle install, bundler will find the Gemfile.lock and skip the dependency resolution step. Instead, it will install all of the same gems that you used on the original machine.

共同開発者(あるいは、自分の別マシン)がコードをチェックアウトする段階の話をしよう。この時、最後に開発していたマシンで使われていたサードパーティの gem のその時のバージョン(`Gemfile.lock` に書かれているものだ)を持ってくることが可能だ。共同開発者あるいは自分の別マシン(とにかく、先程 `bundle install` したのとは別マシン)で `bundle install` を実行すると Bundler は `Gemfile.lock` を見て、依存関係を解決する処理をスキップする。代わりに、元のマシンで使われているものと同じ gem をインストールする。

In other words, you don't have to guess which versions of the dependencies you should install. In the example we've been using, even though rack-cache declares a dependency on rack >= 0.4, we know for sure it works with rack 1.2.1. Even if the Rack team releases rack 1.2.2, bundler will always install 1.2.1, the exact version of the gem that we know works. This relieves a large maintenance burden from application developers, because all machines always run the exact same third-party code.

言い換えると、どのバージョンの依存 gem をインストールすべきかで悩む必要はないということだ。先ほどの例では、`rack-cache` 自体の依存関係には `rack >= 0.4` と書かれているが、`rack 1.2.1` を使えば動くことはわかっている。Rack 開発陣が `rack 1.2.2` をリリースしたとしても、Bundler は `1.2.1` を常にインストールしてくれる。これによりアプリケーション開発者の労力が大きく削減されるだろう。どのマシンでもサードパーティのコードは全て同一になっているのだから。

Updating a Dependency
----

依存関係の更新
----

Of course, at some point, you might want to update the version of a particular dependency your application relies on. For instance, you might want to update rails to 3.0.0 final. Importantly, just because you're updating one dependency, it doesn't mean you want to re-resolve all of your dependencies and use the latest version of everything. In our example, you only have three dependencies, but even in this case, updating everything can cause complications.

これで話は終わりではない。ある依存 gem をバージョンアップしたいと思うことが出てくるはずだ。例えば、`rails` を `3.0.0` final にアップデートしたいとしよう。重要なのは、一つの依存 gem だけをアップデートしたいのであって、依存関係の解決をやり直して全ての gem を最新版にしたいわけじゃないという事だ。引き続き使っている例では、依存 gem が3つだけだが、このケースでは全てをアップデートさせると複雑な事態を引き起こしかねない。

To illustrate, the rails 3.0.0.rc gem depends on actionpack 3.0.0.rc gem, which depends on rack ~> 1.2.1 (which means >= 1.2.1 and < 1.3.0). The rack-cache gem depends on rack >= 0.4. Let's assume that the rails 3.0.0 final gem also depends on rack ~> 1.2.1, and that since the release of rails 3.0.0, the Rack team released rack 1.2.2.

この件を説明するために、以下を考えてみよう。`rails 3.0.0.rc` は `actionpack 3.0.0.rc` に依存しており、その actionpack 3.0.0.rc は `rack ~> 1.2.1` という依存関係を持っている(これは `1.2.1` 以上 `1.3.0` 未満を意味する)。`rack-cache` は `rack >= 0.4` という依存関係を持つ。さらに仮定として、`rails 3.0.0` final も `rack ~> 1.2.1` という依存関係を持ち、`rails 3.0.0` がリリースされてからどこかの時点で Rack 開発陣は `rack 1.2.2` をリリースしたとする。

If we naively update all of our gems in order to update Rails, we'll get rack 1.2.2, which satisfies the requirements of both rails 3.0.0 and rack-cache. However, we didn't specifically ask to update rack-cache, which may not be compatible with rack 1.2.2 (for whatever reason). And while an update from rack 1.2.1 to rack 1.2.2 probably won't break anything, similar scenarios can happen that involve much larger jumps. (see [1] below for a larger discussion)

Rails をアップデートするために単純に全ての gem をアップデートすると、`rails 3.0.0` と `rack-cache` の要求を満たす `rack 1.2.2` がインストールされる。しかし、`rack-cache` は(なんらかの理由で) `rack 1.2.2` とは互換性がないかも知れない。アップデートして良いかを `rack-cache` に明確に尋ねたわけではないのだ。もっとも、`rack` を `1.2.1` から `1.2.2` にアップデートしても何も壊れることは多分ないだろうが、似たシナリオでバージョンがもっと大きく飛ぶような場合は問題が起こるかも知れない(それについてはページ下の [1] でスペースを取って検討している)。

In order to avoid this problem, when you update a gem, bundler will not update a dependency of that gem if another gem still depends on it. In this example, since rack-cache still depends on rack, bundler will not update the rack gem. This ensures that updating rails doesn't inadvertently break rack-cache. Since rails 3.0.0's dependency actionpack 3.0.0 remains compatible with rack 1.2.1, bundler leaves it alone, and rack-cache continues to work even in the face of an incompatibility with rack 1.2.2.

この問題を避けるために、Bundler は「他の gem から依存されている gem はアップデートしない」という策を取る。今の例で言えば、`rack-cache` が `rack` に依存しているので、Bundler は `rack` をアップデートしない。これにより、`rails` をアップデートすることで不用意に `rack-cache` を壊してしまうことがなくなる。`rails 3.0.0` が依存する `actionpack 3.0.0` は `rack 1.2.1` でも依存関係を満たすので、Bundler は問題なしと判断する。これで、`rack-cache` が仮に `rack 1.2.2` と非互換だったとしても罠にハマらずに済む。

Since you originally declared a dependency on rails 3.0.0.rc, if you want to update to rails 3.0.0, simply update your Gemfile to gem 'rails', '3.0.0' and run:

元々の例では依存関係に `rails 3.0.0.rc` と書いていた。これを `rails 3.0.0` にアップデートしたいなら、単に `Gemfile` の内容を `gem 'rails', '3.0.0'` にしてから以下を実行する。

    $ bundle install

As described above, the bundle install command always does a conservative update, refusing to update gems (or their dependencies) that you have not explicitly changed in the Gemfile. This means that if you do not modify rack-cache in your Gemfile, bundler will treat it **and its dependencies** (rack) as a single, unmodifiable unit. If rails 3.0.0 was incompatible with rack-cache, bundler will report a conflict between your snapshotted dependencies (Gemfile.lock) and your updated Gemfile.

上で説明した通り、`bundle install` コマンドは常に保守的なアップデートを心がけるので、`Gemfile` で明示していない gem のアップデートは拒否される。つまり、`Gemfile` で `rack-cache` に関していじっていないなら、Bundler は 「`rack-cache` 自身とその依存する gem (`rack`)」を変更できない一つの塊として扱う。もし `rails 3.0.0` が `rack-cache` と一緒には使えない(依存関係に矛盾がある)と分かれば、Bundler は依存関係のスナップショット(`Gemfile.lock`)とアップデートした `Gemfile` の間にコンフリクトがあることを報告する。

If you update your Gemfile, and your system already has all of the needed dependencies, bundler will transparently update the Gemfile.lock when you boot your application. For instance, if you add mysql to your Gemfile, and have already installed it in your system, you can boot your application without running bundle install, and bundler will persist the "last known good" configuration to the Gemfile.lock snapshot.

`Gemfile` をアップデートしたが既に依存関係が全て満たされていた場合は、次にアプリケーションを起動した時に Bundler は `Gemfile.lock` を静かに書き換える。例えば `Gemfile` に `mysql` を追加し、それが既にシステムにインストールされていたとする。その場合は `bundle install` をせずにアプリケーションを起動でき、Bundler は「最後にうまくいっていたもの」として `Gemfile.lock` にスナップショットを残す。

This can come in handy when adding or updating gems with minimal dependencies (database drivers, wirble, ruby-debug). It will probably fail if you update gems with significant dependencies (rails), or that a lot of gems depend on (rack). If a transparent update fails, your application will fail to boot, and bundler will print out an error instructing you to run bundle install.

これは、最低限必要な gem(データベースドライバ、`wirble`、`ruby-debug` 等)が追加／更新される時に便利だ。そういったものは、たくさんの依存 gem を持つもの(`rails`)やたくさんの gem から依存されているもの(`rack`)をアップデートした時に失敗しやすい。もし静かなアップデートが失敗するなら、アプリケーションの起動も失敗し、Bundler は `bundle install` を実行せよと出力してくる。

Updating a Gem Without Modifying the Gemfile
----

Gemfile を変更せずに gem をアップデートする
----

Sometimes, you want to update a dependency without modifying the Gemfile. For example, you might want to update to the latest version of rack-cache. Because you did not declare a specific version of rack-cache in the Gemfile, you might want to periodically get the latest version of rack-cache. To do this, you want to use the bundle update command:

場合によっては、Gemfile を変更せずに依存 gem をアップデートしたいこともあるだろう。例えば `rack-cache` の最新版を使いたいといった場合だ。`Gemfile` には `rack-cache` のバージョンを明示していないので、`rack-cache` を定期的に最新版にしたいつもりだ。そうであるなら、`bundle update` コマンドを使おう。

    $ bundle update rack-cache

This command will update rack-cache and its dependencies to the latest version allowed by the Gemfile (in this case, the latest version available). It will not modify any other dependencies.

上記コマンドで、`rack-cache` の依存関係は `Gemfile` が許す限り新しいものにアップデートされる(この例では、実際に最新版が使用される)。それ以外の依存 gem は変更されない。

It will, however, update dependencies of other gems if necessary. For instance, if the latest version of rack-cache specifies a dependency on rack >= 1.2.2, bundler will update rack to 1.2.2 even though you have not asked bundler to update rack. If bundler needs to update a gem that another gem depends on, it will let you know after the update has completed.

だがしかし、必要に応じて他の gem がアップデートされることもある。例えば、最新版の `rack-cache` には `rack >= 1.2.2` という依存関係が指定されているとしたら、`rack` をアップデートしてくれと言わなくても Bundler は `rack` を `1.2.2` へアップデートする。もし他の gem から依存されている gem をアップデートせざるを得ない事になったら、それはアップデートが完了した後に知らされる。

If you want to update every gem in the Gemfile to the latest possible versions, run:

Gemfile に載っている全ての gem を可能な限り最新版にしたい場合は、以下を実行する。

    $ bundle update

This will resolve dependencies from scratch, ignoring the Gemfile.lock. If you do this, keep git reset --hard and your test suite in your back pocket. Resolving all dependencies from scratch can have surprising results, especially if a number of the third-party packages you depend on have released new versions since you last did a full update.

これは `Gemfile.lock` を無視して、依存関係の解決を全て最初からやり直す。これをする場合は、`git reset --hard` 等の元に戻す手段や、コードの動作を確認するテストスイートを準備しておこう。依存関係の解決を最初から全部やり直すことは予期しない結果になり得る。特に、前回のフルアップデートから多くのサードパーティ製パッケージの新しいバージョンがリリースされているような時には。

Deploying Your Application
----

アプリケーションのデプロイ
----

When you run bundle install, bundler will (by default), install your gems to your system repository of gems. This means that they will show up in gem list. Additionally, if you are developing a number of applications, you will not need to download and install gems in common for each application. This is nice for development, but somewhat problematic for deployment.

`bundle install` を実行する時、デフォルトでは「システムの gem」へインストールされる。つまり、`bundle install` した gem は `gem list` でリストアップされるようになる。この場合、たくさんのアプリケーションを開発しているなら gem が共通で扱われるので再度ダウンロードしたりインストールしなくて済む。これは開発中は良いのだが、デプロイでは問題になることがある。

訳注：開発中の手元の環境でも、システムの gem に入れるのは嬉しくないという考え方もある。bundle install 時にプロジェクトのディレクトリに gem をインストールしたいなら、bundle install --path vendor/bundle のようにする。--path には vendor/bundle ディレクトリが指定されることが多い(慣習)。このディレクトリはリポジトリにチェックインしてはいけない。

In a deployment scenario, the Unix user you deploy with may not have access to install gems to a system location. Even if the user does (or you use sudo), the user that boots the application may not have access to them. For instance, Passenger runs its Ruby subprocesses with the user nobody, a somewhat restricted user. The tradeoffs in a deployment environment lean more heavily in favor of isolation (even at the cost of a somewhat slower deploy-time bundle install when some third-party dependencies have changed).

デプロイ時にはシステムの gem 置き場へのアクセス権がないユーザでやらなければならない時もある。仮にそれが出来るユーザ(あるいは `sudo` 等)にてデプロイしたとしても、アプリケーションを起動するユーザがアクセス権を持たないかもしれない。例えば Passenger は `nobody` ユーザで Ruby のサブプロセスを生成する。`nobody` は通常のユーザよりも権限がいくらか低い。デプロイ環境の「隔離か統合か」というトレードオフは、隔離の方向へ傾きがちだ(サードパーティの依存関係が変更された時に `bundle install` によりデプロイ時間が長くかかるとしてもだ)。

As a result, bundler comes with a --deployment flag that encapsulates the best practices for using bundler in a deployment environment. These practices are based on significant feedback we have received during the development of bundler, as well as a number of bug reports that mostly reflected a misunderstanding of how to best configure bundler for deployment. The --deployment flags adds the following defaults:

このため、Bundler にはデプロイ環境のための一連のベストプラクティスが `--deployment` フラグとして備わっている。これらのプラクティスは Bundler を開発している時にもらった多くのフィードバックや、デプロイ時の Bundler のベストな設定が分からずにバグとして報告されてきたものがベースとなっている。`--deployment` フラグは以下の動作を追加する。

* Instead of installing gems to the system location, bundler will install gems to vendor/bundle inside your application. Bundler will transparently remember this location when you invoke it inside your application (with Bundler.setup and Bundler.require).

* Bundler will not use gems already installed to your system, even if they exist.

* If you have run bundle pack, checked in the vendor/cache directory, and do not have any git gems, Bundler will not contact the internet while installing your bundle.

* Bundler will require a Gemfile.lock snapshot, and fail if you did not provide one.

* Bundler will not transparently update your Gemfile.lock if it is out of date with your Gemfile

* gem をシステムにインストールする代わりに、アプリケーション内の `vendor/bundle` ディレクトリにインストールする。Bundler はこの場所にインストールしたことを覚えておいてくれて、(`Bundler.setup` や `Bundler.require` が)そこを参照するようになる。

* システムに既にインストールされている gem があっても、それを使わない。

* `bundle pack` を実行して `vendor/cache` へダウンロードしており、それをチェックインしていた場合(つまりデプロイ先でもこのディレクトリがチェックアウトされ、存在している場合)で、かつ git リポジトリを指定した gem がない場合は、gem インストール時にインターネット接続をしない。

* Bundler は `Gemfile.lock` を必須のものとして要求する。なければ動作しない。

* `Gemfile.lock` が `Gemfile` より古くても、Gemfile.lock を静かにアップデートしない。

If you use Capistrano, you should symlink vendor/bundle to shared/vendor_bundle so that bundler will share your installed gems between deployments (making things zippy if you didn't make any changes), but still give you the benefits of isolation from other applications.

Capistrano を使っているなら、`vendor/bundle` から `shared/vendor_bundle` へシンボリックリンクを張るのを忘れずに。こうすると Bundler が各デプロイ同士で gem のインストールを共有でき(gem に何も変更がなければデプロイが高速化できる)、かつ他のアプリケーションから隔離されているという利点は守られる。

By defaulting the bundle directory to vendor/bundle, and installing your bundle as part of your deployment process, you can be sure that the same Unix user that checked out your application also installed the third-party code your application needs. This means that if Passenger (or Unicorn) can see your application, it can also see its dependencies.

デフォルトで gem を入れるディレクトリは `vendor/bundle` であり、そこに gem をインストールすることはデプロイプロセスの一部だ。そしてアプリケーションをチェックアウトしたユーザが、サードパーティーのコードもインストールすることになる。つまり、Passenger(あるいは Unicorn)からアプリケーションが見えるなら、その依存 gem まで見えることを意味する。

The --deployment flag requires an up-to-date Gemfile.lock to ensure that the testing you have done (in development and staging) actually reflects the code you put into production. You can run bundle check before deploying your application to make sure that your Gemfile.lock is up-to-date. Note that it will always be up-to-date if you have run bundle install, successfully booted your application (or run your tests) since the last time you changed your Gemfile.

`--deployment` フラグは `Gemfile.lock` が最新であることを要求する。(開発環境やステージング環境で)テストが完了して、プロダクションに置くべきコードになっているかを確実にするためだ。 また `bundle check` を使うことで、アプリケーションをデプロイする前に `Gemfile.lock` が最新かどうかを確かめることが出来る。最後に Gemfile をいじって bundle install をしてから、アプリケーションの起動(あるいはテストの実行)が成功していれば、Gemfile.lock は常に最新と言える。

FAQ: Why Can't I Just Specify Only = Dependencies?
----

FAQ: なぜ、`=` を使った依存関係の指定だけではダメなのか？
----

Q: I understand the value of locking my gems down to specific versions, but why can't I just specify = versions for all my dependencies in the Gemfile and forget about the Gemfile.lock?

Q: 依存 gem のバージョン指定の範囲を決めなければならないのはわかったが、`Gemfile` に書く依存関係を全て `=` で指定してしまうのではダメなのか？　そうしたら `Gemfile.lock` は不要な気がするが。

A: Many of your gems will have their own dependencies, and they are unlikely to specify = dependencies. Moreover, it is probably unwise for gems to lock down all of *their* dependencies so strictly. The Gemfile.lock allows you to specify the versions of the dependencies that your application needs in the Gemfile, while remembering all of the exact versions of third-party code that your application used when it last worked correctly.

A: あなたが使う gem の多くはそれ自身が依存関係を持っており、それらが `=` で指定されていることは少ない。また、それらのバージョン指定を厳しくすることはあまり賢明ではないだろう。`Gemfile.lock` は、アプリケーションが必要とする gem の特定のバージョンを指定させてくれるものであり、アプリケーションが最後にうまく動いていた時に使っていたサードパーティのコードのバージョンを記録してくれている。

By specifying looser dependencies in your Gemfile (such as nokogiri ~> 1.4.2), you gain the ability to run bundle update nokogiri, and let bundler handle updating **only** nokogiri and its dependencies to the latest version that still satisfied the ~> 1.4.2 version requirement. This also allows you to say "I want to use the current version of nokogiri" (gem 'nokogiri' in your Gemfile) without having to look up the exact version number, while still getting the benefits of ensuring that your application always runs with exactly the same versions of all third-party code.

`Gemfile` にゆるめのバージョン指定(`nokogiri ~> 1.4.2` のような)を使うことで、`bundle update nokogiri` を実行した時に `nokogiri` 「だけを」アップデートし、しかも `~> 1.4.2` という範囲に入る限りで最新版を入れるという操作が可能になる。また、わざわざバージョンを調べなくても「nokogiri の最新版を使いたい」という欲求を満たすこともできるし(`gem 'nokogiri'` を `Gemfile` に書く)、その場合でもサードパーティのコードは全て同じバージョンを確実に使うようにするという利点も損なわれていない。

FAQ: Why Can't I Just Submodule Everything?
----

FAQ: git submodule ではダメなのか？
----

Q: I don't understand why I need bundler to manage my gems in this manner. Why can't I just get the gems I need and stick them in submodules, then put each of the submodules on the load path?

Q: Bundler のやり方で gem を管理しなければならない理由がわからない。必要な gem を取得してそれを git submodule 等で固定し、それぞれの submodule をロードパスに置くのではダメなのか？

A: Unfortunately, that solution requires that you manually resolve all of the dependencies in your application, including dependencies of dependencies. And even once you do that successfully, you would need to redo that work if you wanted to update a particular gem. For instance, if you wanted to update the rails gem, you would need to find all of the gems that depended on dependencies of Rails (rack, erubis, i18n, tzinfo, etc.), and find new versions that satisfy the new versions of Rails' requirements.

A: 残念ながら、その方法では依存関係の解決を全て手でやらなければならない。しかも、依存する gem が依存する gem に関してもだ。またそれが成功したとしても、特定の gem をアップデートしたいと思っただけでまたやり直さなければならない。例えば `rails` をアップデートしたいとしたら、`rails` の依存 gem が依存する gem(`rack`, `erubis`, `i18n`, `tzinfo`等) を自分で探し、それぞれの依存関係を満たすような 新しいバージョンを調べないといけない。

Frankly, this is the sort of problem that computers are good at, and which you, a developer, should not need to spend time doing.

はっきり言ってこれはコンピュータに解かせるべき問題であり、開発者たるあなたが時間を費やすべきことではない。

More concerningly, if you made a mistake in the manual dependency resolution process, you would not get any feedback about conflicts between different dependencies, resulting in subtle runtime errors. For instance, if you accidentally stuck the wrong version of rack in a submodule, it would likely break at runtime, when Rails or another dependency tried to rely on a method that was not present.

もっと言うと、手で依存関係を解決する過程でミスをしてしまっても、依存関係同士の衝突に関してのフィードバックが何もないはずだ。その結果は、事前にわからないランタイムエラーだ。例えば submodule で間違ったバージョンの `rack` を固定してしまった場合に、Rails や他の gem がそのバージョンの rack には存在しないメソッドを呼んだとしたら、実行時エラーを起こしてしまう。

Bottom line: even though it might seem simpler at first glance, it is decidedly significantly more complex.

結論： 一見そっちの方が簡単に見えるのだが、間違いなく大幅に複雑化する。

FAQ: Why Is Bundler Downloading Gems From --without Groups?
----

FAQ: `--without` で指定したグループの gem も Bundler がダウンロードするのは何故？
----

Q: I ran bundle install --without production and bundler is still downloading the gems in the :production group. Why?

Q: `bundle install --without production` を実行したのだが、Bundler は `:production` グループの gem もダウンロードしているようだ。なぜ？

A: Bundler's Gemfile.lock has to contain exact versions of all dependencies in your Gemfile, regardless of any options you pass in. If it did not, deploying your application to production might change all your dependencies, eliminating the benefit of Bundler. You could no longer be sure that your application uses the same gems in production that you used to develop and test with. Additionally, adding a dependency in production might result in an application that is impossible to deploy.

A: Bundler の `Gemfile.lock` は `Gemfile` に書かれた全ての依存 gem の特定のバージョンを含んでいなければならない。あなたがどんなオプションを渡したとしてもだ。でなければ、本番環境にアプリケーションをデプロイするだけで依存関係が変化してしまい、Bundler の利点がなくなってしまう。もはや本番環境のアプリケーションが開発時やテスト時に使っていたのと同じ gem を使っているかどうかは保証できない。さらに言えば、production グループに依存 gem を追加すると、そのアプリケーションはデプロイ不可能になってしまうかもしれない。

For instance, imagine you have a production-only gem (let's call it rack-debugging) that depends on rack =1.1. If we did not evaluate the production group when you ran bundle install --without production, you would deploy your application, only to receive an error that rack-debugging conflicted with rails (which depends on actionpack, which depends on rack ~> 1.2.1).

例えば、`rack =1.1` という依存指定の、本番環境のみで使う gem (`rack-debugging` としよう) があると想像してみて欲しい。`bundle install --without production` を実行した時に production グループが評価されないとすると、そのアプリケーションをデプロイした時に「`rack-debugging` が `rails`( `rack ~> 1.2.1` という依存関係を持つ `actionpack` に依存している) と衝突しています」というエラーが帰ってくるだけだ。

Another example: imagine a simple Rack application that has gem 'rack' in the Gemfile. Again, imagine that you put rack-debugging in the :production group. If we did not evaluate the :production group when you installed via bundle install --without production, your app would use rack 1.2.1 in development, and you would learn, at deployment time, that rack-debugging conflicts with the version of Rack that you tested with.

別の例を挙げよう。`Gemfile` にただ `gem 'rack'` とだけ書いているシンプルな Rack アプリケーションを考えてみる。そして再び、`:production` グループに `rack-debugging` を書いたとしよう。`bundle install --without production` した時に `:production` グループが評価されないとすると、開発環境では `rack 1.2.1` が使われ、デプロイ時にはテストで使っていたバージョンの Rack と `rack-debugging` が衝突を起こしてしまう。

In contrast, by evaluating the gems in **all** groups when you call bundle install, regardless of the groups you actually want to use in that environment, we will discover the rack-debugger requirement, and install rack 1.1, which is also compatible with the gem 'rack' requirement in your Gemfile.

上記の例とは対照的に、実際に指定する環境と関係なく全てのグループの gem を評価することで、`rack-debugger` の依存関係を認識できる。つまり、`gem 'rack'` と書いた依存宣言を満たすように `rack 1.1` をインストールできる。

In short, by always evaluating all of the dependencies in your Gemfile, regardless of the dependencies you intend to use in a particular environment, you avoid nasty surprises when switching to a different set of groups in a different environment. And because we just download (but do not install) the gems, you won't have to worry about the possibility of a difficult **installation** process for a gem that you only use in production (or in development).

まとめると、特定の環境でどれを使いたいかという意図とは関係なく Gemfile にある全ての依存関係を常に評価することで、別の環境で別のグループの集合に切り替えた時に良くない驚きに遭遇するのを避けられる。そしてダウンロードだけをする(インストールはしない)ことで、本番環境だけ(あるいは開発環境だけ)で使う gem の難しい*インストール*プロセスが実現可能かどうかを心配しなくてよくなる。

FAQ: I Have a C Extension That Requires Special Flags to Install
----

FAQ: インストール時にフラグをつける必要がある C 拡張があるのだが。
----

Q: I have a C extension gem, such as mysql, which requires special flags in order to compile and install. How can I pass these flags into the installation process for those gems?

Q: `mysql` をはじめとする C 拡張 の gem があり、コンパイルしてインストールするためにフラグをつける必要がある。Bundler ではこれらの gem のインストールプロセスのどこでフラグを渡せばよいのか？

A: First of all, this problem does not exist for the mysql2 gem, which is a drop-in replacement for the mysql gem. In general, modern C extensions properly discover the needed headers.

A: まず第一に、`mysql` gem を置き換える `mysql2` gem ではこの問題は存在しない。モダンな C 拡張は通常、必要なヘッダを正しく見つけてくれる。

If you really need to pass flags to a C extension, you can use the bundle config command:

それでも C 拡張にフラグを渡さなければならないとしたら、`bundle config` コマンドをこう使う。

    $ bundle config build.mysql --with-mysql-config=/usr/local/mysql/bin/mysql_config

Bundler will store this configuration in ~/.bundle/config, and bundler will use the configuration for any bundle install performed by the same user. As a result, once you specify the necessary build flags for a gem, you can successfully install that gem as many times as necessary.

Bundler はこの設定を `~/.bundle/config` に保存し、同じユーザが `bundle install` を実行する時はこの設定を使ってくれる。よって、ある gem に必要なビルドオプションを一旦指定すれば、あとはもう心配をせずに済む。


Summary
----

要約
----

A Simple Bunler Workflow
----

Bundler の基本的なワークフロー
----

When you first create a Rails application, it already comes with a Gemfile. For another kind of application (such as Sinatra), run:

Rails アプリケーションを作り始める時、`Gemfile` は既についてくる。他の種類のアプリケーション(Sinatra のような)を作る問いは、以下を実行する。

    $ bundle init

The bundle init command creates a simple Gemfile which you can edit.

`bundle init` コマンドはシンプルな `Gemfile` を作成してくれる。

Next, add any gems that your application depends on. If you care which version of a particular gem that you need, be sure to include an appropriate version restriction:

次に、アプリケーションが依存する gem を Gemfile に追加していく。使いたい gem のバージョンについて考えていることがあるなら、そこに正しいバージョン指定を含めるようにする。

    source 'http://rubygems.org'

    gem 'sinatra', '~> 0.9.0'
    gem 'rack-cache'
    gem 'rack-bug'

If you don't have the gems installed in your system yet, run:

gem がまだシステムにインストールされていないなら、以下を実行する。

    $ bundle install

To update a gem's version requirements, first modify the Gemfile:

gem のバージョン要求を更新したいなら、まず Gemfile を編集する。

    source 'http://rubygems.org'

    gem 'sinatra', '~> 1.0.0'
    gem 'rack-cache'
    gem 'rack-bug'

and then run:

それから以下を実行。

    $ bundle install

If bundle install reports a conflict between your Gemfile and Gemfile.lock, run:

もし `bundle install` が `Gemfile` と `Gemfile.lock` の間に衝突があると報告してきたら、次のようにする。

    $ bundle update sinatra

This will update just the Sinatra gem, as well as any of its dependencies

これは Sinatra gem と、その依存 gem だけをアップデートしてくれる。

To update all of the gems in your Gemfile to the latest possible versions, run:

`Gemfile` にある gem を全て可能な限り最新にしたいなら、以下を実行する。

    $ bundle update

Whenever your Gemfile.lock changes, always check it in to version control. It keeps a history of the exact versions of all third-party code that you used to successfully run your application.

`Gemfile.lock` が変更されたときはいつでも、リポジトリにそれをチェックインしよう。アプリケーションがうまく動いていたときに使っていたサードパーティコードのバージョンを、履歴に含めるんだ。

When deploying your code to a staging or production server, first run your tests (or boot your local development server), make sure you have checked in your Gemfile.lock to version control. On the remote server, run:

ステージングや本番サーバにコードをデプロイする時は、まずテストを走らせ(あるいはローカルの開発サーバで走らせる)、リポジトリに `Gemfile.lock` を間違いなくチェックインさせておこう。それからリモートサーバ上で、以下を実行する。

    $ bundle install --deployment


Notes
----

注釈
----

[1] For instance, if rails 3.0.0 depended on rack 2.0, that gem would still satisfy the requirement of rack-cache, which declares >= 1.0 as a dependency. Of course, you could argue that rack-cache is silly for depending on open-ended versions, but these situations exist (extensively) in the wild, and projects often find themselves between a rock and a hard place when deciding what version to depend on. Constrain the dependency too much (rack =1.2.1) and you make it hard to use your project in other compatible projects. Constrain it too little (rack >= 1.0) and a new release of Rack may break your code. Using dependencies like rack ~> 1.2.1 and versioning code in a SemVer compliant way mostly solves this problem, but it assumes universal compliance. Since Rubygems has over 100,000 packages, this assumption simply doesn't hold in practice.

[1] 例えば、`rails 3.0.0` が `rack 2.0` に依存しているとする。この場合でも `rack-cache` の要求する `rack >= 1.0` を満たしてしまっている。無論、`rack-cache` のバージョン指定方法がアホなだけだ、と言うことだって出来る。だが、プロジェクトが依存するバージョンをどうするかでジレンマが起こるのはよくある話で、こういった状況は現実に(広範囲で)存在している。依存関係を厳しくしすぎる(`rack =1.2.1`)とそのプロジェクトを他のプロジェクトで使うのが難しくなる。依存関係を緩くしすぎる(`rack >= 1.0`)と新しい Rack がリリースされた時に自分のコードが壊れてしまうかもしれない。`rack ~> 1.2.1` のような依存指定を使いつつ、[Semantic Versioning](http://semver.org/) に従うことでほぼこの問題を解決できるが、それは漏れがあっては意味の無い話だ。Rubygems は10万個以上あるのだから、この仮定を置くことは現実には難しい。




