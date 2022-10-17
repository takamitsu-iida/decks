<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# pyATSの実行例

<br>

pyATSに関して、日本語で入手できる情報は少なめです。

実際に試してみたほうが早いのですが、少しでもハードルが下がるように動作例を提示します。

<br><br><br>

##### Takamitsu IIDA (@takamitsu-iida)

---

### はじめに

自動化といえばansibleの方がメジャーな地位を確立していますが、ネットワーク機器を使った検証作業はpyATSの方が便利です。

pyATSは動作が俊敏で、ネットワーク機器をテストするための環境、道具立てが整っているのがその理由です。

とはいえ、日本語で得られる情報が少なめなこと、ネットワークの専門家であることが前提になっていることから、少々導入のハードルが高めです。

ここでは実際に動かす例を提示して、そのハードルを少しでも下げようと思います。

---

## pyATSとは

このページにあるIntroduction to pyATSが秀逸なので、ぜひトライしてください。

https://developer.cisco.com/pyats/


---

### 環境づくり

- pyATSの実行環境はWindows上のWSL(Ubuntu)に構築します

- venvを使って個人の環境にインストールします

```bash
$ python3 -m venv .venv
```

- direnvを使って自動でactivateします

```bash
echo 'source .venv/bin/activate' > .envrc
echo 'unset PS1' >> .envrc
direnv allow
```

---

### pyATSのインストール

- pyatsに関連したモジュールを全てインストールするにはこのようにします。

```bash
pip install pyats[full]
pip install rest.connector
pip install yang.connector
```

---

### 検証環境

- eve-ngやCMLを使って環境を組むと検証がはかどります

- ここではeve-ngを使います

![構成図](https://takamitsu-iida.github.io/pyats-practice/img/fig1.PNG "構成図")

---

### 例．コマンドの投げ込み


---

### 例．


---

### 例．


---

### 例．


---

### 例．



---

### 例．

---

### 例．

---

### 例．

---

### 例．

---

### 例．
