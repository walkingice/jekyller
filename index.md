---

layout: default

style: |
    #_ h2 a {
        border-bottom: 1px dotted #eee;
        color: white;
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
    #twblg code {
        display: none;
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
    .slide ul li {
        font-size: 140%;
    }
    .slide ul li li {
        font-size: 1em;
    }
    .slide ul li strong {
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

## 不講故事<br>只講程式
{:.shout #no-stories-just-code}

## PgREST is...

* {:.next}**JSON** document store
* {:.next}Running inside **PostgreSQL**
* {:.next}Working with existing **relational** data
* {:.next}Capable of loading **Node.js** modules
* {:.next}Compatible with MongoLab's **REST API**
* {:.next}= `LiveScript` + `PLV8` + `plv8x` + `OneJS`

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

* {:.next}`V8`: JavaScript engine
* {:.next}`PLV8`: Stored procedures in JavaScript
* {:.next}`plv8x`: Package manager for PLV8
    * {:.next}Turns **NPM** modules into **SQL functions**
    * {:.next}**JSON** expressions with `~>` and `<~`
* {:.next}Code reuse for **browser** + **server** + **database** !

## `3du.tw`
{:.cover #3du}
![](pictures/3du.jpg)
<!-- by Kanko, https://secure.flickr.com/photos/kankan/134663487/ -->

## 3du.tw on Hackpad

<iframe src="data:http://3du.tw/"></iframe>
<!-- Replace with screenshot? -->

## `moedict.tw`
{:.cover #moedict}
![](pictures/moedict.png)

## Live Demo

<iframe src="data:http://moedict.tw/"></iframe>
<!-- Replace with localhost -->

## 5 Stars of Open Data

* ⊙ Open License
* ↔ Structure Data Format
* ▵ Non-Proprietary Format
* ✧ Each items has an URI
* ✩ Link to other data

## `twblg.moedict.tw`
{:.cover #twblg}
![](pictures/seeking.jpg)

## Request Form
![](pictures/twblg-request.jpg)

## `←`&#x1f01d; `Big-5`<br>`→`&#x1f00e; `UTF-8`
{:.shout #big5-utf8}

## 零時黑客<br>集體砍站事件
{:.shout #g0v-scraping}

## 粗略的共識<br>會動的程式
{:.shout #rough-consensus-running-code}

## 宅心仁厚<br>仁者無敵
{:.shout #nerds-without-enemies}

## 科技始終<br>來自於佛性
{:.shout #technology-buddha-nature}

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

## Inner Navigation

1. Lets you reveal list items one by one
2. …To keep some key points
3. …In secret from audience
4. …But it will work only once
5. …Nobody wants to see the same joke twice
