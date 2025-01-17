## Network > VPC > コンソール使用ガイド

この文書ではコンソールでVPCを扱う時に必要な内容を記述しています。

## VPC

VPCは複数のサブネットを持つことができるため、サブネットを分割して使用する場合、十分な大きさのネットワークを設定する必要があります。VPCネットワークは[CIDR 表記](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)を使用して記述できます。全てのVPCは[プライベートネットワーク](https://en.wikipedia.org/wiki/Private_network)を構成できる下記3つのアドレス領域に存在する必要があり、リンクローカルアドレスは使用できません。また少なくとも24bit-256個より大きなネットワーク領域を指定する必要があります。

### プライベートネットワーク

RFC1918 | IPアドレス領域 | 使用可能なアドレス個数
-------- | ---------- | -----------------
24bit block | 10.0.0.0/8 | 16,777,216
20bit block | 172.16.0.0/12 | 1,047,576
16bit block | 192.168.0.0/16 | 65,536

### リンクローカルアドレス

169.254.0.0/16に含まれる65,536個のIPアドレスは使用できません。

### 例

例 | 使用可否
-------- | ---------- 
10.0.0.0/8 | 使用できます。
10.0.0.0/16 | 使用できます。
10.0.0.0/24 | 使用できます。
10.0.0.0/28 | 使用できません。範囲が小さすぎます。
172.16.0.0/16 | 使用できます。
172.16.0.0/8 | 使用できません。使用可能な範囲を超えました。
192.168.0.0/16 | 使用できます。基本使用範囲に指定されます。
192.168.0.0/24 | 使用できます。
192.253.0.0/24 | 使用できません。使用可能な範囲を超えました。 

<br>


最初にComputeとNetwork商品を使用すると、下記のような項目を自動的に構成します。

項目 | 名前 | 要約
-------- | ---------- | --------------
VPC | Default Network | 192.168.0.0/16範囲のVPC 1個が作成されます。
サブネット | Default Network | 192.168.0.0/24範囲のサブネット1個が作成されます。
ルーティングテーブル | vpc-[id] | VPC IDの一部を名前に持つルーティングテーブル1個が作成されます。
インターネットゲートウェイ | ig-[id] | ルーティングテーブルIDの一部を名前に持つインターネットゲートウェイ1個が作成されます。
セキュリティーグループ | default | defaultという名前を持つセキュリティーグループ1個が作成されます。

初期構成ではなく、VPCを追加する場合は下記の項目を構成します。

項目 | 名前 | 要約
-------- | ---------- | --------------
VPC | 指定した名前 | 指定した範囲のVPC 1個が作成されます。
サブネット | - | 作成されません。
ルーティングテーブル | vpc-[id] | VPC IDの一部の名前を持つルーティングテーブル1個が作成されます。
インターネットゲートウェイ | - | 生成されないため、別途生成後に接続する必要があります。
セキュリティーグループ | - | 追加で生成されません。

VPCと各項目で利用可能な上限値は下記のとおりです。

項目 | 最大値
-------- | ---------- 
VPC | 3
サブネット | VPCあたり10 
インターネットゲートウェイ | 3
Floating IP | 制限なし
ルーティングテーブル | VPCあたり10 
リルート | ルーティングテーブルあたり10
ピアリング | 制限なし
NATゲートウェイ | 3


> [参考]
VPCを削除するには、サブネットを全て削除できる状態でのみ削除可能です。その場合はサブネット、ルーティングテーブル、インターネットゲートウェイも一緒に削除されます。

* VPCは他のVPCと完全に隔離されているため、トラフィック上の安全性が保たれます。

* VPCはプライベートネットワークのため、インターネットから直接アクセスできません。

* VPC内の全ての機器においてVLANの構築はできません

* 異なるリージョンを跨ぐトラフィックについてはローカル通信を提供しません。

* インターネットゲートウェイの設定をしないと、 VPC内の全てのインスタンスがインターネットに接続されません。

* 過度に転送される"ブロードキャスト、マルチキャスト、Unknownユニキャスト"は予告なく遮断されることがあります。


## サブネット

VPCはサブネットに分けて、小さなネットワークを複数構成できます。ただしサブネットの場合はVPCアドレスの範囲に含まれ、アドレスの長さがそれ以下である必要があります。例えば192.168.0.0/16の場合、192.168.0.0 ~ 192.168.255.255まで、合計65536個のIPアドレスを使用できます。また最も小さなサブネットは28bitで、これより小さく構成することはできません。サブネットもVPC同様、CIDR表記を使用します。

サブネットが作成されると、VPCに含まれるデフォルトのルーティングテーブルに自動的に接続されます。このとき、ゲートウェイIPは自動的に指定されます。 

> [参考]
サブネットは、インスタンスやロードバランサーなどが該当のサブネットに含まれていない空のサブネットの場合にのみ削除できます。また、サブネットが接続されているルーティングテーブルにそのサブネットに向かうリルートがないことを確認します。

* インスタンスが生成されると、指定したサブネットから1個のIPアドレスを割り当てられます。 (Fixed IPと呼びます)

* インスタンスが起動すると、DHCPを通してIPアドレスがインスタンスに適用されます。

* サブネットのアドレス範囲を修正できません。

* 同じVPC内で異なるサブネットの範囲が重複するように作成することはできません。

* 異なるVPCにおいてはサブネットの範囲を重複することができます。

* インスタンスに割り当てられたMACアドレスではない場合、ネットワーク上で遮断される場合があります。したがってVPNサービスをインスタンスで駆動する場合、動作しない場合があります。

* インスタンスに複数のサブネットを接続する場合、インスタンス内のOSで適切なルーティング設定が必要です。

* 同じVPC内の2個のサブネットは完全に隔離されてはいません。セキュリティーグループを使用してインスタンスを保護してください。

* サブネットは異なるアベイラビリティーゾーンにまたがってローカル通信をサポートします。ローカル通信は課金しません。

サブネットの「静的ルート」設定を利用して、サブネット内のインスタンスが起動した時にインスタンス内のルーティングテーブルに設定する必要があるルーティングルールを伝えることができます。

* 「静的ルート」に登録したルーティングルールは、インスタンスがリクエストしたDHCPリクエストに対するレスポンスの"classless-static-routes"オプションに含まれて送信され、インスタンスで起動中のDHCPクライアントでこのオプションの内容をルーティングテーブルに登録します。

* "classless-static-routes"オプションの反映方式はインスタンスで実行中のOSの種類、ディストリビューション、またはDHCPクライアントバージョンによって異なりますが、一般的にインスタンスの起動などHCPクライアントが初めて起動したときに反映され、DHCPリースの更新時には反映されません。したがってサブネットの「静的ルート」を編集した場合、実行中のインスタンスには変更内容がすぐに適用されない可能性があるため、なるべく実行中の対象インスタンスを再起動することを推奨します。

* 「静的ルート」はルーティングするパケットの宛先CIDRと、対象パケットを転送するゲートウェイ情報で構成されます。<br>CIDRが"0.0.0.0/0"の静的ルートを作成すると、インスタンスのデフォルトゲートウェイをサブネットのゲートウェイ以外のIPに変更できます。<br>ゲートウェイはルーティングテーブルの「ルート」とは異なりテキストで入力し、サブネット内にまだ割り当てられていないIPも指定できます。

## インターネットゲートウェイ

インターネットゲートウェイはルーティングテーブルと接続できます。プライベートネットワークで作られたVPCは外部接続ができませんが、インターネットゲートウェイを利用してインターネットにアクセスできます。インターネットに接続するために各インスタンスは"基本ゲートウェイ"をサブネットのゲートウェイアドレスに設定する必要がありますが、NHN Cloudではこれを自動的に処理します。インターネットゲートウェイを生成する場合、外部ネットワークを選択する必要がありますが、NHN Cloudでは現在"public_network" 1個のみ運営しています。

* インターネットゲートウェイアドレスは、インスタンスが生成されたり、VPCがインターネット接続を要求される状態に自動的に割り当てられ、任意で変更できません。

* インターネットゲートウェイアドレスでインスタンスにアクセスできません。

* インターネットゲートウェイアドレスに流入するトラフィックは全て遮断されます。

* インターネットに接続されたインスタンスがインターネットの方向にトラフィックを誘発すると、使用した分だけ課金します。

* インスタンス間のローカル通信には課金しません。

### サーバーメンテナンスのためのインターネットゲートウェイ再起動ガイド

NHN Cloudは周期的にインターネットゲートウェイサーバーのソフトウェアをアップデートして、基本インフラサービスのセキュリティと安定性を向上させています。
インターネットゲートウェイサーバーのメンテナンスのために、メンテナンス対象サーバーで起動中のインターネットゲートウェイは再起動を行い、メンテナンスが完了したインターネットゲートウェイサーバーに移動する必要があります。

再起動が必要なインターネットゲートウェイは名前の横に**!再起動**ボタンが表示され、このボタンを使用して再起動できます。

メンテナンス対象に指定されたインターネットゲートウェイがあるプロジェクトに移動し、次の手順で再起動を実行します。

1. メンテナンス対象のインターネットゲートウェイを確認します。
    名前の横に**!再起動**ボタンがあるインターネットゲートウェイがメンテナンス対象のインターネットゲートウェイです。
    ![image-001](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-jp-001.png)
    **!再起動**ボタンにマウスオーバーすると、詳細なメンテナンス日程を確認できます。 
    ![image-002](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-jp-002.png)
    
2. メンテナンス対象のインターネットゲートウェイを選択し、名前の横にある**!再起動**ボタンをクリックします。
    再起動が完了するまで、メンテナンス対象のインターネットゲートウェイを使用するインスタンスのインターネット接続が遮断されるため、サービスに影響を与えない時間に実行してください。 
   ただし、Floating IPを接続したインスタンスはインターネットゲートウェイの再起動による影響を受けません。
   
3. インターネットゲートウェイの再起動を確認するウィンドウが表示されたら**確認**ボタンをクリックします。
    ![image-003](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-jp-003.png)

4. 状態表示灯が緑に変わり、**!再起動**ボタンが消えるまで待機します。
    インターネットゲートウェイの状態表示灯が変わらない場合や、**!再起動**ボタンが消えない場合は、「更新」を行ってみてください。
    ![image-004](http://static.toastoven.net/prod_vpc/ConsoleGuide/ig_planned_migration_guide-jp-004.png)

インターネットゲートウェイの再起動中は、該当インターネットゲートウェイを操作できません。
インターネットゲートウェイが正常に再起動しなかった場合は、自動的に管理者に報告され、NHN Cloudから別途連絡いたします。


## NATゲートウェイ
韓国(坪村)リージョンでのみ提供する機能です。
NATゲートウェイはルーティングテーブルのルート設定からゲートウェイに指定できます。 NATゲートウェイをルーティングテーブルのゲートウェイに設定すると、NATゲートウェイのFloating IPをソースIPに変換してインターネットにアクセスできます。

| 項目    | 説明                                                       |
| --------|------------------------------------------------------------ |
| 名前    | NATゲートウェイ名を指定できます。 |
| VPC      | NATゲートウェイを作成するVPCを指定します。 |
| サブネット   | 選択したVPCのサブネットの中からインターネットゲートウェイに接続されているルーティングテーブルに接続されたサブネットを指定します。条件を満たさないサブネットではNATゲートウェイを作成できません。  |
| Floating IP  | NATゲートウェイに割り当てるFloating IPを指定します。インターネットにアクセスする時、このIPがソースIPに変換されます。 |
| 説明    | NATゲートウェイの説明を追加できます。  |

* NATゲートウェイのVPC、サブネット、 Floating IPは変更できません。

* NATゲートウェイに設定されたFloating IPは接続を解除できません。 NATゲートウェイが削除されると自動的に接続が解除されます。

* NATゲートウェイアドレスでインスタンスにアクセスできません。

* NATゲートウェイアドレスに流入するトラフィックは全て遮断されます。

* 1つのNATゲートウェイは、同じVPC内の複数のルーティングテーブルでゲートウェイに指定できます。

* NATゲートウェイの設定時、指定したサブネットと異なるサブネットに接続されたルーティングテーブルもNATゲートウェイをゲートウェイに指定できます。

* ネットワークACLが適用されます。

* ルート設定でIP Prefix 0 (/0)ではないルート対象CIDRに対して、「NATゲートウェイ」をゲートウェイに設定すると、インスタンスにFloating IPが設定されていても「NATゲートウェイ」で通信します。



## ルーティングテーブル

ルーティングテーブルはVPCと同時に生成され、VPCが削除されると一緒に削除されます。ルーティングテーブルはVPCに複数生成することができ、基本ルーティングテーブルでなければ明示的に削除できます。サブネットは少なくとも1つのルーティングテーブルに接続されている必要があり、複数のルーティングテーブルが1つのインターネットゲートウェイを共有できません。

ルーティングテーブルリストを指定する場合、詳細画面に要約された情報が表記され、 "ルート"タブを利用して経路を追加できます。

> [参考]
経路を追加する場合、VPC内に到達可能な領域を指定すると追加できます。それ以外は失敗メッセージが発生します。

* ルーティングテーブルに含まれるサブネットのゲートウェイは自動的に追加されます。

* "基本ルーティングテーブル"は削除できません。VPCを削除すると削除されます。

* サブネットのゲートウェイとインターネットゲートウェイはルートリストから削除できません。

* ルーティングテーブルとインターネットゲートウェイの接続が切れると、インターネット接続が切断されます。

ルーティングテーブルは、作成される場所に応じて「分散型ルーティング(DVR)」方式と「中央集中型ルーティング(CVR)」方式で作成できます。

* 分散型ルーティング(DVR)方式

    * ルーティングテーブルに接続されているサブネットに含まれるインスタンスが配置されているハイパーバイザごとにルーティングテーブルが作成される方式です。ルーティングテーブルが使用される場所に均等に分散されるため、安定性と高可用性が得られます。

* 中央集中型ルーティング(CVR)方式

    * 中央に1つのルーティングテーブルが作成され、ルーティングテーブルに接続されたサブネットに含まれるインスタンスのトラフィックが1つのルーティングテーブルに集中する方式です。<br>ゲートウェイやファイアウォールなど、1つの地点を通じてトラフィックを制御する場合に使われます。

分散型ルーティング(DVR)方式は、NHN Cloudが基本的に提供する方式です。安定性、高可用性およびトラフィック分散の利点があり、特別な場合を除いて分散型ルーティング(DVR)方式を使用することを推奨します。

ルーティングテーブルの方式を変更することも可能です。変更時にルーティングテーブルを再構成し、完了するまで約1分間、外部通信およびサブネット間の通信が切断されるため注意してください。

## ピアリング

ピアリングは異なる2個のVPCを接続する機能です。通常VPCはネットワーク領域が異なるため、互いに通信が不可能で、 Floating IPを利用して接続できますが、これはネットワーク使用量に応じて追加課金されます。したがって2個のVPCを接続する機能を提供しますが、これをピアリングと呼びます。

> [参考]ピアリングは異なる2個のVPCを接続します。他のVPCを渡って、また別のVPCへの接続はサポートしていません。 A <-> B <-> C接続でAとCは接続できません。

* 2個のVPCのIPアドレス領域は重なると使用できません。<br>
IPアドレス領域が、片方がもう片方と包含関係となってはならず、この場合はピアリング作成が失敗します。
* IPアドレス領域のサイズは関係なく、 "基本ルーティングテーブル"に接続されていないサブネットでは通信ができません。

    * 韓国(坪村)リージョンはピアリングを作成した後、ピアリングした両側のVPCのルーティングテーブルに別のルートを設定すると通信が可能です。
        * ルートの「対象CIDR」に相手VPCのIPアドレス領域を入力し、「ゲートウェイ」リストからピアリングの名前を持つ"PEERING"項目を選択してルートを追加します。
        * ルートを追加したルーティングテーブルに接続されたサブネットでのみ通信が可能です。
        * 「基本ルーティングテーブル」ではなく、ルーティングテーブルもルートを追加すると、ルーティングテーブルに接続されているサブネットからピアリング通信が可能です。
        * ピアリングの作成時にサブネットではないVPCを指定すると、ピアリングは作成されますが、実際のピアリングを接続しないため、ルーティングテーブルにルートを設定できません。そのVPCに1つ以上のサブネットを作成した後にルート設定が可能です。


## コロケーションゲートウェイ

コロケーションゲートウェイは、NHN Cloudがハイブリッドで提供する顧客のネットワークを接続するための機能で、韓国(坪村)リージョンにのみ提供されます。NHN Cloudでハイブリッドサービスを使用する場合に限りNHN Cloud Zoneが提供され、コロケーションゲートウェイを利用して構成したVPCにNHN Cloud Zoneを直接接続できます。

* 1つのVPCは1つのNHN Cloud Zoneと1:1で接続されます。
* 接続と同時に課金されます。
