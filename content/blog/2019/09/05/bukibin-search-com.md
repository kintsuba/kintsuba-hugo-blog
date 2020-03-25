---
title: bukibin-search.com ができるまで
date: 2019-09-05 07:27:34
categories:
- Technology
tags:
- 不器用ビンボーダンス
- Nuxt.js
- Buefy
- Netlify
- TypeScript
---

今日はなんと私の誕生日です、きんつばです。  
[めいすきー](https://misskey.m544.net/)で今日誕生日なので祝ってって投稿したら、ちゃんとみんなお祝いの言葉をくれたのが嬉しかったです。

さて誕生日とは何も関係なく、今回は<abbr title="不器用ビンボーダンス">武器瓶</abbr>のファンサイトというかツールのようなものを作ったというお話です。  
不器用ビンボーダンスについてご存じない方は、[前回の記事](/2019/08/03/about-bukiyo-binbo-dance/)で語っていますのでそちらを参照ください。


## できたもの
ひとまず完成したものがこちらになります。

https://bukibin-search.com/

どんなものを作ったのか見ていただいたところで、ここからは完成までの道程を書いていきたいと思います。
<!-- more -->
## きっかけ
作り始めた理由なんですけど、LINEのオープンチャットってご存じでしょうか。ちょっと前に始まったやつですね。  
武器瓶を書かれている[なをををををを](https://twitter.com/70_pocky)さんが、武器瓶について語る用のオープンチャットのグループを作っていて、もちろん私も参加しています。参加するためのリンクは気合入れて探せば見つかります、多分。  
でまぁ、このチャットについて語るだけでも1記事書けるくらいの濃いチャットなのですが、ここで誰かがセリフ検索したいとかそんなことを言っていたのです。  
それを見て、そこまで難しくなさそうだし挑戦するだけ挑戦してみようかな、くらいで作り始めました。

## こうそう
そんな適当な感じで作り始めたとはいえ、一応勝算はありました。最近似たようなものを作っていたからです。  
それがこれです。

Enbu Wiki Linker - https://enbu-wiki-linker.site/

「幻想人形演舞」という東方の二次創作ゲームで使う自分用のツールで、キャラクターの名前から絞り込んで選んで、Wikiのそのキャラのページに飛ぶためのツールです。  
データを引っ張ってきて絞り込んで選ばせるという、基礎的なところは今回作りたいものとほぼ一緒なので、基礎はそのままでこれを作った時の反省を踏まえつつ作ろうと考えました。

というわけで安心と信頼の、というかこれしか知らないので今回も [Nuxt.js](https://ja.nuxtjs.org/) + [Buefy](https://buefy.org/) で作ることにしました。

ちなみにこの時考えていたのは以下のようなものでした。
1. 何かしらにセリフ一覧を登録しておく
2. セリフを引っ張って来て Buefy の autocomplete で一覧表示
3. 選んだセリフの詳細を表示（何話かはもちろん、発言キャラとか、できれば原作漫画とか）
4. 表示・非表示を切り替えられるものに、前後の台詞を表示させる（セリフのやり取りが面白い漫画ですからね）

## じっそう
### せりふ
とにもかくにも、セリフの一覧をどこかに入力しなければ何も始まりません。最初は[Cloud Firestore](https://firebase.google.com/docs/firestore?hl=ja)を使おうと思っていたのですが、以下のような問題がありました。  
- Webコンソール上から入力しようとするとかなり大変で、実質的に代替手段(Webアプリ等)が必要
- セリフが多く複数人で入力できるようにしたいが、それを実装するのは割と面倒。

ここで複数人で入力という点から、[Google スプレッドシート](https://www.google.com/intl/ja_jp/sheets/about/)を使うことを思いつきました。  
調べてみると、入力した表から JSON を生成して取得することもできるようでした。  
（参考: https://qiita.com/YukiIchika/items/778856a7ea92e5a2383c）  
スプレッドシートの利便性は周知の事実ということもあり、今回はこれを採用することにしました。

Nuxt.js側では以下のように取得しています。
```ts
async fetch({ $axios, store }) {
    const quotes: Quote[] = await $axios.$get(
      "https://script.google.com/macros/s/************************************/exec"
    );
    store.commit("setQuotes", quotes);
}
```
axios で GET して vuex に保存するシンプルな実装です。

### ぺーじ
あとはコードをゴリゴリ書いていくだけです。といっても大体は Buefy や Bulma の Doc を見ながら、適当にパーツを配置していくだけなので、そこまで難しいことはありませんでした。。

また、今回初めて Nuxt.js を TypeScript化した状態で開発したのですが、[https://typescript.nuxtjs.org/](https://typescript.nuxtjs.org/) に書いてある通りに進めれば、ほぼほぼ問題なく動かすことができました。  
型のありがたみを改めて実感したので、これから Nuxt.js を開発するときは絶対 TypeScript を導入しようと思います。

### ついったー
ただ、一点詰まった（あるいは現在進行形で詰まっている）ところがありまして、原作漫画の表示の部分です。  
毎日ツイッターに投稿されている作品なので、該当するツイートの ID をシートに入力しておいて、あとはページに埋め込んで完成！と思ったのですが……。

{{% img src="/images/2019/twitter-env.png" w="500" h="641" %}}

お分かりいただけますでしょうか、漫画の下辺が切れてしまっています。  
いろいろと試したのですが、未だに根本的な解決には至っていません。  
現在は漫画を開くと同時に `setTimeout` を発火させて、強制的に CSS を書き加えています。絶対他にもっといい方法あるとは思います……。

ただこの方法、なぜか最初に表示させた漫画には反応しないので、実際のところ何も対応できてないに等しいです。  
漫画の大部分は表示できているので、ひとまずはこのまま放置しようと思っています。

## おわりに
規模は小さいとはいえ、初めて不特定多数の目に触れるものを作れたことに自分の成長を感じることができました。  
本業とは何も関係ないのですがものづくり自体は楽しいので、これからもっと大きなものが作れるようになれたらいいなと思います。

最後に、セリフの入力作業を手伝ってくださった皆さん、またオープンチャットで反応をくださった方々、加えて利用してくださった方々に感謝の気持ちを述べて締めといたします。  
本当にありがとうございました！セリフの入力作業遅れてますけど、まだ更新する気はあるので少々お待ちください！

## 実は
輪ゴムの音だけ表示の仕方が違います。