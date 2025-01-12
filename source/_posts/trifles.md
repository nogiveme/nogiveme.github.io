---
layout: wiki
title: 杂记
wiki: Notes
menu_id: notes
order: 2
---

## 未分类问题

### homebrew下载软件包速度缓慢的解决方案

通过homebrew来下载软件包的时候，下载速度很缓慢。我尝试过更换国内源（中科大源），但是效果仍然很差。最终通过终端代理实现提速，下载速度极快，足够满足需求。

终端代理开启命令和效果因人而异，取决于您的代理客户端和机场速度。

若您是Clash用户，那么可以通过如下操作进行：

1. 首先复制终端代理命令。

<img src="https://cdn.jsdelivr.net/gh/fushunhesir/blog-images@main/imgs/23080701.png" alt="image-20230807143659615" style="zoom:50%;" />

2. 接着在终端执行代理命令即可。



### 华为手机去除APP名称的解决方案

目前市场上绝大部分手机能够修改默认桌面启动器，但是华为禁止该项功能，官方解释为处于安全和隐私考虑。但这对于有定制化需求的用户来说不够合理，比如本人就想将手机上的APP名称给删除，华为系统没有该功能的实现。

因此，下面将记录通过修改华为手机主题文件来实现隐藏应用名称的方法。同时以我目前的了解来讲，很多定制化需求都能通过修改主题文件来实现，读者可自行学习。

1. 首先下载合适的主题。

   **注意**：*请不要在华为商城下载主题*，除开付费缺点外，主题出于版权是不开放源码的，无法进行修改，可以在华为或者荣耀的论坛上下载免费主题。

2. 将下载的hwt文件解压。

   **注意**：hwt文件是华为的主题文件，本质上是zip文件，将其后缀改为.zip即可解压。

3. 解压其中com.huawei.android.launcher文件。

4. 修改其中的theme.xml文件。

5. 修改后，将之前解压的文件倒序压缩，恢复为hwt文件。

6. 在手机上将该hwt文件放在Huawei/Themes文件夹下。

7. 最后在华为的主题中找到新主题，应用即可。

下面是修改代码的示例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<hwthemes>
<!-- 桌面图标投影 -->
<color name="icon_shadow">#00000000</color>
<!-- 一键清理字体颜色 -->
<color name="widget_text_color">#FFFFFFFF</color>
 
<!-- 鸿蒙卡片App文字颜色 -->
<color name="workspace_app_text_color">#00000000</color>（桌面无字前两位色值加00）
<!--桌面图标文字投影颜色-->
<color name="workspace_app_text_shadowColor">#00000000</color>
<!-- 桌面图标文字投影颜色 -->
<color name="icon_shadow">#00000000</color>
<!-- 桌面图标文字阴影开关 -->
<!-- true为显示false为隐藏 -->
<bool name="workspace_app_text_shadow">true</bool>（一键清理统一色值则选择true）
 
<!-- 文件夹内APP文字颜色 -->
<color name="folder_app_text_color">#00000000</color>（文件夹无字前两位色值加00）
 
</hwthemes>
```

**注意**：当您在修改代码时，完全复制本段代码，仅需要将您文件中含text的元素设置颜色为`#00000000`

