# プロトコル草案

@mizunashi

## 名称

このプロトコルは、"Lekeplass Standard Shogi Protocol"である。略称は"LSSP"である。

## プロトコルのフロー

## 基本

プロトコルは、全てUTF-8文字エンコーディングによって解釈される。

## 通信の開始

### クライアントの接続確認

クライアントは、サーバーに接続したあと、直ちにProtocolコマンドを送る必要がある。
クライアントの接続確認を送った後には空行を送る必要がある。

クライアントの接続確認の形式は以下のようになる。

```
<Protocol Command>
<CRLF>
```

#### Protocol

Protocolコマンドはライン形式のみで表現可能で以下のようになる。

```
Protocol: <Protocol Name>/<Protocol Version>
```

### サーバーの接続応答

サーバーは、クライアントの接続確認を受け取った後直ちにAllow-Data-Formatコマンドを送信する。
サーバーの接続応答を送った後には空行を送る必要がある。

サーバーの接続応答の形式は以下のようになる。

```
<Allow-Data-Formats Command>
<CRLF>
```

#### Allow-Data-Formats

Allow-Data-Formatコマンドはライン形式のみで表現可能で以下のようになる。

```
Allow-Data-Formats:*( <Data Format Name>/<Data Format Version>)
```

なお、何も表記されなかった場合以下と同等とする。

```
Allow-Data-Formats: plain/1.0.0
```

## ルールの合意

### クライアントの意思表示

クライアントはまず、Rule-Intensionコマンドで意思表示の指標を送る。
そのあと、任意でSet-Optionsコマンドを送ることができる。
クライアントは、意思表示情報を送ったあとに空行を送る必要がある。

クライアントの意思表示の形式は以下のようになる。

```
<Rule-Intension Command>
<Set-Options? Command>?
<CRLF>
```

#### Rule-Intension

Rule-Intensionコマンドはライン形式のみで表現可能で以下のようになる。

```
Rule-Intension: <Signature of Rule Intension>
```

#### Set-Options?

Set-Optionsコマンドはヒアドック形式のみで表現可能で以下のようになる。

```
Set-Options?: <- <Data Format>
<Option Data>
<CRLF>
```

### サーバーの合意形成

サーバーは、クライアントの意思表示を受け取った後、何らかの処理によってそれを加味したルール情報をクライアントに送る。
サーバーは、まずルール情報として規定のルールを使用することを明示するdeclaration方式と、ルール情報を列挙するcustomize方式のいずれかによって、ルール情報をクライアントに指示する。
次に、サーバーは、任意でデフォルトで使われるデータの記述方式をクライアントに送ることができる。
そして、サーバーは、初期状態の情報をクライアントに送る。
最後に、サーバーは、合意時間の情報をクライアントに送ることができる。
サーバーの合意形成を送った後に、空行を送る必要がある。

サーバーの合意形成の形式は、以下のようになる。

```
<Rule-Declaration Lines>|<Rule-Customize Lines>
<Data-Format? Command>?
<Initial-Context Command>
<Consensus-Time? Command>?
<CRLF>
```

#### Declaration方式によるルール情報の表現

Declaration方式によるルール情報の形式は以下のようになる。
Rule-Declarationコマンドはライン形式のみで表現可能である。

```
Rule-Mode: declaration
Rule-Declaration: <Declaration Name>/<Declaration Version>
```

#### Customize方式によるルール情報の表現

Customize方式によるルール情報の形式は以下のようになる。
Rule-Customizeコマンドはヒアドック形式のみで表現可能である。

```
Rule-Mode: customize
Rule-Customize: <- <Data Format>?
<Rule Data>
<CRLF>
```

#### Data-Format

Data-Formatコマンドはライン形式のみで表現可能で以下の形式になる。

```
Data-Format?: <Data Format Name>/<Data Format Version>
```

#### Initial-Context

Initial-Contextコマンドはヒアドック形式のみで表現可能で以下の形式になる。

```
Initial-Context: <- <Data Format>?
<Initial Context Data>
<CRLF>
```

#### Consensus-Time?

Consensus-Timeコマンドはライン形式のみで表現可能で以下の形式になる。

```
Consensus-Time?: <Consensus Time Seconds>
```

### クライアントの合意

クライアントは、ルールに合意するかの意思を送る必要がある。
合意時間が指定されている場合は、その時間内に意思の送信を完了させる必要がある。
クライアントは、まずルールに合意するかの意思を送る必要がある。
その後、なぜその意思を表明しかの理由を送ることができる。
クライアントは、クライアントの合意の意思を送った後空行を送る必要がある。

```
<Rule-Consensus Command>
<Rule-Consensus-Detail? Command>?
<CRLF>
```

#### Rule-Consensus

Rule-Consensusコマンドはライン形式のみで表現可能で以下の形式になる。

```
Rule-Consensus: agree|reject
```

#### Rule-Consensus-Detail?

Rule-Consensus-Detailコマンドは、ヒアドック形式のみで表現可能で以下の形式になる。

```
Rule-Consensus-Detail?: <- <Data Format>?
<Consensus Detail>
<CRLF>
```

## ゲームの開始

サーバーがクライアントの合意の旨を受け取り、何らかの判断をもってクライアントの合意が得られたとした場合、ゲームを開始する。

### サーバーの準備開始の通知

サーバーはゲームの開始に伴って、クライアントの準備の待機を行う。
まず、サーバーはクライアントに了解をとる旨を通知する。
その後、サーバーは任意でクライアントに準備時間を通知できる。
サーバーは準備開始の通知の後空行を送る必要がある。

```
<Is-Ready Command>
<Ready-Time? Command>?
<CRLF>
```

#### Is-Ready

Is-Readyコマンドはライン形式のみで表現可能で以下の形式になる。

```
Is-Ready: <Is-Ready Detail>
```

#### Ready-Time?

Ready-Timeコマンドはライン形式のみで表現可能で以下の形式になる。

```
Ready-Time?: <Ready Time Seconds>
```

### クライアントの準備完了の通知

クライアントは、準備が完了したらその旨の通知を送る必要がある。
準備時間が指定されている場合は、その時間内に通知の送信を完了させる必要がある。
クライアントは、準備が完了した旨の通知を送る。
クライアントは、通知を送った後空行を送る必要がある。

```
<Ready-Game Command>
<CRLF>
```

#### Ready-Game

Ready-Gameコマンドはライン形式のみで表現可能で以下の形式になる。

```
Ready-Game: <Ready Game Detail>
```

### サーバーのゲーム開始の通知

サーバーが準備完了の通知を受け取り、何らかの判断において準備が完了したとした場合、ゲームの開始の通知を送る。

```
<Game-Start Command>
```

#### Game-Start

Game-Startコマンドはライン形式のみで表現可能で以下の形式になる。

```
Game-Start: <Game Start Detail>
```

## ゲームによる通信

サーバーが何らかの判断によってゲームを終了させるまで、ゲームによる通信を繰り返し行う。

### ゲーム情報の通知

サーバーはクライアントにゲーム情報を通知する。
まず、サーバーはゲーム情報をクライアントに送信する。
サーバーは、任意で応答時間をクライアントに送信できる。
その後、サーバーはクライアントに応答を思考する旨を送信する。
ゲーム情報の通知を送信した後は、空行を送信する必要がある。

```
<Game-Context Command>
<Game-Time? Command>?
<Go Command>
<CRLF>
```

#### Game-Context

Game-Contextコマンドはヒアドック形式でのみ表現可能で以下の形式になる。

```
Game-Context: <- <Data Format>?
<Game Context Data>
<CRLF>
```

#### Game-Time?

Game-Timeコマンドはライン形式でのみ表現可能で以下の形式になる。

```
Game-Time?: <Game Time Seconds>
```

#### Go

Goコマンドはライン形式でのみ表現可能で以下の形式になる。

```
Go: <Go Detail>
```

### クライアントの応答

クライアントはサーバーに応答を行う必要がある。
まず、応答の種類を送る必要がある。
その後に種類に応じた応答を送る必要がある。
クライアントは任意で、応答の理由を送ることができる。
クライアントの応答を送信した後、空行を送信する必要がある。

```
<Game-Action-Move Lines>|<Game-Action-Giveup Lines>|<Game-Action-Mate Lines>|<Game-Action-Extra Lines>
<Game-Action-Detail? Command>?
<CRLF>
```

#### Moveアクションの表現

Moveアクションは以下の形式になる。
Game-Action-Moveコマンドはライン形式のみで表現可能である。

```
Game-Action-Mode: move
Game-Action-Move: <Move Information>
```

#### Resignアクションの表現

Resignアクションは以下の形式になる。

```
Game-Action-Mode: resign
```

#### Mateアクションの表現

Mateアクションは以下の形式になる。

```
Game-Action-Mode: mate
```

#### Extraアクションの表現

Extraアクションは以下の形式になる。

```
Game-Action-Mode: extra
```

#### Game-Action-Detail?

Game-Action-Detailコマンドはヒアドック形式のみで表現可能で以下の形式になる。

```
Game-Action-Detail?: <- <Data Format>?
<Game Action Detail>
<CRLF>
```

### サーバーの中断

クライアントが応答を送る前に、サーバーは任意のタイミングで中断通知を送ることができる。
サーバーはまず中断の旨を送る。
サーバーは任意で中断の理由を送ることができる。
また、サーバーは任意で中断への応答時間を送ることができる。
サーバーの中断通知を送信した後、空行を送る必要がある。

```
<Game-Stop Command>
<Game-Stop-Detail? Command>?
<Game-Stop-Time? Command>?
<CRLF>
```

#### Game-Stop

Game-Stopコマンドはライン形式のみで表現可能で以下の形式になる。

```
Game-Stop: <Game Stop Variable>
```

#### Game-Stop-Detail?

Game-Stop-Detailコマンドはヒアドック形式のみで表現可能で以下の形式になる。

```
Game-Stop-Detail?: <- <Deta Format>?
<Game Stop Detail>
<CRLF>
```

#### Game-Stop-Time?

Game-Stop-Timeコマンドはライン形式のみで表現可能で以下の形式になる。

```
Game-Stop-Time?: <Game Stop Seconds>
```

### 中断に対する応答

クライアントは、サーバーから中断の通知を受け取った場合、受け取った時点でコマンドを発行中の場合は発行し終えた後即座に応答を返す必要がある。
応答時間が指定されている場合応答はその時間内に完了させる必要がある。また、応答を返した後はサーバーからその後の動作の指示があるまで待機する必要がある。
応答を送った後、空行を送る必要がある。

```
<Game-Stop-Received Command>
<CRLF>
```

#### Game-Stop-Received

Game-Stop-Receivedコマンドはライン形式でのみ表現可能で以下の形式になる。

```
Game-Stop-Received: <Game Stop Received Message>
<CRLF>
```

### 判定結果の通知

サーバーはクライアントの応答を受け取るか、中断を通知しクライアントから受け取った旨の応答を受け取った後その応答の判定結果を返す必要がある。
判定結果をサーバーは送信する。
その後サーバーは任意で判定結果の詳細を送信できる。

```
<Game-Status Command>
<Game-Status-Detail? Command>?
```

#### Game-Status

Game-Statusコマンドはライン形式でのみ表現可能で以下の形式になる。

```
Game-Status: continue|end
```

#### Game-Status-Detail?

Game-Status-Detailコマンドはヒアドック形式でのみ表現可能で以下の形式になる。

```
Game-Status-Detail?: <- <Data Format>?
<Game Status Detail>
<CRLF>
```

## ゲームの終了

サーバーは、コマンド発行中でない任意のタイミングでゲームの終了を通知することができる。ゲームの終了を通知した後このプロトコル上での通信は終了する。

### ゲームの終了通知

サーバーは何らかの理由で通信を終了しなければいけない時、またはGame-Statusコマンドでendを通知した時、ルールの合意が得られなかった時、定められた応答時間内に応答がなかった場合にゲームの終了通知を送信する。
ゲームの終了によるこのゲームの最終結果をまず送信する。
その後、サーバーは任意でゲームの終了の理由を送信することができる。
ゲームの終了通知を行った後、空行を送信する必要がある。

```
<Game-End Command>
<Game-End-Detail? Command>?
<CRLF>
```

#### Game-End

Game-Endコマンドはライン形式でのみ表現可能で以下の形式になる。

```
Game-End: win|lose|draw|nogame
```

#### Game-End-Detail?

Game-End-Detailコマンドはヒアドック形式でのみ表現可能で以下の形式になる。

```
Game-End-Detail?: <- <Data Format>?
<Game End Detail>
<CRLF>
```
