# Wireshark Lab:HTTP

在导语实验中已经对wireshark分组嗅探器有了些了解之后，我们现在准备使用wireshark来调查操作中的协议。在这个实验中，我们将探索HTTP协议的一些方面：基本的GET/响应交互，HTTP报文格式，检索嵌入式的HTML文件，以及HTTP验证和安全。在开始这些实验之前，你可能想要复习一下书上的2.2节

1. 基本的HTTP GET/响应交互

   让我们通过下载一个非常简单的HTML文件（非常短，没有绑定的东西）来开始我们的HTTP探索。操作如下：
   1. 打开你的浏览器
   2. 打开wireshark分组嗅探器，就像intro 实验中那样。在筛选窗口处输入HTTP，这样只有捕获到的HTTP报文将被在之后的分组列表窗口中被展示。
   3. 等一分钟多一点，然后启动wireshark分组捕获器
   4. 在浏览器中输入下列网址：http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file1.html 你的浏览器应当显示一个非常简单，只有一行的HTML文件
   5. 停止wiresahrk分组捕获器
   
   你的wireshark窗口应当看起来像图1。如果你没能在一个活动的网络连接上运行wiresahrk，你可以下载一个在以上步骤执行时能被创建的分组追踪器
   
   图1中的例子在分组列表窗口中展示了两个被捕获的HTTP报文：GET报文和从服务器到你的浏览器的响应报文。回忆一下，HTTP报文被通过TCP报文段传输，TCP报文段又通过IP数据段传输，IP数据段由以太网帧传输，wireshark显示帧，以太网，IP和TCP报文信息。我们想最小化非HTTP数据的展示，所以收起帧，以太网，IP和TCP信息。

   通过看HTTP GET和响应报文中的信息，回答下列问题。当回答下列问题时，你应该打印出GET和相应报文并指出你在报文的哪里找到了问题的答案。当你提交作业时，注释清楚你是从哪里获得问题的答案的。
      1. 你的浏览器运行的是HTTP1.0还是1.1？服务器运行的HTTP是哪个版本的？
      2. 你的浏览器向服务器指出哪种语言可以被接受？
      3. 你电脑的IP是什么？gaia.cs.umass.edu服务器的呢？
      4. 服务器向你的浏览器发出的状态码是什么？
      5. 你在服务器检索到HTML文件最后的修改时间?
      6. 你的浏览器被返回了多少字节的内容？
      7. 通过在内容窗口中检查报文中的原始数据，你是否看到包含数据但没有在分组列表中现实的头？如果有，请命名。
   在前五个问题中，你或许会惊讶于发现你检索到的文档上一次被修改是在你下载文档的一分钟前。那是因为，gaia.cs.umass.edu服务器在每隔一分钟将文件的最后修改时间设置为当前时间。因此，如果你在连接之间等一分钟，文件就会变成刚修改过的，你的浏览器就会下载一个新的文档的拷贝。

2. HTTP条件GET/响应交互
   
   回忆2.2节，大多数浏览器进行对象缓存因此会在检索HTTP对象时执行条件GET。在做接下来的步骤前，确保你的浏览器缓存是空的。现在如下操作：
      * 打开你的浏览器，如上，确保你的浏览器缓存清空了。
      * 打开wireshark分组嗅探器
      * 在浏览器中输入下列URL：http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file2.html 你的浏览器应该显示一个非常简单的五行HTML文件
      * 尽快在浏览器中再一次输入相同的URL
      * 停止wireshark分组捕获器，在筛选窗口输入HTTP，这样只有被捕获的HTTP报文会被显示在分组列表窗口中
   回答下列问题：
      8. 检查从你的浏览器到服务器的第一条HTTP GET请求。你是否在HTTP GET中看到一行“IF-MODIFIED-SINCE”？
      9. 检查服务器响应的内容。服务器是否准确返回了文件内容？你怎么知道的？
      10. 现在检查从你的浏览器到服务器的第二个HTTP GET请求。你是否在HTTP GET中看到一行“IF-MODIFIED-SINCE”？如果是，“IF-MODIFIED-SINCE”之后是什么？
      11. 第二个HTTP GET从服务器返回的状态码和字段是什么？服务器清楚的返回了文件的内容吗？请解释。
3. 检索长文档
   
   在我们迄今的例子中，被检索的文档一直是简单和短的HTML文件。让我们来看看当我们下载一个长的HTML文件时会发生什么。操作如下：
      * 打开你的浏览器，确保你的浏览器缓存已清空。
      * 打开wireshark分组嗅探器
      * 在你的浏览器中输入下列URL：http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file3.html 你的浏览器应当会显示相当长的美国权利法案
      * 停止wireshark分组捕获器，在筛选窗口中输入HTTP来使得只有被捕获的HTTP报文会被显示
   
   在分组列表窗口中，你应该看到你的HTTP GET报文后紧跟着多个对你的HTTP GET请求的TCP响应。这多个响应应当做些解释。回忆2.2节，HTTP响应报文包括一行状态，之后是首部行，之后是一行空行，之后是数据部分。如果是HTTP GET，响应中的数据部分完全是请求的HTML文件。就我们的情况而言，HTML文件相当长，4500比特对一个TCP包太长了。单独的HTTP响应因此被TCP分成多个小片，每一片都包含一个独立的TCP报文段。在最新版本的wireshark，wireshark把每一个TCP段当作一个独立的数据报，而“单独的HTTP相依ing被分成多个TCP数据报”是由wireshark视图选项中的“重新组装的PDU的TCP段 指定的。"早期的wireshark使用"Continuation"选项来指定一个HTTP响应分成多个TCP段。我们假定HTTP报文中没有“Continuation”

   回答下列问题：
      12. 你的浏览器发送了多少HTTP GET请求？跟踪到的哪个GET报文请求权利法案？
      13. 哪个响应报文包括了状态码和相关段？
      14. 响应中的状态码和段是什么？
      15. 需要有多少TCP段来传输一个短的HTTP请求和权利法案文本？
   
4. 有绑定对象的HTML文档

   现在我们已经看到wireshark如何显示被捕获的大HTML文件数据报，我们考虑当你的浏览器下载一个有绑定对象的文件时发生了什么。

   操作如下：
      * 打开你的浏览器，确保缓存被清除
      * 打开wireshark分组嗅探器
      * 在浏览器中输入下列URL：http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file4.html 你的浏览器应当显示一个短的HTML文件，其中包含两个图片。这两个图片被基础的HTML文件引用。因此，图片本身没有被包含在HTML文件中。相反，下载的HTML文件中包含图片的URL。正如教材中所讨论的，你的浏览器将从指定的网站检索logo。我们出版商的logo能从gaia.cs.umass.edu检索到。我们第五版书的封面在caite.cs.umass.edu服务器上存储。
      * 停止wireshark分组捕获器，在筛选框中输入HTTP使得只有被捕获的HTTP报文将被显示。

   回答下列问题：
      16. 你的浏览器发出了多少HTTP GET请求？这些GET请求被发往了哪个因特网地址？
      17. 你能说出你的浏览器连续的现在这两张图片还是非连续的吗？给出解释
5. HTTP认证
   
   最后，让我们试试访问一个有密码保护的网站并检查其中交换的HTTP报文。URL http://gaia.cs.umass.edu/wireshark-labs/protected_pages/HTTP-wireshark-file5.html 是被密码保护的。用户名是wireshark-students，密码是network。所以让我们连接这个安全的，带密码保护的网站。操作如下：
      * 确保你的浏览器缓存被清除。然后关闭你的浏览器，然后打开浏览器。
      * 打开wireshark分组捕获器
      * 在浏览器中输入下列URL：http://gaia.cs.umass.edu/wireshark-labs/protected_pages/HTTP-wireshark-file5.html 在顶部的框中输入需要的用户名和密码
      * 停止wireshark分组捕获器，在筛选窗口中输入http来让只有捕获到的HTTP报文在分组列表窗口中被显示

   现在让我们检查wireshark的输出。你可能想先读下HTTP访问身份验证上的指南材料来了解HTTP身份验证框架：http://frontier.userland.com/stories/storyReader$2159

   回答下列问题：
      18. 服务器对来自你的浏览器的初始HTTP GET报文的响应（状态码和段）是什么？
      19. 当你的浏览器第二次发送HTTP GET报文，HTTP GET 报文中新的东西是什么？
   
   你输入的用户名和密码被编码成d2lyZXNoYXJrLXN0dWRlbnRzOm5ldHdvcms=字符串，位于客户端HTTP GET报文中的”Authorization：Basic“之后。他们或许看起来你的用户名和密码被base64格式加密了。但实际上用户名和密码没被加密！为了显示，访问 http://www.motobit.com/util/base64-decoder-encoder.asp 并且输入你被base64编码的字符串（d2l....)然后解码。瞧！你已经将base64编码解码到ASCII编码，这应当能看见你的用户名！为了看密码，输入字符串(...)然后解码。因此任何人都可以下载一个类似wireshark的工具和包嗅探器通过他们的网络适配器，任何人都可以从base64向ASCII转换。显然，万维网上简单密码，除非有其他额外安全措施，否则就是不安全的。

   别怕！正如我们将在第八章看到的，有办法让www连接更安全。然而，我们很显然需要基础http认证框架之外的东西！
