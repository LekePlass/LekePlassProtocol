# 草案

@mizunashi

## SFEN拡張

基本的にSFEN準拠。対局者情報付属などを変更。

## MOVE_INFO

USIの情報に付属して、消費時間を加える

## 例

`>|` : serverからの
`<|` : clientからの
`[`文字`]` : その文字を意味する値
`\\` : 空行
`#` : 改行までコメントアウト

```
>|PING
<|PONG
>|PROTOCOL_INFO/
>|Version:[PROTOCOL_VERSION]
>|Mode:[PROTOCOL_MODE]
>|\\
>|GAME_INFO/
>|YOUR_TURN:[CLIENT_ID]
>|START_POS:[START_POS_SFENE]
>|\\
>|EXT_INFO/
>|/TIME_OPTION/
>|TIME_UNIT:[TIME_UNIT] # N秒かを指定、小数も可
>|TOTAL_TIME:[TOTAL_TIME]
>|BYOYOMI:[BYOYOMI]
>|EVERY_LEAST_TIME:[LEAST_TIME]
>|/LAST_INFO/
>|REMATCH_ON_DRAW:[IS_REMATCH]
>|\\
>|ISOK
<|AGREE # またはREJECT
>|START
>|POSITION:[POS_SFENE]/ # POS_SFENはbeforepos/startposも許容
>|/MOVES:[MOVE_INFO]/
>|/MOVES:[MOVE_INFO] # movesは省略可
>|GO
<|MOVE:[MOVE_INFO] # resign(投了)も許容
>|END
>|RESULT:[GAME_RESULT] # win/lose/drawで指定
>|REASON:[GAME_REASON] # 省略可、勝った場合win
>|QUIT
```

