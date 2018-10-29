# Alexa Skills Kit SDK for Node.jsを使ってスキルの永続アトリビュートをAmazon S3に保存する

## はじめに

Alexaスキルでデータ（アトリビュート）の永続化を行いたい場合、Alexa Skills Kit(ASK) SDKが提供するAttributesManagerというAPIを使うとアトリビュートの永続化レイヤーへの保存と取得を簡単に行うことができます。
データの永続化レイヤーにはデフォルトでAmazon DynamoDBが使用されるようになっています。
永続化レイヤーとの接続にはASK SDKのPersistenceAdapterというAPIが使用されていて、DynamoDB以外の任意のデータ永続化サービスを接続することができます。
ASK SDKには、Amazon S3を永続化サービスとして使えるS3PersistenceAdapterが提供されているので、本記事ではこれの使い方を紹介します。

Alexa Skills Kit(ASK) SDKには、スキルで扱うデータ（アトリビュート）を簡単に扱うための仕組みとして[AttributesManager](https://ask-sdk-for-nodejs.readthedocs.io/ja/latest/Managing-Attributes.html#attributesmanager)というAPIが用意されています。
データの永続化レイヤーは、デフォルトでは[Amazon DynamoDB](https://aws.amazon.com/jp/dynamodb/)が利用されるようになっています。
PersistenceAdaterという仕組みを使うと、DynamoDB以外の任意のサービスを永続化サービスとして使える
SDKには　Amazon S3を永続化サービスとして使えるS3PersistenceAdapterが提供されているので、これの使い方を紹介します。

## 動作確認用のスキル

動作確認用のサンプルとして、カフェの注文を受け付けるダミーのスキルを作成しました。
永続アトリビュートの機能を活用して、前回注文したメニューについては「いつもの」と言うだけで注文できるようになっています。

```
一度目の起動
 ユーザー：「アレクサ、サンプルカフェをひらいて」
アレクサ：「いらっしゃいませ。ご注文はどうしますか？コーヒー、紅茶、緑茶、コーラがありますよ」
ユーザー：「コーヒー」
アレクサ：「コーヒーですね。すぐお作りします！」

二度目の起動：
ユーザー：「アレクサ、サンプルカフェをひらいて」
アレクサ：「いらっしゃいませ。ご注文はどうしますか？コーヒー、紅茶、緑茶、コーラがありますよ」
ユーザー：「いつもの」
アレクサ」「わかりました。いつものコーヒーをお作りしますね！」
```

一度も注文したことがない状態で「いつもの」と、言われたら、注文を促すように話しかけます。

```
ユーザー：「アレクサ、サンプルカフェをひらいて」
アレクサ：「いらっしゃいませ。ご注文はどうしますか？コーヒー、紅茶、緑茶、コーラがありますよ」
ユーザー：「いつもの」
アレクサ：「ご来店ははじめてのようですね！ご注文はどうしますか？コーヒー、紅茶、緑茶、コーラがありますよ」
```

## 準備

- npmでs3-persistence-adapterをインストール
    - npm install ask-sdkにはdynamodo-persistent-adapterは含まれるが、s3の方は含まれないので別途インストールが必要
- S3PersistenceAdapterの初期化
- バケットの作成
    - SDKはやってくれないよね？　ー＞　要確認　ー＞　やってくれなかった。

```
2018-10-17T08:04:32.409Z	46c9517a-d1e3-11e8-993e-957f2de41b1f	Error handled: Could not read item (amzn1.ask.account.AFYPWWJJVCHCWPU4G7E3YJ7Y5N23X5FPJUFD6UUPTZA3CC4WUKH75F5WSMNWUYOKUILYTKIX3J3VG2BJ3FPJJVWS5F3ZS6HLGLJTWNCNAJIHW6QDYMX5JR6WXDMIXRD4HEK4QO2YK7L2TU2W53TV7NS3NEBNK4VFAB3PPOSF6ZNXS4W62QYGZ2XBSQG44KPEICOZBAVRID3GMGY) from bucket (alexa-skill-s3-persistent-adapter-sample2): The specified bucket does not exist
```

- IAMロールの設定
    - S3アクセス権限の追加

## 動作確認用スキルの実装

SkillBuilderオブジェクトをビルドする際にS3PersistenceAdapterオブジェクトをセットすれば、
スキルのソースコード全体はGitHubで公開してありますので、ここではAmazon S3を永続化レイヤーに指定して永続アトリビュートの読み書きを行う部分のみを記述します。

S3 Persistence Adapterモジュールの読み込み
```
import * as Adapter from 'ask-sdk-s3-persistence-adapter';
```

S3PersistenceAdapterオブジェクトの生成
```
const config = {
  bucketName: ENV_S3_BUCKET_NAME_FOR_PERSISTENCE_ATTRIBUTE;
};
const S3Adapter = new Adapter.S3PersistenceAdapter(config);
```

```

```

```
const {attributesManager} = handlerInput;

// 永続アトリビュートオブジェクトの取得
const attributes = await attributesManager.getPersistentAttributes();

// 永続アトリビュートオブジェクトへの値のセット
attributes.drink = drink;
attributesManager.setPersistentAttributes(attributes);

// 永続アトリビュートオブジェクトの保存
await attributesManager.savePersistentAttributes();
```


```
スクショ取る
```

- DynamoDBを永続化レイヤーにした時と同じく、attributesのキーはデフォルトではuserId
- キー単位で別ファイル(オブジェクト)になる
- ファイルの中身はJSON形式
- configにObjectKeyGeneratorを指定すると、userId以外をキーにすることもできる。SDKにはdeviceIdをキーにするObjectKeyGeneratorの実装も含まれている
- 自前で独自のキーを扱うObjedtKeyGeneratorを実装することも可能
    - https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs/blob/10780e0e274bb7db531c20885d7b8c10a1e8b31d/ask-sdk-s3-persistence-adapter/lib/attributes/persistence/S3PersistenceAdapter.ts
    - https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs/blob/10780e0e274bb7db531c20885d7b8c10a1e8b31d/ask-sdk-s3-persistence-adapter/lib/attributes/persistence/ObjectKeyGenerators.ts

## Amazon S3に永続アトリビュートを保存する場合の注意点

このへんの話。
- S3の整合性モデルについて
    - https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/Introduction.html

- DynamoDBの整合性モデルについて
    - https://aws.amazon.com/jp/dynamodb/faqs/


## おわりに

最後まで読んでいただきありがとうございました。参考になるところがありましたら、SNSでシェアしていただけると嬉しいです。コメントもお待ちしています！

## 参照

- [ASK SDK V2 for Node.js - PersistenceAdapter の使い方](https://qiita.com/toshimin/items/7c2ff8d61052cf59824a)

- [Alexaスキル開発トレーニングシリーズ 第4回 データの保存 : Alexa Blogs](https://developer.amazon.com/ja/blogs/alexa/post/4144a8ea-7549-4c44-a4bd-e94cb93807ea/chapter4-jp)

- [スキルのアトリビュート — ASK SDK for Node.js ドキュメント](https://ask-sdk-for-nodejs.readthedocs.io/ja/latest/Managing-Attributes.html#id5)

---

## メモ


- githubにソース一式あげる。
    - あげる前に、バケット名のところをLambdaの環境変数から取るように変える
    - READMEにビルド手順を書く。
        - npm install
        - tsc
        - zip
        - lambdaの環境変数にバケット名を入れる
