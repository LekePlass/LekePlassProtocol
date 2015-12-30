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

<-> = ? US-ASCII hyphen (45) ?
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
```

## コマンド形式

```
field-name = field-word { <-> field-word } [<?>]
field-word = ASC_UPALPHA { ASC_ALPHA | ASC_DIGIT }
```

### ライン形式

```
command-line = field-name <:> { WS } line-value CRLF
line-value = uni_exclude_fmt { UNI_FMT | uni_exclude_fmt }
uni-exclude-fmt = UNI_DIGIT | UNI_LETTER | UNI_MARK | UNI_PUNC | UNI_SYM
```

### ヒアドック形式

```
command-heredoc = field-name <:> { WS } left-arrow { WS } [data-format { WS }]
                  ? any octets not including 2 times CRLF starts with CRLF ?
				  2 * CRLF
left-arrow = "<" "-"
data-format = data-format-name </> data-format-version
data-format-name = data-format-headchr { data-format-headchr | data-format-nonheadchr }
data-format-headchr = ASC_DIGIT | ASC_ALPHA
data-format-nonheadchr = "-" | ":" | ";" | "." | "," | "_" | "@" | "+"
                         "<" | ">" | "{" | "}" | "[" | "]" | "(" | ")"
data-format-version = (ASC_DIGIT | ASC_LOALPHA) { ASC_DIGIT | ASC_LOALPHA | "." }
```