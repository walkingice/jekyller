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
    body .shout h2 {
        color: #222;
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
    body .cover figcaption {
        float: right;
        font-style: normal;
        font-size: 140%;
        color: #ccc;
        margin-top: -0.5em;
        margin-right: 0em;
    }
    body .cover blockquote {
        font-style: normal;
        font-size: 180%;
        margin-left: 4em;

    }
---

# [PgREST](http://pgre.st/)<br>Node.js in the Database {#_}

Audrey Tang

![](pictures/stars.jpg)
<!-- by-nc-sa orkomedix, https://secure.flickr.com/photos/orkomedix/6812055939 -->

## 不講故事<br>只講程式
{:.shout #no-stories-just-code}

## PgREST is...

* **JSON** document store
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
CREATE FUNCTION getKey(obj JSON, key TEXT) returns JSON AS $$
   return JSON.stringify( obj[key] );
$$ LANGUAGE plv8;

SELECT getKey(entry, 'bopomofo') FROM moe;
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
plv8x -E 'plv8.execute """
  SELECT entry FROM moe
""" .0.entry.pinyin.toUpperCase!'
# MÉNG
~~~

## plv8x: Modules

~~~ php
$ npm i -g uax11
$ plv8x -i uax11
$ plv8x -E 'require "uax11" .toFullwidth "méng"'
# ｍｅ́ｎｇ
~~~

~~~ sql
SELECT entry ~> 'require "uax11" .toFullwidth @pinyin' FROM moe;
-- "ｍｅ́ｎｇ"
~~~

## plv8x: Functions

~~~ php
$ plv8x -f 'text fullwidth(text)=uax11:toFullwidth'
$ plv8x -f 'text PINYIN(json)=:&0.pinyin.toUpperCase!'
~~~

~~~ sql
SELECT fullwidth('Ingy döt Net');
-- Ｉｎｇｙ　ｄｏ̈ｔ　Ｎｅｔ
SELECT fullwidth( PINYIN(entry) ) FROM moe;
-- ＭＥ́ＮＧ
~~~

## plv8x: Summary

* `V8`: JavaScript engine
* `PLV8`: Stored procedures in JavaScript
* `plv8x`: Package manager for PLV8
    * Turns **NPM** modules into **SQL functions**
    * **JSON** expressions with `~>` and `<~`
* Code reuse for **browser** + **server** + **database**!

## Interlude

## Request Form
![](pictures/twblg-request.jpg)

## 〈回答〉
{:.cover #answer}

<figure markdown="1">
> 新的轉機和閃閃星斗，<br>
> 正在綴滿沒有遮攔的天空。<br>
> 那是五千年的象形文字，<br>
> 那是未來人們凝視的眼睛。
<figcaption>／北島</figcaption>
</figure>
![](pictures/stars.jpg)
<!-- by-nc-sa orkomedix, https://secure.flickr.com/photos/orkomedix/6812055939 -->
