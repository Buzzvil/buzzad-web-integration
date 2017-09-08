# BuzzAd web繋ぎ込みガイド v3.3

BuzzAd Webの繋ぎこみにはJavascriptを利用した繋ぎこみとServer to Server を利用した繋ぎこみがある。２つの中で１つを選んで繋ぎこみお願い致します。

## 1. Javascript 繋ぎ込み

### 概要
提供されている２種類のJavascriptコードを2つの段階に分けて挿入することで繋ぎこみができる。１つ目のステップ（初期化）は広告の一番初めのページ（LP）を読み込みする際に、2つ目のステップ（アクション達成送信）はアクションが完了された時に呼び出す。

#### 注意事項
- Javascriptの繋ぎこみはlocalStorageを利用するため、LPとアクション完了ページのドメインが一致しないといけない。そうでない場合は、Server to Server繋ぎこみをおすすめします。
- 必ず、提供される繋ぎこみテストを完了したあと、弊社の担当者に繋ぎこみ完了に関する旨をお知らせしてください。
- 繋ぎこみテストが失敗する場合、失敗したステップのFAQを確認する。
- 繋ぎこみテストはStep1が成功したあと、 Step2をする。

#### 広告ポイント付与のフロー

- BuzzAdのインベントリを通じてユーザーがクライアントの広告にランディングした際に、BuzzAdサーバではユーザーのアクションをトラッキングするためのidである `bz_tracking_id` を本来のurlにパラメーターとして付けて送る。

    > 注意 : `bz_tracking_id` は広告ごとに付与さえる固定された値ではなく、ユーザーがクリックする度に作成できる値である。

- Step1 ’初期化’ステップの繋ぎこみでこのパラメータをユーザーのwebブラウザー内のクライアントとメインlocalStorageに保存する。
- Step2 ’アクション達成送信’ステップの繋ぎこみでアクション完了時BuzzAdサーバーに信号を送る時localStorageに保存したbz_tracking_id を一緒に送信する。
- BuzzAdのサーバーからの `bz_tracking_id` 値を利用し、広告に参加したユーザーを特定し、該当するユーザーにポイントを付与する。

### Step1 : 初期化

BuzzAdを通じランディングされるクライアントの最初のページで下記のjavascriptを呼び出す。

> 下記のコードは `bz_tracking_id` というパラメーターが現在のurlの検索クエリにあったら、それをlocalStorageにBuzzAdという名で保存する

```javascript
<script>
	if (/bz_tracking_id/.test(location.search)) { localStorage.BuzzAd = location.search }
</script>
```

#### FAQ
- Q. コードに’test’という単語があることからテスト用の例見たいですが、繋ぎこみの時は変更して使いますか？
    
    > テスト用の例ではなく、正式表現の一つです。スプリプト全体をそのままお使いください。

- Q. 初期化スクリプト入れ、スクリプトが実行できることまで確認しましたが、テストをするとStetp1繋ぎこみ失敗と表示されます。
    
    > コード作業を進めたページのURLが広告用として渡されたURLと同じであるか再確認してください。 広告用に渡されたランディングページですぐにリダイレクトが起こる場合、意図せず渡されたURL以外のページでは、コードの作業をする可能性があります。 `bz_tracking_id` は最初のランディングページのみ付いて送信されるので、指定されたスクリプトコードは、必ずランディングページで実行する必要があり、そうでない場合は、トラッキングID自体が最初から生成されていないように見えて連動が失敗したと表示される場合があります。 もしこの２つが異なる場合は、最初に渡されたURLのページに戻って、コードの作業をするか、広告に登録されるURLを変更するように弊社の広告担当者にお知らせください。

### Step2 : アクション達成送信

アクション達成時下記のjavascriptを呼び出す。

> 下記のコードは先保存した、BuzzAdという名の変数をそのまま呼びたし、サーバーに送信する。その値でユーザーが広告に参加し、アクションを完了したことをBuzzAdサーバーに送信し、ポイントが付与できるようにする。

```javascript
<script>
(function (img) { img.onload = function () {
	var length = localStorage.BuzzAd.length;
    if(localStorage.BuzzAd.indexOf('10023_71ffbffd-ccf1-4edf-9c4c') != -1){
        alert("[Success] Action Completed!");
    };
    //*必要な場合、ここでリダイレクトする*
};
if (localStorage.BuzzAd == null) { localStorage.BuzzAd = ""; }
img.src = "//t.buzzad.io/action/pb/cpa/default/pixel.gif" + localStorage.BuzzAd; }) (new Image())
</script>
```
#### 注意！アクション達成後、すぐに他のページへリダイレクトする場合
BuzzAdサーバーにアクション達成リクエストが完了されたことを確認し、リダイレクトしなければなりません。 上のコード内のコメントがあるところはアクション送信が完了できたら呼び出される関数の一部でリダイレクトを実行すると、安全にリダイレクトを処理することができる。

> コードの関数はBuzzAdを通じで広告に参加したかどうかに関係なく、いつも呼び出されるので、別のところでリダイレクトを処理しなくてもよい。むしろ、別のところでリダイレクトを処理する場合、BuzzAdサーバーんアクション達成送信が全く来ない可能性があるので、ご注意ください。

#### FAQ
- Q. Step2アクション達成送信のコードでalertを送るようになっているが、ユーザーにはこのalertを見せたくないです。コードを修正する必要がありますか？
    
    > そのalertテスト時だけに表示されます。ユーザーには表示されないのでご安心ください。

- Q. アクション完了時に挿入したコードが実行できることを確認しましたが、テスト結果alertがないので繋ぎこみに失敗しらみたいです。

    > まず、アクション完了ページとランディングページのドメインが同じであるかご確認ください。（http/httpsを含め）ドメインが同じであれば、リダイレクト自動ウィンドウを閉じるがアクション達成送信前に起こっていないか確認してください。そのコードは、必ず上記のコードのコメント処理された部分で行われなければいけません。（上記のリダイレクト関連の説明を参照）

### Javascript 繋ぎ込みテスト

下記のリンクで `Buzzad integration test` リンクをブックマークに追加し、テストを進める。詳細は該当ページをご参考ください。

[Javascript Integration Test Page](https://cdn.rawgit.com/Buzzvil/buzzad-web-integration/master/integration_test_ja.html)

#### ブックマークに追加する方法
- ブラウザの設定でbookmarks barが常に見えるように設定する。（次の写真は、Chrome browserの例示である）
![show_bookmarks_bar](show_bookmarks_bar.png)
- リンクをドラッグしてbookmarks barに追加します。

## 2. Server to Server 繋ぎ込み (Javascript の繋ぎ込み場合は必要なし)
 
### アクション達成 API
ユーザーが特定のアクションをすると BuzzAd サーバーにアクションが起こったことを知らせる。 繋ぎ込みは次のようにする。
 
1) 要請方向

クライアント → 媒体主
 
2) HTTP Request method

POST or GET
 
3) HTTP Request URL

https://t.buzzad.io/action/pb/cpa/default/

4) HTTP Request parameters

| Field | Type | Description |
| --- | --- | --- |
| `bz_tracking_id` | String | 広告とユーザートラッキングの為のID。BuzzAdで広告と繋がっているURLで転換時に送られる値。広告webサイトはこの値を保存しておいてアクション達成 API呼び出し時に再び送らなければならない。 |
 
5) Response

JSON 形式に返還
        
| Field | Type | Description |
| --- | --- | --- |
| `code` | Integer | 処理結果コード<br>- 200 : 正常<br>- 9020 : 重複要請<br>- その他 : エラー |
| `msg` | String | 処理結果メッセージ |
 
6) Test bz_tracking_id

bz_tracking_id = 10023_71ffbffd-ccf1-4edf-9c4c
 
eg) https://t.buzzad.io/action/pb/cpa/default/?bz_tracking_id=10023_71ffbffd-ccf1-4edf-9c4c
