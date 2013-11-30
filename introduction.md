#Manifest

什么是离线web应用？这听来是件矛盾的事。web页面需要下载和渲染的。下载就意味着要有网络连接。那么在离线的时候怎么下载资源？当然不能。但是可以在有网络连接的时候下载。这就是HTML5离线应用的工作。

简单的来说，离线应用一个列表的URLS，包括HTML，CSS，JavaScript，图片和一些其他资源。离线应用的首页指向这些文件，叫做manifest文件，是放在web服务器上的一个文本文件。实现了HTML5离线应用的浏览器将读取manifest文件中的URLS列表，下载资源，缓存在本地，自动保存本地直到它们有变化。在没有网络连接时访问web应用时，浏览器就自动切换到本地缓存。

#CACHE MANIFST

一个离线应用需要解析一个manifest文件。什么是manifest文件？当没有网络连接时，需要访问的一组资源列表。为了让进程下载和缓存资源，需要在html标签上定义manifest属性，指向manifest文件。

    <!DOCTYPE html>
    <html manifest="/cache.manifest">
    <body>
    ...
    </body>
    </html>

可以把manifest文件放在服务器上任意的地方，但是必须设置content type是text/cache-manifest。如果是Apache服务器，可以在web应用的根目录的.htaccess文件中增加类型
    
    AddType text/cache-manifest .manifest

确定manifest文件以.manifest结尾。如果用不同的web服务器或者一个不同的Apache配置，根据服务器的佩紫怀黄文档去控制content type。

    问：比如我的web应用多于一个页面。是否需要在每个页面都增加manifest属性吗，还是在在首页增加就可以？
    答：每个页面都增加manifest属性

下面来看下manifest文件的内容，第一行是这样：

    CACHE MANIFEST

所有manifest文件都分成三个部分：explicit、fallback、online whitelist。这三个部分都有自己的头部。如果没有任何头部，那么所有的资源都被认为是explicit部分。

一个有效的manifest文件。包括了三部分资源：一个css文件、一个js文件和一个jpg图片。

    CACHE MANIFEST
    /clock.css
    /clock.js
    /clock-face.jpg

explicit部分被下载和存储到本地，当失去网络连接时使用。浏览器通过加载manifest文件，从web服务器的根目录下载clock.css, clock.js, and clock-face.jpg文件。当失去网络连接，刷新页面时，这些资源都是可用的。

    问：在我们的manifest文件中，需要列出所有的HTML页面吗？
    答：分情况看。如果web应用是一个单页面，那要确保页面给html增加manifest属性，并指向manifest文件。当访问一个manifest属性的页面时，页面假设自己也是web应用的一部分，所以不需要把它自己也列在manifest文件中。如果web应用是多页面时，因为浏览器不知道哪些页面需要下载和缓存，所以那就必须在manifest文件中列出所有的HTML页面。

#NETWORK SECTIONS

举个例子，假设有个时钟应用用来追踪访问者，通过加载img的src属性。那么缓存资源与追踪的目的矛盾，所以这个资源应该不被缓存，离线时也不能使用。那么我们应该这么做：

    CACHE MANIFEST
    NETWORK:
    /tracking.cgi
    CACHE:
    /clock.css
    /clock.js
    /clock-face.jpg

manifest文件中的explicit部分用NETWORK:作为头部，下面是“在线白名单”。这部分资源不会缓存，而且不能离线使用。

#FALLBACK SECTIONS

在这个部分，可以定义当无论什么原因，当cache失效时的替代资源。HTML5规范中的例子如下：

    CACHE MANIFEST
    FALLBACK:
    / /offline.html
    NETWORK:
    *

这部分的是做什么的呢？首先，例如维基百科包含了数以百万计的网页。我们不可能下载整个网站。假设只需要一部分的离线应用。那么怎么决定缓存那些页面呢？可以离线访问的页面都应该是被下载和缓存过的。应该是之前访问过的页面，讨论页面，编辑页面。

这就是manifest的工作。假定维基百科上的每一个页面都被写在manifest文件中。当访问任何在manifest文件中定义的页面，浏览器会认为这是离线应用的一部分，我是否已经下载它了？如果浏览器没有下载这个资源，将建立一个新的appcache（application cache），下载所有在manifest文件中定义的资源，然后把当前页面加到已经存在的appcache中。无论哪种方式，刚刚访问的页面在appcache中存储。这很重要。这意味着有一个后添加页面到的离线应用。不需要在manifest文件中列出所有单独的HTML页面。

现在看fallback部分。在manifest文件中只有简单的一行。它不是一个URL，只是URL的格式。这个 / 将匹配网站上非首页的其他页面。在离线是访问一个页面，浏览器会先在appcache中查找。如果浏览器在appcache中找到了页面(因为在有网络时访问时，已经自动将页面添加到appcache中了)，那么浏览器将显示本地缓存的页面。如果浏览器在appcache中没有找到页面，不是显示错误信息，而是显示 /offline.html 这个页面。

最后，检查一个NETWORK部分。也只有简单的一行，这一行包括一个字符 * 。这个字符有特殊的含义，它叫做“在线白名单通配符”。它的功能是，在有网络连接时，任何不在appcache中的资源都从原始地址下载。这对于一个广泛的离线应用很重要，这意味着当浏览浏览离线应用在线时，正常获取图片、视频和其他资源，即使他们在不同的主域(在大网站这很平常，甚至他们不是离线应用的一部分，HTML页面在服务器，图片、视频在其他域的CDN)。如果没有通配符，离线应用在有网络链接时会很奇怪，它不会从其他的服务器加载图片和视频。

维基百科不仅仅只有HTML文件。还包含css、js和图片。每个资源都需要明确的写在CACHE部分，这样在离线时才能正常使用。在FACKBALL部分，如果想拥有一个广泛的离线应用，需要扩展在manifest文件中已经明确定义的资源。

#THE FLOW OF EVENTS

当浏览器访问一个使用了manifest的页面时，会在window.applicationCache对象上触发一系列事件。

1.  当浏览器发现<html>有manifest属性时，浏览器会触发checking事件。(所有事件列表都是在window.applicationCache对象上触发)不管之前是否访问过这个页面或者任何其他指向相同manifest文件的页面，都会触发checking事件。
2.  如果浏览器之前没有缓存过manifest
    *  触发downloading事件，然后开始下载在manifest文件中的缓存资源。
    *  当触发downloading事件后，浏览器将循环的触发progress事件，包含了有多少文件已经下载和有多少文件即将被下载。
    *  所有资源都下载成功后，浏览器最后触发cached事件。这是一个标志，离线应用已经完全缓存，可以用于离线状态。
3.  如果之前访问过这个页面或者任何其他指向相同manifest文件的页面，浏览器已经知道这个manifest文件。可能有些资源已经在appcache中。它可能已经包括离线应用的所有资源。那么现在的问题是，上次浏览器检查的manifest文件是否发生了变化？
    *  如果没发生变化，浏览器会触发noupdate事件
    *  如果发生变化，浏览器会触发downloading事件，然后开始重新下载每一个在CACHE中的资源
    *  当触发downloading事件后，浏览器将循环的触发progress事件，包含了有多少文件已经下载和有多少文件即将被下载。
    *  所有资源都下载成功后，浏览器会最后触发一个updateready事件。
    *  如果没发生变化，浏览器会触发noupdate事件。这是一个标志，代表新版本离线应用已经完全缓存，为离线状态做好了准备。但是新版本现在还没有使用，为了实现热插拔而不强制用户去重载页面，可以使用window.applicationCache.swapCache()方法。

在这个过程中，如果发生错误，浏览器将触发error事件并停止。下面是一些会产生错误的情况：
    *  manifest文件返回http 404(文件未找到)或410(永久性不可用)
    *  manifest文件发现并且没有发生改变，但是manifest文件中指向的HTML页面不能下载
    *  当更新manifest文件时，manifest文件发生变化。
    *  manifest文件发现并且发生改变，但是浏览器不能下载到manifest文件中的资源之一

#调试的方法

如果manifest文件中的一个资源下载失败，整个缓存进程都将失败，离线应用也将不起作用。浏览器将触发error事件，但是并没有指出实际的错误。这就使调试离线应用让人很迷惑。

必须做精确的检查，是否manifest文件发生变化。

1.  和其他通过HTTP请求访问的文件一样，浏览器在响应头会包括元数据信息，检查文件是否过期。manifest文件也是一样。某些HTTP头(Expires和Cache-Control)会通知浏览器怎样允许缓存文件，而不去服务器校验它是否已经发生变化。这种缓存方式与离线应用无关。
2.  manifest文件已经过期(相对于HTTP头来说)，那么浏览器会问服务器是否有一个新的版本，如果有，那么浏览器会下载它。为了做到这一点，浏览器会发送一个包括了manifest文件最后修改时间(浏览器响应头最后下载manifest文件的时间)的请求。如果服务器发现manifest文件与那个时间的版本没有变化，它将返回304状态(没有变化)。这还是和之前一样，不是特定的离线应用。
3.  如果服务器发现manifest文件与那个时间的版本发生变化，它将返回200状态，随后返回新文件的内容，和Cache-Control头和一个新的最后修改时间，以便让步骤1和步骤2下次正确的运行。一旦这次下载了新的manifest文件，浏览器会和上次下载的资源做检查。如果manifest文件和上次一样，浏览器不会重新下载manifest文件中的资源。

以上三点会使你在开发和测试离线应用时犯错。例如，开发了一个manifest文件，但是10分钟之后，发现需要添加资源，那么添加一行，重新发布。重载页面，浏览器检测到manifest属性，会触发checking事件，……。浏览器发现manifest文件没有发生改变。为什么？因为服务器默认设置，是浏览器在一定时间内会缓存静态资源。(HTTP语法，使用Cache-Control头来控制)。这就意味着浏览器不会执行步骤1。服务器上的文件已经发生变化，但是浏览器并没有给服务器发请求。为什么？因为上次浏览器下载manifest文件时，服务器告诉浏览器在一定时间缓存这些资源。现在，10分钟之后，浏览器才真正知道该做什么。

现在清楚了，这不是一个bug，而是一个特性。每件事都是按照设计好的在执行。如果服务器没用一个方式告诉浏览器去缓存，那么服务器一个晚上就崩溃了。

所以现在有件事必须做：在服务端配置manifest文件不被HTTP缓存。如果在一个apache服务器，在.htaccess文件中修改两行：

    ExpiresActive On
    ExpiresDefault "access"

这会不缓存目录下和所有子目录的文件。但是在产品中，不能这么做，所以你应该文件目录只影响manifest文件，或者创建一个子目录只包含.htaccess文件和manifest文件。

一旦HTTP不缓存manifest文件，当每次访问服务器上的URL时，你将有时间改变在appcache中的资源。现在步骤2又来折磨你。如果manifest文件没有发生变化，那么浏览器将永远也不会发现之前缓存的资源已经变化。参考下面的例子：

    CACHE MANIFEST
    # rev 42
    clock.js
    clock.css

如果改变了clock.css重新发布，将不会看到变化，因为manifest文件没有变化。每次离线应用改变文件后，同时也需要改变manifest文件自身。常用的办法是修改注释，用一个版本号。改变版本号，服务器将返回新的manifest文件，浏览器将重新下载manifest文件中的资源。

    CACHE MANIFEST
    # rev 43
    clock.js
    clock.css

