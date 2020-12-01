<!--markdown-->1. 下载源码，并解压
    在这里下载: http://www.codeblocks.org/downloads/25
    下载解压后进入源码文件夹

2. 安装依赖
  ```
  apt install build-essential libgtk-3-dev libglib2.0-dev zlib1g-dev libgamin-dev libhunspell-dev libtool libwxgtk3.0-gtk3-dev
  ```
3. 编译安装
  ```
  ./bootstrap
  ./configure --with-contrib-plugins=all,-NassiShneiderman,-spellchecker
  make
  make install
  ldconfig
  ```
完毕	