
1. yum search java-1.8.0-openjdk
   ```
    Loaded plugins: fastestmirror
    Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
    Loading mirror speeds from cached hostfile
    * base: ftp-srv2.kddilabs.jp
    * centosplus: ftp-srv2.kddilabs.jp
    * epel: nrt.edge.kernel.org
    * extras: ftp-srv2.kddilabs.jp
    * updates: ftp.jaist.ac.jp
    ======================= N/S matched: java-1.8.0-openjdk ========================
    java-1.8.0-openjdk.i686 : OpenJDK Runtime Environment 8
    java-1.8.0-openjdk.x86_64 : OpenJDK 8 Runtime Environment
    java-1.8.0-openjdk-accessibility.i686 : OpenJDK accessibility connector
    java-1.8.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
    java-1.8.0-openjdk-demo.i686 : OpenJDK Demos 8
    java-1.8.0-openjdk-demo.x86_64 : OpenJDK 8 Demos
    java-1.8.0-openjdk-devel.i686 : OpenJDK Development Environment 8
    java-1.8.0-openjdk-devel.x86_64 : OpenJDK 8 Development Environment
    java-1.8.0-openjdk-headless.i686 : OpenJDK Headless Runtime Environment 8
    java-1.8.0-openjdk-headless.x86_64 : OpenJDK 8 Headless Runtime Environment
    java-1.8.0-openjdk-javadoc.noarch : OpenJDK 8 API documentation
    java-1.8.0-openjdk-javadoc-zip.noarch : OpenJDK 8 API documentation compressed
                                        : in a single archive
    java-1.8.0-openjdk-src.i686 : OpenJDK Source Bundle 8
    java-1.8.0-openjdk-src.x86_64 : OpenJDK 8 Source Bundle

   ```

 2. install jdk development


    yum install java-1.8.0-openjdk-devel.x86_64
   
    java -version

3. setting profile
   如果java -version 没有出来结果，
   配置环境变量
  vi /etc/proflie打开配置文件，按insert切换到输入模式添加这两行内容
  export JAVA_HOME=/usr/local/jdk/jdk1.8.0_241
  export PATH=P A T H : PATH:PATH:JAVA_HOME/bin
