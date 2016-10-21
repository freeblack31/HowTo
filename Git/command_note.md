##build code
/mtkoss/git/mtk_repo init -u http://gerrit.mediatek.inc:8080/alps/platform/manifest -b alps-mp-m0.mp22 -m manifest-hq.xml
**要加 -g default,libcam , 因為有些repo是用binary release的方式, 這樣會sync不到**
/mtkoss/git/mtk_repo init -u http://gerrit.mediatek.inc:8080/alps/platform/manifest -b alps-mp-n0.mp5 -g default,libcam -m manifest-hq.xml

**link CQ**
ln -s /mtkoss/git/hooks/wsd/prepare-commit-msg .repo/repo/hooks/

git config --global commit.template /mtkeda/CI/gitconfig/.gitcommitmsg

**launch**
source build/envsetup.sh

mosesq "/mtkoss/git/mtk_repo sync -j24 --no-tags -c"

lunch|grep k57v1_64_om

mosesq 'make -k -j24 2>&1 | tee ./build.log'

##logcat##
adb logcat -c | adb logcat -v threadtime > b.log

##git checkout and update branch to alps-trunk-n0.tk##
git checkout -b up-to-date remotes/m/alps-trunk-n0.tk

**強制build system.img**
mosesq make snod
mosesq make -j24 -k systemimage 2>&1 | tee b.log

**只build某個module**
mosesq 'make libcam.camshot -j24 -k 2>&1 | tee build_camshot.log'

##backtrace
**32 bits**
arm-linux-androideabi-addr2line -e out/target/product/k57v1_64_om/symbols/system/lib/libcam3_hwnode.so 
arm-linux-androideabi-addr2line -e out_k99v1_64_tee_userdebug/target/product/k99v1_64_tee/symbols/system/vendor/lib/libmtkcam_pipeline.so  00027e8d

**64 bits**
aarch64-linux-android-addr2line -e out/target/product/k57v1_64_om/symbols/system/lib64/libcam3_hwnode.so 
aarch64-linux-android-addr2line -e out/target/product/k57v1_64_om/symbols/system/lib64/vendor/libcam_extension.so


##clear untrack file
git clean -f -d

##抓到某一個commit的版本
 1.git fetch < url > < sha1 >

> Note:
> < url >  -> use the remote_name by executing "git remote". You may get "mediatek" or "origin"  for use.
> < sha1 > -> complete commit id(40 bit)
> Ex.
> git fetch origin f4ad37e08074e16577eb4544c90ea3a7365714e3

 2.git checkout -b new_branch < sha1 >

> Ex.
> git checkout -b test_branch f4ad37e08074e16577eb4544c90ea3a7365714e3

##把HEAD移到某一筆commit
git checkout xxxx

##Revert
取消已經暫存的檔案: 把已經add的檔案to unstage

`git reset HEAD`

取消對檔案的修改: 這指令有點危險

`git checkout -- filename`

已經commit但想要修改

`git reset --soft HEAD^` 這會退回到尚未commit的狀態

強制恢復到上一個版本
`git reset --hard HEAD^`

##Shared codebase
不知道可以查看help
`mtk_sc help`

列出branch
`mtk_sc list -b alps-fpb-m0.mp9`

會開始抓
`mtk_sc init -b alps-fpb-m0.mp9 -v alps-fpb-m0.mp9-of.p50 -m eng`

設定
`Export OUT\_DIR=`out_???

`Lunch|grep `k97v1_camera3…


##Tool
diff tool: Meld
 - 設定meld為主要的difftool
git config --global diff.tool meld
git config --global difftool.prompt false

##git fetch + merge = git pull##
`git fetch mediatek`
```
[mtk08997@mtkslt207 mtkcam]$git fetch mediatek
remote: Counting objects: 19780, done
remote: Finding sources: 100% (218/218)
remote: Total 218 (delta 178), reused 218 (delta 178)
Receiving objects: 100% (218/218), 63.48 KiB | 0 bytes/s, done.
Resolving deltas: 100% (178/178), completed with 152 local objects.
From http://gerrit.mediatek.inc:8080/alps/vendor/mediatek/proprietary/hardware/mtkcam
   a82cf63..0dfab50  alps-trunk-m1.basic-swo2 -> mediatek/alps-trunk-m1.basic-swo2
```
`git merge mediatek/alps-trunk-m1.basic-swo2`

`git checkout -t mediatek/alps-trunk-m1.basic-swo2 -b aa`

##git push
`git push mediatek HEAD:refs/for/refs/heads/alps-trunk-m1.basic-swo2`

##使用公用帳號上code
在下git commit 时采用
`git commit --author "swintegrator <swintegrator@mediatek.com>"`
or 用自己帐号commit 后，再使用
`git commit --amend --author "swintegrator <swintegrator@mediatek.com>"`
即可。


##哪一個才是我的遠端?##
`repo info .`
> [mtk08997@mtkslt207 mtkcam]$repo info . 
> Manifest branch: alps-trunk-m1.basic-swo2
> Manifest merge branch: refs/heads/alps-trunk-m1.tk-swo2
> Manifest groups: all,-notdefault

`git symbolic-ref refs/remotes/m/alps-trunk-m1.tk-swo2`
> refs/remotes/mediatek/alps-trunk-m1.basic-swo2


查看HEAD與遠端之間的關係:
`git lg HEAD mediatek/alps-trunk-m1.basic-swo2  --pretty=short`
或
`git lg`


##.gitconfig##

>  41 [alias]
>  42 lg = log --graph --decorate --abbrev-commit
>  43 lga = log --decorate --abbrev-commit --graph --all
>  44 lga1 = log --decorate --abbrev-commit --graph --all --oneline
>  45 short = --pretty=short
>  46 st = status
>  47 co = checkout
>  48 cmt = commit
>  49 cmta = commit --amend
>  50 cmtaau = commit --amend --author "swintegrator <swintegrator@mediatek.com>"
>  51 cmtau = commit --author "swintegrator <swintegrator@mediatek.com>"


###查詢遠端分支###
`git branch -r`

###diff##
比對「工作目錄」與「索引」之間的差異
`git diff`
針對已經add.的檔案
`git diff --cached`
`git diff --staged`
比較【最新版的前一版】與【最新版】之間的差異
`git diff HEAD^ HEAD`

`adb shell getprop | findstr "jpeg.rotation.enable"列出包含""的属性`

#####dump memory
`adb shell cat /sys/kernel/debug/ion/ion_mm_heap`

#####把out移到其他目錄下站存
`moses-list-disk`
`moses-mv-disk   src(/proj/mtk08997/...)   dst(/proj/mtk08997/_out自訂)`
`moses-mv-disk   dst(/proj/mtk08997/_out自訂)   src(/proj/mtk08997/...) `

####Others command####

**消紅屏**
adb shell aee -c dal

**看記憶體**
adb shell cat proc/meminfo

**install apk**
adb install -r -d Camera.apk

**close ninja**
export USE_NINJA=false

**dump metadata**
adb shell dumpsys media.camera -v 1
adb shell dumpsys media.camera -v 2

**切換v1/v3**
adb shell setprop debug.camera.force_device 1
adb shell setprop debug.camera.force_device 3

**切換api1/2**
adb shell setprop mtk.camera.app.api.version 1
adb shell setprop mtk.camera.app.api.version 2

**open mfll(flash need close),hal3需要切到night mode**
adb shell "setprop mediatek.mfll.force 1 && setprop mediatek.mfll.capture_num 4 && setprop mediatek.mfll.blend_num 4 && setprop mediatek.mfll.full_size_mc 0"

**not use p2featurenode**
adb shell setprop debug.force.p2featurenode 8

**查看手機sensor**
adb shell cat /proc/driver/camera_info

**讓它可以重複push so 以及remount不會fail**
adb disable-verity

###debug log###
    adb shell setprop debug.camera.log 1
	adb shell setprop debug.camera.log.p2node 1
    adb shell setprop debug.camera.log.AppStreamMgr 2
    adb shell setprop log.tag.SurfaceViewTestCase V
    adb shell setprop log.tag.Camera2AndroidTestCase  V
    adb shell setprop log.tag.CameraTestUtils V
    adb shell setprop log.tag.SurfaceViewTestCase V
    adb shell setprop log.tag.CaptureRequestTest V
    adb shell setprop log.tag.CameraDeviceTest V
    adb shell setprop log.tag.PerformanceTest V
    adb shell setprop log.tag.BurstCaptureTest V
    adb shell setprop log.tag.StaticMetadata V
    adb shell setprop log.tag.MultiViewTest V
    adb shell setprop log.tag.RobustnessTest V
    adb shell setprop log.tag.SurfaceViewPreviewTest V
    adb shell setprop log.tag.StillCaptureTest V
    adb shell setprop debug.camera.log.hal3a 1
    adb shell setprop debug.isp_mgr.enable.LSC 1
    adb shell setprop debug.hal3av3.log 259
    adb shell setprop debug.3a.log 1
    adb shell setprop debug.hal3av3.log 256
    adb shell setprop debug.isp_mgr.enable 1
    adb shell setprop debug.paramctrl.enable 1

##關log## 
adb shell setprop persist.log.tag x (reboot 后还有效果)
X可以为: V, D, I, W, E, A (all), S (silent)