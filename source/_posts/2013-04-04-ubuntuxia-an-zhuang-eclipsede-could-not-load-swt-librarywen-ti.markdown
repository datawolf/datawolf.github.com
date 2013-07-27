---
layout: post
title: "Ubuntu下安装eclipse的Could not load SWT library问题"
date: 2013-04-04 15:16
comments: true
categories: [ubuntu, eclipse]
---
## Description

版本：Ubuntu 12.04, Eclipse3.7.2-1, Oracle Java 1.6
 
``` sh
$ sudo apt-get install eclipse
```

## Question

安装后打开eclipse，提示出错
``` plain
An error has occurred. See the log file /home/datawolf/.eclipse/org.eclipse.platform_3.7.0_155965261/configuration/1365058671830.log.
```
打开log文件，看到下面的错误
``` plain
!SESSION 2013-04-04 14:57:51.666 -----------------------------------------------
eclipse.buildId=I20110613-1736
java.version=1.6.0_37
java.vendor=Sun Microsystems Inc.
BootLoader constants: OS=linux, ARCH=x86_64, WS=gtk, NL=en_US
Command-line arguments:  -os linux -ws gtk -arch x86_64

!ENTRY org.eclipse.osgi 4 0 2013-04-04 14:57:53.033
!MESSAGE Application error
!STACK 1
java.lang.UnsatisfiedLinkError: Could not load SWT library. Reasons: 
	no swt-gtk-3740 in java.library.path
	no swt-gtk in java.library.path
	Can't load library: /home/datawolf/.swt/lib/linux/x86_64/libswt-gtk-3740.so
	Can't load library: /home/datawolf/.swt/lib/linux/x86_64/libswt-gtk.so

	at org.eclipse.swt.internal.Library.loadLibrary(Library.java:285)
	at org.eclipse.swt.internal.Library.loadLibrary(Library.java:194)
	at org.eclipse.swt.internal.C.<clinit>(C.java:21)
	at org.eclipse.swt.internal.Converter.wcsToMbcs(Converter.java:63)
	at org.eclipse.swt.internal.Converter.wcsToMbcs(Converter.java:54)
	at org.eclipse.swt.widgets.Display.<clinit>(Display.java:132)
	at org.eclipse.ui.internal.Workbench.createDisplay(Workbench.java:695)
	at org.eclipse.ui.PlatformUI.createDisplay(PlatformUI.java:161)
	at org.eclipse.ui.internal.ide.application.IDEApplication.createDisplay(IDEApplication.java:153)
	at org.eclipse.ui.internal.ide.application.IDEApplication.start(IDEApplication.java:95)
	at org.eclipse.equinox.internal.app.EclipseAppHandle.run(EclipseAppHandle.java:196)
	at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.runApplication(EclipseAppLauncher.java:110)
	at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.start(EclipseAppLauncher.java:79)
	at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:344)
	at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:179)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
	at java.lang.reflect.Method.invoke(Method.java:597)
	at org.eclipse.equinox.launcher.Main.invokeFramework(Main.java:622)
	at org.eclipse.equinox.launcher.Main.basicRun(Main.java:577)
	at org.eclipse.equinox.launcher.Main.run(Main.java:1410)
	at org.eclipse.equinox.launcher.Main.main(Main.java:1386)
```

## Solution

把相关文件拷贝到`~/.swt/lib/linux/x86_64`下即可 

``` sh
$ cp /usr/lib/jni/libswt-*3740.so ~/.swt/lib/linux/x86_64
```
