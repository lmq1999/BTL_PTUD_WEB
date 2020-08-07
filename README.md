# Jitsi

## Giới thiệu
Jitsi là giải pháp hội nghị truyền hình được xây dựng dựa trên một loạt các open-source project cho phép bạn tự triển khai một cách dễ dàng để nâng cao tính bảo mật.

### Tính năng 
Jitsi bao gồm những tính năng chính sau đây :

* Jitsi sẽ truyền trực tiếp toàn bộ video và âm thanh của mọi người thay vì xử lý chúng trước như một số công cụ khác. Kết quả là bạn sẽ có độ trễ thấp hơn, chất lượng được nâng cao hơn và nếu bạn tự chạy Jitsi thì đây sẽ là nền tảng mang lại khả năng mở rộng lớn với chi phí thấp.
* Jitsi tương thích với WebRTC, một tiêu chuẩn mở cho giao tiếp trên nền tảng web.
* Jitsi hỗ trợ trên cả nền tảng điện thoại lẫn PC.
* Hỗ trợ đặt mật khẩu cho từng room.

## Cài đặt Jitsi

### Cầu hình server: 
Chúng ta sử dụng 1 server **Ubuntu 18.04 LTS** với 01 **IP public**
2 core 4GB RAM 40GB HDD
*Sử dụng server được cấp ở: https://bizflycloud.vn/*

### Chuẩn bị


Các thành phần:

```sh
                   +                           +
                   |                           |
                   |                           |
                   v                           |
                  443                          |
               +-------+                       |
               |       |                       |
               | Nginx |                       |
               |       |                       |
               +--+-+--+                       |
                  | |                          |
+------------+    | |    +--------------+      |
|            |    | |    |              |      |
| jitsi-meet +<---+ +--->+ prosody/xmpp |      |
|            |files 5280 |              |      |
+------------+           +--------------+      v
                     5222,5347^    ^5347   4443,10000
                +--------+    |    |    +-------------+
                |        |    |    |    |             |
                | jicofo +----^    ^----+ videobridge |
                |        |              |             |
                +--------+              +-------------+
```

#### Cập nhật repository

Sau khi đăng nhập vào server

Cập nhật server sử dụng lệnh sau: `root@quanlm-jitsi-lab:~# apt update && apt upgrade -y`

#### Đặt tên miền đầy đủ

Đặt tên: `root@quanlm-jitsi-lab:~# hostnamectl set-hostname quanlm-jitsimeet`
Cấu hình file host: `/etc/hosts`

```sh
127.0.0.1 localhost
103.56.156.137     quanlm-jitsimeet
```

Kiểm tra kết nối: 

```sh
root@quanlm-jitsi-lab:~# ping -c 4 "$(hostname)"
PING quanlm-jitsimeet (10.6.169.214) 56(84) bytes of data.
64 bytes from quanlm-jitsimeet (10.6.169.214): icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from quanlm-jitsimeet (10.6.169.214): icmp_seq=2 ttl=64 time=0.042 ms
64 bytes from quanlm-jitsimeet (10.6.169.214): icmp_seq=3 ttl=64 time=0.041 ms
64 bytes from quanlm-jitsimeet (10.6.169.214): icmp_seq=4 ttl=64 time=0.036 ms

--- quanlm-jitsimeet ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3045ms
rtt min/avg/max/mdev = 0.020/0.034/0.042/0.008 ms
```

Mở tường lửa cho những port sau:
* 80 TCP
* 443 TCP 
* 4443 TCP
* 10000 UDP
* 22 TCP

Ở đây chúng ta sử dụng tường lửa ufw:
```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 4443/tcp
sudo ufw allow 10000/udp
sudo ufw allow 22/tcp
sudo ufw enable
```

#### Cài đặt OpenJDK Java Runtime Environment (JRE) 8

Cài đặt JRE 8 sử dụng lệnh: `sudo apt install -y openjdk-8-jre-headless`

Kiểm tra phiên bản:

```sh
root@quanlm-jitsi-lab:~# java -version
openjdk version "1.8.0_265"
OpenJDK Runtime Environment (build 1.8.0_265-8u265-b01-0ubuntu2~20.04-b01)
OpenJDK 64-Bit Server VM (build 25.265-b01, mixed mode)
```

Cấu hình đường dẫn JAVA_HOME:

```sh
echo "JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")" | sudo tee -a /etc/profile
source /etc/profile
```

#### Cài đặt Nginx

Cài đặt nginx sử dụng lệnh sau:
```
sudo apt install -y nginx
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```

Kiểm tra nginx: 

Sử  dụng lệnh: `sudo service nginx status`

```sh
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2020-08-06 15:59:57 UTC; 31s ago
       Docs: man:nginx(8)
   Main PID: 10620 (nginx)
      Tasks: 3 (limit: 4683)
     Memory: 6.1M
     CGroup: /system.slice/nginx.service
             ├─10620 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─10621 nginx: worker process
             └─10622 nginx: worker process

Aug 06 15:59:57 quanlm-jitsimeet systemd[1]: Starting A high performance web server and a reverse proxy server...
Aug 06 15:59:57 quanlm-jitsimeet systemd[1]: Started A high performance web server and a reverse proxy server.
```

Gọi đến địa chỉ IP điểm tra

```sh
root@quanlm-jitsi-lab:~# curl  103.56.156.137
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Cài đặt jitsi-meet sử dụng package


#### Cài đặt repo jitsi-meet

Cài đặt jitsi-repo sử dụng lệnh sau:

```sh
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
sudo sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi-stable.list"
sudo apt update -y
```

Cài đặt gói jitsi-meet:

```sh
sudo apt install -y jitsi-meet
```

Lưu ý : Trong quá trình cài, bạn sẽ được yêu cầu cung cấp hostname. Tại đây, bạn cần điền domain đã được trỏ về IP máy chủ cài đặt.

* Trường hợp bạn không có domain, bạn cũng có thể nhập IP của chính máy chủ đó.

Lựa chọn **Generate a new self-signed certificate (You will later get a chance to obtain a Let’s Encrypt certificate).**
Sau khi quá trình cài đặt hoàn tất, bạn có thể chạy script để cài đặt SS

LTrong quá trình cài đặt, bạn sẽ được yêu cầu nhập email. Phần còn lại script sẽ tự động cài cho bạn.
Lưu ý : Bạn chỉ có thể sử dụng script trên để cài đặt SSL nếu bạn sử dụng domain ở bước trước đó.
Cuối cùng, bạn hãy truy cập vào domain để kiểm tra kết quả.

**Domain**: https://ba67042-jitsi.prebuilt-app.bizflycloud.vn/

### Cài đặt jitsi-meet từ source

#### Cài đặt prosody

Sử dụng lệnh: `apt-get install prosody` để cài dặt prosody

Tạo file cấu hình: `/etc/prosody/conf.avail/jitsi.example.com.cfg.lua`

Nội dung:
    ```lua
    VirtualHost "jitsi.example.com"
    authentication = "anonymous"
    ssl = {
        key = "/var/lib/prosody/jitsi.example.com.key";
        certificate = "/var/lib/prosody/jitsi.example.com.crt";
    }
    modules_enabled = {
        "bosh";
        "pubsub";
    }
    c2s_require_encryption = false
    VirtualHost "auth.jitsi.example.com"
    ssl = {
        key = "/var/lib/prosody/auth.jitsi.example.com.key";
        certificate = "/var/lib/prosody/auth.jitsi.example.com.crt";
    }
    authentication = "internal_hashed"
    admins = { "focus@auth.jitsi.example.com" }
    Component "conference.jitsi.example.com" "muc"
Component "jitsi-videobridge.jitsi.example.com"
    component_secret = "YOURSECRET1"
Component "focus.jitsi.example.com"
    component_secret = "YOURSECRET2"
    ```

Trong đó YOURSECRET1, YOURSECRET2 là secret mình chọn
Virtual Host để là domain mình đặt

Tạo solf-link đến thư mục cấu hình:` ln -s /etc/prosody/conf.avail/jitsi.example.com.cfg.lua /etc/prosody/conf.d/jitsi.example.com.cfg.lua`

Tạo cert cho tên miền: 

```sh
prosodyctl cert generate jitsi.example.com
prosodyctl cert generate auth.jitsi.example.com

ln -sf /var/lib/prosody/auth.jitsi.example.com.crt /usr/local/share/ca-certificates/auth.jitsi.example.com.crt
update-ca-certificates -f
```

Tạo user cho jicofo: `prosodyctl register focus auth.jitsi.example.com YOURSECRET3`

Khởi động lại prosody: `prosodyctl restart`

#### Cài đặt nginx

Cài đặt nginx :`apt-get install nginx`

Cấu hình: `/etc/nginx/sites-available/jitsi.example.com`

```sh
server_names_hash_bucket_size 64;

server {
    listen 0.0.0.0:443 ssl http2;
    listen [::]:443 ssl http2;
    # tls configuration that is not covered in this guide
    # we recommend the use of https://certbot.eff.org/
    server_name jitsi.example.com;
    # set the root
    root /srv/jitsi-meet;
    index index.html;
    location ~ ^/([a-zA-Z0-9=\?]+)$ {
        rewrite ^/(.*)$ / break;
    }
    location / {
        ssi on;
    }
    # BOSH, Bidirectional-streams Over Synchronous HTTP
    # https://en.wikipedia.org/wiki/BOSH_(protocol)
    location /http-bind {
        proxy_pass      http://localhost:5280/http-bind;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $http_host;
    }
    # external_api.js must be accessible from the root of the
    # installation for the electron version of Jitsi Meet to work
    # https://github.com/jitsi/jitsi-meet-electron
    location /external_api.js {
        alias /srv/jitsi-meet/libs/external_api.min.js;
    }
}
```

Cài đặt đường dẫn cho cấu hình:

```sh
cd /etc/nginx/sites-enabled
ln -s ../sites-available/jitsi.example.com jitsi.example.com
```

#### Cài đặt Jitsi Videobridge

Cài đặt JRE: `apt-get install openjdk-8-jre`

Cài đặt và giải nén jitsi-videobridge: 
```
wget https://download.jitsi.org/jitsi-videobridge/linux/jitsi-videobridge-linux-{arch-buildnum}.zip
unzip jitsi-videobridge-linux-{arch-buildnum}.zip
```

Tạo thư mục khởi chạy jitsi-videobridge: `~/.sip-communicator/sip-communicator.properties`

```
mkdir -p ~/.sip-communicator
cat > ~/.sip-communicator/sip-communicator.properties << EOF
org.jitsi.impl.neomedia.transform.srtp.SRTPCryptoContext.checkReplay=false
# The videobridge uses 443 by default with 4443 as a fallback, but since we're already
# running nginx on 443 in this example doc, we specify 4443 manually to avoid a race condition
org.jitsi.videobridge.TCP_HARVESTER_PORT=4443
EOF
```

Khởi chạy với lệnh: `./jvb.sh --host=localhost --domain=jitsi.example.com --port=5347 --secret=YOURSECRET1 &`

#### Cài đựat jicofo

Cài đặt JRE và maven: `apt-get install openjdk-8-jdk maven`

Tải source code, buid và khởi chạy:

```
git clone https://github.com/jitsi/jicofo.git

cd jicofo
mvn package -DskipTests -Dassembly.skipAssembly=false

=======
unzip target/jicofo-1.1-SNAPSHOT-archive.zip
cd jicofo-1.1-SNAPSHOT-archive'
./jicofo.sh --host=localhost --domain=jitsi.example.com --secret=YOURSECRET2 --user_domain=auth.jitsi.example.com --user_name=focus --user_password=YOURSECRET3
```

#### Cài đặt jitsi-meet

```
cd /srv
git clone https://github.com/jitsi/jitsi-meet.git
cd jitsi-meet
npm install
make
```

Cầu hình:

```
var config = {
    hosts: {
        domain: 'jitsi.example.com',
        muc: 'conference.jitsi.example.com',
        bridge: 'jitsi-videobridge.jitsi.example.com',
        focus: 'focus.jitsi.example.com'
    },
    useNicks: false,
    bosh: '//jitsi.example.com/http-bind', // FIXME: use xep-0156 for that
    //chromeExtensionId: 'diibjkoicjeejcmhdnailmkgecihlobk', // Id of desktop streamer Chrome extension
    //minChromeExtVersion: '0.1' // Required version of Chrome extension
};
```

Khởi động lại nginx `nginx -t && nginx -s reload`

## Tính năng

Những tính năng cơ bản của jitsi được viết tại đây: https://docs.google.com/document/d/118cT0L0-TaQQE9Ucf_ouamKKXGaTl94z8XY90f8Lh3Y/edit

## Phân tích code Jitsi

### Phương pháp phân tích:


Chúng ta sẽ sử dụng trình duyệt firefox, truy cập vào trang và bấm f12, khi cần thông tin về code ở mục vào thì ta bấm chuột phải vào chọn inspect element, ta sẽ xem có event nào được thực hiện không, nếu có thì nhảy đến đường dẫn của event, xem hàm, các biến được thực hiện, đồng thời qua repository của jiti ở github tra hàm, biến đó còn có ở đâu nữa, từ đó sẽ tìm được chức năng mà chúng thực hiện.

Source code jitsi tại: https://github.com/jitsi/jitsi-meet

Ta có thể thấy jitsi được viết hầu hết là sử dụng react JS.

Các chức năng tập trung tại https://github.com/jitsi/jitsi-meet/tree/master/react/features

Ví dụ:

Chức năng: Tạo phòng chat khi bấm GO:
Khi bấm GO, hàm này sẽ được gọi khi click (vị trí:https://github.com/jitsi/jitsi-meet/blob/6c4901a8260ed23f64c6a72881126b117f0bf90b/react/features/welcome/components/AbstractWelcomePage.js#L179)

```
    _onJoin() {
        const room = this.state.room || this.state.generatedRoomname;

        sendAnalytics(
            createWelcomePageEvent('clicked', 'joinButton', {
                isGenerated: !this.state.room,
                room
            }));

        if (room) {
            this.setState({ joining: true });

            // By the time the Promise of appNavigate settles, this component
            // may have already been unmounted.
            const onAppNavigateSettled
                = () => this._mounted && this.setState({ joining: false });

            this.props.dispatch(appNavigate(encodeURI(room)))
                .then(onAppNavigateSettled, onAppNavigateSettled);
```

Tên của room được random tại:

https://github.com/jitsi/js-utils/blob/master/random/index.js

Trong đó random tên gồm 4 thành phần, PLURALNOUN, VERB, ADVERB, ADJECTIVE và được hàm random tại

https://github.com/jitsi/js-utils/blob/master/random/randomUtil.js

Ngôn ngữ được dịch bởi i8next

https://github.com/jitsi/jitsi-meet/blob/342a00a6af323d98eef33fc60f4e2a515c8e254d/modules/translation/translation.js

Những tính năng của i8next được cài đặt tại:

https://github.com/jitsi/jitsi-meet/blob/0598e7369b2dfac9c11637c3c356cf634466430f/react/features/base/i18n/i18next.js

Và danh sách ngôn ngữ, bản dịch ở dạng json tại: https://github.com/jitsi/jitsi-meet/blob/0598e7369b/lang/languages-af.json

Trong phòng chat, code review audio tại: 
https://github.com/jitsi/jitsi-meet/blob/0598e7369b2dfac9c11637c3c356cf634466430f/react/features/settings/components/web/audio/AudioSettingsPopup.js
