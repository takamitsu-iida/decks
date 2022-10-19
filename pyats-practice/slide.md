<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# pyATSの実行例

<br>

pyATSはネットワークの検証に大変便利なツールですが、残念ながら日本語で入手できる情報は少なめです。

少しでも導入のハードルが下がるように動作例を提示します。

<br><br>

<div class="textright">
Takamitsu IIDA (@takamitsu-iida)
</div>

---

### はじめに

自動化といえばansibleの方がメジャーな地位を確立していますが、ネットワーク機器を使った検証作業はpyATSの方が便利です。

ですが、日本語で得られる情報が少なめなこと、ネットワークとPython両方の専門家であることが前提になっていることから、少々導入のハードルが高めです。

最初はスクリプトの例をコピペする感じで試してもらえれば幸いです。

---

## pyATSとは

pyATS = python Automated Test System

2017年頃にオープンソースになったCisco社製のツールです。

> Cisco currently run over 2 million test runs per month using the pyATS framework.

<!--
引用元 https://www.rogerperkin.co.uk/network-automation/pyats/pyats-genie-tutorial/ -- >
-->

どんなものなのかは、下記ページにあるIntroduction to pyATSが秀逸なのでぜひトライしてください。
最速で理解できます。

<p><a href="https://developer.cisco.com/pyats/" target="_blank">https://developer.cisco.com/pyats/</a></p>

---

### 環境づくり

オススメはWindows上のWSL(Ubuntu)にPython環境を構築することです。

pyATSはやっぱりいらないな、となるかもしれませんので、venvを使って適当なディレクトリにインストールするといいでしょう。

```bash
$ python3 -m venv .venv
```

direnvをインストールして自動でactivateすると便利です。

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

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex10.execute.py" target="_blank">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex10.log" target="_blank">log</a>]
</p>


接続してコマンドを打ち込む、基本中の基本です。
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

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex13.execute.py" target="_blank">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex13.log" target="_blank">log</a>]
</p>

複数台まとめてログ取りできます。

logディレクトリに装置ごとにログファイルを作ります。

```python
#!/usr/bin/env python

import sys
import os

#
# overwrite standard telnetlib
#
def here(path=''):
  return os.path.abspath(os.path.join(os.path.dirname(__file__), path))

if not here('./lib') in sys.path:
  sys.path.insert(0, here('./lib'))

import telnetlib
if telnetlib.MODIFIED_BY:
    print('modified telnetlib is loaded.')

#
# pyATS
#

# import Genie
from genie.testbed import load
from unicon.core.errors import TimeoutError, ConnectionError, SubCommandFailure

# show commands
command_list = [
    'show version',
    'show cdp neighbors',
    'show ip ospf neighbor',
    'show ip route'
]

# log directory
log_dir = os.path.join(here('.'), 'log')

testbed = load('lab.yml')

uut = testbed.devices['uut']

for name, dev in testbed.devices.items():
    # テストベッド内のすべてのCSR1000vを対象に
    if dev.platform == 'CSR1000v':
        # ファイルをオープンしてログ取り開始
        log_path = os.path.join(log_dir, f'ex13_{name}.log')
        with open(log_path, 'w') as f:
            # connect
            try:
                dev.connect(via='console')
            except (TimeoutError, ConnectionError, SubCommandFailure) as e:
                f.write(str(e))
                continue

            # execute
            for cmd in command_list:
                try:
                    f.write('\n===\n')
                    f.write(cmd)
                    f.write('\n===\n')
                    f.write(dev.execute(cmd))
                    f.write('\n\n')
                except SubCommandFailure:
                    f.write(f'`{cmd}` invalid. Skipping.')

            # disconnect
            if dev.is_connected():
                dev.disconnect()
```

---

### 例．コマンドの実行結果をパース

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex20.parse.py" target="_blank">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex20.log" target="_blank">log</a>]
</p>

uut.execute('show version) を uut.parse('show version) にするだけで、結果が辞書型になります。

```python
#!/usr/bin/env python

#
# 単体のコマンドをパースする
#

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

サポートしているコマンドパーサーの検索は
<a href="https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/parsers" target="_blank">こちら</a>

---

### 例．機能名で一括学習

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex30.learn.py" target="_blank">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex30.log" target="_blank">log</a>]
</p>

単体コマンドをパースするだけでも便利ですが、より抽象的な機能を指定して一括学習させることもできます。
routeを学習すればルーティングテーブルを、ospfを学習すればOSPFに関するあらゆる情報を取得できます。

```python
#!/usr/bin/env python

#
# 抽象的な機能名を指定して学習
#

# import Genie
from genie.testbed import load

testbed = load('lab.yml')

uut = testbed.devices['uut']

# connect
uut.connect(via='console')

# learn routing table
routing = uut.learn('routing')

# disconnect
if uut.is_connected():
    uut.disconnect()

from pprint import pprint
pprint(routing.info)
```

サポートしている機能名一覧は
<a href="https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/models" target="_blank">こちら</a>

---

### 例．パースした結果を検索

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex40.parse_find.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex40.log" target="_blank">log</a>]
</p>

show interfacesをパースすると結果は辞書型になります。このなかには大量の情報が格納されていますので欲しい情報を探すのはなかなか大変です。pyATSには辞書型を検索する機能もあって、簡単に欲しい情報を取り出せます。

```python
#!/usr/bin/env python

#
# find interfaces where out_pkts is 0.
#

from pprint import pprint

# import Genie
from genie.testbed import load

testbed = load('lab.yml')

uut = testbed.devices['uut']

# connect
uut.connect(via='console')

# parse
parsed = uut.parse('show interfaces')

# disconnect
if uut.is_connected():
    uut.disconnect()

# display parsed data
for name, data in parsed.items():
    pprint(data)

# find 力技で見つける
for name, data in parsed.items():
    if 'counters' in data:
       if 'out_pkts' in data['counters']:
           if parsed[name]['counters']['out_pkts'] == 0 :
                print(f"{name} is not used.")
                # uut.configure('int {}\n shutdown'.format(name))

# pyats find
# https://pubhub.devnetcloud.com/media/pyats/docs/utilities/helper_functions.html

from pyats.utils.objects import R, find

# from
# {interface_name: {'counters': {'out_pkts': 0

req = R(['(.*)', 'counters', 'out_pkts', 0])
found = find(parsed, req, filter_=False)
pprint(found)
```

---

### 例．もっと複雑な条件で検索

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex41.learn_find.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex41.log" target="_blank">log</a>]
</p>

たとえばshow interfacesの出力の中からoper_statusがupで、かつ、duplexがfullのインタフェースを探すなら、このように検索します。

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

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex42.learn_find.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex42.log" target="_blank">log</a>]
</p>

スパニングツリーのブロックポートがどこにあるのか、人力で探すのは大変です。

pyATSならテストベッド全体に対してstpを学習させて検索するだけです。

```python
req = R(['(.*)', 'info', 'pvst', 'default', 'vlans', '(.*)', 'interfaces', '(.*)', 'port_state', 'blocking'])
found = find(learnt, req, filter_=False)
```

---

### 例．インタフェースがアップするまで待機

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex43.learn_poll.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex43.log" target="_blank">log</a>]
</p>

条件を満たすまで、繰り返し学習を続けることもできます。

oper_statusが'up'かどうかを確認する関数を作って、learn_poll()するだけです。この例では条件を満たすまで5秒間待機を3回繰り返します。

```python
def verify_interface_status(obj):
    for name in obj.info.keys():
        oper_status = obj.info[name].get('oper_status', None)
        if oper_status == 'up':
            print("verified successfully")
            return
    raise Exception("Could not find any up interface")

intf.learn_poll(verify=verify_interface_status, sleep=5, attempt=3)
```

---

### 例．コピペ感覚でコンフィグを貼り付け

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex50.configure.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex50.log" target="_blank">log</a>]
</p>

投入したいコマンドを羅列して、コピペ感覚で装置に投入できます。

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

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex51.configure.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex51.log" target="_blank">log</a>]
</p>

Pythonのオブジェクトに対して値を設定するだけで、ルータの設定ができてしまいます。これだけでインタフェースGig1のdescriptionが書き換わります。

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

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex60.diff.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex60.log" target="_blank">log</a>]
</p>

- 作業前のコンフィグをlearn()
- 作業後のコンフィグをlearn()
- diffを出力

こんな感じの出力が得られます。+が増えた部分、-が消えた部分です。

```bash
r1#
+Current configuration : 6519 bytes:
-Current configuration : 6520 bytes:
 interface GigabitEthernet1:
+ ip ospf cost 10:
- ip ospf cost 100:
```

---

### 例．OSPFの状態の変化を確認したい

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex61.diff.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex61.log" target="_blank">log</a>]
</p>

コンフィグではなくて、状態データとしての変化を知りたいこともありますね。

OSPFの場合、こんな感じで差分を検出できます。

```bash
 info:
  vrf:
   default:
    address_family:
     ipv4:
      instance:
       1:
        areas:
         0.0.0.0:
          interfaces:
           GigabitEthernet1:
-           cost: 100
+           cost: 10
```

---

### 例．ルーティングテーブルを変化を確認したい

<p>
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/ex62.diff.py">source</a>]　
[<a href="https://github.com/takamitsu-iida/pyats-practice/blob/main/output/ex62.log" target="_blank">log</a>]
</p>

- 作業前のルーティングテーブルをlearn()
- 作業後のルーティングテーブルをlearn()
- diffを出力

こんな感じの出力が得られます。+が増えた部分、-が消えた部分です。

```bash
 info:
  vrf:
   default:
    address_family:
     ipv4:
      routes:
+      192.168.100.0/24:
+       active: True
+       next_hop:
+        outgoing_interface:
+         Null0:
+          outgoing_interface: Null0
+       route: 192.168.100.0/24
+       source_protocol: static
+       source_protocol_codes: S
```

---

### 例．疎通確認結果を判定

結果をわかりやすく判定するならテストフレームワークを使うと便利です。

pingした結果がOKなのかNGなのか、わかりやすく表示します。

<p>
<a href="https://github.com/takamitsu-iida/pyats-practice/tree/main/job01_ping" target="_blank">こちら</a>
</p>

を参照してください。

---

### 例．インタフェースの状態を判定

テストベッド内の全装置に乗り込み、インタフェースの状態を学習。duplexがfullであればOKとして判定します。

<p>
<a href="https://github.com/takamitsu-iida/pyats-practice/tree/main/job03_duplex" target="_blank">こちら</a>
</p>

を参照してください。


---

### 例．ルーティングテーブルの状態を判定

事前に保存しておいたルーティングテーブルの情報と、今現在のルーティングテーブルを比較して、差分がなければOKとして判定します。

<p>
<a href="https://github.com/takamitsu-iida/pyats-practice/tree/main/job04_duplex" target="_blank">こちら</a>
</p>

を参照してください。


---

### さいごに

<br>

楽をするための苦労はいとわない。

<br>

ネットワークの自動化って、そういうことだと思います。
