■ はじめに
このプログラムはTwitter4CをUEFIに移植したものです。
現在、IA32版UEFIで英語のツイートが動作する事を確認しています。
使い方など質問は@syuu1228までお願い致します。
ソースコードのライセンスは、fork元のライセンスに準じます。

■ EDK IIによる開発環境構築とOVMFの実行環境構築方法を含む、Twitter4C for UEFIの使い方

以下のコマンドを順に実行：
sudo apt-get install build-essential subversion uuid-dev iasl gcc-4.4 gcc-4.4-multilib kvm
mkdir ~/src
cd ~/src
svn co https://edk2.svn.sourceforge.net/svnroot/edk2/trunk/edk2
make -C edk2/BaseTools
cd ~/src/edk2
export EDK_TOOLS_PATH=$HOME/src/edk2/BaseTools
 . edksetup.sh BaseTools


Conf/tools_def.txtを以下のdiffの通りに変更：
--- tools_def.txt.orig  2013-02-16 17:33:45.194575221 +0900
+++ tools_def.txt	2013-02-16 17:34:21.289816161 +0900
@@ -2693,15 +2693,15 @@
 # GCC44 IA32 definitions
 ##################
 *_GCC44_IA32_OBJCOPY_PATH         = DEF(GCC44_IA32_PREFIX)objcopy
-*_GCC44_IA32_CC_PATH              = DEF(GCC44_IA32_PREFIX)gcc
+*_GCC44_IA32_CC_PATH              = DEF(GCC44_IA32_PREFIX)gcc-4.4
 *_GCC44_IA32_SLINK_PATH           = DEF(GCC44_IA32_PREFIX)ar
 *_GCC44_IA32_DLINK_PATH           = DEF(GCC44_IA32_PREFIX)ld
 *_GCC44_IA32_ASLDLINK_PATH        = DEF(GCC44_IA32_PREFIX)ld
-*_GCC44_IA32_ASM_PATH             = DEF(GCC44_IA32_PREFIX)gcc
-*_GCC44_IA32_PP_PATH              = DEF(GCC44_IA32_PREFIX)gcc
-*_GCC44_IA32_VFRPP_PATH           = DEF(GCC44_IA32_PREFIX)gcc
-*_GCC44_IA32_ASLCC_PATH           = DEF(GCC44_IA32_PREFIX)gcc
-*_GCC44_IA32_ASLPP_PATH           = DEF(GCC44_IA32_PREFIX)gcc
+*_GCC44_IA32_ASM_PATH             = DEF(GCC44_IA32_PREFIX)gcc-4.4
+*_GCC44_IA32_PP_PATH              = DEF(GCC44_IA32_PREFIX)gcc-4.4
+*_GCC44_IA32_VFRPP_PATH           = DEF(GCC44_IA32_PREFIX)gcc-4.4
+*_GCC44_IA32_ASLCC_PATH           = DEF(GCC44_IA32_PREFIX)gcc-4.4
+*_GCC44_IA32_ASLPP_PATH           = DEF(GCC44_IA32_PREFIX)gcc-4.4
 *_GCC44_IA32_RC_PATH              = DEF(GCC44_IA32_PREFIX)objcopy
 
 *_GCC44_IA32_ASLCC_FLAGS          = DEF(GCC_ASLCC_FLAGS) -m32
 
 
 
Conf/target.txtを以下のdiffの通りに変更：
--- target.txt.orig  2013-02-16 17:33:41.602650613 +0900
+++ target.txt	2013-02-16 17:35:44.996059553 +0900
@@ -23,7 +23,7 @@
 #                                               build. This line is required if and only if the current
 #                                               working directory does not contain one or more description
 #                                               files.
-ACTIVE_PLATFORM       = Nt32Pkg/Nt32Pkg.dsc
+ACTIVE_PLATFORM       = AppPkg/AppPkg.dsc
 
 #  TARGET                List       Optional    Zero or more of the following: DEBUG, RELEASE, NOOPT
 #                                               UserDefined; separated by a space character.
@@ -56,7 +56,7 @@
 #  TAGNAME               List      Optional   Specify the name(s) of the tools_def.txt TagName to use.
 #                                             If not specified, all applicable TagName tools will be
 #                                             used for the build.  The list uses space character separation.
-TOOL_CHAIN_TAG        = MYTOOLS
+TOOL_CHAIN_TAG        = GCC44
 
 # MAX_CONCURRENT_THREAD_NUMBER  NUMBER  Optional  The number of concurrent threads. Recommend to set this
 #                                                 value to one more than the number of your compurter
 
 
 以下のコマンドを順に実行：
cd AppPkg/Applications/
git clone git://github.com/syuu1228/Twitter4C_UEFI.git


AppPkg/AppPkg.dscを以下のdiffの通りに変更：
--- AppPkg/AppPkg.dsc  2013-02-16 17:23:26.891551597 +0900
+++ ../edk2.bak/AppPkg/AppPkg.dsc	2013-02-16 17:11:06.575113028 +0900
@@ -107,6 +107,7 @@
 
 #### After extracting the Python distribution, un-comment the following line to build Python.
 #  AppPkg/Applications/Python/PythonCore.inf
+  AppPkg/Applications/Twitter4C_UEFI/Twitter4C_UEFI.inf
 
 
 ##############################################################################
 
 
以下のコマンドを実行してTwitter4Cをコンパイル：
build


OvmfPkg/build.shを以下のdiffの通りに変更：
@@ -90,6 +90,7 @@
     esac
 esac
 
+TARGET_TOOLS=GCC44
 #
 # Scan command line to override defaults
 #
 
 
http://downloadcenter.intel.com/Detail_Desc.aspx?agr=Y&DwnldID=17515&lang=eng
からPROEFI_v13_5.exeをダウンロード


以下のコマンドを実行してセットアップを実行：
wine PROEFI_v13_5.exe


以下のコマンドを実行してOVMFをビルド：
cp -a ~/.wine/drive_c/Intel13.5 ~/src/edk2/Intel3.5
OvmfPkg/build.sh -a IA32 -D NETWORK_ENABLE -DFD_SIZE_2MB
OvmfPkg/build.sh -a IA32 -D NETWORK_ENABLE -DFD_SIZE_2MB qemu


qemu上でOVMFが立ち上がるのを確認してCTRL+Cで終了


以下の内容を~/src/edk2/bridge.shとして作成：
#!/bin/sh
brctl addbr br0
brctl addif br0 eth0
ifconfig eth0 0.0.0.0 up
ifconfig br0 up
dhclient br0


以下の内容を~/src/edk2/qemu-ifup.shとして作成：
#!/bin/sh
sudo /sbin/ifconfig $1 0.0.0.0 promisc up
sudo /sbin/brctl addif br0 $1


以下の内容を~/src/edk2/qemu-ovmf.shとして作成：
#!/bin/sh
qemu-system-i386 -L Build/OvmfIa32/DEBUG_GCC44/QEMU -hda fat:Build/AppPkg/DEBUG_GCC44/IA32 -net nic,model=e1000 -net tap,ifname=tap0,script=qemu-ifup.sh -debugcon file:debugcon.log -global isa-debugcon.iobase=0x402 -nographic


以下のコマンドを実行：
cd ~/
git clone git://github.com/Plemling138/Twitter4C.git
cd Twitter4C


Makefileを以下のdiffの通りに変更：
diff --git a/Makefile b/Makefile
index 1d2f888..407ca8f 100644
--- a/Makefile
+++ b/Makefile
@@ -1,5 +1,5 @@
 CC     =       gcc
-CFLAGS =       -Os -Wall -I/usr/include
+CFLAGS =       -Os -Wall -I/usr/include -D__INTEL__ -DSIZEOF_LONG=8  -D__x86_64
 DEST   =       /usr/bin
 LDFLAGS        =       -L/usr/lib
 OBJS   =       main.o urlenc.o session.o extract.o


以下のコマンドを実行：
make
./tweet test


OAuth用URLへアクセスしてPINコードを取得、入力
Twitterへ”test"が投稿され、access_token.txtが生成される
これをOVMFから使えるようにコピーする


以下のコマンドを実行：
cp access_token.txt ~/src/edk2/Build/AppPkg/DEBUG_GCC44/IA32/
cd ~/src/edk2
chmod a+rx *.sh
sudo ./qemu-ovmf.sh


UEFI Shell上で以下のコマンドを実行：
ifconfig -s eth0 dhcp
tweet.efi test2


UEFI上のtweet.efiからTwitterへ"test2"が投稿される
