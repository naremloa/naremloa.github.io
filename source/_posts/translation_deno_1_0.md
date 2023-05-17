---
title: Deno 1.0
id: translation-deno-1-0
date: 2020-05-18
updated: 2023-05-16
categories: Deno
tags:
  - Deno
  - JavaScript
---

> 原文: [Deno 1.0](https://deno.land/v1) @written by Ryan Dahl, Bert Belder, and Bartek Iwańczuk
> 譯者: naremloa

動態語言是一種很有用的工具。腳本能使用戶迅速且簡潔的將複雜的系統黏合在一起，並且能在不需要考慮包括像記憶體管理、系統建置等技術細節的環境下充分的展現創意。近幾年，像 Rust、Go 等的程式語言讓用戶能更容易的生產出複雜的原生機器語言，這些專案也在計算機基礎設施(`computer infrastructure`)的發展上有著很重大的作用。然而我們認為，能有一個可以解決多種範圍的問題領域(`problem domains`)的強大的腳本環境，還是很重要的。

Dynamic languages are useful tools. Scripting allows users to rapidly and succinctly tie together complex systems and express ideas without worrying about details like memory management or build systems. In recent years programming languages like Rust and Go have made it much easier to produce sophisticated native machine code; these projects are incredibly important developments in computer infrastructure. However, we claim it is still important to have a powerful scripting environment that can address a wide range of problem domains.

<!-- more -->

JavaScript 是一種最為廣泛使用的動態語言，僅需要一個瀏覽器就能運行在所有設備上。極大數量擅長 JavaScript 的工程師，花了大部分的功夫在優化其執行上。通過像 ECMA 這樣的標準組織，該語言正在小心的且持續的發展中。我們相信，不管是在瀏覽器環境下還是獨立的進程中，JavaScript 都會是動態語言工具的首選。

JavaScript is the most widely used dynamic language, operating on every device with a web browser. Vast numbers of programmers are fluent in JavaScript and much effort has been put into optimizing its execution. Through standards organizations like ECMA International, the language has been carefully and continuously improved. We believe JavaScript is the natural choice for dynamic language tooling; whether in a browser environment or as standalone processes.

我們原本在這塊的作品，Node.js，已是一個非常成功的軟體平台(`software platform`)。大家已經認識到不管是在建置 web 開發工具、建置獨立的 web 伺服器還是其他無數的案例上，它非常的有用。node 設計的時間是在 2009 年，而當時的 JavaScript 相比現在已不可同日而語。出於需要，當時的 node 必須先實現之後被標準組織添加到語言中的部分概念。有關於這方面的細節，在[Design Mistakes in Node](https://www.youtube.com/watch?v=M3BM9TB-8yA)中有所討論。鑑於 node 擁有數量龐大的使用者，想要對這個系統做改變是非常困難且緩慢的。

Our original undertaking in this area, Node.js, proved to be a very successful software platform. People have found it useful for building web development tooling, building standalone web servers, and for a myriad of other use-cases. Node, however, was designed in 2009 when JavaScript was a much different language. Out of necessity, Node had to invent concepts which were later taken up by the standards organizations and added to the language differently. In the presentation [Design Mistakes in Node](https://www.youtube.com/watch?v=M3BM9TB-8yA), this is discussed in more detail. Due to the large number of users that Node has, it is difficult and slow to evolve the system.

隨著 JavaScript 的發展，和新的東西如 TypeScript 的加入，建置 Node 專案變得越發艱鉅，其中就包括管理建置系統和其他繁重的工具。這些都降低了動態腳本語言的趣味。此外，通過 NPM 倉庫來鏈接外部庫這種中心化的機制，並不符合 web 的理念。

With the changing JavaScript language, and new additions like TypeScript, building Node projects can become an arduous endeavor, involving managing build systems and other heavy handed tooling that takes away from the fun of dynamic language scripting. Furthermore the mechanism for linking to external libraries is fundamentally centralized through the NPM repository, which is not inline with the ideals of the web.

我們認為現階段的 JavaScript 和其周圍的軟體基礎設施的改變已經足以開始做簡化了。我們想要找一種有趣且有效的腳本環境，用來應對多種範圍的任務。

We feel that the landscape of JavaScript and the surrounding software infrastructure has changed enough that it was worthwhile to simplify. We seek a fun and productive scripting environment that can be used for a wide range of tasks.

## A Web Browser for Command-Line Scripts

Deno 是一個新的在瀏覽器之外執行 JavaScript 和 TypeScript 的執行環境。

Deno is a new runtime for executing JavaScript and TypeScript outside of the web browser.

Deno 嘗試提供一個標準化工具來迅速的編寫複雜的功能。Deno 是(以後也會是)一個單一的可執行檔案。就像瀏覽器一樣，它知道如何取得外部程式。在 Deno 中，一個單一的檔案可以在不使用其他工具下編寫任意複雜的行為。

Deno attempts to provide a standalone tool for quickly scripting complex functionality. Deno is (and always will be) a single executable file. Like a web browser, it knows how to fetch external code. In Deno, a single file can define arbitrarily complex behavior without any other tooling.

```javascript=
import { serve } from "https://deno.land/std@0.50.0/http/server.ts";
for await (const req of serve({ port: 8000 })) {
  req.respond({ body: "Hello World\n" });
}
```

上述是一個完整的 HTTP 伺服器模組，它在單獨的一行裡新增了一個依賴。你不需要其他的設定檔案或預先安裝東西，只需要執行`deno run example.js`

Here a complete HTTP server module is added as a dependency in a single line. There are no additional configuration files, there is no install to do beforehand, just `deno run example.js`.

就像在瀏覽器那樣，程式預設是執行在一個安全沙箱中。腳本不能讀取硬碟、打開網路連接，或是執行其他在未經授權下可能有的惡意行為。瀏覽器提供 APIs 來連接攝像頭和麥克風，但這一定會先需要取得用戶的授權。Deno 在終端提供類似的行為：上述的這些例子如果沒有在命令行提供`--allow-net`這個參數都會失敗。

Also like browsers, code is executed in a secure sandbox by default. Scripts cannot access the hard drive, open network connections, or make any other potentially malicious actions without permission. The browser provides APIs for accessing cameras and microphones, but users must first give permission. Deno provides analogous behaviour in the terminal. The above example will fail unless the `--allow-net` command-line flag is provided.

Deno 很小心的不偏離標準化瀏覽器的 JavaScript APIs。當然，不是所有的瀏覽器 API 都跟 Deno 相關，但只要有相關的部分，Deno 都不會偏離標準。

Deno is careful to not deviate from standardized browser JavaScript APIs. Of course, not every browser API is relevant for Deno, but where they are, Deno does not deviate from the standard.

## First Class TypeScript Support

我們想要 Deno 能使用在多種範圍的問題領域上：從小小的一行腳本，到複雜的伺服器端業務邏輯(`business logic`)。隨著程式變得越來越複雜，有這某種形式的型別檢查變的日益重要。TypeScript 是一種 JavaScript 的擴展，它允許使用者可選擇的提供型別資訊。

We want Deno to be applicable to a wide range of problem domains: from small one-line scripts, to complex server-side business logic. As programs become more complex, having some form of type checking becomes increasingly important. TypeScript is an extension of the JavaScript language that allows users to optionally provide type information.

Deno 原生支持 TypeScript。`deno types`指令可以查看所有 Deno 提供的型別宣告。Deno 的標準模組都是用 TypeScript 編寫的。

Deno supports TypeScript without additional tooling. The runtime is designed with TypeScript in mind. The `deno types` command provides type declarations for everything provided by Deno. Deno's standard modules are all written in TypeScript.

## Promises All The Way Down

在 Node 設計之初，JavaScript 還未有 Promises 和 async/await 的概念。Node 中對應 promises 的是 EventEmitter。一些重要的 APIs 像 sockets 和 HTTP 則是圍繞在一旁。先不談 async/await 所帶來的好處，EventEmitter 模式有著背壓(`back-pressure`)問題。以 TCP 套接字來舉例。套接字會在收到 incoming packets 時發出`資料`。這些`資料`的回調會以一種不受控制的方式發出，讓事件(`events`)充滿氾濫整個進程。因為 Node 持續不斷的接收新的資料事件，底層的 TCP 套接字沒有相對應的背壓，而遠程的發送方不知道伺服器已過載，仍在持續的發送資料。為了緩解這個問題，新增了`pause()`這個方法。它可以解決這個問題，但需要額外的程式，並且因為 flooding 問題只有出現在進程非常繁忙的時候，許多的 Node 程式都會有資料氾濫的問題。最終導致整個系統有著非常不好的尾部延遲。

Node was designed before JavaScript had the concept of Promises or async/await. Node's counterpart to promises was the EventEmitter, which important APIs are based around, namely sockets and HTTP. Setting aside the ergonomic benefits of async/await, the EventEmitter pattern has an issue with back-pressure. Take a TCP socket, for example. The socket would emit "data" events when it received incoming packets. These "data" callbacks would be emitted in an unconstrained manner, flooding the process with events. Because Node continues to receive new data events, the underlying TCP socket does not have proper back-pressure, the remote sender has no idea the server is overloaded and continues to send data. To mitigate this problem, a `pause()` method was added. This could solve the problem, but it required extra code; and since the flooding issue only presents itself when the process is very busy, many Node programs can be flooded with data. The result is a system with bad tail latency.

在 Deno 中，套接字依舊是非同步的，但在接收新資料上仍要求使用者顯示的調用`read()`。在構建一個接收套接字上不需要一個額外的暫停語意。這不僅僅只有在 TCP 套接字上，在系統的最底的綁定層也是從根本上綁定了 promised - 我們對這些綁定稱之為`ops`。所有在 Deno 任何形式的回調都來自 promises。

In Deno, sockets are still asynchronous, but receiving new data requires users to explicitly `read()`. No extra pause semantics are necessary to properly structure a receiving socket. This is not unique to TCP sockets. The lowest level binding layer to the system is fundamentally tied to promises - we call these bindings "ops". All callbacks in Deno in some form or another arise from promises.

在 Rust 中，它有自己的類 promise 的抽象概念，叫 Futures。通過`op`這個抽象概念，可以很簡單的將在 Rust 中基於 future 的 APIs 與 JavaScript 的 promises 進行綁定。

Rust has its own promise-like abstraction, called Futures. Through the "op" abstraction, Deno makes it easy to bind Rust future-based APIs into JavaScript promises.

## Rust APIs

我們編寫的一個主要的組件是 Deno 命令行介面(CLI)。今天 CLI 的版本來到了 1.0。但 Deno 不是一個單獨的程式，它被設計成是一種 Rust 套件的集合，可以在不同的層次中進行整合。

The primary component that we ship is the Deno command-line interface (CLI). The CLI is the thing that is version 1.0 today. But Deno is not a monolithic program, but designed as a collection of Rust crates to allow integration at different layers.

[deno_core 套件](https://crates.io/crates/deno_core)(`crate`)在 Deno 中是一個非常重要(`very bare bones`)的一塊。它不依賴 TypeScript 和[Tokio](https://tokio.rs/)，只提供了 Op 和資源基礎架構。也就是，它提供了一種系統化的將 Rust futures 綁定到 JavaScript promises 上的方法。CLI 完全就是構建在 deno_core 之上。

The [deno_core](https://crates.io/crates/deno_core) crate is a very bare bones version of Deno. It does not have dependencies on TypeScript nor on [Tokio](https://tokio.rs/). It simply provides our Op and Resource infrastructure. That is, it provides an organized way of binding Rust futures to JavaScript promises. The CLI is of course built entirely on top of deno_core.

[rusty_v8](https://crates.io/crates/rusty_v8)套件提供了將 Rust 綁定到 V8 的 C++ 上的高性能 API。這些 API 盡可能的不偏離原來的 C++ API。這是一種零成本(`zero-cost`)的綁定 - 那些在 Rust 中暴露的物件與在 C++ 中操作的物件完全相同。（例如，之前嘗試在 Rust V8 的綁定中，強制使用 Persistent handles。）這個套件提供在 Github Actions CI 中內置的二進制資料，它還允許使用者從頭開始編譯 V8，並調整各種編譯配置。V8 的原始碼分布在套件中。最終 rusty_v8 試著成為一個完全的接口。它目前還不是 100%安全，但已經很接近了。能夠與像 V8 這樣複雜的 VM 進行交互是很棒的，它讓我們發現了許多在 Deno 中的 bug。

The [rusty_v8](https://crates.io/crates/rusty_v8) crate provides high quality Rust bindings to V8's C++ API. The API tries to match the original C++ API as closely as possible. It's a zero-cost binding - the objects that are exposed in Rust are exactly the object you manipulate in C++. (Previous attempts at Rust V8 bindings forced the use of Persistent handles, for example.) The crate provides binaries that are built in Github Actions CI, but it also allows users to compile V8 from scratch and adjust its many build configurations. All of the V8 source code is distributed in the crate itself. Finally rusty_v8 attempts to be a safe interface. It's not yet 100% safe, but we're getting close. Being able to interact with a VM as complex as V8 in a safe way is quite amazing and has allowed us to discover many difficult bugs in Deno itself.

## Stability

我們承諾在 Deno 上會維護一個穩定的 API。Deno 有很多的接口和組件，因此來明確我們所定義的`穩定`是很重要的。我們實現的，用來與作業系統間進行交互的 JavaScript APIs 都可以在`Deno`這個命名空間中找到(如 `Deno.open()`)。這些都已經過了仔細的審查，我們將不會做出不向下兼容的改動。

We promise to maintain a stable API in Deno. Deno has a lot of interfaces and components, so it's important to be transparent about what we mean by "stable". The JavaScript APIs that we have invented to interact with the operating system are all found inside the "Deno" namespace (e.g. `Deno.open()`). These have been carefully examined and we will not be making backwards incompatible changes to them.

所有還沒穩定下來的動能都隱藏在`--unstable`這個命令行參數下。你可以[在這](https://doc.deno.land/https/raw.githubusercontent.com/denoland/deno/master/cli/js/lib.deno.unstable.d.ts)找到不穩定接口的文檔。在之後的版本中，部分的 APIs 將會被穩定下來。

All functionality which is not yet ready for stabilization has been hidden behind the `--unstable `command-line flag. You can see the documentation for the unstable interfaces [here](https://doc.deno.land/https/raw.githubusercontent.com/denoland/deno/master/cli/js/lib.deno.unstable.d.ts). In subsequent releases, some of these APIs will be stabilized as well.

在全局命名空間你可以發現其他所有的物件（像是`setTimeout()`或 `fetch()`）。我們努力嘗試者讓這些接口與在瀏覽器中保持一致。但如果有發現不兼容的部分我們也會做出改正。是瀏覽器標準定義了這些接口，不是我們。所有我們提出的改正都會是 bug 修復，而非接口變動。如果瀏覽器標準 API 存在不兼容，將會在下一個主版本釋出前做出修正。

In the global namespace you'll find all sorts of other objects (e.g. `setTimeout()` and `fetch()`). We've tried very hard to keep these interfaces identical to those in the browser; but we will issue corrections if we discover inadvertent incompatibilities. The browser standards define these interfaces, not us. Any corrections issued by us are bug fixes, not interface changes. If there is an incompatibility with a browser standard API, that incompatibility may be corrected before a major release.

Deno 也有許多的 Rust APIs，像是 deno_core 或 rusty_v8。但這些 APIs 都還未達到 1.0 版本。我們會持續的迭代他們。

Deno also has many Rust APIs, namely the deno_core and rusty_v8 crates. None of these APIs are at 1.0 yet. We will continue to iterate on them.

## Limitations

認識到 Deno 不是一個 Node 的分支是非常重要的，Deno 完全是一個新的實現。Deno 的開發只有兩年的時間，而 Node 已超過了十年之久。由於各方對 Deno 的興趣，我們期望它能持續的發展成熟。

It's important to understand that Deno is not a fork of Node - it's a completely new implementation. Deno has been under development for just two years, while Node has been under development for over a decade. Given the amount of interest in Deno, we expect it to continue to evolve and mature.

對於部分應用，Deno 會是一個很好的選擇，但不會是對全部的應用。這取決於應用的需求。我們認為透明的闡述這些局限性，可以幫助人們在考慮是否使用 Deno 上做出明智的決定。

For some applications Deno may be a good choice today, for others not yet. It will depend on the requirements. We want to be transparent about these limitations to help people make informed decisions when considering to use Deno.

### Compatibility

不幸的是，對於現有的 JavaScript 工具缺乏兼容性。總體來說，對於 Node(NPM)包，Deno 是不兼容的。在[https://deno.land/std/node/](https://deno.land/std/node/)是有一個處於初期階段的兼容層，但距離完成還有一段距離。

Unfortunately, many users will find a frustrating lack of compatibility with existing JavaScript tooling. Deno is not compatible, in general, with Node (NPM) packages. There is a nascent compatibility layer being built at
[https://deno.land/std/node/](https://deno.land/std/node/) but it is far from complete.

儘管 Deno 在簡化模組系統這塊採取了較為強硬的態度，但 Deno 和 Node 依舊是兩個非常相似的系統，也在解決著相似的問題。隨著時間推移，我們期望 Deno 能夠對越來越多的 Node 程式做到開箱即用。

Although Deno has taken a hardline approach to simplifying the module system, ultimately Deno and Node are pretty similar systems with similar goals. Over time, we expect Deno to be able to run more and more Node programs out-of-the-box.

### HTTP Server Performance

我們持續的追蹤 Deno 的 HTTP 伺服器的[表現](https://deno.land/benchmarks)。一個 hello-world 的 Deno HTTP 伺服器每秒處理大約 25000 個請求，最大延遲為 1.3 毫秒。相比同樣情況下的 Node 服務，每秒能處理大約 34000 個請求，最大延遲浮動在 2-300 毫秒。

[We continuously track the performance of Deno's HTTP server](https://deno.land/benchmarks). A hello-world Deno HTTP server does about 25k requests per second with a max latency of 1.3 milliseconds. A comparable Node program does 34k requests per second with a rather erratic max latency between 2 and 300 milliseconds.

Deno 的 HTTP 伺服器是在原生的 TCP 套接字(`sockets`)上用 TypeScript 實現的。Node 的 HTTP 伺服器是用 C 編寫，並通過高階綁定(`high-level bindings`)的方式暴露給 JavaScript。我們拒絕在 Deno 上綁定原生的 HTTP 伺服器，因為我們希望在未來能優化 TCP 套接字，更通俗的講是 op 接口。

Deno's HTTP server is implemented in TypeScript on top of native TCP sockets. Node's HTTP server is written in C and exposed as high-level bindings to JavaScript. We have resisted the urge to add native HTTP server bindings to Deno, because we want to optimize the TCP socket layer, and more generally the op interface.

Deno 是個還不錯的非同步伺服器，25000 個/秒的請求處理已能滿足大部分的服務。(如果還不行，那 JavaScript 就不會是個最佳選擇。)此外，由於普遍的使用了 promises(上述)，我們期望 Deno 能展示出更佳的尾部延遲。綜上，我們相信在這個系統上還能有更多的優勢，並且希望能在之後的版本中實現這個目標。

Deno is a proper asynchronous server and 25k requests per second is quite enough for most purposes. (If it's not, probably JavaScript is not the best choice.) Furthermore, we expect Deno to generally exhibit better tail latency due to the ubiquitous use of promises (discussed above). All that said, we do believe there are more performance wins to be had in the system, and we hope to achieve that in future releases.

## TSC Bottleneck

Deno 內部使用了微軟的 TypeScript 編譯器來做型別檢查和生產出對應的 JavaScript。相比用 V8 去解析 JavaScript 的時間，它是非常慢的。在專案早期，我們希望`V8 Snapshots`能為這塊部分帶來顯著的提升。`Snapshots`確實有所幫助，但依舊是很慢。我們當然可以在現有的 TypeScript 編譯器上做出改進，但我們明白最終還是需要通過 Rust 來實現型別檢查。這會是一項艱鉅的、非短期能完成的任務。但它能給開發人員在關鍵路徑的體驗(`critical path experienced`)上提升一個量級。`TSC`必須移植到 Rust 上。如果你對這塊問題有興趣，請聯繫我們。

Internally Deno uses Microsoft's TypeScript compiler to check types and produce JavaScript. Compared to the time it takes V8 to parse JavaScript, it is very slow. Early on in the project we had hoped that "V8 Snapshots" would provide significant improvements here. Snapshots have certainly helped but it's still unsatisfyingly slow. We certainly think there are improvements that can be done here on top of the existing TypeScript compiler, but it's clear to us that ultimately the type checking needs to be implemented in Rust. This will be a massive undertaking and will not happen any time soon; but it would provide order of magnitude performance improvements in a critical path experienced by developers. TSC must be ported to Rust. If you're interested in collaborating on this problem, please get in touch.

## Plugins / Extensions

我們有一個處於初期階段的套件(`plugin`)系統，通過客製化設定來擴展 Deno 的執行期。然而，這套接口目前仍處在開發階段並已被標記為不穩定。因此要通過非 Deno 提供的方式訪問原生系統是很困難的。

We have a nascent plugin system for extending the Deno runtime with custom ops. However this interface is still under development and has been marked as unstable. Therefore, accessing native systems beyond that which is provided by Deno is difficult.

## Acknowledgements

Many thanks to the many contributors who helped make this release possible. Espcially: @kitsonk who has had a massive hand in many parts of the system, including (but not limited to) the TypeScript compiler host, deno_typescript, deno bundle, deno install, deno types, streams implementation. @kevinkassimo has contributed countless bug fixes over the whole history of the project. Among his contributions are the timer system, TTY integration, wasm support. Deno.makeTempFile, Deno.kill, Deno.hostname, Deno.realPath, std/node's require, window.queueMircotask, and REPL history. He also created the logo. @kt3k implemented the continuous benchmark system (which has been instrumental in almost every major refactor), signal handlers, the permissions API, and many critical bug fixes. @nayeemrmn contributes bug fixes in many parts of Deno, most notably he greatly improved the stack trace and error reporting, and has been a forceful help towards the stabilizing the APIs for 1.0. @justjavac has contributed many small but critical fixes to align deno APIs with web standards and most famously he wrote the VS Code deno plugin. @zekth has contributed a lot of modules to std, among them the std/encoding/csv, std/encoding/toml, std/http/cookies, as well as many other bug fixes. @axetroy has helped with all things related to prettier, contributed many bug fixes, and has maintained the VS Code plugin. @afinch7 implemented the plugin system. @keroxp implemented the websocket server and provided many bug fixes. @cknight has provided a lot of documentation and std/node polyfills. @lucacasonato built almost the entire deno.land website. @hashrock has done a lot of amazing artwork, like the loading page on doc.deno.land and the lovely image at the top of this page!
