<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# Ansibleもくもく会

<br>
<br>

## メモ

---

資材はここ

<https://network-automation.github.io/linklight/exercises/networking_v2/>


---

## 最初のセットアップ

githubに新規でレポジトリを作成。

```
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/takamitsu-iida/moku2.git
git push -u origin master
```

---

## ローカルのPCと同期

```
git pull
```

あとはVisual Studio Codeで編集。

---

## ex2-0-config

設定を変更

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

