<!--http://gihyo.jp/dev/feature/01/memcached/0004-->

<!-- 第4回 memcachedの分散アルゴリズム # -->

# 第4回 memcachedの分散アルゴリズム #

> 本文出处 [第4回 memcachedの分散アルゴリズム](http://gihyo.jp/dev/feature/01/memcached/0004)

<!--
株式会社ミクシィの長野です。第2回，第3回と前坂がmemcachedの内部について紹介しました。今回は内部構造から離れて，memcachedの分散についての紹介をいたします。
-->

## memcachedの分散 ##

<!--
連載の1回目に紹介しましたが，memcachedは「分散」キャッシュサーバと言われていますが，サーバ側には「分散」の機能は備わっていません。サーバ側には当連載の第2回，第3回で前坂が紹介したメモリストレージの機能のみが組み込まれており，非常にシンプルな実装となっています。では，memcachedの分散はどのように実現しているのかと言うと，すべてクライアントライブラリによって実現されます。この分散方法はmemcachedの大きな特徴です。
-->

“memcached是‘分布式’缓存服务器”虽然在连载的第1章这样介绍过，但是服务器端并不具备“分布式”的功能。正如在第2章、第3章前坂所介绍的那样，服务器端的实现非常简单，只具备内存存储（memory storage）的功能。那么如何实现memcached的分布式呢，答案是所有的分布式方案要由客户端代码库（client library）实现。这种分布式方案是memcached的一大特征。

### memcachedの分散とは ###
ここまで数度「分散」という言葉を用いてきましたが，あまり詳しく触れてきませんでした。ここでは各クライアントの実装に共通する大まかな仕組みから紹介いたします。

至此，虽然已经多次使用了“分布式”一词，但是并没有详细地讲解。下面将从在各个客户端的实现中都在使用的大体结构入手，开始讲解。

memcachedのサーバが，node1〜node3まであり，アプリケーションから「tokyo」「kanagawa」「chiba」「saitama」「gunma」というキー（名前）のデータを保存する場合を想定してみます。

假设有node1～node3，3台memcached服务器，下面要通过应用程序将分别以「tokyo」「kanagawa」「chiba」「saitama」「gunma」为key的数据保存到其中。

図1 分散概要：準備

![図1 分散概要：準備](http://image.gihyo.co.jp/assets/images/dev/feature/01/memcached/0004/thumb/TH400_0004-01.png)

まず「tokyo」をmemcachedにaddします。クライアントライブラリに「tokyo」を渡すと，ライブラリに実装されたアルゴリズムよって「キー」から保存するmemcachedサーバを決定します。サーバが決定したら，そのサーバに対して「tokyo」とその値を保存する命令をします。

首先把「tokyo」add到memcached中。一旦把「tokyo」传递给client library，由library实现的算法就能通过key决定要将其对应的数据保存到哪台memcached服务器上。而一旦确定了服务器，client library就会像这台服务器发送一条指令，要求它保存作为key的「tokyo」以及其对应的数据。

図2 分散概要：追加時
![](http://image.gihyo.co.jp/assets/images/dev/feature/01/memcached/0004/0004-02.png)

同じように，「kanagawa」「china」「saitama」「gunma」もサーバを決定して保存します。

相似地，对于「kanagawa」「china」「saitama」「gunma」也要先（由client library）确定服务器再保存。

次は保存したデータの取得です。取得の場合も，ライブラリに対して，取得したいキー「tokyo」を渡します。ライブラリは保存時と同じアルゴリズムを使い，「キー」からサーバを決定します。同じアルゴリズムを利用しているので保存したときと同じサーバが決定されるので，サーバに対してgetの命令を送るとなんらかの理由でデータが削除されていない限り，保存した値が取得できます。

下面是获取已保存到memcached中的数据。获取数据时，要把想获取的数据对应的key，比如「tokyo」，传递给library。library会使用与保存数据时相同的算法，由key决定服务器。通过使用相同的算法就能找到保存时所使用的服务器，因此只要这个key所对应的数据还存在，向这台服务器发送get指令，就能获取到对应的数据。

図3 分散概要：取得時
![](http://image.gihyo.co.jp/assets/images/dev/feature/01/memcached/0004/thumb/TH400_0004-03.png)

このようにキーごとに異なるサーバへ保存することでmemcachedの分散は実現されています。memcachedのサーバが多くなればキーも分散され，もしmemcachedのサーバが1台障害がおきて接続できない状態になったとしても，ほかのキャッシュに影響はせずにシステム全体は動き続けることができます。

像这样，通过将每个key保存到不同的服务器上就实现了memcached的分布式（存储）。就算memcached服务器的数量不多，key也能被分布（存储），而且即使有1台memcached服务器发生故障无法连接，也不会影响到其他（服务器上的）缓存数据，整个系统还可以继续运转。

次は具体的な実装として，1回目の連載で紹介したPerlのクライアントライブラリであるCache::Memcachedが実装している分散方法紹介いたします。

作为具体的现实方法，下面将介绍在连载的第1章介绍过的Perl语言的client library——Cache::Memcached所实现的分布式（存储）方案。

<!-- 
========================================================
	Page2 http://gihyo.jp/dev/feature/01/memcached/0004?page=2
======================================================== 
-->

## Cache::Memcachedの分散方法 ##

PerlのmemcachedクライアントライブラリであるCache::Memcachedはmemcachedを作成したBrad Fitzpatrick氏によるもので，いわば純正的なライブラリになります。

作为Perl语言的memcached client library的Cache::Memcached是由memcached的作者Brad Fitzpatrick开发的，即所谓的正统的library[^1]。

このライブラリにも分散の機能が備わっており，memcachedの標準的な分散方法になっています。

该library中也具备了分布（存取）功能，是memcached标准的分布存取方案。

### 剰余による分散 ###
Cache::Memcachedの分散方法について簡単に書くと，「サーバの台数の剰余による分散」になります。キーの整数のハッシュ値を求め，その数値をサーバの台数で割った余りでサーバを決定します。Cache::Memcachedのソースコードを単純化したPerlのスクリプトを利用して説明します。

若要简要地写出Cache::Memcached的分布式存取方案，那么就是“通过对服务器台数取余实现分布式存取”。首先求出整数形式的key的哈希值，然后用这个数值除以服务器的台数，最后用余数确定服务器。下面使用已将Cache::Memcached的源代码简化后的Perl语言的脚本来讲解。

```
use strict;
use warnings;
use String::CRC32;

my @nodes = ('node1','node2','node3');
my @keys = ('tokyo', 'kanagawa', 'chiba', 'saitama', 'gunma');

foreach my $key (@keys) {
	my $crc = crc32($key);             # CRC値
	my $mod = $crc % ( $#nodes + 1 );
	my $server = $nodes[ $mod ];       # 剰余からサーバを決定
	printf "%s => %s\n", $key, $server;
}
```

Cache::Memcachedはハッシュ値を求めるために，CRCを利用しています。

在Cache::Memcached中，为了求出哈希值，要用到CRC[^2]。

まず，キーのCRC値を求め，その数値をノードの数で割った余りの数から決まるサーバを利用します。上記のコードを実行すると，以下の出力が得られます。

首先求出key的CRC，然后用该数值除以节点数，最后由所得到的余数决定要使用的服务器。运行上述的代码后，会得到以下的输出结果。

```
tokyo       => node2
kanagawa    => node3
chiba       => node2
saitama     => node1
gunma       => node1
```

この結果から「tokyo」はnode2へ，「kanagawa」はnode3へというように分散されることがわかります。蛇足ですが，Cache::Memcachedではこのようにして求まったサーバに接続できなかった場合に，キーに接続を行った回数を連結して再度ハッシュ値を求めて接続を試みます。この動きをrehashと言います。rehashをさせたくない場合には，Cache::Memcachedのオブジェクトを生成する際に，「rehash=>0」というオプションを指定します。

从结果中可以看到，「tokyo」被分到了node2，「kanagawa」被分到了node3等等。这里要补充一点，在Cache::Memcached中，当通过此算法算出的服务器无法连接时，会把已尝试连接的次数加到key的后面，（然后对由此产生的新key）再次计算哈希值后并再次尝试连接。这个行为称作rehash。当不想进行rehash时，可以在生成Cache::Memcached的对象时，指定选项「rehash=>0」。

### 剰余による分散方法の弱点 ###

この剰余による分散はシンプルでデータの散らばり具合も優れていますが，弱点もあります。その弱点とはmemcachedサーバの追加や削除を行ったときのキャッシュの組み替えのコストが非常に大きいことがあげられます。サーバを追加すると剰余の結果が大きく変わってしまい，その結果データを保存したときと同じサーバへ取得しにいかなくなり，キャッシュのヒット率に影響が出ます。Perlのコードを書いてそのコストを検証してみます。

通过求余现实的分布（算法）很简单，也能实现较好的数据分布程度，但是这种方法也有缺点。缺点是进行完增加或减少memcached服务器的操作后，对已缓存数据的重组所带来的开销很大。由于一旦增加了服务器，求余计算的结果就会发生很大的变化，由此导致无法从保存数据时所使用的服务器上获取数据，降低了缓存的命中率。下面就通过写一段Perl的代码来检验一下这种开销。

```
use strict;
use warnings;
use String::CRC32;

my @nodes = @ARGV;
my @keys = ('a'..'z');
my %nodes;

foreach my $key ( @keys ) {
	my $hash = crc32($key);
	my $mod = $hash % ( $#nodes + 1 );
	my $server = $nodes[ $mod ];
	push @{ $nodes{ $server } }, $key;
}

foreach my $node ( sort keys %nodes ) {
	printf "%s: %s\n", $node,  join ",", @{ $nodes{$node} };
}
```

このPerlのスクリプトは，「a」から「z」までのキーをオプションとして渡したmemcachedに保存・参照するというイメージで作っています。mod.plとして保存し実行しました。

这段Perl脚本的用途是把key为a～z保存到作为参数传进来的memcached服务器中。将这段代码保存到mod.pl中并运行。

まず，最初にサーバが3台のときに
首先，当最初只有3台服务器时：

```
$ mod.pl node1 node2 nod3
node1: a,c,d,e,h,j,n,u,w,x
node2: g,i,k,l,p,r,s,y
node3: b,f,m,o,q,t,v,z
```

となり，node1には，a, c, d, e...，node2には，g, i, k...と1台のサーバに8個から10個のデータが保存されました。

node1中包含key a, c, d, e...，node2中包含key g, i, k...，即1台服务器中存有8～10个数据。

次にmemcachedのサーバを1台増やします。
下面增加1台memcached服务器。

```
$ mod.pl node1 node2 node3 node4
node1: d,f,m,o,t,v
node2: b,i,k,p,r,y
node3: e,g,l,n,u,w
node4: a,c,h,j,q,s,x,z
```

node4を増やして実行しました。このようにノードを増やすと，キーが分散されるサーバが大きく変化するのがわかると思います。26個のデータのうち6個のみが元のサーバと同じところを参照し，ほかのデータは異なるサーバへ移動してしまいます。ヒット率では23%まで低下してしまいます。Webアプリケーションでmemcachedを利用している場合，memcachedを増設した瞬間にキャッシュの効率が落ち，データベースサーバに負荷が集中し正常にサービスが行えないような状態になることが考えられます。

增加了node4后又运行了脚本。我想大家都会注意到，向这样一旦增加了节点，key会分布到哪台服务器上的结果就会发生巨大的变化。26个数据中只有6个数据还在原来的服务器上，其余的数据都移动到了不同的服务器上。命中率下降到23％。可以预见到，在通过Web应用程序调用memcached时，增设了memcached的瞬间，会导致缓存的效率降低，数据库服务器的负担增大，服务无法正常地运转。

mixiのWebアプリケーション運用でもこの問題ありmemcachedサーバの増設ができずにいましたが，新しい分散方法を利用することで現在ではmemcachedを手軽に増設していくことができるようになっています。その分散方法がConsistent Hashingです。

即使在mixi的Web应用程序中，这个问题也存在。因此过去一直无法增设memcached服务器，但是现在由于使用了新的分布算法，变得能够轻松地增设memcached服务器了。这种新的分布算法就是Consistent Hashing。

<!--
==========================================================
Page3 http://gihyo.jp/dev/feature/01/memcached/0004?page=3
==========================================================
-->

## Consistent Hashing ##

<!--Consistet Hashingの考え方は，株式会社ミクシィのエンジニアブログなどで数多く紹介されていますので，ここでは簡単に説明させていただきます。-->

在mixi的工程师博客[^3]等网站上都在频频介绍一致性哈希（Consistet Hashing）的思想[^4]，因此我在这里只是简单地说明一下。

<!--### Consiste Hashingの簡単な説明 ###
Consistent Hasingでは，まずmemcachedのサーバ（ノード）のハッシュ値を求め，0から232までの円（continuum）の中に配置します。そして格納するデータのキーも同じようにハッシュ値を求め，円の上にマッピングします。データはマッピングされた地点から時計回りで最初に見つかるサーバに保存されます。サーバが見つからず232を超えた場合は最初のmemcachedのサーバに保存します。-->

### 一致性哈希的简单介绍 ###

在一致性哈希中，首先求出每台memcached服务器（节点）的哈希值，然后把它们分别安置在0～2<sup>32</sup>的圆环（continuum）上。同样，对被存储数据的键也要计算哈希值并映射到这个圆环上。从映射点开始沿顺时针方向查找，数据将存储到最先找到的服务器上。如果由于找不到服务器而超过了2<sup>32</sup>，那么数据将保存到第一台memcached服务器上（如图4所示）。

<!--図4 Consistent Hashing：基本-->

图4 一致性哈希：初始状态
![图4 一致性哈希：初始状态](http://image.gihyo.co.jp/assets/images/dev/feature/01/memcached/0004/thumb/TH400_0004-04.png)

<!--この状態からmemcachedのサーバを1台追加をします。剰余による分散ではキーが保存されるサーバが大きく変化しキャッシュ率に影響がでていましたが，Consistent-Hashingではcontinuum上でサーバが追加された地点から時計回りとは逆方向で最初に見つかるサーバまでにあるキーのみが影響を受けます。-->

下面在如图4所示的状态下添加1台memcached服务器。如果是通过求余实现分布，一旦保存键的服务器发生较大的变化，那么将会影响缓存命中率，但是在一致性哈希中，从添加的服务器所对应的圆环上的点开始，到沿逆时针方向找到的第一台服务器为止，只有这之间的键才会受影响。

<!--図5 Consistent Hashing：サーバ追加-->

图5 一致性哈希：添加服务器
![图5 一致性哈希：添加服务器](http://image.gihyo.co.jp/assets/images/dev/feature/01/memcached/0004/0004-05.png)

<!--このようにしてConsistent Hashingでは，キーの組み替えを最小限にしています。また，Consistent Hasingの実装によっては仮想ノードという考え方が取り入れられています。通常のhash関数を通す方法ではサーバがマッピングされる場所のばらつきが非常に大きくなります。そこで仮想ノードでは1つのノード（サーバ）に対して100〜200個の点をcontinuum上に作成してそれを利用します。これによりばらつきを押さえ，サーバの増減時のキャッシュの組み替えを最小限にしています。-->

在一致性哈希中，通过这种处理，使键的重新排布达到最小化。另外，在一致性哈希的实现中，还采用了虚拟节点的技术。若使用一般哈希函数计算，服务器的映射点将会非常分散。因此，对于1台服务器（节点），通过虚拟节点在圆环上生成100～200个点，由此来减小分散程度，进一步使服务器的增减所带来的键的重新排布最小化。

<!--後述するConsistent Hashingに対応したmemcachedのクライアントライブラリを利用した実験の結果からは，現在のサーバの台数（n）と増やすサーバの台数（m）から，増設後のヒット率を以下の計算で求めることがわかっています。-->

使用支持一致性哈希的memcached客户端代码库进行实验，从实验结果可以得出如下用于计算增加服务器后命中率的公式：

```
(1 - n / (n + m)) * 100
```

其中，n是当前服务器的台数，m是要增加的服务器台数。

<!--### Consistent Hashing対応のライブラリ ###

この連載でも何回か紹介しているCache::Memcachedでは，Consistent Hashingはサポートされていませんが，既にいくつかのクライアントライブラリがこの新しい分散方法をサポートしています。Consistet Hashingと仮想ノードをサポートしている最初のmemcachedのクライアントライブラリはlibketamaというPHPのライブラリで，last.fmで開発されました。-->

### 支持一致性哈希的代码库 ###

在本连载中经常介绍的Cache::Memcached虽然不支持一致性哈希，但还是有一些客户端代码库已经支持这种新的分布算法了。叫作libketama[^5]的PHP代码库是第一个支持一致性哈希和虚拟节点的memcached客户端代码库，该代码库已在last.fm开发。

<!--Perlのクライアントとしては，連載の1回目で名前だけ紹介したCache::Memcached::FastとCache::Memcached::libmemcachedがConsistent Hashingをサポートしています。-->

对于Perl的客户端，在连载的第1回只提及名字的Cache::Memcached::Fast[^6]和Cache::Memcached::libmemcached[^7]都支持一致性哈希。
  
<!--どちらのモジュールもCache::Memcachedとほぼ同じインターフェイスを持ち，Cache::Memcachedを既に使っている場所であれば簡単に入れ替えを行うことができます。Cache::Memcached::Fastはlibketamaを再実装しており，Consistent Hashingを利用する場合はオブジェクトを作成する際にオプションでketama_pointsを指定します。-->

这两个模块都具有和Cache::Memcached大体相同的接口，所以若已经使用了Cache::Memcached那么就可以进行轻松地替换。Cache::Memcached::Fast封装了libketama，所以若要使用一致性哈希，在创建对象时要通过选项的形式指定ketama_points。

```
my $memcached = Cache::Memcached::Fast->new({
	servers => ["192.168.0.1:11211","192.168.0.2:11211"],
	ketama_points => 150
});
```

<!--また，Cache::Memcached::libmemcachedは，Brian Aker氏によるCのライブラリlibmemcachedを利用しているPerlのモジュールになります。libmemcached自体がいくつかの分散アルゴリズムを組み込んでおり，Consistent Hashingも実装されているので，そのPerlバインディングでもConsistent Hashingがサポートされています。-->

另外，另一个Perl模块Cache::Memcached::libmemcached封装了由Brian Aker开发的C代码库libmemcached[^8]。libmemcached本身包含了多个分布算法，由于也实现了一致性哈希，所以该Perl Bingding也支持一致性哈希。


[^1]: [memcachedを知り尽くす](http://gihyo.jp/dev/feature/01/memcached/0004?page=1)

[^1]: [Cache::Memcached - search.cpan.org](http://search.cpan.org/dist/Cache-Memcached/)

[^2]: [String::CRC32 - search.cpan.org](http://search.cpan.org/dist/String-CRC32/)

[^3]: [mixi Engineers' Blog >> スマートな分散で快適キャッシュライフ](http://alpha.mixi.co.jp/blog/?p=158)

[^4]: [ConsistentHashing - コンシステント・ハッシュ法](http://www.hyuki.com/yukiwiki/wiki.cgi?ConsistentHashing)

[^5]: [libketama - a consistent hashing algo for memcache clients – RJ ブログ - Users at Last.fm](http://www.lastfm.jp/user/RJ/journal/2007/04/10/rz_libketama_-_a_consistent_hashing_algo_for_memcache_clients)

[^6]: [Cache::Memcached::Fast - search.cpan.org](http://search.cpan.org/dist/Cache-Memcached-Fast/)

[^7]: [Cache::Memcached::libmemcached - search.cpan.org](http://search.cpan.org/dist/Cache-Memcached-libmemcached/)

[^8]: [Tangent Software: libmemcached](http://tangent.org/552/libmemcached.html)

