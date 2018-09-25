# Gửi cảnh báo qua Telegram

# 1. Yêu cầu.

- Đã cài đặt check_mk 1.4

- Python

- Đã tạo tài khoản telegram.

# 2. Tạo bot và tìm chat ID.

Đầu tiên chat với `@BotFather` để lấy token :

```sh
/start
/newbot
duc             (tên bot)
ducnm37bot      (user name cho bot)
```

Để Bot có thể gửi cho client thì Clien cần cho phép Bot bằng cách. Đứng phía Client gửi tin nhắn bất kỳ cho Bot :

<img src="https://i.imgur.com/FLJXkMW.png">

Truy cập 
`https://api.telegram.org/bot{your_token}/getUpdates                (thay token của bạn vào)`
ta sẽ thấy được các tin nhắn mà Bot nhận được và ID của Client. Ta sẽ lấy ID của client này để truyền vào `User` :

![lay-id](/images/lay-id.png)

# 3. Cấu hình check_mk .

Tạo file `telegram.py` :

```sh
vi /omd/sites/monitoring/share/check_mk/notifications/telegram.py
```

Nội dung file `telegram.py` như sau :

```sh
#!/usr/bin/env python
# Gui Canh Bao Telegram
import json
import requests
import os

TOKEN = "your_token"
URL = "https://api.telegram.org/bot{}/".format(TOKEN)


def get_url(url):
    response = requests.get(url)
    content = response.content.decode("utf8")
    return content


def get_json_from_url(url):
    content = get_url(url)
    js = json.loads(content)
    return js


def get_updates():
    url = URL + "getUpdates"
    js = get_json_from_url(url)
    return js


def get_last_chat_id_and_text(updates):
    num_updates = len(updates["result"])
    last_update = num_updates - 1
    text = updates["result"][last_update]["message"]["text"]
    chat_id = updates["result"][last_update]["message"]["chat"]["id"]
    return (text, chat_id)


def send_message(text, chat_id):
    url = URL + "sendMessage?text={}&chat_id={}".format(text, chat_id)
    get_url(url)


#text, chat = get_last_chat_id_and_text(get_updates())
#send_message(text, chat)
mess = os.environ['NOTIFY_LASTSERVICESTATE']+ '->' + os.environ['NOTIFY_SERVICESTATE'] + ' Host:' + os.environ['NOTIFY_HOSTNAME'] + ' IP:' + os.environ['NOTIFY_HOSTADDRESS'] + ' Service:' + os.environ['NOTIFY_SERVICEDESC'] + ' Time:' + os.environ['NOTIFY_SHORTDATETIME']
send_message(mess, os.environ['NOTIFY_CONTACT_TELEGRAM_CHAT_ID'])
```

- Thay `TOKEN` bằng TOKEN chúng ta lấy được qua chat box `@BotFather`

Phân quyền cho file `telegram.py` :

```sh
chmod +x /omd/sites/monitoring/share/check_mk/notifications/telegram.py
```

Restart lại omd server :

```sh
omd restart
```

# 4. Cấu hình trên WATO.

Tạo một `Attributes User`.

- Tại `WATO` chọn `User` => `Custom User Attributes`:

![attb](/images/attb.png)

- Tại `WATO` chọn `Users` => `New User` :

![tao-user](/images/tao-user.png)

- Tại `WATO` chọn `Notifications` => `New Rule` :

![rule-notify](/images/rule-notify.png)

# 5. Kiểm tra.

- Gửi cảnh báo kiểm tra.

![gui-canh-bao](/images/gui-canh-bao.png)

- Kết quả :

![kq-telegram](/images/kq-telegram.png)
