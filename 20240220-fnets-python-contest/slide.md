<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD012 -->
<!-- markdownlint-disable MD036 -->

# FNETSのPythonコンテストに参加してみた

<br>

たいへんおもしろかったです。

企画、運営にご尽力頂いた皆様、ありがとうございました。

<br><br>

<div class="textright">
Takamitsu IIDA (@takamitsu-iida)
</div>

---

### どんなスクリプトにしたか

UDPやTCPを使ってデータを送受信するなら、import socket するのが基本ですが、より高レベルなAPIがあります。

```python
import asyncio
```

asyncioは非同期でI/O処理を実現するための標準ライブラリです。

今回はこれを使いました。

---

## main()はこんな感じ

connect() と send_once() は別に作った関数です。

```python
if __name__ == '__main__':

    HOST = '10.77.165.166'
    PORT = 5002
    MESSAGE = 'iida@fujitsu.com'

    async def main():

        while True:
            try:
                connect_task = asyncio.create_task(connect(host=HOST, port=PORT))
                await connect_task
                reader, writer = connect_task.result()

                send_task = asyncio.create_task(send_once(reader, writer, MESSAGE))
                await send_task

                await asyncio.sleep(5)  # 5秒待つ
            except KeyboardInterrupt:
                logger.info('KeyboardInterrupt')
                break
            except Exception as e:
                logger.error(f'Unexpected exception - {e}')
                break

    # 実行
    asyncio.run(main())
```

---

## connect()はこんな感じ

asyncio.open_connection()で読み込み用、書き込み用のストリームが返ってきます。

```python
async def connect(host: str, port: int):
    try:
        reader, writer = await asyncio.open_connection(host=host, port=port)
        return reader, writer
    except ConnectionRefusedError:
        logger.error(f'ConnectionRefusedError - {host}:{port}')
    except Exception as e:
        logger.error(f'Unexpected exception - {e}')
```

---

## send_once()はこんな感じ

```python
async def send_once(reader: asyncio.StreamReader, writer: asyncio.StreamWriter, input_message: str):
    # サーバにメッセージを送信
    writer.write(input_message.encode('utf-8'))
    await writer.drain()

    # サーバから結果を受信
    recv_data = await reader.read(4096)
    if recv_data == b'':
        logger.info(f'Connection closed by peer - {writer.get_extra_info("peername")}')

    # 受信したメッセージを表示
    print(recv_data.decode('utf-8'))

    # 接続を閉じる
    if not writer.is_closing():
        writer.close()
        await writer.wait_closed()
```

---

## asyncioの何がいいの？

処理対象が複数の場合に便利です。

今回は登るタワーが４個ありましたので、４個のスクリプトを書いて別々に実行してもよいのですが、
asyncioを使って一つのスクリプトで同時に４個のタワーを攻略することもできます。

ネットワークの検証作業において、TCPで疎通できるか、をいろんな宛先に調べるような場合も、順番に処理するよりも同時に実行しちゃった方が速いですし、スクリプトもシンプルです。

---

## copilotにも手伝ってもらいました

answer.txtを取得するにはrequestsモジュールを使うのが手っ取り早いので、copilotに例を出してもらいました。

プロンプト

```
pythonのrequestsモジュールを使ってテキストファイルを取得する例を提示してください
```

---

## copilotの回答

Pythonのrequestsモジュールを使用してテキストファイルを取得する基本的なコードは以下の通りです

```
import requests

# ファイルのURL
url = "http://example.com/path/to/file.txt"

# GETリクエストを送信
response = requests.get(url)

# レスポンスのテキストを取得
text = response.text

# テキストを表示
print(text)
```

コメントを付けてくれるので便利ですね～

---

## 次回に向けて

他の人がどんなコードを書いているか、リアルタイムに分かるようにGithubを使うといいかもしれませんね。

Pythonの実行部分はGithub Actionsで対処するようにすると、実行環境が標準化されてよいと思います。
