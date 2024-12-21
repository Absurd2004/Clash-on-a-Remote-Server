
<center>

### 远程服务器配置clash

</center>

在使用远程服务器的时候，很多时候只能连到校园网，当需要使用pip，huggingface，或者一些api的时候就会很慢或者连接不上。

这个时候一般有两种方法，让远程服务器和本地共享代理端口，或者在远程重新配置vpn。当程序需要后台长时间运行的时候，和本地共享代理端口可能很容易断开，不稳定，因此这里介绍直接在远程配置clash的方法。由于

#### 配置步骤

1. **创建 clash 文件夹**
   执行以下命令在用户目录下创建并进入 `clash` 文件夹：
    ```bash
    mkdir clash && cd clash
    ```
2. **下载Clash文件**
   可以在https://github.com/doreamon-design/clash/releases 或者https://github.com/doreamon-design/clash/releases 等其他网址下载与自己系统相符的安装包，比如clash-linux-amd64-v1.18.0.gz。再上传到服务器刚才创建的clash文件夹目录下

   执行以下命令进行解压和重命名
   ```bash
   gzip -d clash-linux-amd64-v1.18.0.gz && mv clash-linux-amd64-v1.18.0 clash
   ```
   对于tar.gz压缩文件，解压命令使用
   ```bash
   tar -xvzf 文件名.tar.gz
   ```
3. **启动Clash**
   在clash目录下执行
   ```bash
   ./clash -d .
   ```
   如果显示权限问题，先执行
   ```bash
   chmod +x clash
   ```
   此时由于目录下没有config文件和Contry.mmdb文件，会显示正在下载这两个文件，等下载完成后，会发现文件夹里有三个文件cache.db、config.yaml和Country.mmdb。这时候使用Ctrl+C退出Clash程序。

4. **配置config文件**
   
   在本地的clash配置文件夹中，如果用的是学校的vpn找到WallessPKU.yaml这个文件。把其中内容复制到远程服务器的config.yaml里。对于clash里使用自己的托管配置的，也只要找到对应的yaml文件即可。在配置文件中，默认的一般是mixed-port: 7890，external-controller: 127.0.0.1:9090这两个端口。

   如果想临时使用代理，直接在命令行输入
   ```bash
   export http_proxy=http://127.0.0.1:7890
   export https_proxy=http://127.0.0.1:7890
   ```
   如果想长时间使用，需要更改bashrc文件
   ```bash
   vim ~/.bashrc
   export http_proxy=http://127.0.0.1:7890
   export https_proxy=http://127.0.0.1:7890
   source ~/.bashrc
   ```
   之后再进行步骤3，启动Clash

5. **端口占用问题**
   在远程服务器上这两个端口很可能被其他人配置clash的时候使用过，因此在启动clash的时候可能会报错
   ```bash
   ERRO[0000] External controller listen error: listen tcp 0.0.0.0:9090: bind: address already in use 
   ERRO[0000] create addr 127.0.0.1:7890 tcp listener error: listen tcp 127.0.0.1:7890: bind: address already in use
   ```
   这种报错显示端口已经被占据，这个时候只要更改配置文件里的这两个选项即可，选取未被使用的两个端口。比如mixed-port: 7898，external-controller: 127.0.0.1:9092。其中port端口是 Clash 提供的 HTTP/HTTPS 代理端口。所有需要通过代理访问网络的流量都需要发送到这个端口。external-controller用于与 Clash 的外部控制器通信。通过这个端口，你可以使用 RESTful API 或其他工具（例如 Clash Dashboard）来管理和控制 Clash 的运行状态，例如切换代理模式、查询日志等。

   重新运行clash，这个时候如果没有刚才的报错就说明运行成功啦。想要进一步确认clash连接的节点的情况，比如是否能连接上google，可以使用命令(均以7890和9090为例，如果有更改请切换成自己设置的端口)
   ```bash
   curl -x http://127.0.0.1:7890 https://www.google.com
   ```
   如果想知道节点定位的地理位置，可以使用命令
   ```bash
   curl -x http://127.0.0.1:7890 https://api.ipify.org
   ```
6. **更改代理设置**
   通过Clash Dashboard可以很方便的更改代理设置，比如节点和连接规则。由于external-controller: 127.0.0.1:9090是在远程服务器上设置的，为了让本地的端口和远程服务器的端口能够接上，需要先在本地终端输入命令
   ```bash
   ssh -L 9090:127.0.0.1:9090 user@<server-ip>
   ```
   使得远程的9090和本地的9090端口接上

   之后访问https://clash.razord.top/#/proxies ，可以更改代理设置。
   

7. **更改vscode的代理设置**
   在正确启动clash之后，代码中可以使用requests 库设置代理，比如使用gemini的api的时候，可以用
   ```python
   # Configure proxies for requests
    proxies = {
        'http': 'http://127.0.0.1:7890',
        'https': 'http://127.0.0.1:7890',
    }

    # Overwrite requests' session with the proxy settings
    session = requests.Session()
    session.proxies = proxies
    genai._session = session
   ```
   如果想要让远程服务器的copilot这些扩展也可以使用代理端口，需要更改vscode的代理设置。

   找到vscode左下角的齿轮按钮（Manage/管理），然后选择 Settings（设置）。在设置的搜索栏中输入 “proxy”。找到 Http: Proxy ，在这里输入你的代理地址（如 http://127.0.0.1:7890）。之后应该就可以让copilot连接上代理了。

