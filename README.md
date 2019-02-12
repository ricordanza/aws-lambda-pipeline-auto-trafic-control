Lambda CI/CD サンプル オートトラフィックコントロール
====

## Description
CodePipelineを利用して CodeCommit → CodeBuild → CFn → Lambda -> CodeDeployの流れでLambdaを更新するサンプル  
Lambdaのバージョン管理を行いAPIGatewayとの間でAliasを利用しCodeDeployでトラフィックコントロールを行う  
AWS SAM使う方がトラフィックコントロールをシンブルに設定できるのでケースバイケースで使い分けること  
**2019/0212 AWS::Lambda::VersionはCFnによるアップデートをサポートしておらず、本実装ではLambdaのバージョンとAliasを更新する事ができない**  
**その為、LambdaのVersion作成を専用のLambdaで行い、そのままDeployAPIを呼び出してトラフィックコントロールをしている**
**Piplineでcreate_deploy相当のアクションの設定を見つける事ができなかったための妥協策。[ドキュメント](https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/reference-pipeline-structure.html#action-requirements)**

## Install
### Preparation
cfn/_variable.shを自分のインストールしたい環境の設定に合わせて編集する

### Create Resource
```
$ cd /PATH/TO/THIS
$ sh ./cfn/_deploy.sh
```
サンプル環境を用意する  

## Usage
### サンプルコードをCodeCommitにプッシュ
```
$ cd /PATH/TO/THIS
$ git remote add origin ssh://git-codecommit.{$AWS_DEFAULT_REGION}.amazonaws.com/v1/repos/{$NAME}-{$STAGE}
$ git add .
$ git commit -m "first commit"
$ git push origin master
```
これでmasterへのコードの変更を起点としてCodePipeline({$NAME}-{$STAGE}-pipeline)が動き出す  

## Uninstall
```
$ cd /PATH/TO/THIS
$ sh ./cfn/_remove.sh
```  

## Limitations
- APIGWに対するアクセス制限などは未実装
- LambdaのAlias間のトラフィックコントロールはPipelineの管理外となっている（エラー検知がPiplineからできない）ため、厳密な管理を望むならこの部分は手で追加した方が良いかも
- 初回実行時はバージョンが複数存在しないのにトラフィックをコントロールしようとしてエラーになるが割り切り

## Versioning
バージョンはv1.2.3という形でtagを付けることにより管理する  
v: 接頭辞として固定  
1: メジャバージョン  
2: マイナバージョン  
3: ビルドバージョン  
masterに 修正が加えられる毎にビルドバージョンを増加  
後方互換性がない修正が入る場合にはマイナバージョンを増加させて、ビルドバージョンは0にリセットする  

## Contribution
- Fork it
- Create your feature branch (git checkout -b my-new-feature)
- Commit your changes (git commit -am 'Add some feature')
- Push to the branch (git push origin my-new-feature)
- Create new Pull Request
