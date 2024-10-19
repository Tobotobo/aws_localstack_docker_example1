# aws_localstack_docker_example1

## 概要
* AWS エミュレータの LocalStack を Docker Compose で構築する
* aws-cli を使って構築した LocalStack 上に S3 バケットを作成する

LocalStack  
https://www.localstack.cloud/  
AWS のクラウドサービスをローカルマシーンでエミュレートできるサービス  

LocalStack - docs - Starting LocalStack with Docker-Compose  
https://docs.localstack.cloud/getting-started/installation/#docker-compose  

dockerhub - localstack/localstack  
https://hub.docker.com/r/localstack/localstack  

## 環境
* Ubuntu 24.04
* Docker version 27.3.1, build ce12230
* aws-cli/2.18.6 Python/3.12.6 Linux/6.8.0-45-generic exe/x86_64.ubuntu.24

## 準備

### AWS CLIの最新バージョンのインストールまたは更新  
https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html  

Command line installer - Linux x86 (64-bit)
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

```
$ aws --version
aws-cli/2.18.6 Python/3.12.6 Linux/6.8.0-45-generic exe/x86_64.ubuntu.24
```

### プロファイル作成
※ダミーの設定で OK
```
$ aws configure --profile localstack
AWS Access Key ID [None]: dummy
AWS Secret Access Key [None]: dummy
Default region name [None]: ap-northeast-1                
Default output format [None]: json
```

### LocalStack サーバー起動

```
$ docker compose up
[+] Running 26/1
 ✔ localstack Pulled                                                                                                                                                                                             14.9s 
[+] Running 3/3
 ✔ Network aws_localstack_docker_example1_default             Created                                                                                                                                             0.1s 
 ✔ Volume "aws_localstack_docker_example1_localstack-volume"  Created                                                                                                                                             0.0s 
 ✔ Container localstack-main                                  Created                                                                                                                                             0.8s 
Attaching to localstack-main
localstack-main  | 
localstack-main  | LocalStack version: 3.8.1
localstack-main  | LocalStack build date: 2024-10-08
localstack-main  | LocalStack build git hash: 529aba7d8
localstack-main  | 
localstack-main  | Ready.
```

### LocalStack を GUI で操作・確認
※要 LocalStack アカウント  
※aws-cli を使用した CUI の操作については後述

* LocalStack のサイトに Sign in すると Dashboard が表示される
* 左側のメニューの LocalStack Instances の Default Instance から localhost:4566 で接続可能な LocalStack に対して状態の確認や操作が可能  
  * localhost:4566 以外に接続したい場合は、 LocalStack Instances をクリックすると表示される LocalStack Instance Management から Add Bookmark で追加する
* 下図は Resource Browser で S3 バケットを表示した際のキャプチャ  
  * 右上の Region の指定を S3 バケットを作成したリージョンにすることを忘れずに！(2枚目の画像)  
  * 最新の状態が表示されない場合は、Buckets の横の更新ボタンを押す(2枚目の画像)
* 画面にある Create ボタンから GUI でリソースの作成が可能(2枚目の画像)
  * その隣の Actions にはリソースの削除などがある(2枚目の画像)

![alt text](images/README/image.png)

![alt text](images/README/image-1.png)

## S3 バケット

LocalStack - docs - Simple Storage Service (S3)  
https://docs.localstack.cloud/user-guide/aws/s3/  

### S3 バケット作成
```
$ aws s3 mb s3://sample-bucket --endpoint-url=http://localhost:4566 --profile localstack
make_bucket: sample-bucket
```

### S3 バケット一覧取得
```
$ aws s3 ls --endpoint-url=http://localhost:4566 --profile localstack
2024-10-20 00:19:53 sample-bucket
```

### S3 バケットにファイルをアップロード
```
$ aws s3 cp ./index.html s3://sample-bucket/ --endpoint-url=http://localhost:4566 --profile localstack
upload: ./index.html to s3://sample-bucket/index.html      
```

http://localhost:4566/sample-bucket/index.html  

![alt text](images/README/image-2.png)

※LocalStack だとデフォルトで外部からの接続が許可されてる？（要確認）

### S3 バケット内のオブジェクト一覧取得
```
$ aws s3 ls s3://sample-bucket/ --endpoint-url=http://localhost:4566 --profile localstack
2024-10-20 01:36:10        245 index.html
```

### S3 バケット内のオブジェクト削除
```
$ aws s3 rm s3://sample-bucket/index.html --endpoint-url=http://localhost:4566 --profile localstack
delete: s3://sample-bucket/index.html
```

### S3 バケットの削除
```
$ aws s3 rb s3://sample-bucket/ --force --endpoint-url=http://localhost:4566 --profile localstack
remove_bucket: sample-bucket
```
※`--force` バケット内にオブジェクトが残っていても強制的に削除（残っていたオブジェクトは自動削除）