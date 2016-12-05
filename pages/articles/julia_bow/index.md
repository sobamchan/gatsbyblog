---
title: "julia nlp入門 [1]"
date: "2016-12-05"
layout: post
path: "/julia_bow/"
category: "julia"
description: "bowもどきをしながらjuliaに入門してみる"
---

## 経緯
数値計算や機械学習に強い言語として最近julia langがよく出てくるので入門してみたい

## やること
テキストファイルを読み込んでテキストごとの単語出現回数をカウントする。  
- テキストファイルの読み込み
- 辞書型を使ってみる
- 配列型を使ってみる

## やらないこと
- typeを使った型の定義

## 使うパッケージ
今回は何も使いません

## ソースコード
[ここにあります]()

## 実践
### ファイルの読み込み
```
function readfile(filename::String)
    docs = []
    open(filename) do f
        for doc in eachline(f)
            push!(docs, split(doc))
        end
    end

    return docs
end
```
ほとんどpythonと同じように読み出すことができました。  
関数定義内の引数の型指定はなくても大丈夫でした。(`filename::String`の所)  
- `split()`は文字列を引数にとって、スペースで区切って配列にしてくれます。
- `push!(array, hoge)`はarrayにhogeをpushします。juliaでは慣習的に破壊的メソッドには`!`を記述するようです

### 文章中の単語にidを振り、カウントする
```
function get_corpus(docs::Array)
    corpus = Dict() # {id => word}
    counts = Dict() # {id => count}
    word2id = Dict() # {word => id}
    for doc in docs
        for word in doc
            # exists
            if get(word2id, word, 0) != 0
                counts[word2id[word]] += 1
            # new
            else
                corpus[length(corpus)] = word
                word2id[word] = length(corpus)
                counts[word2id[word]] = 1
            end
        end
    end
    
    return corpus, word2id, counts
end
```
のちにidに割り振られた単語が何か調べることができるようにword2idを用意しています。  
単語のidに当たるのもはcorpus追加時のcorpus内の単語数にしています(重複していない整数であれば何でも別にいい)  
- `get(dict, string, 0)`はdict内にkeyがstringのものがあるか調べています。第三引数に当たる`0`は該当がなかった際に返される値です。該当がある場合はvalueを返します。


### 文章をとって、{id => count}を作成する
```
function get_vector(docs::Array, corpus, word2id, counts)
    bow_corpuses = [] # {id => count}
    for doc in docs
        bow_corpus = Dict() # {id => count}
        for word in doc
            # exists
            if get(bow_corpus, word2id[word], 0) != 0
                bow_corpus[word2id[word]] += 1
            # new
            else
                bow_corpus[word2id[word]] = 1
            end
        end
        push!(bow_corpuses, bow_corpus)
    end

    return bow_corpuses
end
```
さっきの内容とほとんど同じです。


### 上記で作成した関数を呼び出す
```
docs = readfile("./docs.txt")
corpus, word2id, counts = get_corpus(docs)
bow_corpuses = get_vector(docs, corpus, word2id, counts)
```

## 感想
今回はしっかりとコードを書くということはせず何となく書いてみたが、python(ruby)を知っている人なら馴染めるかなという感じだった。  
速度面の測定等はしなかったが早いというのは確かみたいなので、この雰囲気でPythonより高速で走らせることができるならばこれからの選択肢には全然入ってきそうな印象。  
ただクラスをpythonのように書けず少しトリッキーな表現方法になりそうなので調査が必要。

## 今後
juliaの大きな特徴の一つである、行列計算とニューラルネットワーク(フレームワーク選定を含む)の記述をしてみたい。
juliaのおお