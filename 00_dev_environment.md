# 开发环境

Ionic 可以用来构建夸平台应用。他利用 Cordova 抽象了 iOS、Android 底层的原生 APIs的特性，使得一套代码可以生成两个移动平台下的 App；又借助 Electron 使得可以创建 PC、Linux、MacOS的桌面应用。

Ionic的安装需要 NodeJS/NPM, 本身是由 TypeScript 构建，故需要掌握 TypeScript 编程语言。Android App、Linux 桌面应用可在 Linux 下编写、调试与构建；iOS、MacOS App则需要在 MacOS 下构建。

开发环境涉及 Node/NPM、Ionic、Java JDK/JRE、Android Studio及Gradle的安装。

## Node/NPM 的安装

使用 [nvm-sh/nvm](https://github.com/nvm-sh/nvm) 可以在不需要 `root` 权限的情况下，就能获得 `node/npm`。

1. 使用该项目提供的安装脚本安装/更新 `nvm`

    ```console
    $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
    ```

    ```console
    $ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
    ```

2. 创建/修改`~/.npmrc`文件，使得其为以下内容:

    ```config
    registry=https://mirrors.huaweicloud.com/repository/npm/
    always-auth=false
    disturl=https://mirrors.huaweicloud.com/nodejs
    SASS_BINARY_SITE=https://mirrors.huaweicloud.com/node-sass
    PHANTOMJS_CDNURL=https://mirrors.huaweicloud.com/phantomjs
    CHROMEDRIVER_CDNURL=https://mirrors.huaweicloud.com/chromedriver
    OPERADRIVER_CDNURL=https://mirrors.huaweicloud.com/operadriver
    ELECTRON_MIRROR=https://mirrors.huaweicloud.com/electron/
    PYTHON_MIRROR=https://mirrors.huaweicloud.com/python
    ## 这是因为在华为 NPM 源上找不到 ionic 
    @ionic:registry=https://registry.npmjs.org/
    ```

3. 使用 `nvm` 安装最新的长期支持版（`LTS`）的 `node`

    - 使用`nvm ls-remote`可以查看可供安装的`node` 版本
        
        ```console
        $nvm ls-remote
        ...
                 v12.9.0
                 v12.9.1
                v12.10.0
                v12.11.0
                v12.11.1
                v12.12.0
        ->      v12.13.0   (Latest LTS: Erbium)
                 v13.0.0
                 v13.0.1
                 v13.1.0
        ```

    - 使用`nvm install --lts` 安装当前的 LTS 版本

        ```console
        $ which npm
        /home/peng/.nvm/versions/node/v12.13.0/bin/npm
        $ which node
        /home/peng/.nvm/versions/node/v12.13.0/bin/node
        ```

## 安装 Ionic/Cordova

使用 `npm i -g ionic cordova` 全局安装 Ionic，得到：

    ```console
    $ which ionic
    /home/peng/.nvm/versions/node/v12.13.0/bin/ionic
    $ which cordova
    /home/peng/.nvm/versions/node/v12.13.0/bin/cordova
    ```

## 安装 Java JDK/JRE

__网上的使用 `ppa:webupd8team/java` 方式已不可用，得用下面的手动安装方法。__

前往 [Java SE Development Kit 8 - oracle.com](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 网页查看最新的 Java JDK 8 二进制包版本，复制文件名（这里是`jdk-8u231-linux-x64.tar.gz`），然后Google下载该文件。

1. 将下载到的`tar.gz`压缩包解压到`/opt`

    ```console
    sudo tar -zxvf jdk-8u231-linux-x64.tar.gz -C /opt
    ```

2. 将以下内容添加到 `~/.bashrc` 最后

    ```bash
    #set oracle jdk environment
    export JAVA_HOME=/opt/jdk1.8.0_231 ## 这里要注意目录要换成自己解压的jdk 目录
    export JRE_HOME=${JAVA_HOME}/jre  
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
    export PATH=${JAVA_HOME}/bin:$PATH
    ```

    _切勿添加其他环境变量，否则会导致运行`$ gradle -v`、`$ sdkmanager --update`等命令时出现`找不到或无法加载主类：java.se.ee`错误_

3. 使环境变量马上生效：

    ```console
    $ source ~/.bashrc
    ```

4. __运行下面的命令，往系统中注册该JDK__：

    ```console
    $ sudo update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_231/bin/java 300
    ```

5. 运行下面的命令，配置系统使用的 JDK:

    ```console
    $ sudo update-alternatives --config java
    ```

6. 检查`java`、`javac`版本：

    ```console
    $ java -version
    java version "1.8.0_231"
    Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
    $ javac -version
    javac 1.8.0_231
    ```

## 安装 Android Studio

前往 [developer.android.com/studio](https://developer.android.com/studio)下载 Android Studio 的压缩包。

1. 解压到 `/opt`:

    ```console
    $ sudo tar -zxvf android-studio-ide-191.5977832-linux.tar.gz /opt
    ```

2. 修改`android-studio/`权限

    ```console
    $ cd /opt
    $ sudo chown -R peng:peng android-studio/
    ```

运行 `/opt/android-studio/bin/studio.sh`可启动 Android Studio, 然后可以创建桌面快捷方式。

## 安装 Android SDK

Android SDK 是在 Android Studio 中安装的。


__以上两步完成后，要在 `~/.bashrc` 中加入 `ANDROID_SDK_ROOT` 等环境变量__：


    ```bash
    # Export the Android SDK path
    export ANDROID_SDK_ROOT=$HOME/Android/Sdk
    export PATH=$PATH:$ANDROID_SDK_ROOT/tools/bin
    export PATH=$PATH:$ANDROID_SDK_ROOT/platform-tools
    ```

## 安装 Gradle

Gradle 是 Android Studio 使用的构建工具。其安装是借助 [SDKMAN!](https://sdkman.io/)完成的。

1. 安装 SDKMAN!

    ```console
    ## 下载 SDKMAN! 的安装脚本
    $ curl -s "https://get.sdkman.io" | bash
    ## 安装 SDKMAN!
    $ source "$HOME/.sdkman/bin/sdkman-init.sh"
    $ sdk version

    SDKMAN 5.7.4+362
    ```

2. 安装 Gradle

    ```console
    $ sdk list gradle
    ================================================================================
    Available Gradle Versions
    ================================================================================
         6.0-rc-3            4.10.1              3.4.1               2.2            
         6.0-rc-2            4.10                3.4                 2.1            
         6.0-rc-1            4.9                 3.3                 2.0            
     > * 5.6.4               4.8.1               3.2.1               1.12           
         5.6.3               4.8                 3.2                 1.11           
         5.6.2               4.7                 3.1                 1.10           
         5.6.1               4.6                 3.0                 1.9            
         5.6                 4.5.1               2.14.1              1.8            
         5.5.1               4.5                 2.14                1.7            
         5.5                 4.4.1               2.13                1.6            
         5.4.1               4.4                 2.12                1.5            
         5.4                 4.3.1               2.11                1.4            
         5.3.1               4.3                 2.10                1.3            
         5.3                 4.2.1               2.9                 1.2            
         5.2.1               4.2                 2.8                 1.1            
         5.2                 4.1                 2.7                 1.0            
         5.1.1               4.0.2               2.6                 0.9.2          
         5.1                 4.0.1               2.5                 0.9.1          
         5.0                 4.0                 2.4                 0.9            
       * 4.10.3              3.5.1               2.3                 0.8            
         4.10.2              3.5                 2.2.1               0.7            

    ================================================================================
    + - local version
    * - installed
    > - currently in use
    ================================================================================
    ```

3. 安装最新的稳定版（`5.6.4`）

    ```console
    $ sdk install gradle 5.6.4
    $ which gradle
    /home/peng/.sdkman/candidates/gradle/5.6.4/bin/gradle
    $ gradle -v

    ------------------------------------------------------------
    Gradle 5.6.4
    ------------------------------------------------------------

    Build time:   2019-11-01 20:42:00 UTC
    Revision:     dd870424f9bd8e195d614dc14bb140f43c22da98

    Kotlin:       1.3.41
    Groovy:       2.5.4
    Ant:          Apache Ant(TM) version 1.9.14 compiled on March 12 2019
    JVM:          1.8.0_231 (Oracle Corporation 25.231-b11)
    OS:           Linux 5.0.0-32-generic amd64

    ```


_大功告成!_





