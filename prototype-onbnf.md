# プロトコル草案

## EBNF

## 基本

```
CR = ? US-ASCII CR, carriage return (13) ?
LF = ? US-ASCII LF, linefeed (10) ?
SP = ? US-ASCII SP, space (32) ?
HT = ? US-ASCII HT, horizontal tab (9) ?

ASC_UPALPHA = ? any US-ASCII uppercase letter "A".."Z" ?
ASC_LOALPHA = ? any US-ASCII lowercase letter "a".."z" ?
ASC_ALPHA = ASC_LOALPHA | ASC_UPALPHA
ASC_DIGIT = ? any US-ASCII digit "0".."9" ?
ASC_SPECIFY = "-" | ":" | ";" | "." | "," | "_" | "@" | "+" | "'"
              "<" | ">" | "{" | "}" | "[" | "]" | "(" | ")"

<-> = ? US-ASCII hyphen (45) ?
<.> = ? US-ASCII dot (46) ?
</> = ? US-ASCII slash (47) ?
<:> = ? US-ASCII colon (58) ?
<?> = ? US-ASCII question mark (63) ?

UNI_ND = ? any Unicode decimical digit Nd Category ?
UNI_DIGIT = UNI_ND

UNI_LU = ? any Unicode uppercase letter Lu Category ?
UNI_LL = ? any Unicode lowercase letter Ll Category ?
UNI_LT = ? any Unicode titlecase letter Lt Category ?
UNI_LM = ? any Unicode modifier letter Lm Category ?
UNI_LO = ? any Unicode other letter Lo Category ?
UNI_NL = ? any Unicode number letter Nl Category ?
UNI_LETTER = UNI_LU | UNI_LL | UNI_LT | UNI_LM | UNI_LO | UNI_NL

UNI_MN = ? any Unicode nonspacing mark Mn Category ?
UNI_MC = ? any Unicode spacing combining mark Mc Category ?
UNI_MARK = UNI_MN | UNI_MC

UNI_PC = ? any Unicode connector punctuation Pc Category ?
UNI_PD = ? any Unicode dash punctuation Pd Category ?
UNI_PS = ? any Unicode open punctuation Ps Category ?
UNI_PE = ? any Unicode close punctuation Pe Category ?
UNI_PI = ? any Unicode initial punctuation Pi Category ?
UNI_PF = ? any Unicode final quote punctuation Pf Category ?
UNI_PO = ? any Unicode other punctuation Po Category ?
UNI_PUNC = UNI_PC | UNI_PD | UNI_PS | UNI_PE | UNI_PI | UNI_PF | UNI_PO

UNI_NO = ? any Unicode number symbol No Category ?
UNI_SM = ? any Unicode math symbol Sm Category ?
UNI_SC = ? any Unicode currency symbol Sc Category ?
UNI_SK = ? any Unicode modifier symbol Sk Category ?
UNI_SO = ? any Unicode other symbol So Category ?
UNI_SYM = UNI_NO | UNI_SM | UNI_SC | UNI_SK | UNI_SO

UNI_ZS = ? any Unicode other space Zs Category ?
UNI_CF = ? any Unicode other format Cf Category ?
UNI_FMT = UNI_ZS | UNI_CF | HT

CRLF = CR LF
WS = SP | HT

WSS = { WS }
LSEP = <:> WSS
LEND = WSS CRLF
HSEP = <:> WSS "<" "-" WSS
HEND = CRLF
```

### バージョン番号

```
version-value = exclude_dot { exclude_dot | <.> }
exclude_dot = ASC_DIGIT | ASC_LOALPHA
```

### 時間表示

```
time-value = ASC_DIGIT { ASC_DIGIT }
```

## コマンド形式

```
field-name = field-word { <-> field-word } [<?>]
field-word = ASC_UPALPHA { ASC_ALPHA | ASC_DIGIT }
```

### ライン形式

```
command-line = field-name LSEP line-value LEND
line-value = uni_exclude_fmt { UNI_FMT | uni_exclude_fmt }
uni-exclude-fmt = UNI_DIGIT | UNI_LETTER | UNI_MARK | UNI_PUNC | UNI_SYM
```

### ヒアドック形式

```
command-heredoc = field-name HSEP [data-format WSS]
                  ? any octets not including 2 times CRLF
                    starts with CRLF and ends with CRLF ?
                  HEND
data-format = data-format-name </> data-format-version
data-format-name = data-format-headchr { data-format-headchr | ASC_SPECIFY }
data-format-headchr = ASC_DIGIT | ASC_ALPHA
data-format-version = version-value
```

## 通信の開始

### クライアントの接続確認

クライアントは、サーバーに接続したあと、直ちにProtocolコマンドを送る必要がある。
クライアントの接続確認を送った後には空行を送る必要がある。

クライアントの接続確認の形式は以下のようになる。

```
protocol-command
CRLF
```

#### Protocol

Protocolコマンドはライン形式のみで表現可能で以下のようになる。

```
protocol-command = "Protocol" LSEP
                   protocol-name </> protocol-version
                   LEND
protocol-name = protocol-headchr { protocol-headchr | ASC_SPECIFY }
protocol-headchr = ASC_DIGIT | ASC_ALPHA
protocol-version = version-value
```

### サーバーの接続応答

サーバーは、クライアントの接続確認を受け取った後直ちにAllow-Data-Formatコマンドを送信する。
サーバーの接続応答を送った後には空行を送る必要がある。

サーバーの接続応答の形式は以下のようになる。

```
allow-data-formats-command
CRLF
```

#### Allow-Data-Formats

Allow-Data-Formatコマンドはライン形式のみで表現可能で以下のようになる。

```
allow-data-formats-command = "Allow-Data-Formats" LSEP
                            data-formats
                            LEND
data-formats = { data-format WSS }
```

なお、何も表記されなかった場合以下と同等とする。

```
data-formats = "plain/1.0.0"
```

## ルールの合意

### クライアントの意思表示

クライアントはまず、Rule-Intensionコマンドで意思表示の指標を送る。
そのあと、任意でSet-Optionsコマンドを送ることができる。
クライアントは、意思表示情報を送ったあとに空行を送る必要がある。

クライアントの意思表示の形式は以下のようになる。

```
rule-intension-command
[ set-options-command ]
CRLF
```

#### Rule-Intension

Rule-Intensionコマンドはライン形式のみで表現可能で以下のようになる。

```
rule-intension-command = "Rule-Intension" LSEP
                         rule-intension-data
                         LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
rule-intension-data = line-value
```

#### Set-Options?

Set-Optionsコマンドはヒアドック形式のみで表現可能で以下のようになる。

```
set-options-command = "Set-Options?" HSEP data-format WSS
                      set-options-data
                      HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
set-options-data = ? any octets not including 2 times CRLF
                    starts with CRLF and ends with CRLF ?
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
( rule-declaration-lines | rule-customize-lines )
[ data-format-command ]
initial-context-command
[ consensus-time-command ]
CRLF
```

#### Declaration方式によるルール情報の表現

Declaration方式によるルール情報の形式は以下のようになる。
Rule-Declarationコマンドはライン形式のみで表現可能である。

```
rule-declaration-lines = rule-mode-declaration-command
                         rule-declaration-command
rule-mode-declaration-command = "Rule-Mode" LSEP
                                "declaration"
                                LEND
rule-declaration-command = "Rule-Declaration" LSEP
                           declaration-name </> declaration-version
                           LEND
declaration-name = declaration-headchr { declaration-headchr | ASC_SPECIFY }
declaration-headchr = ASC_DIGIT | ASC_ALPHA
declaration-version = version-value
```

#### Customize方式によるルール情報の表現

Customize方式によるルール情報の形式は以下のようになる。
Rule-Customizeコマンドはヒアドック形式のみで表現可能である。

```
rule-customize-lines = rule-mode-customize-command
                       rule-customize-command
rule-mode-cutomize-command = "Rule-Mode" LSEP
                             "customize"
                             LEND
rule-customize-command = "Rule-Customize" HSEP [ data-format WSS ]
                         rule-customize-data
                         HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
rule-customize-data = ? any octets not including 2 times CRLF
                      starts with CRLF and ends with CRLF ?
```

#### Data-Format

Data-Formatコマンドはライン形式のみで表現可能で以下の形式になる。

```
data-format-command = "Data-Format?" LSEP
                      data-format
                      LEND
```

#### Initial-Context

Initial-Contextコマンドはヒアドック形式のみで表現可能で以下の形式になる。

```
initial-context-command = "Initial-Context" HSEP [ data-format WSS ]
                          initial-context-data
                          HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
initial-context-data = ? any octets not including 2 times CRLF
                       starts with CRLF and ends with CRLF ?
```

#### Consensus-Time?

Consensus-Timeコマンドはライン形式のみで表現可能で以下の形式になる。

```
consensus-time-command = "Consensus-Time?" LSEP
                         time-value
                         LEND
```

### クライアントの合意

クライアントは、ルールに合意するかの意思を送る必要がある。
合意時間が指定されている場合は、その時間内に意思の送信を完了させる必要がある。
クライアントは、まずルールに合意するかの意思を送る必要がある。
その後、なぜその意思を表明したかの理由を送ることができる。
クライアントは、クライアントの合意の意思を送った後空行を送る必要がある。

```
rule-consensus-command
[ rule-consensus-detail-command ]
CRLF
```

#### Rule-Consensus

Rule-Consensusコマンドはライン形式のみで表現可能で以下の形式になる。

```
rule-consensus-command = "Rule-Consensus" LSEP
                         ( "agree" | "reject" )
                         LEND
```

#### Rule-Consensus-Detail?

Rule-Consensus-Detailコマンドは、ヒアドック形式のみで表現可能で以下の形式になる。

```
rule-consensus-detail-command = "Rule-Consensus-Detail?" HSEP [ data-format WSS ]
                                rule-consensus-detail-data
                                HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
rule-consensus-detail-data = ? any octets not including 2 times CRLF
                             starts with CRLF and ends with CRLF ?
```

## ゲームの開始

サーバーがクライアントの合意の旨を受け取り、何らかの判断をもってクライアントの合意が得られたとした場合、ゲームを開始する。

### サーバーの準備開始の通知

サーバーはゲームの開始に伴って、クライアントの準備の待機を行う。
まず、サーバーはクライアントに了解をとる旨を通知する。
その後、サーバーは任意でクライアントに準備時間を通知できる。
サーバーは準備開始の通知の後空行を送る必要がある。

```
is-ready-command
[ ready-time-command ]
CRLF
```

#### Is-Ready

Is-Readyコマンドはライン形式のみで表現可能で以下の形式になる。

```
is-ready-command = "Is-Ready" LSEP
                   is-ready-data
                   LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
rule-intension-data = line-value
```

#### Ready-Time?

Ready-Timeコマンドはライン形式のみで表現可能で以下の形式になる。

```
ready-time-command = "Ready-Time?" LSEP
                     time-value
                     LEND
```

### クライアントの準備完了の通知

クライアントは、準備が完了したらその旨の通知を送る必要がある。
準備時間が指定されている場合は、その時間内に通知の送信を完了させる必要がある。
クライアントは、準備が完了した旨の通知を送る。
クライアントは、通知を送った後空行を送る必要がある。

```
ready-game-command
CRLF
```

#### Ready-Game

Ready-Gameコマンドはライン形式のみで表現可能で以下の形式になる。

```
ready-game-command = "Ready-Game" LSEP
                     ready-game-data
                     LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
ready-game-data = line-value
```

### サーバーのゲーム開始の通知

サーバーが準備完了の通知を受け取り、何らかの判断において準備が完了したとした場合、ゲームの開始の通知を送る。

```
game-start-command
```

#### Game-Start

Game-Startコマンドはライン形式のみで表現可能で以下の形式になる。

```
game-start = "Game-Start" LSEP
             game-start-data
             LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
game-start-data = line-value
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
game-context-command
[ game-time-command ]
go-command
CRLF
```

#### Game-Context

Game-Contextコマンドはヒアドック形式でのみ表現可能で以下の形式になる。

```
game-context-command = "Game-Context" HSEP [ data-format WSS ]
                       game-context-data
                       HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
game-context-data = ? any octets not including 2 times CRLF
                    starts with CRLF and ends with CRLF ?
```

#### Game-Time?

Game-Timeコマンドはライン形式でのみ表現可能で以下の形式になる。

```
game-time-command = "Game-Time?" LSEP
                    time-value
                    LEND
```

#### Go

Goコマンドはライン形式でのみ表現可能で以下の形式になる。

```
go-command = "Go" LSEP
             go-data
             LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
go-data = line-value
```

### クライアントの応答

クライアントはサーバーに応答を行う必要がある。
まず、応答の種類を送る必要がある。
その後に種類に応じた応答を送る必要がある。
クライアントは任意で、応答の理由を送ることができる。
クライアントの応答を送信した後、空行を送信する必要がある。

```
( game-action-move-lines
| game-action-giveup-lines
| game-action-mate-lines
| game-action-extra-lines
)
game-action-detail-command
CRLF
```

#### Moveアクションの表現

Moveアクションは以下の形式になる。
Game-Action-Moveコマンドはライン形式のみで表現可能である。

```
game-action-move-lines = game-action-mode-move-command
                         game-action-move-command
game-action-mode-move-command = "Game-Action-Mode" LSEP
                                "move"
                                LEND
game-action-move-command = "Game-Action-Move" LSEP
                           game-action-move-data
                           LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
game-action-move-data = line-value
```

#### Resignアクションの表現

Resignアクションは以下の形式になる。

```
game-action-resign-lines = game-action-mode-resign-command
game-action-mode-resign-command = "Game-Action-Mode" LSEP
                                  "resign"
                                  LEND
```

#### Mateアクションの表現

Mateアクションは以下の形式になる。

```
game-action-mate-lines = game-action-mode-mate-command
game-action-mode-mate-command = "Game-Action-Mode" LSEP
                                "mate"
                                LEND
```

#### Extraアクションの表現

Extraアクションは以下の形式になる。

```
game-action-extra-lines = game-action-mode-extra-command
game-action-mode-extra-command = "Game-Action-Mode" LSEP
                                 "extra"
                                 LEND
```

#### Game-Action-Detail?

Game-Action-Detailコマンドはヒアドック形式のみで表現可能で以下の形式になる。

```
game-action-detail-command = "Game-Action-Detail?" HSEP [ data-format WSS ]
                             game-action-detail-data
                             HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
game-context-data = ? any octets not including 2 times CRLF
                    starts with CRLF and ends with CRLF ?
```

### サーバーの中断

クライアントが応答を送る前に、サーバーは任意のタイミングで中断通知を送ることができる。
サーバーはまず中断の旨を送る。
サーバーは任意で中断の理由を送ることができる。
また、サーバーは任意で中断への応答時間を送ることができる。
サーバーの中断通知を送信した後、空行を送る必要がある。

```
game-stop-command
[ game-stop-detail-command ]
[ game-stop-time-command ]
CRLF
```

#### Game-Stop

Game-Stopコマンドはライン形式のみで表現可能で以下の形式になる。

```
game-stop-command = "Game-Stop" LSEP
                    game-stop-data
                    LEND
```

ここで、データの内容は以下のことを満たすデータである。

```
game-stop-data = line-value
```

#### Game-Stop-Detail?

Game-Stop-Detailコマンドはヒアドック形式のみで表現可能で以下の形式になる。

```
game-stop-detail-command = "Game-Stop-Detail?" LSEP [ data-format WSS ]
                           game-stop-detail-data
                           CRLF
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
game-stop-detail-data = ? any octets not including 2 times CRLF
                        starts with CRLF and ends with CRLF ?
```

#### Game-Stop-Time?

Game-Stop-Timeコマンドはライン形式のみで表現可能で以下の形式になる。

```
game-stop-time-command = "Game-Stop-Time?" LSEP
                         time-value
                         LEND
```

### 中断に対する応答

クライアントは、サーバーから中断の通知を受け取った場合、受け取った時点でコマンドを発行中の場合は発行し終えた後即座に応答を返す必要がある。
応答時間が指定されている場合応答はその時間内に完了させる必要がある。また、応答を返した後はサーバーからその後の動作の指示があるまで待機する必要がある。
応答を送った後、空行を送る必要がある。

```
game-stop-received-command
CRLF
```

#### Game-Stop-Received

Game-Stop-Receivedコマンドはライン形式でのみ表現可能で以下の形式になる。

```
game-stop-received-command = "Game-Stop-Received" LSEP
                             game-stop-received-data
                             LEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
game-stop-received-data = ? any octets not including 2 times CRLF
                          starts with CRLF and ends with CRLF ?
```

### 判定結果の通知

サーバーはクライアントの応答を受け取るか、中断を通知しクライアントから受け取った旨の応答を受け取った後その応答の判定結果を返す必要がある。
判定結果をサーバーは送信する。
その後サーバーは任意で判定結果の詳細を送信できる。

```
game-status-command
game-status-result-command
```

#### Game-Status

Game-Statusコマンドはライン形式でのみ表現可能で以下の形式になる。

```
game-status-command = "Game-Status" LSEP
                      ( "continue" | "end" )
                      LEND
```

#### Game-Status-Result

Game-Status-Resultコマンドはヒアドック形式でのみ表現可能で以下の形式になる。

```
game-status-result-command = "Game-Status-Result" HSEP [ data-format WSS ]
                             game-status-result-data
                             HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
game-status-result-data = ? any octets not including 2 times CRLF
                          starts with CRLF and ends with CRLF ?
```

## ゲームの終了

サーバーは、コマンド発行中でない任意のタイミングでゲームの終了を通知することができる。ゲームの終了を通知した後このプロトコル上での通信は終了する。

### ゲームの終了通知

サーバーは何らかの理由で通信を終了しなければいけない時、またはGame-Statusコマンドでendを通知した時、ルールの合意が得られなかった時、定められた応答時間内に応答がなかった場合にゲームの終了通知を送信する。
ゲームの終了によるこのゲームの最終結果をまず送信する。
その後、サーバーは任意でゲームの終了の理由を送信することができる。
ゲームの終了通知を行った後、空行を送信する必要がある。

```
game-end-command
[ game-end-detail-command ]
CRLF
```

#### Game-End

Game-Endコマンドはライン形式でのみ表現可能で以下の形式になる。

```
game-end-command = "Game-End" LSEP
                   ( "win" | "lose" | "draw" | "nogame" )
                   LEND
```

#### Game-End-Detail?

Game-End-Detailコマンドはヒアドック形式でのみ表現可能で以下の形式になる。

```
game-end-detail-command = "Game-End-Detail?" HSEP [ data-format WSS ]
                          game-end-detail-data
                          HEND
```

ここで、データの内容は以下のことを満たすデーターフォーマットに即したデータであり、このフォーマットはそれぞれのデータフォーマットによって定められる。

```
game-end-detail-data = ? any octets not including 2 times CRLF
                        starts with CRLF and ends with CRLF ?
```
