<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# Ansibleもくもく会

<br>
<br>

## 現場メモ

<br>
<br>

##### @takamitsu-iida

---

資材はここ

<https://network-automation.github.io/linklight/exercises/networking_v2/>


---

## 最初のセットアップ

githubに新規でレポジトリを作成。

/home/student8/networking-workshop/以下をgithubにpush

```
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/takamitsu-iida/moku2.git
git push -u origin master
```

---

## ローカル側

pullで取ってくる。

```
git pull
```

あとはVisual Studio Codeで編集。

markdownもvs codeの中で見れるので便利。

編集したらcommit, push

---

## ex2-0-config

ルータの設定を変更するプレイブック。

設定内容はファイルから。

```yml
  tasks:

    - name: ENSURE THAT ROUTERS ARE SECURE
      ios_config:
        src: secure_router.cfg
```

---

## ex2-0-config

この設定の場合、本当は親子関係を指定した方がいいんじゃなかろうか？

```yml
line con 0
 exec-timeout 5 0
line vty 0 4
 exec-timeout 5 0
 transport input ssh
ip ssh time-out 60
ip ssh authentication-retries 5
service password-encryption
service tcp-keepalives-in
service tcp-keepalives-out
```

---

## ex2-1-backup

素直にshow running-configを採取した方がいいのではなかろうか？

```yml
  tasks:
    - name: BACKUP THE CONFIG
      ios_config:
        backup: yes
      register: config_output

    - name: RENAME BACKUP
      copy:
        src: "{{config_output.backup_path}}"
        dest: "./backup/{{inventory_hostname}}.config"

    - name: REMOVE NON CONFIG LINES
      lineinfile:
        path: "./backup/{{inventory_hostname}}.config"
        line: "Building configuration..."
        state: absent

    - name: REMOVE NON CONFIG LINES - REGEXP
      lineinfile:
        path: "./backup/{{inventory_hostname}}.config"
        regexp: 'Current configuration.*'
        state: absent
```

---

## ex2-2-backup

一つのフォルダbackupにファイルが保存される。

なんどやっても一つだけしかできない。
古いのは自動で削除してるみたい。

```bash
[student8@ansible 2-1-backup]$ tree backup
backup
├── rtr1.config
├── rtr1_config.2019-01-17@11:12:39
├── rtr2.config
├── rtr2_config.2019-01-17@11:12:39
├── rtr3.config
├── rtr3_config.2019-01-17@11:12:39
├── rtr4.config
└── rtr4_config.2019-01-17@11:12:39

0 directories, 8 files
[student8@ansible 2-1-backup]$
```

---

## ex2-2-restore

scpコマンドでルータに直接ファイルを送り込む！

ずいぶん斬新なやり方だなぁ・・・

まずex2-1で採取したバックアップフォルダをコピー。

```yml
[student8@ansible 2-2-restore]$ cp -r ../2-1-backup/backup .
```

タスクはこんな感じ。

```yml
  tasks:
    - name: COPY RUNNING CONFIG TO ROUTER
      command: scp ./backup/{{inventory_hostname}}.config {{inventory_hostname}}:/{{inventory_hostname}}.config

    - name: CONFIG REPLACE
      ios_command:
        commands:
          - config replace flash:{{inventory_hostname}}.config force
```

---

## ex2-2-restore

bootflashのファイルを確認。

```bash
[student8@ansible 2-2-restore]$ ssh rtr1

rtr1#dir flash:
Directory of bootflash:/

   11  drwx            16384  Jul 27 2018 16:29:16 +00:00  lost+found
   12  -rw-        392479704  Jul 27 2018 16:30:53 +00:00  csr1000v-mono-universalk9.16.09.01.SPA.pkg
   13  -rw-         40201438  Jul 27 2018 16:30:53 +00:00  csr1000v-rpboot.16.09.01.SPA.pkg
   14  -rw-             1941  Jul 27 2018 16:30:53 +00:00  packages.conf
292609  drwx             4096  Jan 16 2019 07:29:59 +00:00  .installer
162561  drwx             4096  Jan 16 2019 07:29:57 +00:00  core
   15  -rw-              128  Jan 16 2019 07:29:51 +00:00  iid_check.log
48769  drwx             4096  Jan 16 2019 07:29:52 +00:00  .prst_sync
276353  drwx             4096  Jan 16 2019 07:29:57 +00:00  .rollback_timer
357633  drwx             8192  Jan 17 2019 11:20:23 +00:00  tracelogs
463297  drwx             4096  Jan 16 2019 07:31:00 +00:00  .dbpersist
32513  drwx             4096  Jan 16 2019 07:30:16 +00:00  virtual-instance
   16  -rw-               30  Jan 16 2019 07:31:03 +00:00  throughput_monitor_params
   17  -rw-             5919  Jan 16 2019 07:30:59 +00:00  cvac.log
   18  -rw-                1  Jan 16 2019 07:30:57 +00:00  .cvac_version
   19  -rw-               16  Jan 16 2019 07:30:57 +00:00  ovf-env.xml.md5
   20  -rw-              209  Jan 16 2019 07:30:58 +00:00  csrlxc-cfg.log
40641  drwx             4096  Jan 16 2019 07:30:58 +00:00  onep
65025  drwx             4096  Jan 17 2019 10:38:49 +00:00  syslog
73153  drwx             4096  Jan 16 2019 07:31:04 +00:00  iox
   21  -rw-             6275  Jan 17 2019 11:20:13 +00:00  rtr1.config

7897378816 bytes total (7062708224 bytes free)
rtr1#
```

---

## ex3-0-templates

templateモジュールはtemplatesディレクトリの下を自動で探しに行くのでファイル名だけを指定すればよい。

```yml
    - name: RENDER FACTS AS A REPORT
      template:
        src: os_report.j2
        dest: reports/{{ inventory_hostname }}.md
```

---

## ex3-0-templates

出力されるレポートはこんな感じ。

```bash
[student8@ansible 3-0-templates]$ cat network_os_report.md
RTR1
---
9G55Z8J8HRC : 16.09.01
RTR2
---
9RXLXYW5JBK : 16.09.01
RTR3
---
9PM3U2U9REA : 16.09.01
RTR4
---
92PQ5DJXGKN : 16.09.01[student8@ansible 3-0-templates]$
```

---

## ex3-1-parser

ロールを取ってくる。

```bash
[student8@ansible 3-1-parser]$ ansible-galaxy install ansible-network.network-engine
- downloading role 'network-engine', owned by ansible-network
- downloading role from https://github.com/ansible-network/network-engine/archive/v2.7.2.tar.gz
- extracting ansible-network.network-engine to /home/student8/.ansible/roles/ansible-network.network-engine
- ansible-network.network-engine (v2.7.2) was installed successfully
[student8@ansible 3-1-parser]$
```

---

## ex3-1-parser

ロールをプレイブックから呼び出す。

この書き方、古いので私はやらない。

```yml
  roles:
    - ansible-network.network-engine
```

---

## ex3-1-parser

タスク全容。

```yml
  tasks:
    - name: CAPTURE SHOW INTERFACES
      ios_command:
        commands:
          - show interfaces
      register: output

    - name: PARSE THE RAW OUTPUT
      command_parser:
        file: "parsers/show_interfaces.yaml"
        content: "{{ output.stdout[0] }}"

    - name: DISPLAY THE PARSED DATA
      debug:
        var: interface_facts
```

---

## ex3-1-parser

parsers/show_interfaces.ymlがどこにあるのかわからん。

```bash
[student8@ansible 3-1-parser]$ ansible-playbook interface_report.yml

PLAY [GENERATE INTERFACE REPORT] ***************************************************************************************************************************************************************************

TASK [CAPTURE SHOW INTERFACES] *****************************************************************************************************************************************************************************
ok: [rtr4]
ok: [rtr3]
ok: [rtr1]
ok: [rtr2]

TASK [PARSE THE RAW OUTPUT] ********************************************************************************************************************************************************************************
fatal: [rtr3]: FAILED! => {"msg": "src [parsers/show_interfaces.yaml] is either missing or invalid"}
fatal: [rtr1]: FAILED! => {"msg": "src [parsers/show_interfaces.yaml] is either missing or invalid"}
fatal: [rtr2]: FAILED! => {"msg": "src [parsers/show_interfaces.yaml] is either missing or invalid"}
fatal: [rtr4]: FAILED! => {"msg": "src [parsers/show_interfaces.yaml] is either missing or invalid"}
	to retry, use: --limit @/home/student8/networking-workshop/exercises/3-1-parser/interface_report.retry

PLAY RECAP *************************************************************************************************************************************************************************************************
rtr1                       : ok=1    changed=0    unreachable=0    failed=1
rtr2                       : ok=1    changed=0    unreachable=0    failed=1
rtr3                       : ok=1    changed=0    unreachable=0    failed=1
rtr4                       : ok=1    changed=0    unreachable=0    failed=1
```

---

## ex3-1-parser

ここかな？

```bash
/home/student8/.ansible/roles/ansible-network.network-engine/tests/command_parser/command_parser/parser_templates/ios/show_interfaces.yaml
```

---

## ex3-1-parser

やっぱダメだ。

```bash
[student8@ansible 3-1-parser]$ ansible-playbook interface_report.yml

PLAY [GENERATE INTERFACE REPORT] ***************************************************************************************************************************************************************************

TASK [CAPTURE SHOW INTERFACES] *****************************************************************************************************************************************************************************
ok: [rtr2]
ok: [rtr4]
ok: [rtr1]
ok: [rtr3]

TASK [PARSE THE RAW OUTPUT] ********************************************************************************************************************************************************************************
fatal: [rtr4]: FAILED! => {"msg": "invalid value for export_as, got None"}
fatal: [rtr3]: FAILED! => {"msg": "invalid value for export_as, got None"}
fatal: [rtr1]: FAILED! => {"msg": "invalid value for export_as, got None"}
fatal: [rtr2]: FAILED! => {"msg": "invalid value for export_as, got None"}
	to retry, use: --limit @/home/student8/networking-workshop/exercises/3-1-parser/interface_report.retry

PLAY RECAP *************************************************************************************************************************************************************************************************
rtr1                       : ok=1    changed=0    unreachable=0    failed=1
rtr2                       : ok=1    changed=0    unreachable=0    failed=1
rtr3                       : ok=1    changed=0    unreachable=0    failed=1
rtr4                       : ok=1    changed=0    unreachable=0    failed=1

[student8@ansible 3-1-parser]$
```
