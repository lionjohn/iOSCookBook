# iOS之widget开发(Today Extension)

### 前言

**extension**是iOS8新开放的一种对几个固定系统区域的扩展机制，它可以在一定程度上弥补iOS的沙盒机制对应用间通信的限制。

**extension**的出现，为用户提供了在其它应用中使用我们应用提供的服务的便捷方式，比如用户可以在**Today Extension**中查看应用展示的简略信息，而不用再进到我们的应用中，同样可以快捷操作app的功能，这将是一种全新的用户体验。

今天我们介绍一下给工程添加**Today Extension**的步骤。

### 添加Today Extension工程

在原有的工程基础上，想要使用Today Extension，我们需要创建一个新的target，点击**File-->New-->Target-->Today Extention**，如下图所示：

![](media/1240-12.)



添加Target



![](media/1240-13.)

添加成功后项目的目录会如下图所示：

![](media/1240-14.)

运行项目会看到如下图所示的效果：

![](media/1240-15.)

### 定制UI

由于我习惯使用纯代码写UI，所以我会选择删除默认创建的MainInterface.storyboard，并在info.plist中删除NSExtensionMainStoryboard，添加NSExtensionPrincipalClass为TodayViewController，如下图所示：

![](media/1240-16.)

我们可以使用以下方法配置视图的大小

```
//配置Today Extension展示视图的大小
self.preferredContentSize = CGSizeMake([UIScreen mainScreen].bounds.size.width, 100);
```

实现下面的协议，配置边距，否则会发现一个问题：绘制的内容与左侧边界有一定距离。

```
- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets {

    //配置边距为0
    return UIEdgeInsetsMake(0.0, 0.0, 0.0, 0.0);

}
```

我们创建一个lable来充满视图，并且点击可打开我们的app

```
//Today Extension的页面加一个可点击打开containingAPP的label
UILabel *openAppLabel = [[UILabel alloc] init];
openAppLabel.textColor = [UIColor colorWithRed:(97.0/255.0) green:(97.0/255.0) blue:(97.0/255.0) alpha:1];
openAppLabel.backgroundColor = [UIColor clearColor];
openAppLabel.frame = CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, 100);
openAppLabel.textAlignment = NSTextAlignmentCenter;
openAppLabel.text = @"点击打开app";
openAppLabel.font = [UIFont systemFontOfSize:15];
[self.view addSubview:openAppLabel];

openAppLabel.userInteractionEnabled = YES;
UITapGestureRecognizer *openURLContainingAPP = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(openURLContainingAPP)];
[openAppLabel addGestureRecognizer:openURLContainingAPP];
```

### 打开app

Today Extension只能通过openURL的方式来调起app，并且需要在info.plist文件中配置参数URL types，
app工程中配置如下

![](media/1240-17.)

app schemes



Today Extension如下图

![](media/1240-18.)

跳转URL types设置



URL identifier为app的bundle ID，URL Schemes配置为app的scheme
并且调用以下代码来打开app

```
//通过openURL的方式启动Containing APP
- (void)openURLContainingAPP
{
    //scheme为app的scheme
    [self.extensionContext openURL:[NSURL URLWithString:@"scheme://xxxx"]
                 completionHandler:^(BOOL success) {
                     NSLog(@"open url result:%d",success);
                 }];
}
```

[demo代码](https://github.com/suxiaoyao/SXYWidgetDemo)，由于后面的步骤是需要苹果开发者账号才能操作，所以demo的代码到这里为止。

### 数据共享

首先需要去[苹果开发者中心](https://developer.apple.com/)的APP Groups中创建一个APP Group，命名方式"group.com.companyName.xxx"，如下图

![](media/1240-19.)

创建App Group



完成之后你还要做以下修改

1. 编辑你的contain app的APP ID，Service中选中App Groups，并且点击右边的Edit按钮选中刚刚创建的group，返回后，点击Done完成APP ID的编辑
2. 此时contain app的Provisioning Profiles文件会显示为无法使用，需要更新下文件，并且下载下来覆盖安装

Today Extension工程与app工程的配置都如下图所示


![](media/1240-20.)

App Groups设置



通过App Groups提供的同一group内app共同读写区域，可以用NSUserDefaults和NSFileManager两种方式实现Today Extension和containing app之间的数据共享。

**通过NSUserDefaults共享数据**

```
- (void)saveDataByNSUserDefaults
{
    NSUserDefaults *shared = [[NSUserDefaults alloc] initWithSuiteName:@"group.com.xxx.xxx"];
    [shared setObject:@"test" forKey:@"widget"];
    [shared synchronize];
}

- (NSString *)readDataFromNSUserDefaults
{
    NSUserDefaults *shared = [[NSUserDefaults alloc] initWithSuiteName:@"group.com.xxx.xxx"];
    NSString *value = [shared valueForKey:@"widget"];

    return value;
}
```

**通过NSFileManager共享数据**

```
- (BOOL)saveDataByNSFileManager
{
    NSError *error = nil;
    NSURL *containerURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.com.xxx.xxx"];
    containerURL = [containerURL URLByAppendingPathComponent:@"Library/Caches/test"];

    NSString *value = @"test";
    BOOL result = [value writeToURL:containerURL atomically:YES encoding:NSUTF8StringEncoding error:&error];
    if (!result) {
        NSLog(@"%@",error);
    } else {
        NSLog(@"save value:%@ success.",value);
    }

    return result;
}

- (NSString *)readDataByNSFileManager
{
    NSError *error = nil;
    NSURL *containerURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.com.xxx.xxx"];
    containerURL = [containerURL URLByAppendingPathComponent:@"Library/Caches/test"];
    NSString *value = [NSString stringWithContentsOfURL:containerURL encoding:NSUTF8StringEncoding error:&error];

    return value;
}
```

这样就实现了Today Extension与app的数据共享

### 真机调试与打包

我们把Today Extension当作一个单独的app，各自有自己的App ID和Provisioning profile. 在Xcode里是两个Target给不同的target设置自己的bundleID和Provisioning profile。所以你需要按以下步骤操作，才能真机调试以及打包

去[苹果开发者中心](https://developer.apple.com/)操作以下步骤

1. 需要为Today Extension创建一个APP ID，一般命名方式为你的contain app的bundle id加上你创建的Today Extension工程名"com.companyName.xxx.xxx"，App Services中勾选上App Groups，完成创建。如下图

1. 
    ![](media/1240-21.)

    Today Extension APP ID设置

    

2. 去Provisioning Profiles中创建Today Extension对应的profile文件，下载下来，安装，真机调试和打包需要用到，如下图

1. ![](media/1240-22.)

    Today Extension profile文件

    

2. 将Today Extension的bundleID修改为刚刚为Today Extension创建的APP ID
3. Today Extension版本号与contain app配置一致，否则审核上传的时候会有警告
4. 打包或者真机调试的时候contain app与Today Extension选择各自的profile文件。

完成以上的准备工作之后，我们就可以开始真机调试以及打包了。

### 总结

本篇暂时只是Today Extension简单的功能实现，我会在后面更新iOS10的适配，以及其他功能使用。如果有错误的地方欢迎指出~谢谢~

### 扩展阅读

1. [WWDC2014之App Extensions学习笔记](http://foggry.com/blog/2014/06/23/wwdc2014zhi-app-extensionsxue-xi-bi-ji/)

希望对您有帮助，如果文章中有问题，欢迎评论留言~，谢谢支持~欢迎**关注**，我会在空余时间更新技术文章~

