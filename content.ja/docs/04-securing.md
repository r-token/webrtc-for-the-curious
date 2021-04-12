---
title: セキュリティ対策
type: docs
weight: 5
---


# WebRTCにはどんなセキュリティがありますか？

WebRTCの接続はすべて認証され、暗号化されています。第三者があなたの送信内容を見たり、偽のメッセージを挿入したりすることはないので安心です。また、セッション記述を生成したWebRTCエージェントが、通信相手であることも確かです。

誰もこれらのメッセージに手を加えないことが非常に重要です。第3者が転送中のセッション記述を読んでも問題ありません。しかし、WebRTCにはセッション記述が変更されることに対する保護がありません。攻撃者は ICE Candidates を変更し、証明書フィンガープリントを更新することで、あなたに対して中間者攻撃を行うことができます。

## どのような仕組みになっているのですか？

WebRTCは、Datagram Transport Layer Security ([DTLS](https://tools.ietf.org/html/rfc6347))とSecure Real-time Transport Protocol ([SRTP](https://tools.ietf.org/html/rfc3711))という2つの既存のプロトコルを使用しています。

DTLSは、セッションをネゴシエートした後、2つのピア間で安全にデータを交換することができます。DTLSは、HTTPSを実現する技術であるTLSと兄弟関係にありますが、DTLSはトランスポート層としてTCPではなくUDPを使用します。つまり、このプロトコルは、信頼性の低い配信を処理しなければならないということです。SRTPは、特にメディアを安全に交換するために設計されています。DTLSの代わりにSRTPを使用することで、いくつかの最適化が可能になります。

DTLSは最初に使用されます。ICEが提供する接続に対してハンドシェイクを行います。DTLSはクライアント/サーバー型のプロトコルなので、ハンドシェイクはどちらか一方が開始する必要があります。クライアント／サーバーの役割は、シグナリング時に選択されます。DTLSのハンドシェイクでは、双方が証明書を提示します。
ハンドシェイクが完了すると、この証明書は「セッション記述」にある証明書のハッシュ値と比較されます。これは、ハンドシェイクが期待していたWebRTCエージェントで行われたことを確認するためです。これで、DTLS 接続が DataChannel の通信に使用できるようになります。

SRTPセッションを作成するには、DTLSで生成されたキーを使用してセッションを初期化します。SRTPにはハンドシェイク機構がないため、外部の鍵を使ってブートストラップを行う必要があります。これが完了すると、SRTP で暗号化されたメディアを交換することができます。

## セキュリティ101
本章で紹介する技術を理解するには、まずこれらの用語を理解する必要があります。暗号は難しいテーマなので、他の資料も参考にしてください。

#### 暗号

暗号とは、平文を暗号文に変換する一連の手順のことです。その後、暗号を元に戻すことができるため、暗号文を平文に戻すことができます。暗号は通常、その動作を変えるための鍵を持っています。別の用語では、暗号化と復号化があります。

簡単な暗号はROT13です。各文字が13文字分前に移動します。暗号を解除するには、13文字を後ろに移動します。平文の`HELLO`は暗号文の`URYYB`になります。この場合、暗号はROT、鍵は13となります。

#### 平文/暗号文

平文とは暗号の入力である。暗号文とは、暗号の出力である。

#### ハッシュ

ハッシュは、ダイジェストを生成する一方通行のプロセスです。入力があると、毎回同じ出力を生成します。出力が可逆的でないことが重要です。出力があれば、その入力を特定できないようにする必要があります。ハッシュ化は、メッセージが改ざんされていないことを確認したい場合に有効です。

単純なハッシュは、1文字おきに取るだけのもので、`HELLO`は`HLO`になります。HELLO」が入力であると仮定することはできませんが、「HELLO」が一致することは確認できます。

#### 公開鍵/秘密鍵暗号方式

公開鍵/秘密鍵暗号方式は、DTLS と SRTP が使用する暗号の種類を説明します。このシステムでは、公開鍵と秘密鍵の2つの鍵を持ちます。公開鍵は、メッセージを暗号化するためのもので、共有しても安全です。
秘密鍵は復号化のためのもので、決して共有してはいけません。公開鍵で暗号化されたメッセージを復号化できる唯一の鍵です。

#### ディフィー・ヘルマン交換

ディフィー・ヘルマン交換は、初対面の2人のユーザがインターネット上で安全に共有秘密を作成することができます。ユーザ`A`は、盗聴の心配をすることなく、ユーザ`B`に秘密を送ることができます。これは、離散対数問題を解く難しさによります。
この仕組みを完全に理解する必要はありませんが、これがDTLSのハンドシェイクを可能にしていることを知っておくと役立ちます。

ウィキペディアには、この動作の例があります [リンク](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange#Cryptographic_explanation).

#### 擬似乱数関数

擬似乱数関数(PRF)とは、ランダムに見える値を生成するためにあらかじめ定義された関数です。複数の入力を受け取り、1つの出力を生成することができます。

#### 鍵導出関数

鍵導出は、疑似乱数関数の一種です。 鍵導出は、鍵を強化するために使用される関数です。 一般的なパターンの1つは、キーストレッチです。

例えば、8バイトの鍵が与えられたとします。KDFを使ってより強力なものにすることができます。

#### ノンス
ノンスとは、暗号機への追加入力です。これは、同じメッセージを複数回暗号化した場合でも、暗号から異なる出力を得ることができるようにするためです。

同じメッセージを 10 回暗号化すると、暗号は同じ暗号文を 10 回生成します。ノンスを使用すれば、同じ鍵を使用しながら、異なる入力を得ることができます。メッセージごとに異なるノンスを使用することが重要です。そうしないと、その価値の多くが否定されてしまいます。

#### メッセージ認証コード
メッセージ認証コード（Message Authentication Code）は、メッセージの最後に置かれるハッシュです。MACは、そのメッセージが期待したユーザーからのものであることを証明します。

MACを使用しない場合、攻撃者は無効なメッセージを挿入することができます。復号化しても、相手は鍵を知らないので、ただのゴミになってしまいます。


#### キー・ローテーション
キーローテーションとは、一定期間ごとに鍵を交換することです。これにより、盗まれた鍵の影響を少なくすることができます。鍵が盗まれたり漏れたりした場合、復号できるデータの数は少なくなります。

## DTLS
DTLS(Datagram Transport Layer Security)は、2つのピアが既存の設定なしに安全な通信を確立することができます。たとえ誰かが会話を盗み聞きしていたとしても、メッセージを解読することはできません。

DTLSクライアントとサーバーが通信するためには、暗号と鍵に合意する必要があります。これらの値は、DTLSのハンドシェイクを行うことで決定されます。ハンドシェイクの間、メッセージは平文である。
DTLSクライアント/サーバーが暗号化を開始するのに十分な情報を交換したとき、`Change Cipher Spec`を送信します。このメッセージの後、後続の各メッセージは暗号化されます。

### パケットフォーマット
全てのDTLSパケットはヘッダーから始まります。

#### コンテンツタイプ
以下のようなタイプが想定されます。

* チェンジサイファースペック (Change Cipher Spec) - `20`
* ハンドシェイク (Handshake) - `22`
* アプリケーションデータ (Application Data) - `23`

`ハンドシェイク` は、セッションを開始するための詳細情報を交換するために使用されます。`チェンジサイファースペック`は、相手にすべてのデータが暗号化されることを通知するために使用します。`アプリケーションデータ`は、暗号化されたメッセージです。

#### バージョン
バージョンは `0x0000feff` (DTLS v1.0) または `0x0000fefd` (DTLS v1.2) のいずれかで、v1.1 はありません。

#### エポック
エポックは `0` から始まりますが、`Change Cipher Spec` を実行すると `1` になります。エポックが0でないメッセージはすべて暗号化されます。

#### シーケンス番号
シーケンス番号はメッセージを順番に並べるために使われます。メッセージを送信するたびに、シーケンス番号が増加します。エポックが増加すると、シーケンス番号は最初から始まる。

#### 長さとペイロード
ペイロードは`コンテンツタイプ`ごとに異なります。`アプリケーションデータ`の場合、`ペイロード`は暗号化されたデータです。`ハンドシェイク`の場合は、メッセージによって異なります。

長さは`ペイロード`の大きさを表します。

### ハンドシェイクのステートマシン
ハンドシェイクの間、クライアントとサーバーは一連のメッセージを交換します。これらのメッセージはフライトに分類されます。各フライトには複数のメッセージが含まれることがあります（1つだけの場合もあります）。
フライトは、そのフライトに含まれるすべてのメッセージを受信するまで完了しません。各メッセージの目的については、以下で詳しく説明します。

![Handshake](../images/04-handshake.png "Handshake")

#### ClientHello

ClientHello は、クライアントが送信する最初のメッセージです。これは属性のリストを含んでいます。これらの属性は、クライアントがサポートしている暗号や機能をサーバーに伝えます。WebRTC の場合、これは SRTP Cipher を選択する方法でもあります。また、セッションの鍵を生成するために使用するランダムデータも含まれています。

#### HelloVerifyRequest
HelloVerifyRequestは、サーバーからクライアントに送信されます。HelloVerifyRequestは、サーバーからクライアントに送信されます。これは、クライアントがリクエストの送信を意図しているかどうかを確認するためです。その後、クライアントは、HelloVerifyRequestで指定されたトークンを使用して、ClientHelloを再送信します。

#### ServerHello
ServerHello は、このセッションの設定に対するサーバからの応答です。このセッションが終了したときに使用される暗号を含んでいます。また、サーバーのランダムデータも含まれています。

#### Certificate
Certificate は、クライアントまたはサーバーの証明書を含みます。この証明書は、通信相手を一意に識別するために使用されます。ハンドシェイクが終わった後、この証明書をハッシュ化したものが`セッション記述`のフィンガープリントと一致するかどうかを確認します。

#### ServerKeyExchange/ClientKeyExchange
これらのメッセージは、公開鍵を送信するために使用されます。起動時には、クライアントとサーバーの両方がキーペアを生成します。ハンドシェイクの後、これらの値は `プレマスターシークレット` の生成に使用されます。

#### CertificateRequest
CertificateRequestは、サーバがクライアントに証明書が必要であることを通知するために送信されます。サーバは、証明書を要求することも要求することもできます。

#### ServerHelloDone
ServerHelloDone は、サーバーがハンドシェイクを終了したことをクライアントに通知します。

#### CertificateVerify
CertificateVerify は、送信者が Certificate メッセージで送られたプライベートキーを持っていることを証明する方法です。

#### ChangeCipherSpec
ChangeCipherSpec は、このメッセージの後に送信されるすべてのものが暗号化されることを受信者に知らせます。

#### Finished
Finished は暗号化され、すべてのメッセージのハッシュを含みます。これはハンドシェイクが改ざんされていないことを保証するためです。

### 鍵の生成
ハンドシェイクが完了すると、暗号化されたデータの送信が可能になります。暗号はサーバーが選択し、ServerHelloに入っています。では、その鍵はどのようにして選ばれたのでしょうか？

まず、`プレマスターシークレット`を生成します。この値を得るために、`ServerKeyExchange`と`ClientKeyExchange`で交換された鍵にDiffie-Hellmanを使用します。 詳細は選択した暗号によって異なります。

次に、`マスターシークレット`が生成されます。DTLSの各バージョンには、定義された`疑似乱数関数`があります。DTLS 1.2では、この関数は`プレマスターシークレット`と、`ClientHello`と`ServerHello`に含まれるランダムな値を受け取ります。
`似乱数関数`を実行した結果の出力は`マスターシークレット`です。`スターシークレット`は、暗号に使用される値です。

### アプリケーションデータの交換
DTLSの主力となるのが`ApplicationData`です。初期化されたCipherがあれば、暗号化して値を送信することができます。

`ApplicationData`メッセージは、前述のようにDTLSヘッダを使用します。`Payload` には暗号文が格納されています。これでDTLSセッションが動作し、安全に通信できるようになりました。

DTLSには、再ネゴシエーションなど、さらに興味深い機能がたくさんあります。WebRTCでは使用されていないので、ここでは説明しません。

## SRTP
SRTP は、RTP パケットの暗号化に特化して設計されたプロトコルです。SRTP セッションを開始するには、鍵と暗号を指定します。DTLSとは異なり、ハンドシェイクの仕組みはない。すべての設定と鍵は、DTLSのハンドシェイク中に生成されます。

DTLSでは、別のプロセスで使用するために鍵をエクスポートする専用のAPIを提供しています。これは[RFC 5705](https://tools.ietf.org/html/rfc5705)で定義されています。

### セッションの作成
SRTPでは、入力に使用される鍵導出関数を定義しています。SRTPセッションの作成時には、入力をこの関数に通して、SRTP暗号用の鍵を生成します。この後、メディアの処理に移ることができます。

### メディアの交換
各 RTP パケットには、16 ビットのシーケンス番号があります。このシーケンス番号は、主キーのように、パケットの順序を保つために使用されます。通話中、これらの番号はロールオーバーします。SRTPはそれを追跡し、これをロールオーバーカウンタと呼ぶ。

SRTPは、パケットを暗号化する際に、ロールオーバーカウンタとシーケンス番号を nonceとして使用します。これは、同じデータを 2 回送信しても、暗号文が異なることを保証するためです。これは、攻撃者がパターンを特定したり、リプレイ攻撃を試みたりするのを防ぐために重要です。