# SORACOM UG 信州 #7 2019/8/31

## ハンズオンの内容
SORACOM LTE-M Button for Enterprise（通称：しろボタン）を使い、ボタンを押したらTwilio経由で電話が掛かってくるようにする仕組みを作成します
[SORACOM LTE-M Button powered by AWS（通称：あのボタン）を使う場合はこちらです](https://kizawa2020.github.io/iot-showcase/events/soracomug-shinshu6/)

### ハンズオンの構成
![soracomug_shinshu7 / Twilio全体像](https://kizawa.info/wp-content/uploads/2018/11/soracombutton-arch3.png)

## 本日の貸し出し機材
* SORACOM LTE-M Button for Enterprise（と、ボタンに紐付くSORACOMアカウント）

## お客様の持ち物
* Wifi に繋がるPC
* クレジットカード（AWSアカウント作成に必要になります）
* 会場で着信可能な携帯電話
   * AWSのアカウント作成時に電話音声による認証が必要となります(非通知からの電話に対する着信拒否設定の解除が必要)
   * Twilioのアカウント作成時にSMSによる認証が必要となります
   * ボタンを押したら電話がかかってくる宛先となります

## 参加費用
無料（ただしAWSの利用料金、数円が発生します）

## 目次
1. [AWS アカウントの作成](#content1)
2. [Twilio アカウントの作成](#content2)
3. [Twilio セットアップ](#content3)
4. [Twilio Studio Flowの作成](#content4)
5. [AWS Lambda関数の設置](#content5)
6. [AWS IAMの設定](#content6)
7. [SORACOM Funkの設定](#content7)
8. [動作確認](#content8)
9. [お片付け](#contentE)

## プログラム
<h3 id="content1">1. AWS アカウントの作成</h3>
(※既にアカウントをお持ちの方は読み飛ばして下さい)  
<a href="https://aws.amazon.com/jp/register-flow/" target="_blank">AWS アカウント作成の流れ (AWS のページに飛びます)</a> から進み作成してください。  

**アカウント作成時のポイント**  
電話音声による認証が必要となります
「非通知」からの着信となるため、必要ならば非通知着信が可能になるようにしてください

**AWSサポートプランについて**  
理由がない限り「ベーシックプラン」を選択してください。それ以外のプランは費用がかかります

<h3 id="content2">2. Twilio アカウントの作成</h3> (※既にアカウントをお持ちの方は読み飛ばして下さい)

TwilioはAPIによってコントロールできる電話サービスです。  
登録することで電話番号を取得でき、その電話番号での受発信がAPIでコントロールできます。今回は発信のコントロールを行います。

**アカウントの取得**  
[Twilioのページ](https://twilio.kddi-web.com/)を開き、無料サインアップのアイコンをクリックします。  
![soracomug_shinshu6 / 2-1 Twilio-Top](https://kizawa.info/wp-content/uploads/2018/11/twilio-1-768x377.png)

利用規約に同意します。  
![soracomug_shinshu6 / 2-2 Twilio-signup1](https://kizawa.info/wp-content/uploads/2019/03/twilio-1.png)

氏名・メールアドレス・パスワードの情報を入力します。  
パスワードは14桁以上必要となります。  
![soracomug_shinshu6 / 2-3 Twilio-signup2](https://kizawa.info/wp-content/uploads/2019/03/twilio-2.png)

身元検証用に携帯電話の電話番号の入力を行います。  
携帯電話番号は最初のゼロを取り、80xxxxxxxx のように入力して下さい。  
なお、トライアル（無料）アカウントではここで登録した電話番号のみに発信が可能であるため、今回のハンズオンで電話を着信したい携帯電話の番号を入力して下さい。  
（有料アカウントにアップグレードすることで任意の番号に発信可能となります）  
![soracomug_shinshu6 / 2-4 Twilio-signup3](https://kizawa.info/wp-content/uploads/2019/03/twilio-3.png)

入力した携帯電話に対してSMSで検証コードが送信され認証が完了です。  
![soracomug_shinshu6 / 2-5 Twilio-Validate1](https://kizawa.info/wp-content/uploads/2018/11/twilio-2.png)

チュートリアル画面が出ます。今回は「Skip to dashboard」をクリックしてください。  
![soracomug_shinshu6 / 2-6 Twilio-Validate2](https://kizawa.info/wp-content/uploads/2019/03/twilio-4.png)

トップ画面が表示されます。  
![soracomug_shinshu6 / 2-7 Twilio-Validate3](https://kizawa.info/wp-content/uploads/2019/03/twilio-5.png)

<h3 id="content3">3. Twilio セットアップ</h3>

**電話番号の取得**  
Twilioで利用できる電話番号を取得します。  
[電話番号の取得画面](https://jp.twilio.com/console/phone-numbers/getting-started)にアクセスします。  
![soracomug_shinshu6 / 3-1 Twilio-GetNumber1](https://kizawa.info/wp-content/uploads/2018/11/twilio-3-768x208.png)

「最初のTwilio電話番号を取得」をクリックし電話番号を取得します。  
気に入らない場合は選び直しが可能です。  
なおトライアルアカウントで取得可能な電話番号は050のIP電話番号のみとなります。  
![soracomug_shinshu6 / 3-2 Twilio-GetNumber2](https://kizawa.info/wp-content/uploads/2018/11/twilio-4.png)

電話番号が取得できました。終了ボタンをクリックします。  
作成した電話番号はメモ帳等にコピーしておきます。  
![soracomug_shinshu6 / 3-3 Twilio-GetNumber3](https://kizawa.info/wp-content/uploads/2019/03/twilio-6.png)

**クレデンシャル情報の確認**  
AWS側からAPIで呼び出しできるよう、クレデンシャル情報を確認しておきます。  
[Twilioの設定画面](https://jp.twilio.com/console/project/settings)にアクセスします。  
![soracomug_shinshu6 / 3-4 Twilio-Credential](https://kizawa.info/wp-content/uploads/2018/11/twilio-5-768x533.png)  
ライブクレデンシャルの ACCOUNT SID と AUTH TOKENの内容をメモ帳等にコピーしておきます。この情報がLambda関数からの呼び出しに必要になります。

<h3 id="content4">4. Twilio Studio Flowの作成</h3>
電話を受けた後、任意のメッセージを再生できるよう、Studio Flowの作成を行います。

[Studio Dashboard](https://jp.twilio.com/console/studio)にアクセスし、Create a flowをクリックします。  
![soracomug_shinshu6 / 4-1 Twilio-Studio](https://kizawa.info/wp-content/uploads/2018/11/tstudio-1-1024x371.png)

続けてフローの名称を入力します。  
![soracomug_shinshu6 / 4-2 Twilio-Studio](https://kizawa.info/wp-content/uploads/2018/11/tstudio-2.png)

作成方法はStart from scratchとします。  
![soracomug_shinshu6 / 4-3 Twilio-Studio](https://kizawa.info/wp-content/uploads/2018/11/tstudio-3.png)

下のようなフロー作成画面が開きます。  
右側のWIDGET LIBRARYから、”Make Outgoing Call” と “Say/Play” のウィジェットをドラッグ、フローを繋げて以下のように設定します。  
また、Say/Playのウィジェットでは読み上げるメッセージと言語(Japanese)を設定します。  
![soracomug_shinshu6 / 4-4 Twilio-Studio](https://kizawa.info/wp-content/uploads/2018/11/tstudio-5-1024x657.png)

作成できたら、右上にあるPublishボタンを押して完成です。  
Studio Dashboardに戻りますので、作成したフローのSIDをメモ帳等にコピーしておきます。  
![soracomug_shinshu6 / 4-5 Twilio-Studio](https://kizawa.info/wp-content/uploads/2018/11/tstudio-7-1024x221.png)

<h3 id="content5">5. AWS Lambda関数の設置</h3>

SORACOM LTE-M Buttonが押された際にTwilioを呼び出すプログラム(Lambda関数)を設置します。  
（※今回はハンズオン時間の関係上、作成済の関数をアップロードして設置します）
[Lambda関数一式(zip)](https://github.com/kizawa2020/iot-showcase/raw/master/events/soracomug-shinshu7/lambda.zip)をダウンロードしてください。

[AWS マネジメントコンソール](https://console.aws.amazon.com/console/home)を開きログインした後、リージョンをオレゴンに変更し、Lambdaのコンソールを開きます。  
![soracomug_shinshu6 / 5-1 Lambda1](https://kizawa.info/wp-content/uploads/2019/03/lambda-1.png)

関数の作成をクリックします。  
![soracomug_shinshu6 / 5-2 Lambda2](https://kizawa.info/wp-content/uploads/2019/03/lambda-0.png)

関数の作成画面が開きます。以下の情報を入力します。
* 作成方法：一から作成  
* 関数名：任意の名前
* ランタイム：Python3.7
* 実行ロール：基本的なLambdaアクセス権限で新しいロールを作成
![soracomug_shinshu6 / 5-3 Lambda2](https://kizawa.info/wp-content/uploads/2019/03/lambda-3.png)

続いて関数の編集画面が開きます。以下の設定を行います  
* 関数コードのセクション
    * コードエントリタイプ：.zipファイルをアップロード
    * 関数パッケージ：アップロードボタンをクリックし、先程ダウンロードしたzipファイルを指定
* 基本設定のセクション
    * タイムアウト時間を 3秒⇒10秒 に変更
![soracomug_shinshu6 / 5-4 Lambda2](https://kizawa.info/wp-content/uploads/2019/03/lambda-4.png)  
![soracomug_shinshu6 / 5-5 Lambda2](https://kizawa.info/wp-content/uploads/2019/03/lambda-5.png)  

**環境変数の設定**  
引き続き画面下の方にある環境変数のセクションで以下の情報を入力します。

| 属性の名前 | デフォルト値 |
|:--------------|:------------|
| twilio_sid    | (TwilioのACCOUNT SID)     |
| twilio_token  | (TwilioのAUTH TOKEN)      |
| twilio_flowid | (Twilio Studio FlowのSID) |
| twilio_number | (Twilioで取得した電話番号 +8150xxxxxxxx の形式) |
| twilio_callto | (電話の発信先：Twilioに登録したご自身の携帯電話番号 +81xxxxxxxxxx の形式) |

最後に画面上方にある「保存」ボタンをクリックして下さい。
保存後、画面上にある arn番号をコピーしメモしておきます。
以上で完了です。

<h3 id="content6">6. AWS IAMの設定</h3>

SORACOM Funkからアクセスできるよう、IAMユーザを作成し権限を付与します。

**ポリシーの作成**  
ポリシーを作成します。



**IAMユーザの作成**  





<h3 id="content7">7. SORACOM Funkの設定</h3>

**SIMグループの作成**  



**SORACOM Funkの設定**  




<h3 id="content8">8. 動作確認</h3>

ここまでの作業で AWS IoT 1-Click を通じて Twilio経由で電話がかけられるようになりました。  
ボタンを押すと、ご自身の携帯電話に電話が掛かってくるようになったでしょうか。  
（※うまく動作しない方はスタッフまでご相談ください）

<h3 id="contentE">9. お片付け</h3>

**※ あのボタン貸出枠で参加の方は、ボタン返却のため以下の手順の実施をしてください。**

SIMグループを削除します。



こちらでハンズオンは終了です。お疲れ様でした。

