# Mac app和pkg签名公证盖章流程手册
## 前言
在我不算很长的开发经历中，MacOS 平台上的软件分发是最让人头大的。作为一个门外汉，即使有着官方文档作为指引，也非常容易卡在某些稀奇古怪的地方（是的，我知道你耳熟能详的企业和机构中有人被卡了几周乃止几个月）。比未知的关卡更可怕的是落后的工具，我刚开始接手开发工作时，需要使用命令行将一个 App 编译成 .pkg 并执行某些脚本和操作，这种体验堪比上古时代的刀耕火种。

为了打破这种不幸，我写下了这本操作手册。原本它只是我为某个机构的非技术人员撰写的培训手册，但是随着时间的推移，越来越多的同事和非同事向我索要这本手册。既然流传已经不可避免，使用的资料也可以公开查询到，索性公开以造福广大同行。其中必然有错漏的地方，欢迎指正。

## 你需要预先知道的知识
在 MacOS 上，应用程序以 .app 的形式存放于用户电脑上。应用程序分发有两种。
1. 可以通过官方的App Store下载，直接下载下来就是一个app。它需要遵循应用商店的审核规范和沙盒机制。
2. 在你的网站上放一个链接，用户通过这个链接下载 .pkg 或者 .dmg 格式的文件，这个文件包含了 .app文件。下载完成之后，再双击这个文件，进行手动的安装。pkg文件的好处在于方便执行安装之后或者安装之前的脚本，比如安装之前卸载旧 App ,检测系统版本，安装完成之后自动打开 App ,乃至修改用户的某些系统设置，这些都可以，也无需遵守沙盒机制。因此，需要较高权限的软件通常采用第二种形式的pkg格式进行分发。本文也主要针对此种形式进行讲解。

在 .app 和 .pkg 上，你都需要执行三件事：
1. 签名。
2. 公证。
3. 盖章。

其中，如果是以 .pkg 中包含了 .app 的形式进行分发，那么对 .app 的操作可以省略。但是我仍然建议你不要省略这几步，因为**问题暴露的时间越早越好。**

## 签名

### 什么叫签名

代码/应用签名是一种 macOS 安全技术，用于证明应用程序的来源。一旦应用程序被签名，系统就可以检测到应用程序的任何更改。无论更改是意外引入的还是由恶意代码引入的，比如对应用程序进行任何修改都会使签名失效。

发布pkg安装包时，需要使用的签名证书有两个：Developer ID application 和Developer ID installer。其中，Developer ID application用于给pkg安装包内部的app做签名， Developer ID installer 用于给pkg安装包本身做签名。

Developer ID 证书自创建之日起 5 年内有效；2017 年 2 月 22 日之前生成的 Developer ID 预置描述文件会随 Developer ID 证书一起到期。

对于pkg文件，官方文档：如果用于签名的 Developer ID installer证书已过期，则必须使用有效的 Developer ID installer 证书重新签名，pkg才能正常安装。实际上，经测试，只要签名该 pkg 文件的证书在签名时有效，那么即使在安装时该证书已过期，仍然可以被用户正常安装。但是我仍然建议你及时更新过期的证书及 .pkg ,放任过期证书泛滥并不是负责的做法。一旦苹果修改这个机制，你可能就需要通宵上线了。

撤销删除 Developer ID App 证书 并不是随意操作的，不管你的苹果开发者账号的角色是什么，均需通过 product-security@apple.com 向 Apple 发送请求，才可以删除。
对于所有 Developer ID App，如果用于签名的证书已被撤销，那么相应的 App 就无法再进行安装，已安装的 App 也会无法启动。

### 获取Bundle ID
打开下面这个网址并登陆：
`https://developer.apple.com/account/resources/identifiers/list`
1. 在“Certificates, Identifiers & Profiles (英文)”(证书、标识符和描述文件) 中，点按侧边栏中的“Identifiers”(证书)，再点按左上方Identifiersy右边的添加按钮 (+)。
   
<img width="1027" alt="截屏2024-02-26 下午11 40 44" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/32321cc3-26d9-4aa8-87af-3fc73f2a87b0">


如图，选择第一个“App IDs”。 点击继续。

<img width="1309" alt="截屏2024-02-26 下午11 40 58" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/5bf597f2-a2f6-4250-be06-26ee44cdb231">


3. 如下图，填写描述与bundle ID。

<img width="1306" alt="截屏2024-02-26 下午11 41 40" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/deb06f90-1d97-4b6f-920d-008acf70d901">


4. 在能力（Capabilities）中勾选你需要的权限，例如需要“访问WiFi信息(Access WiFi Information)”“网络扩展(Network Extensions)”“系统扩展(System Extension)”，就在这些选项前打勾。点击继续。
5. 在部署详情里选择正在早期开发中。因为不需要使用内部分发配置文件,也不需要上架。

<img width="454" alt="image" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/8fd45e46-4614-40ec-9790-47f43d2f54aa">


7. 继续，点击注册。即完成了操作。

### 获取Developer ID 证书
如果已有证书，可以忽略这一步。
注意，只有账户持有人才能创建Developer ID 证书。
1. 打开下面这个网址并登陆。
`https://developer.apple.com/account/resources/certificates/list`
2. 在“Certificates, Identifiers & Profiles (英文)”(证书、标识符和描述文件) 中，点按边栏中的“Certificates”(证书)。点按右侧的添加按钮 (+)。

<img width="1293" alt="image" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/4a16a5a9-8a33-49a1-bf79-2f312525c127">

3. 在“Software”(软件) 下面，选择“Developer ID”，然后点按“Continue”(继续)。
**注意，下图演示的是Developer ID Application 的获取，pkg签名需要 Developer ID Installer，操作完全一致，只是需要选中Developer ID Installer，而不是 Developer ID Application 。**

<img width="456" alt="image" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/4b8920a4-9ee3-4c48-a257-b6754657cc8c">

4．按照说明创建证书签名请求。

   1) 启动位于mac的“钥匙串访问”。

   2) 选取“钥匙串访问”>“证书助理”>“从证书颁发机构请求证书”。


  <img width="834" alt="截屏2024-02-26 下午11 44 22" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/9fa1efac-b704-42c1-a9e4-b37fc67b946c">


   3)	在“证书助理”对话框中，在“用户电子邮件地址”栏位中输入电子邮件地址。
   4)	在“常用名称”栏位中，输入密钥的名称。
   5)	将“CA 电子邮件地址”栏位留空。
   6)	选取“存储到磁盘”，然后点按“继续”。
     
  <img width="608" alt="截屏2024-02-26 下午11 46 11" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/66e7543b-658a-4282-ac32-d61a007b7c55">

  
5. 点按“Choose File”(选取文件)。选择上一步创建并保存的证书签名请求。
 
6. 在出现的对话框中，选择证书请求文件 (文件扩展名为 .certSigningRequest 的文件)，然后点按“Choose”(选取)。
7. 点按“Continue”(继续)。
8. 点按“Download”(下载)。这个证书文件 (文件扩展名为 .cer 的文件) 会出现在“下载”文件夹中。
要在钥匙串中安装这个证书，请双击已下载的证书文件。这个证书会出现在“钥匙串访问”中的“我的证书”类别中。

<img width="862" alt="截屏2024-02-26 下午11 22 23" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/ff29b510-8193-48f5-88a6-c8a984e370ed">

正常应显示如上。注意一个关键：**Developer ID 证书下应包含一个“专用密钥”**，没有这个专用密钥，是无法进行签名的。这个专用密钥其实在前述第四步生成证书签名请求时会保存，默认会保存到“钥匙串访问”中的“登录”类别中。而如果你的在第八步下载的证书文件没有保存到“钥匙串访问”中的“登录”类别，而是在“系统”类别，就会出现无法找到专用密钥的问题。

### 获取provisioning profiles
进入苹果开发者网页并登录。在 Profiles 中点击“+”。

<img width="1302" alt="截屏2024-02-26 下午11 48 30" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/1c4eff1e-8957-4f77-9eab-ea5fad1dd04b">

选择`Distribution`最下方的`Developer ID`，点击 `Continue`

<img width="1324" alt="截屏2024-02-26 下午11 49 55" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/e2634f21-b2f9-4430-9920-3d2c3e221f95">

选择对应的app id ,填入“获取Bundle ID” 中自定义的ID.

<img width="1242" alt="image" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/24c77d06-a26a-411e-9397-49198101366f">

选择对应的证书。如果没有会让你重新生成一遍证书。

<img width="1229" alt="image" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/da980823-af6f-4784-b183-e4b1d958a34c">

为Provisioning Profile命名。点击下载描述文件.

在Xcode中导入Provisioning Profile 。注意，不要勾选Automatically manage signing。

<img width="760" alt="截屏2024-02-26 下午11 59 14" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/7fea9a87-b4f5-4f20-a5a2-11f9fda98d23">

### 如果你需要和他人分享你的证书
虽然不建议多人共同持有签名证书，尤其是在商业开发的模式下，会造成一定的风险。但是如果你需要导出并分享，那么请看：
1. 导出P12文件。进入mac的 “钥匙串访问”，在“我的证书”中选择要导出的Developer ID Application 证书并点击导出。

<img width="867" alt="截屏2024-02-27 上午12 03 29" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/dcb376bb-8ed7-4765-ac50-3b65c80db0fb">

2. 点击存储。

<img width="450" alt="image" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/5833c627-d632-4bad-ba72-735a1832ab0b">

3. 接下来会要求分别输入p12文件的密钥和电脑的密码。依次输入即可。

4. 完成之后，就可以在桌面找到这个p12文件并传输。接收者输入p12文件的密码即可安装对应的证书。

5. 在Xcode 的target-Build Setting中，搜索Code Signing Identity，在Release中选择本地钥匙串中的证书（需要与生成Provisioning  Profile时选择的证书对应）。 

<img width="911" alt="截屏2024-02-27 上午12 07 40" src="https://github.com/stuartofmine/DistributeYourMacApp/assets/25903841/44ab1da0-07ee-47e4-a9c2-8e1414f10760">

恭喜你🎉，你完成了签名这一工作的主要工作！

