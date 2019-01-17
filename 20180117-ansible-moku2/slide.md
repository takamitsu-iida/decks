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

## ローカルのPCと同期

```
git pull
```

あとはVisual Studio Codeで編集。

## ex2-0

設定を変更

```yml
  tasks:

    - name: ENSURE THAT ROUTERS ARE SECURE
      ios_config:
        src: secure_router.cfg
```

本当は親子関係を指定した方がいいんじゃなかろうか？

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
