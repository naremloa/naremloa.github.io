---
title: 文章隨筆(200520
id: jotting-200520
date: 2020-05-20
updated: 2023-05-16
categories: HTTP
tags:
  - HTTP
  - 隨筆
---

> 原文: [（建议精读）HTTP 灵魂之问，巩固你的 HTTP 知识体系](https://juejin.im/post/5e76bd516fb9a07cce750746#heading-91)

- `HTTP`一般分為：`請求`和`響應`兩種。
- `HTTP`的報文結構，同樣也分兩種。
- `HTTP`各類狀態碼，及含義。
- `URI`結構，以及`URI`編碼。尤其是`URI`的編碼機制。
- `HTTP`的特點：
  - 靈活和高擴展性。格式上自由，傳輸方式多樣（不侷限於文本）
  - 基於`TCP`，繼承`TCP`部分特點，如可靠性等。
  - 請求-響應 特點。
  - 無狀態，每次請求都是獨立，無上下文相關。

<!-- more -->

- `HTTP`的缺點：
  - 在長連接場景下，無狀態帶來不便。
  - 明文傳輸，造成安全問題。
  - 長連接場景下的隊頭阻塞問題。
- `MIME`標準。與`HTTP`引入`MIME`標準解決的問題。涉及`Content-Type`和`accept`字短。
- `Content-Type`是對發送端而言，告訴接受端發送的數據格式。`accept`則是對接受端而言，告訴發送端自己想要的格式。
- 壓縮方式也是用同樣的方式處理。涉及字段`Content-Encoding`和`Accept-Encoding`。兩者的區別同樣也是用在發送端和接受端的區別。
- 支持語言。涉及字段`Content-Language`和`Accept-Language`。同上。
- 字符集。涉及字段`Content-Type`的`charset`屬性和`Accept-Charset`。
  ```
  // 發送端
  Content-Type: text/html; charset=utf-8
  // 接收端
  Accept-Charset: charset=utf-8
  ```
- HTTP 傳輸的數據，分為定長和不定長。
- 定長需要通過`Content-Length`指定長度，過小會截斷，過長會中斷響應。
- 不定長需要設置`Transfer-Encoding: chunked`。表示會分塊傳輸數據。這個字段會使`Content-Length`失去作用，並使請求基於長連接的方式動態推送內容。
- 對於大文件傳輸，HTTP 採取`範圍請求`方案。涉及字段`Accept-Ranges: none`。
- 對於`Range`字段，格式為: `bytes=x-y`
- 大文件請求，分為請求單段數據，和多段數據。多段數據應用在長連接場景。
- 單段數據報文：
  ```
  HTTP/1.1 206 Partial Content
  Content-Length: 10
  Accept-Ranges: bytes
  Content-Range: bytes 0-9/100
  i am xxxxx
  ```
- 多段數據報文：

  ```
  HTTP/1.1 206 Partial Content
  Content-Type: multipart/byteranges; boundary=00000010101
  Content-Length: 189
  Connection: keep-alive
  Accept-Ranges: bytes


  --00000010101
  Content-Type: text/plain
  Content-Range: bytes 0-9/96

  i am xxxxx
  --00000010101
  Content-Type: text/plain
  Content-Range: bytes 20-29/96

  eex jspy e
  --00000010101--
  ```

  注意：`Content-Type`的屬性`boundary`指定了多段響應的分隔符。

- HTTP 存在兩種表單的提交：`application/x-www-form-urlencoded`和`multipart/form-data`。一般為`POST`請求。
- `application/x-www-form-urlencoded`格式特點：
  - 數據會被編碼成以`&`分隔的鍵值對。
  - 字符以`URL`方式編碼。
- `multipart/form-data`格式特點：
  - `Content-Type`字段會包含`boundary`屬性。且`boundary`的值有瀏覽器默認指定。
  - 數據會分為多個部分。每個之間通過分隔符隔開。且每一部分會有頭部報文的一部分。
  - 數據的最後一部分，在分隔符最後會在加上`--`作結尾。
  - 由於上述特點，表單中的每一個元素都會是獨立的資源表述。
- 通常，傳送圖片等，就是用`multipart/form-data`做傳輸。
- 對於隊頭阻塞，目前有兩個方案來解決。對一個域名`增加併發連接`數量。對一個域名做`域名切片`，也就是新增其子域名，多個子域名指向同一個伺服器。
- 由於`HTTP`的無狀態特點，引入了`Cookie`來處理狀態保留問題。
- 客戶端通過請求頭中的`Cookie`向伺服器發送帶狀態的請求，伺服器也可以通過響應頭的`Set-Cookie`對客戶端進行`Cookie`寫入動作。
- `Cookie`的主要屬性：
  `生存週期`：`Expires`, `Max-Age`
  `作用域`：`Domain`, `path`
  `安全相關`：`Secure`, `HttpOnly`, `SameSite`,
- `Cookie`的缺點：
  - 容量過小。上限為`4kB`
  - 性能缺陷。沒有特定設定下，每個請求都會帶上完整的`Cookie`信息，該問題可以通過指定`Domain`和`Path`解決。
  - 安全缺陷。由於`Cookie`是通過純文本的方式傳送，容易在請求過程中被截取和竄改。
- `HTTP代理`功能：
  - 負載均衡。
  - 保證集群的穩定。
  - 緩存代理。
- `HTTP代理`相關的頭部字段：
  - `Via`：在請求中，帶上經手過的代理伺服器。
  - `X-Forwarded-For`：紀錄請求方的`IP位址`。
  - `X-Real-IP`：紀錄請求源頭的`客戶端IP`。
  - `X-Forwarded-Host`：功能同上，紀錄`客戶端域名`。
  - `X-Forwarded-Proto`：功能同上，紀錄`協議名`。
- `緩存代理`的控制分為兩部分：源伺服器的控制和客戶端的控制。
- `Cache-Control`：通用字段，伺服器和客戶端都可以通過該字段操作緩存的工作。
- `private`：響應頭。禁止代理伺服器進行緩存。
- `public`：響應頭。允許代理伺服器進行緩存。
- `proxy-revalidate`：響應頭。要求代理伺服器的緩存過期後要到源伺服器中獲取。
- `s-maxage`：響應頭。限定緩存在代理伺服器中可以存放多久。
- `max-stale`：請求頭。就是緩存過期了，還是可以繼續緩存和響應。
- `min-fresh`：請求頭。緩存過期前，如果在指定時間之前，還是正查功能響應，如果在指定時間內，雖然還在緩存期限中，但就不會正常響應。
- `only-if-cached`：請求頭。客戶端只接受代理緩存，不接受源伺服器的響應。
- 同源：`scheme`, `host`, `port`都相同。
- 非同源站點的限制：
  - 不能讀取和修改對方的`DOM`。
  - 不能讀取對方的`Cookie`, `IndexDB`, `LocalStorage`。
  - 限制`XMLHttpRequest`請求。
- `CORS`，跨域資源共享。
- [簡單請求](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS#%E7%B0%A1%E5%96%AE%E8%AB%8B%E6%B1%82)只能符合以下條件：
  - 請求方法為：`GET`, `POST`, `HEAD`。
  - 請求頭的取值範圍：`Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `Last-Event-ID`, `DPR`, `Save-Data`, `Viewport-Width`, `Width`。
  - `Content-Type`標頭值：`application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`。
  - 沒有事件監聽器被註冊到任何用來發出請求的`XMLHttpRequestUpload`物件（經由`XMLHttpRequest.upload`屬性取得）上。
  - 請求中沒有`ReadableStream`物件被用於上傳。
- 瀏覽器在發送`簡單請求`前，會自動添加一個`origin`字段，用來說明來自哪裡。伺服器在拿到請求後，在響應會對應添加一個`Access-Control-Allow-Origin`字段，如果`origin`不在這個範圍內，則瀏覽器會將響應攔截（伺服器已成功響應了）。
- `Access-Control-Allow-Credentials`：在`簡單請求`驗證成功過`origin`後，能用來標示是否允許發送`Cookie`。
- `Access-Control-Expose-Headers`：在`簡單請求`驗證成功過`origin`後，除了`簡單請求`規定下的請求頭字段可以拿到外，還能拿到這個字段聲明的響應頭字段。
- 對`非簡單請求`，會先發送`預檢請求`，之後才是`跨域請求`。
- `預檢請求`的方法是`OPTIONS`，同時會加上`Origin`源地址, `Host`目標地址, `Access-Control-Request-Method`列出跨域請求用到哪個`HTTP`方法, `Access-Control-Request-Headers`列出跨域請求將要加上什麼請求頭。
- `預檢請求`的響應格式大致如下：
  ```
  HTTP/1.1 200 OK
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET, POST, PUT
  Access-Control-Allow-Headers: X-Custom-Header
  Access-Control-Allow-Credentials: true
  Access-Control-Max-Age: 1728000
  Content-Type: text/html; charset=utf-8
  Content-Encoding: gzip
  Content-Length: 0
  ```
- `預檢請求`的響應頭中的關鍵字段：
  - `Access-Control-Allow-Origin`: 表示可以允許請求的源，可以填具體的域名，或是用`*`表示允許任意請求源。
  - `Access-Control-Allow-Methods`: 表示允許的請求方法列表。
  - `Access-Control-Allow-Credentials`: 同簡單請求的相同字段。
  - `Access-Control-Allow-Headers`: 表示允許發送的請求頭字段。
  - `Access-Control-Max-Age`: 預檢請求的有效期。在此期間，不用發出另外一條預檢請求。
- 在`預檢請求`的響應返回後，如果請求不滿足`響應頭`的條件，會觸發`XMLHttpRequest`的`onerror`方法。瀏覽器不再發送後面的的`跨域請求`。
- 到了發送`跨域請求`的階段時，跟`簡單請求`的狀況就是一樣的了。
- `JSONP`原理：`script`標籤的`src`字段不遵循同源政策，可以根據`src`上的目標地址發送`GET`請求，並拿到響應。
- `JSONP`的兼容性好，可以支援更早期的瀏覽器。但支持的請求方法過於單一。
- 通過`nginx`的反向代理來支持跨域請求。
- `HTTPS` = `HTTP` + `SSL/TLS`
- `SSL`，即安全套接層（`Secure Sockets Layer`）。處於`OSI`模型的會話層。發展到第三個大版本被標準化，成為`TLS`(傳輸層安全,`Transport Layer Security`)。主流版本為`TLS/1.2`，2018 推出了`TLS/1.3`。
- `TLS/1.2`握手過程：
  - `Client Hello`: 客戶端會先發送`client_random`, `TSL版本`, `加密套件列表`。
  - `Server Hello`: 伺服器發送`server_random`, `server_params`, `確認TLS版本、使用的加密套件列表、伺服器使用的證書`。
  - `Client驗證證書`: 客戶端驗證伺服器傳來的`證書`和`簽名`是否通過，通過則傳`client_params`給伺服器
  - `Client生成secret`: 客戶端通過`client_params`, `server_params`, `client_random`, `server_random`和`加密套件列表`生成最終的`secret`。
  - `Server生成secret`: 伺服器通過相同的方式生成最終`secret`。
  - 之後傳輸兩邊都用各自的最終`secret`作為密鑰進行加解密。
- `TLS/1.2`握手過程中需要注意的點：
  - 加密套件列表的一般格式:
    ```
    TLS_ECDHE_WITH_AES_128_GCM_SHA256
    ```
  - 加密套件列表含義：`TLS`握手過程中，使用`ECDHE`算法生成`pre_random`。採用`128`位的`AES`算法進行對稱加密，在對稱加密過程中使用主流的`GCM`分組模式。哈希摘要算法則是採用`SHA256`算法。
  - 哈希摘要算法在過程中主要是用來驗證身分用的。涉及到數字簽名的部分。
  - 以上面列出的`加密套件列表`為例，`Client`生成`secret`這個階段，最先是通過`ECDHE`算法算出`pre_random`，在這個過程中要傳入兩個參數: `server_params`和`client_params`。再用`pre_random`, `server_random`, `client_random`三者通過一個偽隨即數來計算出最終的`secret`。
  - `TLS`握手是一個`雙向認證`的過程。
  - 客戶端生成`secret`後，會給伺服器發送一個`收尾`的消息，告訴伺服器之後的都用對稱加密，對稱加密的算法就用第一次約定的。伺服器生成完`secret`也會向客戶端發送一個`收尾`消息，告訴客戶端之後直接使用對稱加密來通信。這個`收尾`的消息包括兩部分：`Changer Cipher Spec`，意味著後面都是加密傳輸。`Finished`，這個消息是對之前所有發送的數據做的`摘要`，對`摘要進行對稱加密`，讓對方驗證一下。當雙方都驗證通過之後，握手過程才算是正式結束。後面的`HTTP`就正式開始傳輸加密報文。
  - 使用`ECDHE`還有一個特點，客戶端發送完`收尾`消息後，可以提前`搶跑`，不必等`收尾`消息，直接發送加密的`HTTP`報文，節省了一個`RTT`(`round-trip time`, 客戶端到伺服器往返所花時間)。這個也叫`TLS False Start`
- 2018 年推出的`TLS/1.3`，對`TLS/1.2`做出了一系列的改進，主要分為：`強化安全`, `提高性能`。
- 在`TLS/1.3`中廢除了非常多的加密算法，最終只保留了五個加密套件：
  - TLS_AES_128_GCM_SHA256
  - TLS_AES_256_GCM_SHA384
  - TLS_CHACHA20_POLY1305_SHA256
  - TLS_AES_128_GCM_SHA256
  - TLS_AES_128_GCM_8_SHA256
- 在`TLS/1.3`中，改進了握手流程，伺服器不必等客戶端驗證證書之後才能拿到`client_params`，而是在一開始就能拿到。相比`TLS/1.2`少了一個 RTT，但相對的在第一次握手時客戶端要傳輸更多的信息。由於握手過程對客戶端來說只剩下了一去一回，這種握手方式也叫`1-RTT握手`。
- `1-RTT`的優化方式之一，會話複用。雙方連接成功後，伺服器加密會話信息，用`session ticket`消息發送給客戶端，讓客戶端存下來。在下一次重連時，直接把這個`session ticket`進行解密，驗證它有沒有過期，如果沒過期則直接恢復之前的會話狀態。這個方式在保證`1-RTT`的同時，節省了重新生成會話密鑰這些算法所消耗的時間，也是一筆可觀的性能提升。
- `1-RTT`的優化方式之二，`PSK`。在發送`session ticket`的同時帶上應用數據，不用等到伺服器確認，這種方式稱為`Pre-Shared Key`。通過這樣的方式可以使重連優化到`0-RTT`。
- 由於`HTTPS`對於安全方面的處理做的非常好，`HTTP`的改進關注點放在性能方面。對於`HTTP/2`，性能方面的提升在兩點：
  - 頭部壓縮
  - 多路復用
- `HPACK`, `HTTP/2`的頭部壓縮算法。
  在`HTTP/2`之前，`請求體`一般會有響應的壓縮編碼過程，但頭部字段本身卻沒有。當請求字段非常複雜，尤其對於`GET`請求，請求報文幾乎都是請求頭。`HTTP/2`針對頭部字段，採用了對應的壓縮算法`HPack`。
- `HPACK`的主要兩點：
  - 首先會在伺服器和客戶端之間建立哈希表，將用到的字段放在這張表中。在傳輸的時候，對於之前出現過的值，只需要把索引傳給對方即可。
  - 對整數和字串進行`哈夫曼編碼`。`哈夫曼編碼`的原理就是將所有出現過的字符建立一張索引表，對出現次數多的字符其對應的索引盡可能短，傳輸的時候也是傳輸這樣的索引序列，可以達到非常高的壓縮率。
- `二進制分幀`，`HTTP/2`的多路複用。解決隊頭阻塞問題。
  原來`Headers + Body`的報文格式被拆分成了一個個二進制的幀。用`Headers幀`存放頭部字段，`Data幀`存放請求體數據。拆分後，伺服器看到的不再是一個個完整的`HTTP`請求報文，而是一堆亂序的`二進制幀`。這些`二進制幀`不存在先後關係，因為也就沒有了`HTTP`的隊頭阻塞的問題。
  通信雙方都可以給對方發送`二進制幀`，這種`二進制幀`的`雙向傳輸的序列`，也叫做`流`。`HTTP/2`用`流`在一個`TCP`連接上，進行多個`數據幀`的通信，也就是`多路複用`的概念。
- 上面提到的`亂序`的`二進制幀`。這邊的`亂序`指的是不同`ID`的`流`是亂序的，但同一個`ID`的`流`是按順序傳輸的。
- 在`HTTP/2`中，伺服器不再是完全被動的接收請求，響應請求。它也能新建`流`來給客戶端發送消息，也就是`伺服器推送`(`server push`)。當一個`TCP`連接建立後，瀏覽器請求一個`HTML`文件，伺服器可以在返回`HTML`文件的基礎上，同時將`HTML`中引用到的其他資源文件一個返回給瀏覽器，減少瀏覽器的等待。
- `HTTP`結構參考：
  ![](https://i.imgur.com/PTFOAin.png)
- 二進制幀的幀結果大致如下：
  ![](https://i.imgur.com/vUsIxDN.png)
- 每個幀分為`幀頭`(`Frame Header`)和`幀體`(`Frame Payload`)。
- `幀長度`，表示`幀體`的長度。3 個字節。
- `幀類型`，分為`數據幀`和`控制幀`兩種。`數據幀`用來存放`HTTP`報文，`控制幀`用來管理`流`的傳輸。1 個字節。
- `幀標誌`，一共有 8 個標誌位，常用的有`END_HEADERS`表示頭數據結束，`END_STREAM`表示單方向數據發送結束。1 個字節。
- `流標示符`，也就是`Stream ID`。接收方在亂序的`二進制幀`中選出`ID`相同的幀，按順序組裝成請求/響應報文。4 個字節。
- `流`通過標誌位的改變來實現具體的狀態改變。通常的過程是
  - 剛開始兩者都是空閒。
  - 客戶端發送`HEADERS幀`，開始分配`Stream ID`，客戶端的`流`打開。
  - 伺服器接收之後，伺服器的`流`也打開。
  - 兩端的`流`打開後，可以互相傳遞`數據幀`和`控制幀`。
  - 當客戶端要關閉時，向伺服器發送`END_STREAM`幀，進入`半關閉狀態`。這時候客戶端只能接收數據，不能發送。
  - 伺服器接收到這個`END_STREAM`，也進入`半關閉狀態`。此時，伺服器只能發送數據，不能接收數據。
  - 伺服器向客戶端發送`END_STREAM`幀，表示數據發送完畢，雙方進入`關閉狀態`。
  - 如果下次要開啟新的`流`，`Stream ID`需要自增，直到上限為止。到達上限後，開一個新的`TCP`連接重頭開始計數。`流標示符`長度為 4 個字節，最高位保留，範圍是 0~2 的 31 次方，大約 21 億個。
- `流`的特性：
  - 並發性。一個`HTTP/2`連接上可以同時發送多個幀。
  - 自增性。`Stream ID`不可重用，會按順序向上自增。
  - 雙向性。客戶端和伺服器端都可以創建流，互不干擾。
  - 可設置優先級。可以設置數據幀的優先級，讓伺服器先處理重要資源，優化用戶體驗。
