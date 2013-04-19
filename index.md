---

layout: default

style: |
    #_ h2 a {
        border-bottom: 1px dotted #eee;
        color: white;
    }
    @font-face {
        font-family: 'Symbola';
        src: url('Symbola602.otf') format('truetype');
    }
    @font-face {
        font-family: 'Ubuntu Mono';
        src: url('UbuntuMono-R.ttf') format('truetype');
    }
    @font-face {
        font-family: 'Ubuntu';
        src: url('Ubuntu-R.ttf') format('truetype');
    }
    #_ h2 {
        margin:65px 0 0;
        color:#FFF;
        text-align:center;
        font-size:70px;
        }
    #_ p {
        margin: 2em 0 0;
        text-align: center;
        color: #FFF;
        font-style: italic;
        font-size: 150%;
        }
        #_ p a {
            color:#FFF;
            }
    .cover h2 {
        color:#FFF;
        }
    #big5-utf8.shout div h2 {
        font-family: 'Symbola' !important;
    }
    #twblg code {
        display: none;
    }
    span[style="color:#710"] {
        color: #800 !important;
        background: #fff !important;
    }

    span[style="color:#D20"] {
        color: #800 !important;
        background: #ffe !important;
    }
    span[style="color:#F00;background-color:#FAA"] {
        color: #000 !important;
        background: #ffe !important;
    }
    .cover h2 code {
        background: transparent;
        color: white;
        margin-left: -1em;
        font-weight: normal;
        font-family: 'Ubuntu Mono', 'Consolas', 'Menlo', monospace !important;
        }
    body .shout h2 {
        color: #222;
        font-weight: normal;
    }
    body .shout code {
        background: transparent;
        font-weight: normal;
    }
    pre {
        white-space: pre !important;
        font-family: 'Ubuntu Mono', 'Consolas', 'Menlo', monospace !important;
        }
    pre .line-numbers { display: none }
    body .slide:after { display: none }
    .slide img {
        border-radius: 0.5em;
          box-shadow: 0em 0.2em 0.3em 0px #999999;
    }
    .slide ul, p {
        font-family: 'Ubuntu', sans-serif;
    }
    .slide ul li, .slide ol li {
        font-size: 140%;
    }
    .slide ul li li, .slide ol li li {
        font-size: 1em;
    }
    .slide ul li strong, .slide ol li strong {
        color: #c00;
    }
    .shout {
        background: #eee;
    }
    iframe {
        width: 800px;
        height: 400px;
    }
    #SeeMore h2 {
        font-size:100px;
        }
    #SeeMore img {
        width:0.72em;
        height:0.72em;
        }
        br {
            line-height: 1.5em;
        }

    body .cover figure {
        color: white;
    }
    body figcaption {
        float: right;
        font-style: normal !important;
        font-size: 140%;
    }
    body .cover figcaption {
        color: #ccc;
        margin-top: -0.4em;
        margin-right: 0em;
    }
    body blockquote {
        font-style: normal !important;
        font-size: 150%;
    }
    body .cover blockquote {
        font-style: normal;
        font-size: 180%;
        margin-left: 3em;

    }
---

# [PgREST](http://pgre.st/) <br>Node.js in the Database {#_}

Audrey Tang

![](pictures/stars.jpg)
<!-- by-nc-sa orkomedix, https://secure.flickr.com/photos/orkomedix/6812055939 -->

## 只講程式<br>不講故事
{:.shout #just-code-no-stories}

## PgREST is...

* …**JSON** document store
* …Running inside **PostgreSQL**
* …Working with existing **relational** data
* …Capable of loading **Node.js** modules
* …Compatible with MongoLab's **REST API**
* …= `LiveScript` + `PLV8` + `plv8x` + `OneJS`

## JSON

~~~ json
{ "title":    "萌",
  "bopomofo": "ㄇㄥˊ",
  "pinyin":   "méng",
  "definitions": [
    { "type": "名", "def": "草木初生的芽。" },
    { "type": "名", "def": "事物發生的開端或徵兆。" },
    { "type": "名", "def": "人民。" } ] }
~~~

## PostgreSQL

~~~ sql
CREATE TABLE moe ( "entry" JSON );
INSERT INTO moe VALUES ($$
{ "title":"萌", "bopomofo": "ㄇㄥˊ", "pinyin": "méng",
  "definitions": [ { "type": "名", "def": "草木初生的芽。" },
                   { "type": "名", "def": "事物發生的開端或徵兆。" },
                   { "type": "名", "def": "人民。" } ] } $$);

INSERT INTO moe VALUES ('這不是 ㄓㄟ ㄙㄣˇ'); -- type error
~~~

## PLV8

~~~ sql
CREATE EXTENSION plv8;
CREATE FUNCTION get_json_key(obj JSON, key TEXT) returns JSON AS $$
   return JSON.stringify( obj[key] );
$$ LANGUAGE plv8;

SELECT get_json_key(entry, 'bopomofo') FROM moe;
-- "ㄇㄥˊ"
~~~

## plv8x: Operators

~~~ sql
SELECT entry |> 'this.bopomofo' FROM moe;
-- "ㄇㄥˊ"
SELECT entry ~> '@bopomofo' FROM moe;
-- "ㄇㄥˊ"
SELECT '@bopomofo' <~ entry FROM moe;
-- "ㄇㄥˊ"
SELECT ~> 'new Date' FROM moe;
-- "2013-04-17T12:31:57.523Z"
~~~

## plv8x: Command Line

~~~ php
npm i -g plv8x
export PLV8XCONN=dbname
plv8x -r script.ls # .js works too
plv8x -E 'plv8.execute("SELECT entry FROM moe").0.entry.definitions'
# [ { type: '名', def: '草木初生的芽。' },
#   { type: '名', def: '事物發生的開端或徵兆。' },
#   { type: '名', def: '人民。' } ]
~~~

## plv8x: Modules

~~~ php
npm i -g uax11
plv8x -i uax11
plv8x -E 'require "uax11" .toFullwidth "méng"'
# ｍｅ́ｎｇ
~~~

~~~ sql
SELECT entry ~> 'require "uax11" .toFullwidth @pinyin' FROM moe;
-- "ｍｅ́ｎｇ"
~~~

## plv8x: Functions

~~~ php
plv8x -f 'text fullwidth(text)=uax11:toFullwidth'
plv8x -f 'text PINYIN(json)=:&0.pinyin.toUpperCase!'
~~~

~~~ sql
SELECT fullwidth('Ingy döt Net');
-- Ｉｎｇｙ　ｄｏ̈ｔ　Ｎｅｔ
SELECT fullwidth( PINYIN(entry) ) FROM moe;
-- ＭＥ́ＮＧ
~~~

## Summary

* …`V8`: JavaScript engine
* …`PLV8`: Stored procedures in JavaScript
* …`plv8x`: Package manager for PLV8
    * …Turns **NPM** modules into **SQL functions**
    * …**JSON** expressions with `~>` and `<~`
* …Code reuse for **browser** + **server** + **database** !

## Cutting out the Middleware


## `3du.tw`
{:.cover #3du}
![](pictures/3du.jpg)
<!-- by Kanko, https://secure.flickr.com/photos/kankan/134663487/ -->

## The Revised MoE Dictionary (1994)

<iframe src="data:http://dict.revised.moe.edu.tw/"></iframe>

## The Good

* 160,000+ entries
* …Official, high quality sources
* …Rich etymology and historical usage
* …Full text search with regular expressions
* …Still frequently updated!

## The Bad

* Results are not bookmarkable
* …Requires N clicks to get to a definition
* …Rare characters become low-res bitmaps
* …Difficult to use on mobile devices
* …"Optimized for IE 5.0 and Netscape 4.7+"!?

## The Sad

<figure markdown="1">
> 本會非常歡迎各位來連結「國語辭典」，但是本會目前只開放以超連結 (hyperlink) 的方式與國語辭典 **首頁** 連結，至於其他方式本會並未對外開放授權。若還有疑問或建議，歡迎來信。 
<figcaption>／教育部國語推行委員會〈有關授權〉</figcaption>
</figure>

## .…and the Very Crazy

> 不需登入的網頁，會自動把你登出！

![](pictures/dict-logout.png)

## Yeh's Ping, 2013.1.26.

<figure markdown="1">
> 所以我要響應零時政府 g0v.tw 的活動，來做 3du.tw，把字、詞、成語、定義、例句等等正體中文資料，用開放的文字 API 釋放出來，加上索引和搜尋的功能，讓任何想加值的個人或公司都可以使用。 
<figcaption>／葉平〈還文於民〉</figcaption>
</figure>

## 零時黑客<br>集體砍站事件
{:.shout #g0v-scraping}

## Hackpad for 3du.tw

![](pictures/hackpad.png)

## g0v hackath1n, 2013.1.27.

* …Scraped 3000 characters into HTML (au)
* …Scraped 2741 idioms into HTML (TonyQ, MnO2)
* …Designed JSON schema from samples (Ping)
* …Designed SQL schema from samples (albb0920)
* …Parsed HTML into JSON & SQLite (kcwu)

## `←`&#x1f01d; `Big-5`<br>`→`&#x1f00e; `UTF-8`
{:.shout #big5-utf8}

## Crowd-sourcing 1000+ glyphs

![](pictures/thinking.png)

## Finished in 24 hours!

Thanks to: Favonia, Jun-Yuan Yan, Yao Wei, Yaoting Huang, Poka, Caasi Huang, Daniel Liang, Grey Lee, Irvin Chen, Gugod, Schee…
![](pictures/unimap.png)

## 粗略的共識<br>會動的程式
{:.shout #rough-consensus-running-code}

## Applications

* XUL Desktop App (@racklin)
* OS X Dictionary (@yllan)
* Windows 8 App (@wenpei)
* iOS Client (@tomjpsun, @jamessa, @pct)
* iOS App (@zonble)
 
## Integrations

* AngularJS Client+Server (@viirya)
* Rails API server (@albb0920)
* Chrome Extension (@tonytonyjan)
* Sublime Text plugin (@zonble)
* WinRT Component (@eriksk)

## `moedict.tw`
{:.cover #moedict}
![](pictures/moedict.png)

## 5 Stars of Open Data

1. ⊙ **Open** License
2. ↔ **Structured** Data
3. ▵ **Non-Proprietary** Format
4. …✧ Each Item has an **URI**
5. …✩ **Linking** between Items

## URI Endpoints

* `https://moedict.tw/#文字`
* 3 APIs (for __non-Unicode__ characters):
    * `/raw/文字.json` ⇒ `{[8ff0]}` 
    * `/uni/文字.json` ⇒ `⿰亻壯`
    * `/pua/文字.json` ⇒ `U+F8FF0`

## Private-Use Web Fonts

* Initially based on Hán Nôm font (Yao Wei)
    * Subset everything outside Big5 range
    * Hand-drawn for PUA chars like ⿰亻壯
* …Later on, switched to Hanazono 花園明朝 font
    * 75,619 + 8,236 glyphs
    * From 花園大学国際禅学研究所

## 科技始終<br>來自於佛性
{:.shout #technology-buddha-nature}

## Live Demo

<iframe src="data:http://moedict.tw/"></iframe>
<!-- Replace with localhost -->

## Auto Linking

## `twblg.moedict.tw`
{:.cover #twblg}
![](pictures/seeking.jpg)

## Request Form
![](pictures/twblg-request.jpg)


## 宅心仁厚<br>仁者無敵
{:.shout #nerds-without-enemies}

## When is Transparency Useful?

<figure markdown="1">
> 眾人為了共同目標聚在一起，才能做出改變，科技人很難獨力完成。<br>
> 衡量成功的標準，可以是有多少人的生命因你獲得改善，而不只是有多少人看你架的網站。
<figcaption>— Aaron Swartz, «Open Government»</figcaption>
</figure>

## 開站一時<br>開源一輩子
{:.shout #site-a-time-source-a-lifetime}

## Thank you!
{:.cover #answer}

<figure markdown="1">
> 新的轉機和閃閃星斗，<br>
> 正在綴滿沒有遮攔的天空。<br>
> 那是五千年的象形文字，<br>
> 那是未來人們凝視的眼睛。
<figcaption>／北島〈回答〉</figcaption>
</figure>
![](pictures/stars.jpg)
<!-- by-nc-sa orkomedix, https://secure.flickr.com/photos/orkomedix/6812055939 -->
