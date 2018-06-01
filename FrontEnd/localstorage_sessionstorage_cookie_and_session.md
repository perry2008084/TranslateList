文章翻译自stackoverflow上的一个问题: [what-is-the-difference-between-localstorage-sessionstorage-session-and-cookies](https://stackoverflow.com/questions/19867599/what-is-the-difference-between-localstorage-sessionstorage-session-and-cookies)

# localStorage、sessionStorage、cookie和session的优缺点

This is an extremely broad scope question, and a lot of the pros/cons will be contextual to the situation.
这是一个很宽泛的问题，有很多优缺点与实际情况相关。

In all cases these storage mechanisms will be specific to an individual browser on an individual computer/device. Any requirement to store data on an ongoing basis across sessions will need to involve your application server side - most likely using a database, but possibly XML or a text/CSV file.
所有情况下，这些存储机制都是指某一个单独的电脑或设备上的一个单独的浏览器。任何需要在会话中进行的数据存储，需要有你的应用服务端的介入，大多数情况是一个数据库，但也可以是XML或者一个文本/CSV文件。

localStorage, sessionStorage and cookies are all client storage solutions. Session data is held on the server where it remains under your direct control.
localStorage, sessionStorage和cookies都是客户端存储解决方案。 Session数据是存储在服务端，但依然由你的直接控制。

# localStorage and sessionStorage

localStorage and sessionStorage are relatively new APIs (meaning not all legacy browsers will support them) and are near identical (both in APIs and capabilities) with the sole exception of persistence. sessionStorage (as the name suggests) is only available for the duration of the browser session (and is deleted when the tab or window is closed) - it does however survive page reloads (source  [DOM Storage guide - Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Storage)).
localStorage和sessionStorage是相对较新的API（也就是说不是所有旧浏览器都支持他们），并且近乎相同（API和功能上），唯一的不同在持久性。sessionStorage（如名字提示）只有在浏览器会话期间可用（当窗口或者标签页关闭即删除）- 在页面刷新时依然存在。（参考[MDN存储指南](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Storage)）

Clearly, if the data you are storing needs to be available on an ongoing basis then localStorage is preferable to sessionStorage - although you should note both can be cleared by the user so you should not rely on the continuing existence of data in either case.
明显的，如果你需要持续存储数据，那么localStorage比sessionStorage更合适，由于这两者均可被用户手动清除，因此不能依赖任何一个来保证持续存在。

localStorage and sessionStorage are perfect for persisting non-sensitive data needed within client scripts between pages (for example: preferences, scores in games). The data stored in localStorage and sessionStorage can easily be read or changed from within the client/browser so should not be relied upon for storage of sensitive or security related data within applications.
localStorage和sessionStorage适用于页面间的客户端脚本存储非敏感性的数据持久化（比如，偏好，赛事比分等）。保存在localStorage和sessionStorage中的数据很容易的从客户端或浏览器进行读取和修改，因此不要存储敏感或者安全相关的数据。

# Cookies

This is also true for cookies, these can be trivially tampered with by the user, and data can also be read from them in plain text - so if you are wanting to store sensitive data then session is really your only option. If you are not using SSL, cookie information can also be intercepted in transit, especially on an open wifi.
对于cookies，用户也可以轻松的操作，数据可以通过纯文本读取，因此如果你想要保存敏感数据的话session是你的唯一选择。如果你不是使用SSL，cookie信息可能会在传输中被截获，尤其是在一个开放的wifi情况下。

On the positive side cookies can have a degree of protection applied from security risks like Cross-Site Scripting (XSS)/Script injection by setting an HTTP only flag which means modern (supporting) browsers will prevent access to the cookies and values from JavaScript (this will also prevent your own, legitimate, JavaScript from accessing them). This is especially important with authentication cookies, which are used to store a token containing details of the user who is logged on - if you have a copy of that cookie then for all intents and purposes you  _become_  that user as far as the web application is concerned, and have the same access to data and functionality the user has.
好的一面是，cookies拥有一定程度的安全防护机制，比如跨站脚本(XSS)或者脚本注入，通过设置仅HTTP标志，现代的浏览器将会阻止通过JavaScript（也会阻止你合法JavaScript来访问他们）访问cookies和值。这对认证相关的cookies来说尤其重要，用于保存哪些用户登录的详情，如果你拥有相应目的的cookie拷贝，你就变成了相关应用的用户，并且可以访问对应用户可以访问的数据和功能。

As cookies are used for authentication purposes and persistence of user data,  **all**  cookies valid for a page are sent from the browser to the server for  **every**  request to the same domain - this includes the original page request, any subsequent Ajax requests, all images, stylesheets, scripts and fonts. For this reason cookies should not be used to store large amounts of information. Browser may also impose limits on the size of information that can be stored in cookies. Typically cookies are used to store identifying tokens for authentication, session and advertising tracking. The tokens are typically not human readable information in and of themselves, but encrypted identifiers linked to your application or database.
由于cookies是用于认证母的和持久化用户数据，页面中**所有**有效cookies都将会在**每个**同域请求下从浏览器发送到服务器，这包括原始域页面请求，任何后续的Ajax请求，所有的图片、样式、脚本和字体。由于这个原因，cookies不应该存储大量的信息。浏览器也强制限制了可以存储在cookies中的数据量。比较典型的coolies使用场景，存储认证的标识符token，会话session和广告跟踪。tokens一般是不适合阅读的信息，而是应用或者数据相关联的加密标识符。

# localStorage vs. sessionStorage vs. Cookies

In terms of capabilities, cookies, sessionStorage and localStorage only allow you to store strings - it is possible to implicitly convert primitive values when setting (these will need to be converted back to use them as their type after reading) but not Objects or Arrays (it is possible to JSON serialise them to store them using the APIs). Session storage will generally allow you to store any primitives or objects supported by your Server Side language/framework.
在功能方面，cookies、sessionStorage和localStorage只允许你存储字符串，可以通过隐式的转换基础数据当设置时（读取时需要将数据转换回原类型进行使用），但是对象和数据不行（可行的方案是通过JSON序列化相关API来存储他们）。会话Session存储通常允许你存储任意基础或者对象类型，只要你的服务端语言或框架支持。

# Client-side vs. Server-side

As HTTP is a stateless protocol - web applications have no way of identifying a user from previous visits on returning to the web site - session data usually relies on a cookie token to identify the user for repeat visits (although rarely URL parameters may be used for the same purpose). Data will usually have a sliding expiry time (renewed each time the user visits), and depending on your server/framework data will either be stored in-process (meaning data will be lost if the web server crashes or is restarted) or externally in a state server or database. This is also necessary when using a web-farm (more than one server for a given website).
由于HTTP是一个无状态的协议，web应用无法判断一个用户是否是从上一次访问后重新回来，会话session数据依赖于cookies token通常用于判断用户是否再次访问（尽管有少部分使用URL参数来达到同样目的）。数据通常有一个变化的过期时间（每次用户访问时刷新），并且依赖于你的服务器或框架，数据可能存储在进程（意味着数据在web服务器宕机或重启时可能丢失）或者外部的状态服务器或者数据库中。这在使用web群（一个网站用于多于1个服务器的情况）时同样是必须的。

As session data is completely controlled by your application (server side) it is the best place for anything sensitive or secure in nature.
由于会话session数据有你的应用完全控制（服务端），因此这是最合适存储任何天生敏感或安全的数据。

The obvious disadvantage with server side data is scalability - server resources are required for each user for the duration of the session, and that any data needed client side must be sent with each request. As the server has no way of knowing if a user navigates to another site or closes their browser, session data must expire after a given time to avoid all server resources being taken up by abandoned sessions. When using session data you should therefore be aware of the possibility that data will have expired and been lost, especially on pages with long forms. It will also be lost if the user deletes their cookies or switches browsers/devices.
服务端数据最大的劣势是可伸缩性，服务器资源在每个用户的会话session期间都是必须的，每个请求是客户端需要的数据都会进行发送。由于服务端没办法知道一个用户是否跳转到其他网站或者关闭浏览器，会话session数据必须要经过一段特定时间过期，避免服务器资源被废弃的会话session占用。当使用会话session数据，你要注意数据可能会过期或丢失的可能性，尤其是有较长表单的页面。另外，如果用户删除了他们的cookies或切换浏览器设备时也可能丢失数据。

Some web frameworks/developers use hidden HTML inputs to persist data from one page of a form to another to avoid session expiration.
有一些web框架或开发者使用隐藏的HTML input来从一个页面表单到另一个页面，避免会话session的过期。

localStorage, sessionStorage and cookies are all subject to "same-origin" rules which means browsers should prevent access to the data except from the domain that set the information to start with.
localStorage, sessionStorage和cookies都是服从同一域名准则，意味着浏览器会阻止除非是一开始设置的域名代码来访问数据。
