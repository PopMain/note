

http://gityuan.com/2016/08/13/android-os-env/

#### repo 安装

>mkdir ~/bin PATH=~/bin:$PATH
>
> curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
>
>chmod a+x ~/bin/repo

#### Repo init

>repo init -u https://android.googlesource.com/platform/manifest 
>
>repo init -u https://android.googlesource.com/platform/manifest -b android-7.0.0_r1

#### repo sync

> repo sync
>
> repo sync -c -j4 
>
> repo sync platform/frameworks/base -c -j4

#### fatal: Cannot get https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/clone.bundle

fatal: error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1056)

   打开 ~/bin/下的repo文件，vim进入。添加：

   import ssl

   ssl._create_default_https_context = ssl._create_unverified_context

