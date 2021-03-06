## ★今回行いたいこと★


Amazon ECS を使ってコンテナアプリをデプロイする。

![meetup-v8](https://user-images.githubusercontent.com/66664167/87710416-b6b2e300-c7e0-11ea-8e21-ed44795b4cbc.jpg)

## ① EC2 向け IAMロールの作成

コンテナのデプロイに必要になるIAMロールを作成します。
ロールが付与されるとロールに設定されている権限に応じて他のAWSリソースを操作することができるようになります。

まずは コンテナの実行環境になる EC2 に付与するロールを作成しましょう。

AWS マネジメントコンソールにログインして `IAM` のサービスを検索します。

![img001](https://user-images.githubusercontent.com/66664167/87532939-d823aa00-c6ce-11ea-82e7-ee3c4213a863.png)

ロールを選択します。

![img002](https://user-images.githubusercontent.com/66664167/87533962-62204280-c6d0-11ea-9007-7fce1629d90d.png)

ロールの作成を選択。

![img003](https://user-images.githubusercontent.com/66664167/87533998-6e0c0480-c6d0-11ea-9f2f-5e299bbf3495.png)

AWS サービス > Elastic Container Service > EC2 Role for Elasitc Container Service

![img004](https://user-images.githubusercontent.com/66664167/87534060-7feda780-c6d0-11ea-8407-4597c2c6a9b0.png)

![img005](https://user-images.githubusercontent.com/66664167/87534055-7ebc7a80-c6d0-11ea-8375-0b5923ae2e66.png)

`AmazonEC2ContainerServiceforEC2Role` が付与されていることを確認して次のステップ

任意のタグを設定します。
今回はキーに `created-on` を、値に `meetup-v8` を設定しました。

![img007](https://user-images.githubusercontent.com/66664167/87534119-9267e100-c6d0-11ea-82b9-58e3ec44661b.png)

ロール名に `meet-ec2-role` を設定してロールの作成

![img008](https://user-images.githubusercontent.com/66664167/87534122-93007780-c6d0-11ea-9824-56fe9b4a7f26.png)

これで EC2 インスタンスに割り当てるロールが作成できました。


## ② ECS 向け IAMロールの作成

続いて ECS に割り当てるロールを作成しましょう。
EC2 のときと同様に ロールの作成

![img003](https://user-images.githubusercontent.com/66664167/87533998-6e0c0480-c6d0-11ea-9f2f-5e299bbf3495.png)

AWS サービス >  Elasitc Container Service > Elastic Container Service Task

![img004](https://user-images.githubusercontent.com/66664167/87534060-7feda780-c6d0-11ea-8407-4597c2c6a9b0.png)

![img009](https://user-images.githubusercontent.com/66664167/87534124-93007780-c6d0-11ea-80cf-f8c81b133ee3.png)

`AmazonECSTaskExecutionRolePolicy` を検索してチェックします。

![img010](https://user-images.githubusercontent.com/66664167/87534127-93990e00-c6d0-11ea-9814-c14f5fba7b7e.png)

任意のタグを設定します。
こちらもキーに `created-on` を、値に `meetup-v8` を設定しました。

![img007](https://user-images.githubusercontent.com/66664167/87534119-9267e100-c6d0-11ea-82b9-58e3ec44661b.png)

ロール名に `meet-task-role` を設定してロールの作成

![img011](https://user-images.githubusercontent.com/66664167/87534128-93990e00-c6d0-11ea-8f1b-709158345f04.png)

これで ECS に付与するロールが作成できました。
準備完了です。ECSの作成に入りましょう。


## ③ ECSクラスタの作成

ECSクラスターはコンテナを実行するためのマシンリソースを定義します。
マシンが1台だけだと障害があった際にサービスが停止してしまうので、
複数台をまとめてクラスターにするのが一般的です。

サービスから `ecs` を検索します。

![img012](https://user-images.githubusercontent.com/66664167/87534129-9431a480-c6d0-11ea-90c4-11dae250725f.png)

クラスターからクラスターの作成を選択します。

![img013](https://user-images.githubusercontent.com/66664167/87534130-9431a480-c6d0-11ea-88a4-1b74609a696f.png)

EC2 Linux + ネットワーキング を選択します。

![img014](https://user-images.githubusercontent.com/66664167/87534132-94ca3b00-c6d0-11ea-89d4-8bffd295e0f9.png)

以下のパラメータを設定します。

|param|val|
|:--:|:--:|
|クラスタ名|meet-cluster|
|プロビジョニングモデル|オンデマンドインスタンス|
|EC2 インスタンスタイプ|t2.micro|
|インスタンス数|2|
|EC2 Ami Id|Amazon Linux 2 AMI|
|Root EBS Volume Size (GiB)|30|
|キーペア|なし|
|vpc|新しいVPCの作成|
|CIDR ブロック|10.0.0.0/16|
|サブネット1|10.0.1.0/24|
|サブネット2|10.0.2.0/24|
|セキュリティグループ|新しいセキュリティグループ|
|インバウンドルール|0.0.0.0/0|
|ポート|80|
|コンテナインスタンスIAMロール|新しいロールの作成|
|Tags (キー)|created-on|
|Tags (値)|meetup-v8|

![img015](https://user-images.githubusercontent.com/66664167/87534135-9562d180-c6d0-11ea-9fe8-2533a8b5949c.png)

これでECSクラスターが作成できました。
次にタスクの定義を作成しましょう。


## ④ タスク定義の作成

タスク定義ではESCクラスターでコンテナを実行するための設定を作成します。

タスク定義 から 新しいタスク定義の作成 を選択します。
![img016](https://user-images.githubusercontent.com/66664167/87534136-9562d180-c6d0-11ea-9a58-e7727833e2d9.png)

起動タイプは EC2 を選択します。

![img017](https://user-images.githubusercontent.com/66664167/87534139-95fb6800-c6d0-11ea-80cf-b7eba485bc02.png)

以下のパラメータを設定します。

|param|val|
|:--:|:--:|
|タスク定義名|meet-task-def|
|タスクロール|meet-task-role|
|ネットワークモード|default|
|タスク実行ロール|meet-task-role|
|タスクメモリ (MiB)|128|
|タスク CPU (単位)|128|

![img018](https://user-images.githubusercontent.com/66664167/87534141-9693fe80-c6d0-11ea-8e0a-5a86303d8a2a.png)

コンテナの追加からコンテナの定義を追加します。

|param|val|
|:--:|:--:|
|コンテナ名|別途公開します|
|イメージ|別途公開します|
|プライベートレジストリの認証| |
|メモリ制限 (MiB)|128|
|ポートマッピング|host:80, container:3000|

![img019](https://user-images.githubusercontent.com/66664167/87534142-9693fe80-c6d0-11ea-9da9-5867cd2fb71d.png)

その他は未入力で最下部の追加ボタンでコンテナの定義を追加します。

任意のタグを設定します。
例のごとくキーには `created-on` を、値に `meetup-v8` を設定しました。

![img021](https://user-images.githubusercontent.com/66664167/87534146-972c9500-c6d0-11ea-96b6-6b689ef943ab.png)

これでタスク定義が作成できました。
次にサービスを作成します。


## ⑤ サービスの作成

ECSクラスターとタスク定義を関連付けるためにサービスを作成します。

クラスターから meet-cluster の詳細を開きサービスタブの作成を選択します。

![img022](https://user-images.githubusercontent.com/66664167/87534148-97c52b80-c6d0-11ea-8610-10bfb66b3d76.png)

以下のパラメータを設定します。

|param|val|
|:--:|:--:|
|起動タイプ|EC2|
|タスク定義 - ファミリー|meet-task-def|
|タスク定義 - リビジョン|1(latest)|
|クラスター|meet-cluster|
|サービス名|meet-service|
|サービスタイプ|DAEMON|
|最小ヘルス率|0|
|デプロイメントタイプ|ローリングアップデート|

![img023](https://user-images.githubusercontent.com/66664167/87708725-1e1b6380-c7de-11ea-9020-8620b02fa6e3.png)

|param|val|
|:--:|:--:|
|ロードバランシング|なし|
|サービスの検出の統合の有効化| |

![img024](https://user-images.githubusercontent.com/66664167/87534152-985dc200-c6d0-11ea-8b2c-7991c56f19b2.png)

設定を確認してサービスを作成します。
サービスが作成されるとクラスターに登録されているEC2インスタンス上でコンテナが起動します。


## ⑥ インスタンスの確認

クラスターの詳細ページで状態を確認しましょう。

![img026](https://user-images.githubusercontent.com/66664167/87708941-76eafc00-c7de-11ea-96d5-94d63d829b91.png)

実行中のタスクが 2 になっていますね。
次にコンテナが実行されているEC2インスタンスを確認しましょう。

メニューから `EC2` のサービスを検索してインスタンスを表示します。

![img028](https://user-images.githubusercontent.com/66664167/87709368-21631f00-c7df-11ea-867a-5213e4163456.png)

パブリックIPにアクセスして画面を確認しましょう。

`http://xxx.xxx.xxx.xxx/static/index.html`


## ⑦ リソースの削除

最後に画面の表示が確認できたら課金が発生しないうちに今回作成したリソースを削除しましょう。

1. ECSリソース
    + ECSクラスタ
2. IAMリソース
    + IAMロール


**ECSリソース**

ECSクラスタを削除しましょう。
EC2インスタンスが起動したままだと課金が発生してしまいます。
サービスから `ecs` を検索します。

![img012](https://user-images.githubusercontent.com/66664167/87534129-9431a480-c6d0-11ea-90c4-11dae250725f.png)

クラスターの詳細画面を表示します。
![img029](https://user-images.githubusercontent.com/66664167/87709678-92a2d200-c7df-11ea-8d3f-6b80789fa937.png)

クラスターを削除しましょう。
![img030](https://user-images.githubusercontent.com/66664167/87709709-9afb0d00-c7df-11ea-83f6-ac98b5938ddc.png)


**IAMリソース**

作成したロールを削除します。 
IAMリソースに課金は発生しません。

`IAM` のサービスを検索します。
![img001](https://user-images.githubusercontent.com/66664167/87532939-d823aa00-c6ce-11ea-82e7-ee3c4213a863.png)

ロールを選択します。
![img002](https://user-images.githubusercontent.com/66664167/87533962-62204280-c6d0-11ea-9007-7fce1629d90d.png)

`meet-ec2-role` を検索して削除します。
![img031](https://user-images.githubusercontent.com/66664167/87709730-a3ebde80-c7df-11ea-8a7e-32d7a5f4134e.png)

`meet-task-role`, `ecsInstanceRole` も同様に検索して削除します。


以上となります。
お疲れさまでした。
