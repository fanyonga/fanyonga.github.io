---
layout: post
category: blog
title: 在ubuntu下安装搜狗输入法
description: 如何在ubunru下安装搜狗输入法和配置sublime text3的中文输入
---

##目录

* 更改键盘输入类型
* 下载并安装搜狗输入法
* 登出登入,使用`fcitx-configtool`命令,配置输入法
* 检测并安装GTK库
* 复制sublime_imfix.c文件,并编译成`libsublime-imfix.so`库
* 将库复制到sublime text3相应的位置,并修改sublime text的配置文件

### 实际操作

* 进入System Settings(系统设置)->Language Support(语言支持),设置键盘输入类型为fcitx,实际如图所示:
  
  ![Alt Text](/images/suogouinput_01.png)

然后运行以下命令更新:

	sudo apt-get update

进入system setting（系统设置） -> Keyboard（键盘） -> Shortcuts（快捷方式） -> Typing（打字),修改快捷键

![Alt Text](/images/suogouinput_02.png)

避免与fcitx切换输入法冲突

* 下载[搜狗输入法](http://pinyin.sogou.com/linux/?r=pinyin),并点击deb安装包,安装.

* 安装完成后,`login out`再次`login in` ,即可看到搜狗输入法正在被使用.

* 运行`fcitx-configtool`命令,配置输入法,注意一定要保留这两种输入法(如果删除第二种,则中英文无法正常切换)

 ![Alt Text](/images/suogouinput_03.png)

* 如果没有自动安装选择搜狗输入法,则点击+号,去除Only Show Current Language（只显示当前语言）的选项，然后在搜索框中输入so选择Sougou Pinyin即可.

![Alt Text](/images/suogouinput_04.png)

* 至此,搜狗输入法已安装完毕,再开始配置sublime text 3无法输入中文的问题.
 
* 安装[sublime text 3](https://www.sublimetext.com/3),点击deb安装包安装即可.
 
* 查看并安装 `libgtk2.0-dev`,运行以下命令:

```sh
pkg-config --modversion gtk+ (查看1.2.x版本)
pkg-config --modversion gtk+-2.0 (查看 2.x 版本)
pkg-config --version (查看pkg-config的版本)
pkg-config --list-all grep gtk (查看是否安装了gtk)
```

如果没有,则运行以下命令安装GTK基本库:  

	sudo apt-get install libgtk2.0-dev

 * 保存以下代码到文件sublime_imfix.c

```cpp
/*
 * sublime-imfix.c
 * Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
 * By Cjacker Huang <jianzhong.huang at i-soft.com.cn> *
 *
 * gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
 * LD_PRELOAD=./libsublime-imfix.so sublime_text
 */
#include <gtk/gtk.h>
#include <gdk/gdkx.h>

typedef GdkSegment GdkRegionBox;

struct _GdkRegion
{
    long size;
    long numRects;
    GdkRegionBox *rects;
    GdkRegionBox extents;
};

GtkIMContext *local_context;

void
gdk_region_get_clipbox (const GdkRegion *region,
                        GdkRectangle    *rectangle)
{
    g_return_if_fail (region != NULL);
    g_return_if_fail (rectangle != NULL);

    rectangle->x = region->extents.x1;
    rectangle->y = region->extents.y1;
    rectangle->width = region->extents.x2 - region->extents.x1;
    rectangle->height = region->extents.y2 - region->extents.y1;
    GdkRectangle rect;
    rect.x = rectangle->x;
    rect.y = rectangle->y;
    rect.width = 0;
    rect.height = rectangle->height;

    //The caret width is 2;
    //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
    if (rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
        gtk_im_context_set_cursor_location(local_context, rectangle);
    }
}

//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.

static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
    XEvent *xev = (XEvent *)xevent;

    if (xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
        GdkWindow *win = g_object_get_data(G_OBJECT(im_context), "window");

        if (GDK_IS_WINDOW(win)) {
            gtk_im_context_set_client_window(im_context, win);
        }
    }

    return GDK_FILTER_CONTINUE;
}

void gtk_im_context_set_client_window (GtkIMContext *context,
                                       GdkWindow    *window)
{
    GtkIMContextClass *klass;
    g_return_if_fail (GTK_IS_IM_CONTEXT (context));
    klass = GTK_IM_CONTEXT_GET_CLASS (context);

    if (klass->set_client_window) {
        klass->set_client_window (context, window);
    }

    if (!GDK_IS_WINDOW (window)) {
        return;
    }

    g_object_set_data(G_OBJECT(context), "window", window);
    int width = gdk_window_get_width(window);
    int height = gdk_window_get_height(window);

    if (width != 0 && height != 0) {
        gtk_im_context_focus_in(context);
        local_context = context;
    }

    gdk_window_add_filter (window, event_filter, context);
}
```

运行以下命令,将它编译成`libsublime-imfix.so`库:  

	gcc -shared -o libsublime-imfix.so sublime-imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC

* 复制库`libsublime-imfix.so`到sublime安装位置,一般为`/opt/sublime_text`

运行以下命令:

	sudo cp libsublime-imfix.so /opt/sublime_text/libsublime-imfix.so

修改`/usr/bin/subl`文件，在第一行加入

	export LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so

修改sublime-text.desktop

	sudo vim /usr/share/applications/sublime_text.desktop

参照如下信息进行修改

	[Desktop Entry]
	Version=1.0
	Type=Application
	Name=Sublime Text
	GenericName=Text Editor
	Comment=Sophisticated text editor for code, markup and prose
	Exec=/usr/bin/subl %F        #这里修改执行路径为/usr/bin/subl,subl文件刚才已经修改过，大家应该记得
	Terminal=false
	MimeType=text/plain;        
	Icon=sublime-text
	Categories=TextEditor;Development;
	StartupNotify=true
	Actions=Window;Document;

	[Desktop Action Window]
	Name=New Window
	Exec=/usr/bin/subl -n       #这里修改执行路径为/usr/bin/subl,subl文件刚才已经修改过，大家应该记得
	OnlyShowIn=Unity;

	[Desktop Action Document]
	Name=New File
	Exec=/usr/bin/subl new_file    #这里修改执行路径为/usr/bin/subl,subl文件刚才已经修改过，大家应该记得
	OnlyShowIn=Unity;

修改以上三处代码，保存。Sublime Text 3即可完全正常使用搜狗输入法输入中文.