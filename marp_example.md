---
marp: true
# header: 'header'
footer: 'internal use only, 2020 Fujitsu LTD'

theme: gaia
# theme: default

# size: 4:3
size: 16:9

page_number: true
paginate: true

style: |
  h1, h2, h3, h4, h5, header, footer {
    color: black;
  }
  section {
    justify-content: start;
    background-color: white;
    font-family: 'Yu Gothic UI';
    color: black;
  }
  section.blue {
    background-color: #0078D7;
    color: white;
  }
  section.blue > h1, h2, h3, h4, h5, header, footer{
    color: white;
  }


---

<!-- paginate: true -->
<!-- header: '' -->
<!-- footer: '' -->
<!-- class: '' -->
<!-- color: '' -->
<!-- backgroundColor: '' -->

<!-- _class: blue -->
# Marp for VS Code スライドサンプル

<br>
<br>
<br>

2020/07/20
takamitsu-iida

---

## もくじ

1. Marp for VS Codeとは
2. 使い方
3. ユースケース

---

## 1. Marp for VS Codeとは

- VS Codeの拡張機能
  マークダウンでスライドが作れる

---

## 2. 使い方

1. リスト
2. 背景画像

---

## 2.1 リスト

- One
- Two
- Three

---

## 2.2 背景画像

![](ubuntudde.png)

![bg right:50%](https://picsum.photos/720?image=29)

---

## 画像

```
* 画像を挿入
![](画像ファイルパス)

* 画像の大きさをスライドサイズの比率で指定する
![比率%](画像ファイルパス)

* 背景画像を挿入
![bg](画像ファイルパス)

* 背景画像の大きさをスライドサイズの比率で指定する
![bg 比率%](画像ファイルパス)

* 背景画像を連続して書くと水平方向（横）に分割される
![bg](画像ファイルパス)
![bg](画像ファイルパス)
```

---

## 数式

$$I_{xx}=\int\int_Ry^2f(x,y)\cdot{}dydx$$


---

## きれいな背景画像

<https://unsplash.com/>
