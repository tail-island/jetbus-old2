「Androidアプリの開発は、地球が砕け散ってもイヤです」

上司の問い掛けにノータイムで断りをいれて、でもしがないサラリーマンの私の我儘が通るはずはなくて、Androidアプリの開発があまりに辛くて滂沱のごとく涙した数年前の私。私の立場はあの時と同じヒラのままですし、やっぱり上司の命令には逆らえないですけど、Androidアプリの開発は大きく変わりました。

「Androidアプリの開発を、ぜひワタクシメに！」

今の私は、Androidアプリの開発の仕事大歓迎です。だって、今時のAndroidアプリ開発って、とても楽チンで美味しい仕事なんだもの。そう、Android Jetpackを使うならね。あと、Kotlinとコルーチンも。……え？　信じられない？　わかりました。実際にアプリを作って、その楽チンさを証明しましょう。

# Androidアプリ開発が面倒だった理由と、その対策

さて、Androidアプリの開発が面倒なのは、大昔の貧弱なスマートフォンのリソースでも動作するように初期のバージョンが作成されたためだと考えます。そして、複数のメーカーの様々なAndroid端末があったので後方互換性を考慮しなければならなくて、初期バージョンの古い設計を引きずらざるを得なかったためでしょう。

たとえば、Androidアプリの基本的な構成単位である`Activity`は、いつ終了させられてもよいように作らなければなりません。少ないメモリを有効活用するためには、バックグラウンドの`Activity`を終了させてメモリを解放させることが重要ですから。でも、だからといって、ただ終了させるのでは次にアプリがフォアグラウンドになった時に処理を継続できなくなってしまう。だから、`onSavedInstanceState()`で状態を退避できて`onRestoreInstanceState()`で状態を復帰できるようにしよう。待てよ、これが前提なのだから、端末を縦から横に回転したときに画面のレイアウトをやり直すようなCPUを消費する処理はやめて、`Activity`を再生成してしまうことにしよう。このような流れで出来上がったのが、`Activity`のライフサイクルという、あの複雑怪奇な図です。

図

この面倒な図を正確に理解していないと、Androidアプリの開発はできないらしい。でも、メモリが豊富な今時のスマートフォンなら、こんな仕組みは無駄なのではないでしょうか？　LinuxでもMac OSでもWindowsでも、こんな考慮はしていませんよね？

まさにその通り。でも、後方互換性を考えると、APIを丸ごと書き換えるのは困難です。だから、APIの上にライブラリの層を被せることにしましょう。先ほどの問題は`Activity`そのものではなく、`Activity`の「状態」の問題なので、状態だけを抽出して、その状態については`Activity`が破棄されても破棄されないようにすればよい。そのクラスの名前はそう、MVVM（Model View ViewModel）アーキテクチャの`ViewModel`から借りてこよう。そして、プログラミング業界ではオブジェクトの生成から破棄までをライフサイクルという言葉で表現するので、ライブラリ名はLifecycleとしよう。ほら、これで、あの複雑怪奇なライフサイクルを（それほど）意識しないで済むようになって、開発は楽チンになりました。

でもちょっと待って。画面回転の問題は`Fragment`で解決できるのではという、従来のAndroid開発に詳しい方がいらっしゃるかもしれません。でもね、`Fragment`を使う時って、画面遷移はどうしていました？　Androidそのものは`Activity`が遷移していくように作られていて、だから`Fragment`を含む`Activity`が遷移していくように作ることになって、結果として`Activity`と`Fragment`にコードが分散して見通しが悪くなるだけになっていませんでしたか？　この問題は、`Fragment`が遷移する仕組みを導入することで解消できます。その仕組みをシンプルなコードで実現するライブラリがNavigationです。これから先の人生では、`Fragment`のことだけを考えれば済むというわけ。気難しいActivityさんと離れられて、さらに楽チンです。

データの管理も大変でした。RDBMS（Relational Database Management System）のSQLiteをAndroidシステムに組み込んでくれたのはとてもありがたいのですけど、そのアクセスには素のSQLを使えってのはいただけません。オブジェクト指向のObjectとRDBMSのRelationをマップする、O/Rマッパーが欲しい。ならば与えようってことでRoomという公式のO/Rマッパーが提供されて、ライブラリ選択に頭を悩ます必要がなくなって楽チン。

本項では紹介しませんけど、これ以外にも便利機能はいっぱいあって、Googleは[Android Jetpack](https://developer.android.com/jetpack)としてまとめて提供しています。バックグラウンド・ジョブ管理のWorkManagerとか、面倒でしょうがなかったカメラ制御を楽にしてくれるCameraXなんてのもあります。新機能をなかなか取り入れられない、過去に縛られたダサいAPIとはオサラバできるんです。

あと、Support Libraryも。Androidでは端末が最新OSにバージョン・アップされていることを期待できないので、OSそのものに新しい機能が組み入れられてもすぐに使うことはできません。だから、Support Libraryという形で、OSとは別に機能を提供してきました。ところが、このSupport Libraryにもバージョンの不整合という問題がでてしまったんです。`android.support.v4`とか`android.support.v7`とかね。これはもう心機一転してやりなおそうってことで、`androidx`で始まる新しいライブラリ群にまとめなおされました。前述したLifecycleやNavigationも、実は`androidx`ファミリーの一員なんです。Lifecycleは`androidx.lifecycle`ですし、Navigationは`androidx.navigation`です。当然、新しい`androidx.*`では、LifecycleやNavigationなどのJetpackの機能を前提にした機能強化がなされていて、これでもう本当に楽チンです。

ただね、`androidx.*`は、それぞれがそれぞれを前提としているので、整合性を保って良い感じに組み合わせて使わなければならないんですよ。個々の要素を理解するだけでは不十分で、どのように組み合わせるべきかを理解することが重要なんです。だから、Googleは[アプリのアーキテクチャ・ガイド](https://developer.android.com/jetpack/docs/guide?hl=ja)というアプリ全体をどのように作るのかのガイドと、[Sunflower](https://github.com/googlesamples/android-sunflower)というサンプル・アプリを提供しています。これらはとても良いモノなのですけど、私には、Jetpackに都合が良い美しい世界にとどまっていて、現実の様々な問題への対応が不十分なように感じられます。あと、読者への前提が厳しすぎます。ガイドの序文に「Androidフレームワークの基本について熟知していることを前提としています」って書いてありますけど、Jetpack以降は無駄になるような古い知識をとりあえず習得してこいってのは酷いと思いませんか？

というわけで、もう少し現実的なサンプル・アプリと、ゆるめのガイドを作成してみました。……思いっきり蛇足な気がするけど、気のせいだよね？

# サンプル・アプリ

私が欲しいからという理由で、東京の都営バスの車両の接近情報を表示するアプリを作成します（私が生まれ育った群馬県で力を合わせている200万人をはじめとする地方在住のみなさま、ごめんなさい）。というのもですね、都営バスは[tobus.jp](https://tobus.jp/)というサイトで車両接近情報を提供していて、このサイトはグラフィカルでとてもよくできているんですけど、提供側の都営バスの論理からか情報が路線単位なんですよ。で、私の住んでいるアパートの周りにはいくつかのバス停があってそれぞれ路線が異なるので、会社から帰るときには複数の路線の車両接近情報を調べなければならなくてこれが実にかったるい。

図

なので、複数の路線の車両の接近情報を集めて表示するアプリを作成します。タイミングがよいことに、2019年5月31日に、[都営交通の運行情報等をオープンデータとして提供開始します](https://metro.tokyo.jp.tosei/hodohappyo/press/2019/05/31/04.html)というニュースがありました。[公共交通オープンデータセンター開発者サイト](https://developer.odpt.org)でユーザー登録すれば、誰でも無料でデータを使用できます（利用者への通知が必要などの、いくつかの制約はありますけど）。Webサイトのクローリングが不要になってアプリが簡単になりますから、サンプル・アプリに適しているんじゃないかな。

具体化しましょう。出発バス停を指定して、到着バス停を指定すると、それらのバス停を結ぶ全ての路線の接近情報が一覧表示されます。あと、毎回出発バス停と到着バス停を選ぶのでは面倒すぎるので、ブックマークの機能を持たせます。以下のような感じ。

図

本ガイドでは、このアプリをちゃちゃっと作っていきます。

最終的なコードは、[GitHub](https://github.com/tail-island/jetbus/)で公開しています。必要に応じて参照してみてください。ただし、このコードをビルドして動かすには、公共交通オープンデータセンター開発者サイトにユーザー登録すると貰えるアクセス・トークンをが必要です。コードをcloneしたら`./app/src/main/res/values`ディレクトリに`odpt.xml`を作成し、以下を参考に`consumerKey`を設定してください。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="consumerKey">公共交通オープンデータセンターから取得したアクセス・トークン</string>
</resources>
~~~

あと、Android Studioのバージョンは3.5を使用しました。これより古い場合は、Android Studioをバージョンアップしてください。

# プロジェクトの作成

長い前置きがやっと終わって早速プログラミング……の前に、Androidアプリの開発ではプロジェクトを作成しなければなりません。Android Studioを起動して、プロジェクトを作成しましょう。

Android Studioを起動して、[Start a new Android Studio project]を選択します。

図

作成するプロジェクトは、余計なコードが生成されない「Empty Activity」にします。

図

[Name]にアプリ名、[Package name]にドメイン名＋アプリ名を入力します。アプリ名は、JetpackのサンプルでBusの接近情報ということで、jetbusにしてみました。あ、英語ダメダメで風呂おbathと乗り物のbusの区別がつかなかったわけじゃなくて、イタリーのミラノのJetbus社のファンなだけですよ（たぶん、Jetbusの創業者も、私と同じくらい英語ダメなんだと思う……）。[Language]は、もちろん「Kotlin」にします。Javaの256倍くらい良いプログラミング言語ですし、Google I/O 2019で「Kotlinファースト」が表明されましたし、Kotlinならさらに有効活用できるJetpackの機能もあるためです。あと、[Minimum API Level]は、「API 23: Android 6.0 (Mashmallow)」にしました。Android 6.0で権限確認の方法が変更になったので、これ以前のバージョンだと権限の確認の処理が面倒なためです（今回は関係ないけどね）。古い端末を平気で使う海外までを対象にしたアプリを作るならもっと古いバージョンにすべきでしょうけど、国内が対象なら、まぁ大丈夫じゃないかな。

図

ともあれ、これで無事にプロジェクトが作成されました。でも、まだJetpackが組み込まれていません。さっそく組み込む……前に、ビルド・システムの説明をさせてください。

## ビルド・システム

人類がウホウホ言いながら樹上で暮らしていた大昔、ライブラリの組み込みというのはインターネットからダウンロードしたファイルをプロジェクトのディレクトリにセーブするという作業でした。Windowsでアプリケーションをインストールするために、Webサイトからパッケージをダウンロードしてセットアップするのと似た感じ。

こんな面倒な作業はやってられませんから、Linuxでは`apt`とか`yum`とか、Mac OSでは`MacPorts`とか`HomeBrew`とかのパッケージ管理システムを使って、アプリケーションをセットアップします。たとえば、LinuxディストリビューションのUbuntuで今私が本稿の作成に使用しているEmacsをセットアップする場合は、ターミナルから`sudo apt install emacs`と入力するだけ。これだけで、パッケージ管理システムが全自動でEmacsをダウンロードし、セットアップしてくれます。AndroidのPlay Store、iPhoneのApp Storeと同じですな。

で、今時の開発では様々なライブラリを使用するのが当たり前で、ライブラリ毎にWebサイトを開いてダウンロードして解凍してセーブなんて作業はやってられませんから、ライブラリの組み込みにも自動化が必要でしょう。ライブラリは他のライブラリに依存していることが多くて、その依存関係を辿る作業を手動でやるなんてのは非現実的ですもんね。

あと、今時の開発だと、ライブラリを組み込み終わったあとのビルドもなかなかに複雑な作業となります。ビルドついでにテストしたいとか、事前にコード生成させたいとか。このように考えると、ライブラリの管理とビルドを自動化するシステムが必要で、これがいわゆるビルド・システムとなります。Android Studioは、このビルド・システムとしてGradleというソフトウェアを使用しているわけです。

## Jetpackの組み込み

Jetpackをビルド・システムのGradleに組み込む方法は、各ライブラリのリリース・ノートに書いてあります。たとえばNavigationなら、[https://developer.android.com/jetpack/androidx/releases/navigation](https://developer.android.com/jetpack/androidx/releases/navigation)です。基本はこのリリース・ノートの記載に従うのですけど、いくつか注意点があります。

* ライブラリ名-ktxという名前のライブラリがある場合は、¥*-ktxを指定してください。¥*-ktxは、Kotlinならではの便利機能が入ったライブラリです。Kotlin向けの機能が入っていない基本バージョンのライブラリは、依存関係があるので自動で組み込まれます。
* 当面は、ベータ版やアルファ版まで含めたできるだけ新しいバージョンを指定してください。どんなバージョンがあるかは、[Google's Maven Repository](https://dl.google.com/dl/android/maven2/index.html)で調べることができます。ベータやアルファのバージョンを指定しないと、Jetpackのドキュメントに記載されているレベルのコードですらビルドできないことがありました。そのうち開発が落ち着いたらベータやアルファを使わないで済むようになるでしょうけど、今（2019年8月）はまだダメっぽい。
* Jetpackではコード生成を多用しているのですけど、KotlinやJavaではコード生成の制御にアノテーション（annotation。クラスやメソッドの前に書く`@Foo`みたいなアレ）を使用していて、Javaの場合はGradleのビルド・スクリプトに`annotationProcessor`と書きます。Kotlinの場合は、Kotlin Annotation Processor Toolの略で`kapt`と書いてください。リリース・ノートに`annotationProcessor`の記述があったあとに、Kotlinではkaptを使ってねという知っている人しかわからない意味不明なコメントが書いてある場合は、`kapt`の出番となります。

と、以上の注意を踏まえて、Android Studioの[Project]ビューのbuild.gradeをダブル・クリックして開いて、修正してみましょう。

図

で、以下が修正した結果。修正した行は、修正内容をコメントで書いています。

~~~ gradle:build.gradle(Project:jetbus)
buildscript {
    ext.kotlin_version = '1.3.50'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        classpath 'androidx.navigation:navigation-safe-args-gradle-plugin:2.2.0-alpha01'  // 追加
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
~~~

~~~ gradle:build.gradle(Module:app)
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'  // 追加
apply plugin: 'androidx.navigation.safeargs.kotlin'  // 追加

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "com.tail_island.jetbus"
        minSdkVersion 23
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    dataBinding {  // 追加
        enabled true  // 追加
    }  // 追加
}

dependencies {
    kapt 'androidx.lifecycle:lifecycle-compiler:2.2.0-alpha03'  // 追加
    kapt 'androidx.room:room-compiler:2.2.0-alpha02'  // 追加

    implementation fileTree(dir: 'libs', include: ['*.jar'])

    implementation 'androidx.appcompat:appcompat:1.1.0-rc01'  // バージョン変更
    implementation 'androidx.constraintlayout:constraintlayout:2.0.0-beta2'  // バージョン変更
    implementation 'androidx.core:core-ktx:1.2.0-alpha03'  // バージョン変更
    implementation 'androidx.fragment:fragment-ktx:1.2.0-alpha02'  // 追加
    implementation 'androidx.legacy:legacy-support-v4:1.0.0'  // 追加
    implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0-alpha03'  // 追加
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0-alpha03'  // 追加
    implementation 'androidx.navigation:navigation-fragment-ktx:2.2.0-alpha01'  // 追加
    implementation 'androidx.navigation:navigation-ui-ktx:2.2.0-alpha01'  // 追加
    implementation 'androidx.room:room-ktx:2.2.0-beta01'  // 追加
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"

    testImplementation 'junit:junit:4.12'

    androidTestImplementation 'androidx.test:runner:1.2.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
}
~~~

これで、デモ・アプリに必要なJetpackのライブラリの組み込みが完了しました。やっとプログラミングです。まず最初に、作成してやったた感が大きそうな、画面まわりをやってみましょう。

# Navigation

まぁ、画面といっても、かっこいい画面を作るほうではなく、ダミー画面を使用した画面遷移の実装という、見栄えがあまりよくない部分のプログラミングだけどな。Navigationが火を吹きますよ。

## `Activity`と`Fragment`とNavigation

さて、例によって歴史の話から。`Acticity`と`Fragment`の話、あと、Navigationが作られた経緯です。

`Activity`ってのは、ぶっちゃけアプリの画面1つ分です。で、一般にアプリは複数の`Activity`で構成されます。Androidはアプリ間連携（アプリを操作していたら別のアプリの画面に遷移して、で、戻るボタンで最初のアプリに戻れる）ができるところがとても素晴らしいと私は思っているのですけど、この機能は`Activity`の遷移として実現されています。同様に、アプリ内でも`Activity`の遷移で画面遷移を実現していました。

で、大昔のスマートフォンのアプリの単純な画面だったらこれであまり問題なかったんですけど、高機能なアプリを作ったりタブレットのような大きな画面を効率よく使おうとしたりする場合は、この方式だとコードの重複という問題が発生してしまうんです。タブレットの大きな画面では、左に一覧表示して、右にその詳細を表示するような画面が考えられます。でも、スマートフォンの小さな画面では、一覧表示する画面と、それとは別の詳細を表示する画面に分かれて、画面遷移する形で表現しなければなりません。

図

これを`Activity`で実現しようとすると、タブレット用の`Activity`を1つと、スマートフォン用の`Acticvity`を2つ作らなければなりません。そして、ほとんどのコードは重複してしまうでしょう。UIの問題なら`View`（UIコンポーネント）で解決すれば……って思うかもしれませんけど、画面に表示するデータをデータベースから取得してくるような機能を`View`に持たせるのは、`View`の責任範囲を逸脱しているのでダメです。

ではどうすればよいかというと、`Activity`の構成要素になりえる「何か」を追加してあげればよいわけ。この「何か」こそが、`Fragment`なのです。

でもね、コードの重複が発生しないようなケースで`Fragment`を使って`Activity`で画面遷移をさせると、`Activity`のコードの多くを`Fragment`に移して、で、`Activity`に`Fragment`を管理するコードを追加して、そしてもちろん`Fragment`にも自分自身の初期化処理等が必要となって、結局、コード量が増えただけで誰も得しないという状況になってしまうんです。

これでは無意味なので、1つの`Activity`の中で、複数の`Fragment`が遷移するというプログラミング・スタイルが編み出されました。このスタイルを実現するのが`FragmentTransaction`というAPIで、`Acticity`から`Fragment`を削除したり追加したり入れ替えたりできます。なんと[戻る]ボタンへの対応機能付き……なのですけど、高機能な分だけ使い方が複雑で、`Activity`の遷移（`Intent`のインスタンスを引数に`startActivity()`するだけ）と比べると面倒だったんです。

なので、Navigationが作成されました。GUIツール（私はほとんど使わないけど）で画面遷移を定義できるかっこいいライブラリです。ただ単に`FragmentTransaction`を呼び出しているだけのような気もしますけど、まぁ、いろいろ楽チンなのでよし。SafeArgsという便利機能もありますしね。

## プロジェクトにFragmentを追加する

以上により、画面遷移は`Fragment`遷移ということになりました。だから、プロジェクトに`Activity`ではなく`Fragment`を追加しましょう。jetbusに必要な`Fragment`は、以下の5つとなります。

* SplashFragment（起動処理をする間に表示するスプラッシュ画面）
* BookmarksFragment（ブックマークの一覧を表示する画面）
* DepartureBusStopFragment（出発バス停を指定する画面）
* ArrivalBusStopFragment（到着バス停を指定する画面）
* BusApproachesFragment（バスの接近情報を表示する画面）

さっそく追加します。Android Studioのメニューから、[New] - [Fragment] - [Fragment (Blank)]を選択してください。

図

`Fragment`の名前を入力して、[Include fragment factory methods?]チェックボックスと[Include interface callbacks?]チェックボックスを外して、[Finish]ボタンを押します（このチェックボックスを外さないと、余計なコードが生成されてしまいます）。この手順を5回繰り返して、必要な`Fragment`をすべて生成してください。

図

ふう、完了。

## navigationリソースを作成する

さて、Navigation。Navigationでは、画面の遷移をres/navigationの下のXMLファイルで管理します。このファイルを作りましょう。Android Studioのメニューから、[File] - [New] - [Android Resource Directory]を選びます。

図

[Resource type]を「navigation」にして、[OK]ボタンを押してください。これで、navigationディレクトリが生成されます。

次。画面の遷移を管理するXMLファイルです。Android Studioのメニューから、[File] - [New] - [Android Resource File]を選びます。

図

[Resource type]を「Navigation」にして、[File name]に「navigation」と入力して、[OK]ボタンを押します。これで、navigation.xmlが生成されました。

図

GUIツールはかったるいという私の趣味嗜好により、[Text]タブをクリックして、以下のXMLファイルを入力します。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<navigation
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/splashFragment">

    <fragment
        android:id="@+id/splashFragment"
        android:name="com.tail_island.jetbus.SplashFragment"
        android:label="@string/app_name"
        tools:layout="@layout/fragment_splash">

        <action
            android:id="@+id/splashFragmentToBookmarksFragment"
            app:destination="@id/bookmarksFragment" />

    </fragment>

    <fragment
        android:id="@+id/bookmarksFragment"
        android:name="com.tail_island.jetbus.BookmarksFragment"
        android:label="ブックマーク"
        tools:layout="@layout/fragment_bookmarks">

        <action
            android:id="@+id/bookmarksFragmentToDepartureBusStopFragment"
            app:destination="@id/departureBusStopFragment" />

        <action
            android:id="@+id/bookmarksFragmentToBusApproachesFragment"
            app:destination="@id/busApproachesFragment" />

    </fragment>

    <fragment
        android:id="@+id/departureBusStopFragment"
        android:name="com.tail_island.jetbus.DepartureBusStopFragment"
        android:label="出発バス停"
        tools:layout="@layout/fragment_departure_bus_stop">

        <action
            android:id="@+id/departureBusStopFragmentToArrivalBusStopFragment"
            app:destination="@id/arrivalBusStopFragment" />

    </fragment>

    <fragment
        android:id="@+id/arrivalBusStopFragment"
        android:name="com.tail_island.jetbus.ArrivalBusStopFragment"
        android:label="到着バス停"
        tools:layout="@layout/fragment_arrival_bus_stop">

        <argument
            android:name="departureBusStopName"
            app:argType="string" />

        <action
            android:id="@+id/arrivalBusStopFragmentToBusApproachesFragment"
            app:destination="@id/busApproachesFragment"
            app:popUpTo="@id/bookmarksFragment" />

    </fragment>

    <fragment
        android:id="@+id/busApproachesFragment"
        android:name="com.tail_island.jetbus.BusApproachesFragment"
        android:label="バス接近情報"
        tools:layout="@layout/fragment_bus_approaches">

        <argument
            android:name="departureBusStopName"
            app:argType="string" />

        <argument
            android:name="arrivalBusStopName"
            app:argType="string" />

    </fragment>

</navigation>
~~~

どのような`Fragment`があるのかを、XMLのタグで表現します。`<fragment>`タグでですね。`android:name`属性で`Fragment`を実装するクラスを、`android:lanbel`属性で画面に表示するタイトルを設定します。`tools:layout`属性は、GUIツールでグラフィカルに表示する場合向けの、プレビュー用のレイアウトの指定です。

`<fragment>`タグの中の`<action>`タグは、画面遷移を表現します。`app:destination`属性は、遷移先を指定します。`arrivalBusStopFragmentToBusApproachesFragment`で指定されている`app:popUpTo`属性は、[戻る]ボタンが押された場合の行き先を指定しています。出発バス停を選んで、到着バス停を選んで、バスの接近情報が表示されたあとに[戻る]ボタンを押す場合は、多分その路線の情報はもういらない場合でしょうから、`app:popUpTo`属性を使用してブックマーク画面まで一気に戻るようにしました。

`<fragment>`の中の`<argument>`は、フラグメントに遷移する際のパラメーターです。選択された出発バス停を使って到着バス停の選択肢を抽出しないと、到着バス停を選ぶところで路線がつながっていないバス停が表示されてしまうでしょ？　だから、`arrivalBusStopFragment`では`departureBusStopName`というパラメーターを指定しました。

あとはそう、`android:id`の説明を忘れていました。AndroidのリソースのXMLでは、`@+id/`の後に続ける形で識別子を設定します。上のXMLの`android:id="@+id/splashFragment"`みたいな感じ。`@+id/`の部分が何だか分からなくて気持ち悪いという方のために補足しておくと、Androidアプリの開発ではR.javaというファイルが内部で自動生成されていて、R.javaの中にリソースの識別子が32bit整数で定義されています。XMLのような文字列の識別子では照合作業でCPUを大量に消費してしまうからという貧乏性実装、でも、CPUを消費するということは電池を消費するということで、電池は今でもとても貴重な資源なので今でも素晴らしい実装です。このR.idにIDを追加して適当な32bit整数を割り振っておいてくださいねって指示が、`@+id/`なんです。ちなみに、割り振られた32bit整数を参照してくださいってのは`@id/`です。`<action>`の中の`app:destination="@id/bookmarksFragment"`みたいな感じになります。

## layout

続けて、作成した<navigation>を使うように、画面を定義していきましょう。画面遷移をXMLで表現したように、Androidアプリの開発では、画面の構造もXMLで表現します。文字列を表示するなら`<TextView ... />`みたいな感じです。で、この画面定義の際に重要な属性は`androud:layout_width`と`android:layout_height`です。

`android:layout_width`と`android:layout_height`は、UIコンポーネントの幅と高さとなります。ここに指定するのは、具体的な大きさ（`64dp`など）か、`match_parent`、`wrap_content`になります。`match_parent`は階層上の親と同じところまで大きくするようにとの指示、`wrap_content`はコンテンツが入る大きさでお願いしますという指示です。

## ConstraintLayout

あと、AndroidのUIコンポーネントでは、他のコンポーネントを子要素として持てるものを`ViewGroup`、`ViewGroup`を継承して何らかのレイアウト機能を追加したものをLayoutと呼びます。Layoutには、後述するApp barを作るための`AppBarLayout`のような目的に特化したものと、`LinearLayout`(縦や横に直線状に並べる）のような汎用的なものがあります。で、今ここで話題にしたいのは、汎用的なほうのLayout。

貧弱なCPUを考慮した結果だと思う（作るのが面倒だったからだとは思いたくない）のですけど、Android SDKはとても単純な機能のLayoutしか提供しません。直線状に並べる`LinearLayout`とか並べないで重ねる`FrameLayout`みたいなのとか。これらの単純なLayoutの組み合わせで複雑なレイアウトを実現するのがAndroidアプリ開発者の腕の見せ所……だったのは遠い昔の話で、今は、`androidx.constraintlayout.widget.ConstraintLayout`だけを覚えれば大丈夫になりました。

`ConstraintLayout`は、コンポーネントの位置を他のコンポーネントとの関係で表現します。「住所入力欄は名前入力欄の下」みたいな感じです。具体的には、`app:layout_constraintTop_toBottomOf="@id/nameTextField"`のようになります。左右については少し注意が必要で、アラビア語のように右から左に書く言語にも対応するために、LeftとRightではなくStartとEndで表現します。「住所入力欄のStartは名前入力欄のStartに合わせる」なら、`app:layout_constraintStart_toStartOf="@id/nameTextField"`にするというわけ。

## NavHostFragmentを追加する

これで一般論が終わりました。Projectビューの[app] - [res] - [layout]の下の「activityu\_main.xml」を開いて、[Text]タブを選択して、以下を入力してください。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/navHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:name="androidx.navigation.fragment.NavHostFragment"
        app:navGraph="@navigation/navigation"
        app:defaultNavHost="true" />

</androidx.constraintlayout.widget.ConstraintLayout>
~~~

注目していただきたいのは`<fragment>`タグの`android:name`属性のところ。`android:name`属性で実装のクラスを指定できるのはnavigationのときと同じで、今回のXMLで指定しているのは「androidx.navigation.fragment.NavHostFragment」です。Googleの人が作ってくれたクラスですな。このクラスが何なのかリファレンスで調べてみると、Navigationのためのエリアを提供すると書いてありますので、ここにNavigation配下の画面が表示されるようになったわけ。あとは、`app:navGraph`属性で先程作成したnavigation.xmlを指定して、`app:defaultNavHost`属性でデフォルトに指定しておきます。

次は、ダミーの画面レイアウトを作りましょう。そうしておかないと、遷移したのかどうか分からないですもんね。

## layout、再び

さて、これから`Fragment`向けのダミーのlayoutを作成していくわけですけど、その際に、Android Studioが自動で生成したXMLとはルート要素を変更します。Android Studioが生成したXMLではルート要素は具体的な`Layout`（私の環境では`FrameLayout`でした）だったのですけど、これを`<layout>`タグに変更します。

なんでこんなことをするのかというと、後述する説明するデータ・バインディングをやりたいから。データ・バインディングをやる際には`<data>`タグでデータを指定するのですけど、`<...Layout>`タグの下には`<data>`タグを書けませんもんね。あと、ルート要素を`<layout>`タグにしておくと、Bindingクラスが生成されるようになってプログラミングがちょっと楽になるというのがあります。Bindingクラスについては後述することにして、代表例として、fragment_bookmarks.xmlを載せます。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    tools:context=".BookmarksFragment">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:layout_marginStart="16dp"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            android:text="Bookmarks" />

        <Button
            android:id="@+id/departureBusStopButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            app:layout_constraintTop_toBottomOf="@id/textView"
            app:layout_constraintStart_toStartOf="@id/textView"
            android:text="DEPARTURE BUS STOP" />

        <Button
            android:id="@+id/busApproachesButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            app:layout_constraintTop_toBottomOf="@id/departureBusStopButton"
            app:layout_constraintStart_toStartOf="@id/departureBusStopButton"
            android:text="BUS APPROACHES" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
~~~

ConstraintLayoutのところでUIコンポーネントは`app:layout_constraint...`属性でレイアウトすると説明しておきながら、そこでは使わなかった`app:layout_constraint...`が出てきてくれました（activity\_main.xmlでは、`android:layout_width`属性も`android:layout_height`属性も「match_parent」だったので、制約をつける必要がなかったんですよ）。自分がなんの画面なのかを表示する`TextView`を左上に、その下に`Button`を表示するようになっています。`android:layout_marginTop`属性はマージンです。マージン分だけ隙間を開けてくれるわけですな。

こんな感じでルート要素が`<layout>`になるように、他の`Fragment`のレイアウトも同様に修正してください。

## Fragmentの実装

レイアウトの修正が終わったら、ロジックの修正です。代表例は先ほどと同じ`BookmarksFragment`で。

~~~ kotlin
package com.tail_island.jetbus

import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.navigation.fragment.findNavController
import com.tail_island.jetbus.databinding.FragmentBookmarksBinding

class BookmarksFragment: Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return FragmentBookmarksBinding.inflate(inflater, container, false).apply {
            departureBusStopButton.setOnClickListener {
                findNavController().navigate(BookmarksFragmentDirections.bookmarksFragmentToDepartureBusStopFragment())
            }

            busApproachesButton.setOnClickListener {
                findNavController().navigate(BookmarksFragmentDirections.bookmarksFragmentToBusApproachesFragment("日本ユニシス本社前", "深川第八中学校前"))
            }
        }.root
    }
}
~~~

importの最後で`com.tail_island.jetbus.databinding.FragmentBookmarksBinding`をインポートしていますけど、この`FragmentBookmarksBinding`クラスは、先程fragment_bookmarks.xmlのルート要素を`<layout>`タグに変更したことで生成されたクラスです（XMLのファイル名からクラス名を決定するので、`BookmarksFragmentBinding`ではなくて`FragmentBookmarksBinding`になります。不整合がキモチワルイけど、Android Studioのデフォルトはこのファイル名なのでしょうがない……）。あと、もしコードを書いているときにAndroid Studioがそんなクラスはないよと文句をつけてきたら、レイアウトXMLの変更を忘れたか、レイアウトXMLの変更後にビルドをしていないか（ビルドのときに自動生成されます）です。確認してみてください。

次。`Fragment`を初期化する処理は、コンストラクタではなく、`onCreateView()`メソッドの中に書くことに注意してください。`Fragment`は画面と結び付けられていて、画面というのは複雑で初期化が大変なソフトウェア部品です。だから、初期化のステージに合わせて、小刻みにメソッドが呼ばれる方式が採用されました。`Fragment`を作成するときに呼ばれる`onCreate()`メソッドとか、`Fragment`の`View`を生成するときに呼ばれる`onCreateView()`メソッドとか。で、一般に`Fragment`の初期化というのは画面の初期化なので、画面を構成する`View`がない状態では初期化の作業ができません。だから、`onCreate()`メソッドではなくて、`onCreateView()`メソッドに初期化のコードを書きます。

さて、`onCreateView()`メソッドでやらなければならない作業は、`Fragment`の`View`の生成です。Android Studioが生成する`Fragment`のコードでは、引数で渡ってくる`inflater: LayoutInflater`の`inflate()`メソッドをレイアウトXMLを引数にして呼び出すコードが生成されるのですけど、将来のデータバインディングのために、`FragmentBookmarksBinding`の`inflate()`メソッドを使用します。で、`...Binding`の`inflate()`メソッドで生成されるのは`...Binding`なので、`View`である`root`プロパティを返します。コードにすると、こんな感じ。

~~~ kotlin
return FragmentBookmarksBinding.inflate(inflater, container, false).root
~~~

## スコープ関数

でもね、私達は画面の初期化をしたいわけで、初期化にはボタンが押されたときの`Listener`の登録も含まれます。以下のようなコードになるでしょうか。

~~~ kotlin
val binding = FragmentBookmarksBinding.inflate(inflater, container, false)

binding.departureBusStopButton.setOnClickListener {
    // ボタンが押された場合の処理
}

return binding.root
~~~

`...Binding`では、レイアウトXMLで定義されたUIコンポーネントを`android:id`と同じプロパティ名で参照するためのコードが生成されていますから、`binding.departureBusStopButton`で画面のボタンを参照できます。ボタンがクリックされたときのリスナーを設定するメソッドは`setOnClickListener`で、その引数は関数です。で、関数はラムダ式で定義することができます。あと、Kotlinには、最後のパラメータがラムダ式の場合はそのパラメーターはカッコの外に指定するという慣習があります。さらに、ラムダ式だけを引数にする場合は、括弧を省略できる。というわけで、上のようなシンプルなコードになるわけです。

ただね、このようなコードはよく見るのですけど、実はこれ、とても悪いコードなんです。その理由は、ローカル変数（val binding）を使っているから（グローバル変数を使えと言っているわけじゃないですよ、念の為）。

ローカル変数は、そのブロックを抜けるまで有効です。長期間に渡ってコードに影響を与えるというわけ。`val`にしてイミュータブル（不変）にした場合でも、状態遷移という影響は減るけれども変数があることを覚えて置かなければならないのは一緒で、コードを読むのが大変になってしまう。私のような、記憶力が衰えまくっているおっさんには、ローカル変数は辛いんですよ。だから、変数のスコープを小さくします。関数の最初で変数を宣言しなければならなかくてその変数が関数の最後まで有効な大昔のプログラミング言語より、どこでも変数を宣言できてブロックが終わると変数のスコープが切れる今どきの言語の方が使いやすいですよね？

と、このような、変数のスコープを短く、かつ、分かりやすい形で制御したい場合に使えるKotlinの便利ライブラリが、スコープ関数なんです。

たとえば、`foo()`の戻り値を使用したい場合は、

~~~ kotlin
val x = foo()

x.bar()
~~~

と書くのではなくて、

~~~ kotlin
foo().let { it.bar() }
~~~

と書きます。`let()`スコープ関数は、自分自身を引数にしたラムダ式を呼び出すというわけ。あ、ラムダ式の中の`it`は、ラムダ式のパラメーターを宣言しなかった場合の暗黙の名前です。`let()`スコープ関数は自分自身を引数にしますから、この場合の`it`は`foo()`の戻り値になります。

他のスコープ関数には、`apply()`（`this`を自分自身に設定したラムダ式を呼び出して、自分自身を返す。初期化等で便利）、`also()`（自分自身を引数にしたラムダ式を呼び出して、自分自身を返す。自分自身を使う他のオブジェクトの初期化等で便利）、`run()`（自分自身を`this`にしたラムダ式を実行して、ラムダ式の戻り値を返す。その場でメソッドを定義する感じ）があります。とにかく便利なので、ぜひ使い倒してください。

というわけで、今回は`apply()`を使用して、`FragmentBookmarksBinding`のボタンへのリスナー登録という初期化処理を書いています。

~~~ kotlin
return FragmentBookmarksBinding.inflate(inflater, container, false).apply {
    departureBusStopButton.setOnClickListener {
        // ボタンが押された場合の処理
    }
}.root
~~~

`apply()`なので`this`は`FragmentBookmarksBinding`になっていて、だから`FragmentBookmarksBinding`の`departureBusStopButton`に修飾なしでアクセスできて便利。スコープの範囲がインデントされて判別しやすいのも、認知機能が衰えた私のようなおっさんには嬉しいです。

## 画面遷移

Navigationでの画面の遷移は、`findNavController()`で取得できる`NavController`クラスの`navigate()`メソッドで実施します。リファレンスを見てみると`navigation()`メソッドにはオーバーロードされた様々なバリエーションがあって、色々な引数を取れるようになっています。今回、この中で注目していただきたいのは、`NavDirections`を引数に取るバージョンです。ナビゲーションのXMLで`<action>`を作成するとこのNavDirectionsが自動生成されるので、とても簡単に呼び出せます。というわけで、画面遷移のコードは以下のようになります。

~~~ kotlin
findNavController().navigate(BookmarksFragmentDirections.bookmarksFragmentToDepartureBusStopFragment())
~~~

あと、記憶力が良い方は、ナビゲーションのXMLを作成したときに、`<argument>`で`Fragment`への遷移のパラメーターを定義したことを覚えているかもしれません。`<argument>`を定義しておくと、`NavDirections`を生成するときにパラメーターを追加してくれます。記憶力が壊滅している私のようなおっさんの場合でも、パラメーターをしていしないとコンパイル・エラーになるから思い出せますな。というわけで、以下のようなコードになります。

~~~ kotlin
findNavController().navigate(
    BookmarksFragmentDirections.bookmarksFragmentToBusApproachesFragment("日本ユニシス本社前", "深川第八中学校前")
)
~~~

## Navigationを試してみる

こんな感じですべての`Fragment`をプログラミングして、動かしてみると以下のようになります。

動画

[到着バス停]画面から[バス接近情報]画面に遷移した後、[戻る]ボタンを押すとレイアウトのXMLの`app:popUpTo`属性が効いて、[ブックマーク一覧]画面に戻ってくれていてとても嬉しい。

でも、なんか画面が寂しい気がします……。どうにかならないかな？　大体、ナビゲーションのXMLでわざわざ設定した`android:label`属性はどうなったんでしょうか？

## App barとNavigation drawer

さて、AndroidアプリのUIは、[Material Design](https://material.io)というデザイン・ガイドに従うことになっています。このMaterial Designにはいろいろなコンポーネントがあるのですけど、App barというコンポーネントで画面の情報を表示して、Navigation drawerでメニューを実現する方式が一般的みたいです。

図

図

今の画面でもApp barっぽいのはありますけど、Navigation drawerがありません。作ってみましょう。[Project]ビューの[app] - [res] - [layout]の下の「activity\_main.xml」を開いて、[Text]タブを選択していますね？　activity\_main.xmlを以下に変更します。`Fragment`のレイアウトと統一するために、ルート要素は`<layuout>`に変更しました。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".MainActivity">

    <androidx.drawerlayout.widget.DrawerLayout
        android:id="@+id/drawerLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <com.google.android.material.appbar.AppBarLayout
                android:id="@+id/appBarLayout"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toTopOf="parent">

                <androidx.appcompat.widget.Toolbar
                    android:id="@+id/toolbar"
                    android:layout_width="match_parent"
                    android:layout_height="?attr/actionBarSize" />

            </com.google.android.material.appbar.AppBarLayout>

            <fragment
                android:id="@+id/navHostFragment"
                android:layout_width="match_parent"
                android:layout_height="0dp"
                app:layout_constraintTop_toBottomOf="@id/appBarLayout"
                app:layout_constraintBottom_toBottomOf="parent"
                android:name="androidx.navigation.fragment.NavHostFragment"
                app:navGraph="@navigation/navigation"
                app:defaultNavHost="true" />

        </androidx.constraintlayout.widget.ConstraintLayout>

        <com.google.android.material.navigation.NavigationView
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            app:headerLayout="@layout/item_navigation_header"
            app:menu="@menu/menu_navigation"
            style="@style/Widget.MaterialComponents.NavigationView">

            <androidx.constraintlayout.widget.ConstraintLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="16dp"
                    android:layout_marginEnd="16dp"
                    android:layout_marginBottom="16dp"
                    app:layout_constraintBottom_toBottomOf="parent"
                    android:textSize="10sp"
                    android:text="本アプリが使用する公共交通データは、公共交通オープンデータセンターにおいて提供されるものです。\n\n公共交通事業者により提供されたデータを元にしていますが、必ずしも正確／完全なものとは限りません。本アプリの表示内容について、公共交通事業者に直接問い合わせないでください。\n\n本アプリに関するお問い合わせはrojima1@gmail.comにお願いします。" />

            </androidx.constraintlayout.widget.ConstraintLayout>

        </com.google.android.material.navigation.NavigationView>

    </androidx.drawerlayout.widget.DrawerLayout>

</layout>
~~~

……長くてごめんなさい。でも、思い出してください。我々はNavigationを使用していますので、画面といえば`Fragment`なわけです。`Activity`はこれ一つだけ。なので、まぁ、一回だけならば長くても許容できるかなぁと。次のアプリの開発でも、ほぼコピー＆ペーストでいけますし。私もこれ、コピペで作成して、`<NavigationView>`の子要素の部分を付け加えただけで作っています。本アプリで子要素を追加するという面倒な作業が追加になった理由は、[公共交通オープンデータセンター](https://www.odpt.org/)からデータを取得する際には「本アプリが使用する……」のような通知を書く必要があったためです。

でも、あれ？　`<com.google.android.material.navigation.NavigationView>`タグの属性を見ていくと、`app:headerLayout="@layout/item_navigation_header"`や`app:menu="@menu/menu_navigation"`と書いてあって、こんなリソースは無いとエラーになっています。以下の図のようにNavigation drawerはヘッダーとメニューとそれ以外で構成されていて、それ以外は子要素で定義したので、残りのヘッダーとメニューを定義しなければならないんですね。

図

というわけで、ヘッダーを作成します。[File] - [New] - [Android Resource File]メニューを選択し、[File name]に「item\_navigation\_header」を入力して[Resource type]を「Layout」に設定し、作成されたitem\_navigation\_header.xmlに以下を入力します。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingBottom="16dp"
        android:background="@color/colorPrimary">

        <ImageView
            android:id="@+id/imageView"
            android:layout_width="64dp"
            android:layout_height="64dp"
            android:layout_marginStart="16dp"
            android:layout_marginTop="16dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            android:src="@mipmap/ic_launcher_round" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            app:layout_constraintStart_toEndOf="@id/imageView"
            app:layout_constraintTop_toTopOf="@id/imageView"
            app:layout_constraintBottom_toBottomOf="@id/imageView"
            android:text="@string/app_name" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
~~~

次。メニュー。[File] - [New] - [Android Resource Directory]メニューを選択し、[Resource type]を「Menu」に設定してres/menuを作成します。そのうえで、[File] - [New] - [Android Resource File]メニューを選択して、[File name]に「menu\_navigation」を入力して[Resource type]を「Menu」に設定し、作成されたmenu\_navigation.xmlに以下を入力します。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group>
        <item android:id="@+id/clearDatabase" android:title="バス停と路線のデータを再取得" />
    </group>
</menu>
~~~

[公共交通オープンデータセンター](https://www.odpt.org/)から取得したデータのキャッシュが古くなった場合に、再取得するためのメニューですね。

いろいろと作業してきましたが、ごめんなさい、でもまだ終わりません。元の画面でもApp barっぽいのがあったのに、activity\_main.xmlに新たに`<com.google.android.material.appbar.AppBarLayout>`が追加されたことが不思議ではありませんでしたか？　こんなことをした理由は、元の画面でのApp barっぽいのは制御ができない（少なくとも私は制御のやり方をしらない）から。だから、新しいのを追加したわけですな。

新しいのを追加した以上は古いのを削除しなければならないわけで、そのためにはAndroidManifest.xmlを修正します。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.tail_island.jetbus">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <activity android:name=".MainActivity" android:theme="@style/AppTheme.NoActionBar">  <!-- android:theme属性を追加 -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
~~~

`AppTheme.NoActionBar`というスタイルにするように指定しているわけ……なのですけど、この`AppTheme.NoActionBar`は自分で作らなければならないんですよ。無駄に感じる作業が続いてかなり腹が立ってきた頃かと思いますが（私は、毎回この作業の途中で独り言で文句をいっているらしく、周囲に不気味がられました）、アプリ開発につき1回だけ、しかもコピー＆ペーストで済む作業ですので頑張りましょう。[Project]ビューのres/values/styles.xmlを、以下に変更します。

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="BaseAppTheme" parent="Theme.MaterialComponents.Light.DarkActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="AppTheme" parent="BaseAppTheme" />

    <style name="AppTheme.NoActionBar">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>

    <style name="AppTheme.AppBarOverlay" parent="ThemeOverlay.MaterialComponents.Dark.ActionBar" />

    <style name="AppTheme.PopupOverlay" parent="ThemeOverlay.MaterialComponents.Light" />

</resources>
~~~

App barとNavigation drawerとライブラリのNavigationを組み合わせましょう。MainActivity.ktを開いて、以下に変更してください。

~~~ kotlin
package com.tail_island.jetbus

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import androidx.core.view.GravityCompat
import androidx.databinding.DataBindingUtil
import androidx.navigation.findNavController
import androidx.navigation.ui.AppBarConfiguration
import androidx.navigation.ui.NavigationUI
import com.tail_island.jetbus.databinding.ActivityMainBinding

class MainActivity: AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Bindingを生成します
        binding = DataBindingUtil.setContentView<ActivityMainBinding>(this, R.layout.activity_main).also {
            // NavigationControllerを初期化します
            findNavController(R.id.navHostFragment).apply {
                // レイアウトで設定したToolbarとDrawerLayoutと協調させます。また、BookmarksFragmentをルートにします。itは、ActivityMainBindingのインスタンスです
                NavigationUI.setupWithNavController(it.toolbar, this, AppBarConfiguration(setOf(R.id.bookmarksFragment), it.drawerLayout))

                // SplashFragmentでは、ツールバーを非表示にします
                addOnDestinationChangedListener { _, destination, _ ->
                    it.appBarLayout.visibility = if (destination.id == R.id.splashFragment) View.GONE else View.VISIBLE
                }
            }
        }
    }

    // 不整合の辻褄をあわせます。なんで我々がと思うけど我慢……。
    override fun onBackPressed() {
        if (binding.drawerLayout.isDrawerOpen(GravityCompat.START)) {  // Navigation drawerが開いているときは、[戻る]ボタンでクローズします
            binding.drawerLayout.closeDrawer(GravityCompat.START)
            return
        }

        if (findNavController(R.id.navHostFragment).currentDestination?.id == R.id.bookmarksFragment) {  // AppBarConfiguration()したのでツールバーはハンバーガー・アイコンになっていますが、それでもバック・ボタンでは戻れちゃうので、チェックします
            finish()
            return
        }

        super.onBackPressed()
    }
}
~~~

このコードは少し複雑ですから、解説を。

NavigationとNavigation drwerとApp barはとても良くできていて、上のコードのように`NavigationUI.setupWithNavController()`で結合できるのですけど、少しだけ、AndroidのAPIとの不整合があります。不整合その1は、Navigation drawerが開いているときに[戻る]ボタンが押されたときの動作です。Navigation drawerが表示されるというのは`<com.google.android.material.navigation.NavigationView>`に画面遷移したように見えるのですけど、内部的には画面遷移になっていないので、[戻る]ボタンを押してもNavigation drawerは閉じられません。これではユーザーが混乱しますから、`onBackPressed()`をオーバーライドして「Navigation drawerが開いている場合は閉じる」処理を追加しました。不整合その2は、本アプリでは、NavigationのXMLでのトップ／レベルの画面は`SplashFragment`なのですけど、アプリ的には`BookmarksFragment`がトップ・レベルであることです。なので、上のコードの`AppBarConfiguration(setOf(R.id.bookmarksFragment), ...)`でトップ・レベルの指定をしているのですけど、やっぱり[戻る]ボタンでの動作がおかしくなっちゃう。`BookmarksFragment`ではApp barの左がハンバーガー・アイコンになって戻れない様になっていて正しいのですけど、[戻る]ボタンを押すと`SplashFragment`に戻ってしまいます。なのでやっぱり、`onBackPressed()`の中に遷移を制御する処理を追加しました。

で、`onBackPressed()`はメソッドですから、`onCreate()`と`Binding`を共有したい場合は属性を追加しなければなりません。なので`binding`という属性を追加したのですけど、この`binding`は、`onCreate()`が呼ばれるまで値を設定できないという問題があります。しょうがないので最初は`null`を設定する……のは、KotlinのようなNull安全を目指している言語では悪手です。今回のように、`binding`の値が設定される前に使用されないことを保証できる（`onBackPressed()`は`onCreate()`が終わった後にしか呼び出されないので、保証できます）場合は、プログラマーの責任で属性を`lateinit`として定義できます。`lateinit`にすると初期値を設定しなくてよいので、ほら、`binding`の型を`null`を許容「しない」`ActivityMainBinding`に設定できました。

あと、上のコードでやっているのは、`SplashFragment`でApp barを消していること。App barに[戻る]アイコンが表示されてしまいますし、調べた限りでは、他のアプリでもスプラッシュ画面にはApp barがありませんでしたから。

ともあれ、これで作業完了のはず。試してみましょう

動画

うん、完璧ですな。Navigationよ今夜もありがとう。Navigationくらいに楽チンな、App barとNavigation drawerをどうにかしてくれるライブラリを作ってくれないかなぁ……。

# Retrofit2

本アプリは[公共交通オープンデータセンター](https://www.odpt.org/)が提供してくれるデータが無いと動きようがありませんので、個々の画面の機能を作っていく前に、HTTP通信でWebサーバーからデータを取得する処理を作ってみましょう。ただ、残念なことにJetpackにはHTTP通信の機能がありませんので、Retrofit2を使用します。

## Retrofit2の組み込み

Retrofit2を組み込むために、build.gradelを変更します。

~~~ xml
dependencies {
    ...

    implementation 'androidx.room:room-ktx:2.2.0-beta01'
    implementation 'com.squareup.retrofit2:retrofit:2.6.1'  // 追加
    implementation 'com.squareup.retrofit2:converter-gson:2.6.1'  // 追加
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"

    ...
}
~~~

これで、いつでもHTTP通信できるわけですけど、さて、公共交通オープンデータセンターからどんなデータを取得できるのでしょうか？

## 高交通オープンデータセンター

[公共交通オープンデータセンター](https://odpt.org/)のサイトを開いて、下の方にある[[公共交通オープンデータセンター開発者サイト](https://developer.odpt.org/ja/info)]リンクをクリックします。で、真ん中あたりにある[ユーザ登録のお願い]ボタンを押して、もろもろ入力して[この内容で登録を申請する]ボタンを押して、で、「最大2営業日」で登録完了のメールが送られてきます……。なんで最大2営業日なんだろ？

登録が完了したらログインして、右上の[Account]メニューの[アクセストークンの確認・追加]を選ぶと、アクセストークンが表示されます。まずはこれをコピーしてください。で、この情報の保存先なのですけど、今回はリソースを使用しましょう。Androidアプリの開発では、様々な定数をリソースとしてコードの外に定義できます。このリソースをうまく使うと、国際化もできてとても便利です。

# Room

# Dagger

# Lifecycle

# RecyclerView

# データバインディング

# Asset Studioでアイコンを作れば、完成！
