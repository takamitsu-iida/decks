<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# Ansible Role

<br>

## cisco_show_run

show running-configの出力をファイルに保存するロールです。

<br><br>

##### Takamitsu IIDA (@takamitsu-iida)

---

### はじめに

<br>
<br>

ネットワーク機器の設定を変更する場合、作業の前後でログをファイルに保存します。

ログ採取を簡単にできるようにするロールを作っておくと何かと便利です。

---

### ロールの置き場

- 作成したロールはgithubに置いておくと便利です。
- githubのリポジトリ名はansible-role-を先頭に付けるのが慣例のようです。
- このロールの場合はansible-role-cisco_show_runがリポジトリ名です。

---

### ロールのインストール

- ansible-galaxyコマンド一発でロールをインストールできます。

```bash
ansible-galaxy install -p ./roles
 git+https://github.com/takamitsu-iida/ansible-role-cisco_show_run.git
```

- cisco_show_runというフォルダ名でインストールされます。
- ご希望のロール名に合わせてフォルダ名を変えてください。

---

### 使い方・その１

- 設定変更するタスクの前後に挿入するとよいでしょう。

```yml
  tasks:

    - include_role:
        name: cisco_show_run

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
```

---

### 結果・その１

- このようにログファイルが生成されます。

```bash
log
├── r1_show_run_2019-01-11@12:07:13.txt
├── r1_show_run_2019-01-11@12:07:30.txt
├── r2_show_run_2019-01-11@12:07:13.txt
├── r2_show_run_2019-01-11@12:07:30.txt
├── r3_show_run_2019-01-11@12:07:13.txt
├── r3_show_run_2019-01-11@12:07:30.txt
├── r4_show_run_2019-01-11@12:07:13.txt
└── r4_show_run_2019-01-11@12:07:30.txt
```

- 全ての対象ノードのファイルをlogフォルダに集めていますので、どれが作業前なのか判別しづらいです。

---

### 使い方・その２

- ファイル名の先頭で区別を付けたい場合は、ロールを呼び出すときにPREFIX変数を上書きします。

```yml
  tasks:

    - include_role:
        name: cisco_show_run
      vars:
        PREFIX: before_

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
      vars:
        PREFIX: after_
```

---

### 結果・その２

- ファイル名の先頭をみれば、作業前なのか、作業後なのか識別できます。

```bash
log/
├── after_r1_show_run_2019-01-11@12:05:42.txt
├── after_r2_show_run_2019-01-11@12:05:43.txt
├── after_r3_show_run_2019-01-11@12:05:43.txt
├── after_r4_show_run_2019-01-11@12:05:43.txt
├── before_r1_show_run_2019-01-11@12:05:26.txt
├── before_r2_show_run_2019-01-11@12:05:26.txt
├── before_r3_show_run_2019-01-11@12:05:26.txt
└── before_r4_show_run_2019-01-11@12:05:26.txt
```

---

### 使い方・その３

- 作業前と作業後でログ置き場を変えたい場合は、変数LOG_PATHを変更します。

```yml
  tasks:

    - include_role:
        name: cisco_show_run
      vars:
        LOG_PATH: "{{ lookup('env', 'PWD') + '/log/before' }}"

    - 設定変更するタスク

    - include_role:
        name: cisco_show_run
      vars:
        LOG_PATH: "{{ lookup('env', 'PWD') + '/log/after' }}"
```

---

### 結果・その３

- 作業前のログはbeforeに、作業後はafterにファイルが作られます。

```bash
log
├── after
│   ├── r1_show_run_2019-01-11@12:10:53.txt
│   ├── r2_show_run_2019-01-11@12:10:53.txt
│   ├── r3_show_run_2019-01-11@12:10:53.txt
│   └── r4_show_run_2019-01-11@12:10:54.txt
└── before
    ├── r1_show_run_2019-01-11@12:10:37.txt
    ├── r2_show_run_2019-01-11@12:10:37.txt
    ├── r3_show_run_2019-01-11@12:10:37.txt
    └── r4_show_run_2019-01-11@12:10:37.txt
```

---

### まとめ

<br/><br/>

- ログをファイルに保存するロールを作っておくと便利です。
- 作成したロールはgitで管理すると便利です。

---

# ロール作成のコツ

---

## 実は意外にめんどくさい

<br/><br/>

- ログをファイルに残すだけでも、ゼロから作ると面倒です。
- 以下、豆知識です。

---

### カレントディレクトリの取得方法

- ロールの中ではrole_pathからの相対パスになります。
- いまいる場所にログフォルダを作りたいなら、制御ノード上でカレントディレクトリの情報を採取しないといけません。
- このロールでは環境変数PWDの情報を取り出して利用しています。

```yml
LOG_PATH: "{{ lookup('env', 'PWD') }}"
```

---

### ログを保管するディレクトリの作成

- ログを残すのは制御ノード上ですので **delegate_to** を使ってローカルホストでファイルを作成します。
- フォルダの作成は一度やれば十分なので、**run_once** をtrueにします。これも忘れがちです。

```yml
- name: create log directory if not exists
  delegate_to: localhost
  file:
    path: "{{ LOG_PATH | default('./log') }}"
    state: directory
    recurse: true
  run_once: true
```

---

### 現在時刻の取得

- このロールでは **lookup** でpipeを呼び出してdateコマンドを実行しています。

```yml
- name: get current time
  set_fact:
    date: "{{ lookup('pipe', 'date +%Y-%m-%d@%H:%M:%S') }}"
```

- この出力形式は 2018-08-28@11:17:34 という感じです。

---

### 失敗してもログを残す

- **block** と **rescue** を使って失敗時でもファイルを保存します。

```yml
-
  block:
    - name: run exec commands on remote nodes bia SSH
      ios_command:
        commands: "{{ CMD_LIST }}"
      register: result

  rescue:
    - name: log failed result
      delegate_to: localhost
      copy:
        content: "{{ result | to_nice_json(indent=2) }}"
        dest: "{{ LOG_PATH + '/' + '_FAILED_' + filename }}"
```

---

### コマンド出力を保存する・その１

- ios_commandモジュールの戻り値は **stdout配列** です。
- この中に実行したコマンドの情報は含まれていません。
- したがってstdout配列の中身だけをベタにファイルに書き込むと、なんのコマンドの出力なんだっけ？ということになってしまいます。

---

### コマンド出力を保存する・その２

- 打ち込んだコマンドの配列と、結果のstdout配列の両方をループで回してあげる必要があります。

```yml
- name: save
  delegate_to: localhost
  copy:
    content: |
      {% for item in CMD_LIST -%}
      === {{ item }} ===
      {{ stdout[loop.index0] }}
      {% endfor %}
    dest: "{{ LOG_PATH + '/' + filename }}"
```

---

### コマンド出力を保存する・その３

- 結果はこのようになります。

```bash
=== show running-config ===
Building configuration...

Current configuration : 3143 bytes
!
! Last configuration change at 13:57:03 UTC Sun Apr 14 2019 by cisco
!
```

- 区切り（=== コマンド ===）は後から加工しやすいように、もう少し工夫した方がいいかもしれません。
