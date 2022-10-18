<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# pyATSの実行例

<br>

pyATSに関して、日本語で入手できる情報は少なめです。

実際に試してみたほうが早いのですが、少しでもハードルが下がるように動作例を提示します。

<br><br>

##### Takamitsu IIDA (@takamitsu-iida)

---

### はじめに

自動化といえばansibleの方がメジャーな地位を確立していますが、ネットワーク機器を使った検証作業はpyATSの方が便利です。

ですが、日本語で得られる情報が少なめなこと、ネットワークの専門家であることが前提になっていることから、少々導入のハードルが高めです。

ここでは実際に動かす例を提示して、そのハードルを少しでも下げようと思います。

---

## pyATSとは

pyATS = python Automated Test System

2017年頃にオープンソースになったCiscoの社内ツールです。

このページにあるIntroduction to pyATSが秀逸なのでぜひトライしてください。最速で理解できます。

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

<img src="https://takamitsu-iida.github.io/pyats-practice/img/fig1.PNG" height=60%/>

---

### 例．コマンドの投げ込み

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex10.execute.py

接続して、コマンドを打ち込む、基本中の基本です。
たったこれだけのコードで動きます。

```python
#!/usr/bin/env python

# import Genie
from genie.testbed import load

testbed = load('lab.yml')

uut = testbed.devices['uut']

# connect to the uut
uut.connect(via='console')

# execute command
output = uut.execute('show version')

# print output
from pprint import pprint
pprint(output)
```

---

### 例．複数装置・複数コマンド

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex13.execute.py

ログ取りです。

1. 対象装置に接続

2. コマンドを順番に投げ込み

3. ファイルに保存

---

### 例．コマンドの実行結果をパース

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex20.parse.py

コマンドを実行した結果のテキストを読み取る処理は、コマンドごとにフォーマットが違うので、自分で実装するのは大変です。

pyATSのGenieにはパーサーが豊富に揃っていますので、簡単にPythonの辞書型に変換してくれます。

`uut.execute('show version)` を `uut.parse('show version)` にするだけで、結果が辞書型になります。

```python
#!/usr/bin/env python

# import Genie
from genie.testbed import load

testbed = load('lab.yml')

uut = testbed.devices['uut']

# connect
uut.connect(via='console')

# parse "show version"
output = uut.parse('show version')

# disconnect
if uut.is_connected():
    uut.disconnect()

from pprint import pprint
pprint(output)
```

パーサーの検索はここから

https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/parsers

---

### 例．機能名で一括学習

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex30.learn.py

単体コマンドをパースするだけでも便利ですが、より抽象的な機能を指定して一括学習させることもできます。

`route`を学習すればルーティングテーブルを、`ospf`を学習すればOSPFに関するあらゆる情報を取得できます。


機能名一覧はここから

https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/models

---

### 例．パースした結果を検索

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex40.parse_find.py

`show interfaces`をパースすると結果は辞書型になります。

そのなかから欲しい情報を探すのは、なかなか大変です。

pyATSには辞書型を検索する機能もあります。

out_pktsが0になっているインタフェースを見つけるのは、たったこれだけです。

```python
req = R(['(.*)', 'counters', 'out_pkts', 0])
found = find(parsed, req, filter_=False)
pprint(found)
```

---

### 例．もっと複雑な条件で検索

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex41.learn_find.py

oper_statusがupで、かつ、duplexがfullのインタフェースを探すなら、こうします。

```python
req3 = [
    R(['info', '(?P<interface>.*)', 'oper_status', 'up']),
    R(['info', '(?P<interface>.*)', 'duplex_mode', 'full'])
]
intf_up_full = find(intf, *req3, filter_=False)
print("up and full duplex interfaces")
pprint(intf_up_full)
```

---

### 例．どのスイッチのどのポートがブロック？

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex42.learn_find.py

スパニングツリーでは、どこにブロックポートができているか、人力で探すのは大変です。

pyATSならこれだけです。

```python
req = R(['(.*)', 'info', 'pvst', 'default', 'vlans', '(.*)', 'interfaces', '(.*)', 'port_state', 'blocking'])
found = find(learnt, req, filter_=False)
```

---

### 例．インタフェースがアップするまで待機

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex43.learn_poll.py

条件を満たすまで、繰り返し学習を続けることもできます。

oper_statusが'up'かどうかを確認する関数を作って、

```python
def verify_interface_status(obj):
    for name in obj.info.keys():
        oper_status = obj.info[name].get('oper_status', None)
        if oper_status == 'up':
            print("verified successfully")
            return
    raise Exception("Could not find any up interface")
```

learn_poll()するだけです。この例では条件を満たすまで5秒間待機を3回繰り返します。

```python
intf.learn_poll(verify=verify_interface_status, sleep=5, attempt=3)
```

---

### 例．コピペ感覚でコンフィグを貼り付け

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex50.configure.py

```python
#!/usr/bin/env python

# import Genie
from genie.testbed import load

testbed = load('lab.yml')

uut = testbed.devices['uut']

# connect to the uut
uut.connect(via='console')

# configure
output = uut.configure('''
interface Gig1
description "configured by pyats"
exit
interface Gig2
description "configured by pyats"
exit
''')

# disconnect
if uut.is_connected():
    uut.disconnect()

from pprint import pprint
pprint(output)
```

---

### 例．Pythonicに設定を投入

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex51.configure.py

Pythonのオブジェクトに対して値を設定するだけで、ルータの設定ができてしまいます。

たったこれだけでインタフェースGig1のdescriptionが書き換わります。

```python
from genie.conf.base import Interface

gig1 = Interface(device=uut, name='GigabitEthernet1')

gig1.description = "configured by Genie Conf Object"

# verify config
print(gig1.build_config(apply=False))

# apply config
gig1.build_config(apply=True)
```

---

### 例．設定が書き換わったか確認したい

https://github.com/takamitsu-iida/pyats-practice/blob/main/ex60.diff.py

作業前のコンフィグ、作業後のコンフィグでdiffをとって確認しましょう。

```python
# learn configuration
pre_conf = uut.learn('config')

# change ospf config, cost 100 -> 10
uut.configure('''
interface Gig1
ip ospf cost 10
exit
''')

# learn current config
post_conf = uut.learn('config')

# generate diff
config_diff = Diff(pre_conf, post_conf)
config_diff.findDiff()
print(config_diff)
```

```bash
r1#
+Current configuration : 6519 bytes:
-Current configuration : 6520 bytes:
 interface GigabitEthernet1:
+ ip ospf cost 10:
- ip ospf cost 100:
```

---

### 例．疎通確認をOKかNGかわかりやすく！

テストフレームワークを使うと便利です。


---

### 例．インタフェースの状態が正しいかわかりやすく！





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
