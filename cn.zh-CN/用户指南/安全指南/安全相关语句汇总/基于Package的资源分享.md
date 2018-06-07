# 基于Package的资源分享 {#concept_kyz_531_wdb .concept}

## 分享资源 {#section_ofl_bk1_wdb .section}

|语句|说明|
|:-|:-|
|create package `<pkgName>`|创建一个Package|
|delete package `<pkgName>`|删除一个Package|
|add `<objType>` `<objName>` to package `<pkgName>` \[with privileges privs\]|向Package中添加需要分享的资源|
|remove `<objType>` `<objName>` from package `<pkgName>`|从Package中删除已分享的资源|
|allow prOject `<prjName>` to install package `<pkgName>` \[using label `<num>`\]|许可某个项目空间使用您的Package|
|disallow project `<prjName>` to install package `<pkgName>`|禁止某个项目空间使用您的Package|

## 使用资源 {#section_sg2_dk1_wdb .section}

|语句|说明|
|:-|:-|
|install package `<pkgName>`|安装Package|
|uninstall package `<pkgName>`|卸载Package|

## 查看Package {#section_fyp_2k1_wdb .section}

|语句|说明|
|:-|:-|
|show packages|列出所有创建和安装的Packages|
|describe package `<pkgName>`|查看package的详细信息|

