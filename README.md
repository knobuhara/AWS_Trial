# AWS_Trial
- [宣原AWSネットワーク構成図]
  - 稼働中サービス一覧：
    -  会員登録サンプル(宣原オリジナル)：http://nobuhara.tk/PHPWebApp/index.php
    -  Wordpressサンプル(宣原オリジナル)：http://nobuhara.tk/wordpress/
    -  mysqlツール：http://nobuhara.tk/phpMyAdmin/index.php
  - ※ssl設定済み、上記アクセスは全てhttpsにリダイレクトされる
<img src="https://user-images.githubusercontent.com/88915966/146662342-120131de-97ed-45cb-b110-2430983ac577.png">

#### EC2
- インスタンス作成
  - リージョン：オレコン  ※アメリカリージョンだと最新サービスを受けられる
  - インスタンスタイプ：t2.micro  
  - EBS：サイズ8Gib 
  - アベイラビリティーゾーン：us-west-2a
  - プラットフォーム：Amazon Linux
- セキュリティーグループ①
  - インバウンドルール HTTP：::/0
  - インバウンドルール SSH：0:0:0:0/0
  - インバウンドルール HTTPs：0:0:0:0/0
  - インバウンドルール HTTP：0:0:0:0/0
- セキュリティーグループ②
  - インバウンドルール MyQLAurora：カスタム/セキュリティーグループID①
#### RDB
- データベース作成
  - エンジン：MySQL Community
  - リージョンとAZ：us-west-2a
  - サイズ：db.t2.micro
- サブネットグループ作成
  - AZゾーン：us-west-2b CIDRブロック：10.0.21.0/24
  - AZゾーン：us-west-2a CIDRブロック：10.0.20.0/24
  - AZゾーン：us-west-2a CIDRブロック：10.0.10.0/24
- DB接続（えがおAWS）
  - DB名：egao_aws_dev
  - 認証：egaoadmin/ Egao241241
  - エンドポイント：egao-aws-mysql.cnw7dr5ayn8e.us-west-2.rds.amazonaws.com
#### VPC
- バーチャルネットワーク作成
  - IPv4 CIDR：10.0.0.0/16
  - インターネットゲートウェイ作成
- ルートテーブル①
  - 送信先：10.0.0.0/16 ターゲット：Local
  - 送信先：0.0.0.0/0 ターゲット：igw-xxxxxxx
  - サブネット(アプリケションサーバ用)：10.0.10.0/24
- ルートテーブル②
  - 送信先：10.0.0.0/16 ターゲット：Local
  - サブネット(DBサーバ用)：10.0.20.0/24
  - サブネット(DBサーバ、サブ用)：10.0.21.0/24
#### Route53
- ホストゾーン作成
  - Domain名：nobuhara.tk
  - レコードタイプA：ルーティングポリシー：シンプル　トラフィックルーティング先：EC2グローバルIPアドレス
  - レコードタイプNS：ルーティングポリシー：シンプル　トラフィックルーティング先：ドメイン業者から割り当てられたnameサーバー名
  - レコードタイプSOA：ルーティングポリシー：シンプル　トラフィックルーティング先：＊＊＊
#### ロードバランサーとSSL/TLS設定
- 証明書の作成　Certificate Manager
  - Domain名：nobuhara.tk
  - Route53レコード作成（CNAMEレコード）
- ロードバランサー設定
  - Target groups/Target type:Instance
  - Target groups/Protocol : Port HTTP: 80
  - リスナー/HTTP : 80 -> Redirect to https 443
  - リスナー/HTTPs : 443 -> Target groups指定、certificateをnobuhara.tkの証明書を指定
- Route53設定
  - AレコードはIPアドレス指定から、エイリアス指定に変更、ロードバランサーを指定
