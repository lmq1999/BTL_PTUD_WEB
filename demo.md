Phương pháp phân tích:

Chúng ta sẽ sử dụng trình duyệt firefox, truy cập vào trang và bấm f12, khi cần thông tin về code ở mục vào thì ta bấm chuột phải vào chọn inspect element, ta sẽ xem có event nào được thực hiện không, nếu có thì nhảy đến đường dẫn của event, xem hàm, các biến được thực hiện, đồng thời qua repository của jiti ở github tra hàm, biến đó còn có ở đâu nữa, từ đó sẽ tìm được chức năng mà chúng thực hiện.



Tên của room được random tại:

`https://github.com/jitsi/js-utils/blob/master/random/index.js`

Trong đó random tên gồm 4 thành phần, PLURALNOUN, VERB, ADVERB, ADJECTIVE và được random tại 

`https://github.com/jitsi/js-utils/blob/master/random/randomUtil.js`


Ngôn ngữ được dịch bởi i8next

https://github.com/jitsi/jitsi-meet/blob/342a00a6af323d98eef33fc60f4e2a515c8e254d/modules/translation/translation.js

Những tính năng của i8next được cài đặt tại: 

https://github.com/jitsi/jitsi-meet/blob/0598e7369b2dfac9c11637c3c356cf634466430f/react/features/base/i18n/i18next.js

Và danh sách ngôn ngữ, bản dịch ở dạng json tại: https://github.com/jitsi/jitsi-meet/blob/0598e7369b/lang/languages-af.json
