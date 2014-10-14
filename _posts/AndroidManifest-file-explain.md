title: AndroidManifest file explain
date: 2014-10-03 17:04:21
tags: android
categories:
- Android

---

> 参考： [Android学习笔记之AndroidManifest.xml文件解析](http://www.cnblogs.com/pilang/archive/2011/04/20/2022932.html)

`AndroidManifest.xml`是`Android`应用程序中必须的文件。它描述了应用代码包中的全局数据，在`application`的根目录下，包括配置了应用的组件（activity\service等），同时还配置了启动位置、服务等。

{% code AndroidManifest.xml注释版本 %} 
    
    <?xml version="1.0" encoding="utf-8"?>
    <!--
        manifest 根节点，描述应用的具体信息，包括包名等
            package:            应用内主程序包的包名.
            sharedUserId:       数据权限
                                    因为默认情况下，Android给每个APK分配一个唯一的UserID，
                                    所以是默认禁止不同APK访问共享数据的。若要共享数据，
                                    第一可以采用Share Preference方法，
                                    第二种就可以采用sharedUserId了，
                                    将不同APK的sharedUserId都设为一样，则这些APK之间就可以互相共享数据了
            sharedUserLabel:    一个共享的用户名，它只有在设置了sharedUserId属性的前提下才会有意义 
            versionCode:        是给设备程序识别版本(升级)用的必须是一个interger值代表app更新过多少次
                                    比如第一版一般为1，之后若要更新版本就设置为2，3等等
            versionName:        这个名称是给用户看的，你可以将你的APP版本号设置为1.1版
                                    后续更新版本设置为1.2、2.0版本等等
            installLocation:    安装参数，是Android2.2中的一个新特性，installLocation有三个值可以选择：
                                    1. internalOnly 系统会优先考虑将APK安装到SD卡上
                                                    (SD卡满了的话，则安装在内存存储上)
                                    2. auto         系统将会根据存储空间自己去适应
                                    3. preferExternal   必须安装到内部才能运行
    -->
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.package.xxx"
        android:sharedUserId="string"
        android:sharedUserLabel="string resource"
        android:versionCode="integer"
        android:versionName="string"
        android:installLocation=["auto" | "internalOnly" | "preferExternal"] >
        <!-- 指定当前应用兼容的最低SDK版本 -->
        <uses-sdk
                android:minSdkVersion="10"
                android:targetSdkVersion="14"/>
        <!-- 请求您的package正常运行所需要赋予的安全许可，可以多个 -->
        <uses-permission android:name="android.permission.INTERNET"/>
        <!-- 声明了安全许可来限制那些程序能够使用应用的组件和功能 -->
        <permission/>
        <!-- 声明了用来测试当前应用包或者其他包指定组件的代码 -->
        <instrumentation/>
        <!-- 
            包括package中application级别组件声明的根节点。
            这个元素也可包含application中的全局和默认的属性，
            比如 标签,icon，主题,必要的权限等等，只能有最多一个这个元素。
            1. allowClearUserData: 用户是否能选择自行清除数据，默认为true
                                程序管理器包含一个选择允许用户清除数据
                                当为true时，用户可自己清理用户数据，反之亦然
            2. allowTaskReparenting: 是否允许activity更换从属的任务，比如从短信息任务切换到浏览器任务
            3. backupAgent: 设置该APP的备份，属性值应该是一个完整的类名
            4. debuggable: 当设置为true时，表明该APP在手机上可以被调试
            5. description/label:  此两个属性都是为许可提供的，均为字符串资源
                                当用户去看许可列表(android:label)
                                或者某个许可的详细信息(android:description)时，
                                这些字符串资源就可以显示给用户
            6. enabled: Android系统是否能够实例化该应用程序的组件
            7. hasCode: 表示此APP是否包含任何的代码
                        默认为true，
                        若为false，则系统在运行组件时，不会去尝试加载任何的APP代码
            8. icon:   声明整个APP的图标,图片一般都放在drawable文件夹下
            9. name:   应用程序所实现的Application子类的全名
                        当应用程序进程开始时，该类在所有应用程序组件之前被实例化
            10. permission: 设置许可名，这个属性若在<application>上定义的话，
                        是一个给应用程序的所有组件设置许可的便捷方式，
                        当然它是被各组件设置的许可名所覆盖的
            11. persistent: 该应用程序是否应该在任何时候都保持运行状态,默认为false
            12. process:    应用程序运行的进程名，
                        它的默认值为<manifest>元素里设置的包名，
                        当然每个组件都可以通过设置该属性来覆盖默认值。
            13. restoreAnyVersion:  用来表明应用是否准备尝试恢复所有的备份
            14. taskAffinity: 拥有相同的affinity的Activity理论上属于相同的Task
                            应用程序默认的affinity的名字是<manifest>元素中设定的package名
            15. theme: 一个资源的风格,它定义了一个默认的主题风格给所有的activity
            16. allowBackup: 是否允许备份用户基础数据
         -->
        <application
                android:allowClearUserData=["true" | "false"]
                android:allowTaskReparenting=["true" | "false"]
                android:backupAgent="string"
                android:debuggable=["true" | "false"]
                android:description="string resource"
                android:enabled=["true" | "false"]
                android:hasCode=["true" | "false"]
                android:icon="drawable resource"
                android:killAfterRestore=["true" | "false"]
                android:label="string resource"
                android:manageSpaceActivity="string"
                android:name="com.pack.ApplicationName"
                android:permission="string"
                android:persistent=["true" | "false"]
                android:process="string"
                android:restoreAnyVersion=["true" | "false"]
                android:taskAffinity="string"
                android:theme="resource or theme" 
                android:allowBackup=["true" | "false"]>
            <!-- 
                用来与用户交互的界面。
                当开启一个应用时，初始界面就是一个activity，
                大部分被使用到的其他界面都是由不同的activity所实现并在界面中进行导航调用的。 
                1. name:
                2. allowTaskReparenting: 
                        是否保留状态不变,比如切换回home, 再从新打开，activity处于最后的状态。
                3. alwaysRetainTaskState:
                4. clearTaskOnLaunch: 
                        比如 P 是 activity, Q 是被P 触发的activity,然后返回Home,重新启动 P，是否显示 Q
                5. configChanges : 
                        当配置list发生修改时,是否调用 onConfigurationChanged()方法,比如旋转时界面自适应.
                        1. 正常情况下. 如果手机旋转了.当前Activity后杀掉,然后根据方向重新加载这个Activity.
                            就会从onCreate开始重新加载.
                        2. 如果你设置了这个选项，当手机旋转后,当前Activity之后调用onConfigurationChanged()方法.
                        而不跑onCreate方法等.
                6. enabled : 
                7. excludeFromRecents :  
                        是否可被显示在最近打开的activity列表里，默认是false
                8. exported : 
                9. finishOnTaskLaunch : 
                        当用户重新启动这个任务的时候，是否关闭已打开的activity，默认是false
                10. icon : 
                11. label : 
                12. launchMode : 
                        在多Activity开发中, 有可能是自己应用之间的Activity跳转，或者夹带其他应用的可复用Activity。
                        可能会希望跳转到原来某个Activity实例，而不是产生大量重复的Activity。
                        这需要为Activity配置特定的加载模式，而不是使用默认的加载模式.有4中加载模式，默认为standard
                        1. standard： 就是intent将发送给新的实例，所以每次跳转都会生成新的activity
                        2. singleTop：也是发送新的实例，但不同standard的一点是，在请求的Activity正好
                                      位于栈顶时(配置成singleTop的Activity)，不会构造新的实例
                        3. singleTask:  和后面的singleInstance都只创建一个实例
                                        当intent到来，需要创建设置为singleTask的Activity的时候，
                                        系统会检查栈里面是否已经有该Activity的实例。如果有直接
                                        将intent发送给它。
                        4. singleInstance:  singleInstance模式就是将该Activity单独放入一个栈中，
                                            这样这个栈中只有这一个Activity，不同应用的intent都由
                                            这个Activity接收和展示，这样就做到了共享。
                    第四种模式的示例解释:
                    首先说明一下task这个概念，Task可以认为是一个栈，可放入多个Activity。
                    比如启动一个应用，那么Android就创建了一个Task，然后启动这个应用的入口Activity，
                    那在它的界面上调用其他的Activity也只是在这个task里面。那如果在多个task中共享一个Activity的话
                    怎么办呢。举个例来说，如果开启一个导游服务类的应用程序，里面有个Activity是开启GOOGLE地图的，
                    当按下home键退回到主菜单又启动GOOGLE地图的应用时，显示的就是刚才的地图，实际上是同一个Activity，
                    实际上这就引入了singleInstance。
                13. multiprocess : 是否允许多进程，默认是false
                14. noHistory : 
                        当用户从Activity上离开并且它在屏幕上不再可见时，
                        Activity是否从Activity stack中清除并结束。默认是false。Activity不会留下历史痕迹
                15. permission : 
                16. process : 
                17. screenOrientation : activity显示的模式
                        1. 默认为unspecified：由系统自动判断显示方向
                        2. landscape横屏模式，宽度比高度大
                        3. portrait竖屏模式, 高度比宽度大
                        4. user模式，用户当前首选的方向
                        5. behind模式：和该Activity下面的那个Activity的方向一致(在Activity堆栈中的)
                        6. sensor模式：有物理的感应器来决定。如果用户旋转设备这屏幕会横竖屏切换
                        7. nosensor模式：忽略物理感应器，这样就不会随着用户旋转设备而更改了
                18. stateNotNeeded : activity被销毁或者成功重启时是否保存状态
                19. taskAffinity : 
                20. theme : 
                21. windowSoftInputMode : activity主窗口与软键盘的交互模式，可以用来避免输入法面板遮挡问题
            -->
            <activity 
                    android:name="com.package.Activity"
                    android:allowTaskReparenting=["true" | "false"]
                    android:alwaysRetainTaskState=["true" | "false"]
                    android:clearTaskOnLaunch=["true" | "false"]
                    android:configChanges=["mcc", "mnc", "locale",
                                     "touchscreen", "keyboard", "keyboardHidden",
                                     "navigation", "orientation", "screenLayout",
                                     "fontScale", "uiMode"]
                    android:enabled=["true" | "false"]
                    android:excludeFromRecents=["true" | "false"]
                    android:exported=["true" | "false"]
                    android:finishOnTaskLaunch=["true" | "false"]
                    android:icon="drawable resource"
                    android:label="string resource"
                    android:launchMode=["multiple" | "singleTop" |
                                  "singleTask" | "singleInstance"]
                    android:multiprocess=["true" | "false"]
                    android:noHistory=["true" | "false"]  
                    android:permission="string"
                    android:process="string"
                    android:screenOrientation=["unspecified" | "user" | 
                                         "behind" | "landscape" |  "portrait" | "sensor" | "nosensor"]
                    android:stateNotNeeded=["true" | "false"]
                    android:taskAffinity="string"
                    android:theme="resource or theme"
                    android:windowSoftInputMode=["stateUnspecified",
                                           "stateUnchanged", "stateHidden",
                                           "stateAlwaysHidden", "stateVisible",
                                           "stateAlwaysVisible", "adjustUnspecified",
                                           "adjustResize", "adjustPan"] >
                <!-- 
                    指定一个组件所支持的intent的值
                    1. priority: 有序广播主要是按照声明的优先级别
                                  如A的级别高于B，那么，广播先传给A，再传给B。
                    - action: 只有android:name这个属性
                       name: android.intent.action.MAIN: 
                            表明此activity是作为应用程序的入口
                    - category: 只有android:name这个属性
                        name:android.intent.category.LAUNCHER:
                            决定应用程序是否显示在程序列表里
                 -->
                <intent-filter 
                    android:icon="drawable resource"
                    android:label="string resource"
                    android:priority="integer">
                    <action android:name="android.intent.action.MAIN"/>
                    <category android:name="android.intent.category.LAUNCHER"/>
                    <data />
                </intent-filter>
            </activity>
            <!-- 任何时间都在后台执行的服务组件 -->
            <service android:name="com.package.Service" 
                    android:enabled=["true" | "false"]
                    android:exported[="true" | "false"]
                    android:icon="drawable resource"
                    android:label="string resource"
                    android:name="string"
                    android:permission="string"
                    android:process="string">
                <intent-filter >
                    <action/>
                </intent-filter>
            </service>
            <!-- 能够将application获得数据的改变或发生的操作 -->
            <receiver/>
            <!-- 用来管理持久化数据并发布给其他应用程序使用的组件 -->
            <provider/>
        </application>
    </manifest>
 {% endcode %}

服务说明 **Service**  ： 
- service与activity同级，与activity不同的是，它不能自己启动的，运行在后台的程序，如果我们退出应用时，Service进程并没有结束，它仍然在后台运行。比如听音乐，网络下载数据等，都是由service运行的
- service生命周期：Service只继承了onCreate(),onStart(),onDestroy()三个方法，第一次启动Service时，先后调用了onCreate(),onStart()这两个方法，当停止Service时，则执行onDestroy()方法，如果Service已经启动了，当我们再次启动Service时，不会在执行onCreate()方法，而是直接执行onStart()方法
- ervice与activity间的通信
    Service后端的数据最终还是要呈现在前端Activity之上的，因为启动Service时，系统会重新开启一个新的进程，这就涉及到不同进程间通信的问题了(AIDL)，Activity与service间的通信主要用IBinder负责。具体可参照：[http://zhangyan1158.blog.51cto.com/2487362/491358](http://zhangyan1158.blog.51cto.com/2487362/491358)
