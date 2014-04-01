---
layout: post
title: "APNs推送实践"
date: 2014-04-01 23:01:49 +0800
comments: true
categories: 技术
---


关于APNs苹果官方有详细介绍，之前我也整理了一份翻译篇[《苹果推送通知服务》](http://victorjiang.github.io/blog/2014/04/01/yi-ping-guo-tui-song-tong-zhi-fu-wu/)，由于水平有限，翻译可能存在错误，不过可以起到参考作用，还请指出错误之处，以便及时纠正。

下面就APNs的一些实践操作进行介绍，包括前期的证书申请以及代码操作：

##一、申请证书
###1.CSR文件
通过Keychain Access申请一个CSR文件（CertificateSigningRequest.certSigningRequest），保存到磁盘。
<!--添加图片1-->
->![apns_prac_1](http://victorjiang.github.io/images/2014/apns_prac_1.jpg)<-

###2.创建App ID
Bundle ID不能使用通配的，通配的Bundle ID不能用来推送。
<!--添加图片2-->
->![apns_prac_2](http://victorjiang.github.io/images/2014/apns_prac_2.jpg)<-

App Services勾选Push Notifications
<!--添加图片3-->
->![apns_prac_3](http://victorjiang.github.io/images/2014/apns_prac_3.jpg)<-

###3.创建开发/发布证书
根据情况选择创建开发还是发布证书
<!--添加图片4-->
->![apns_prac_4](http://victorjiang.github.io/images/2014/apns_prac_4.jpg)<-
  
  或者从App ID里创建
<!--添加图片5-->
->![apns_prac_5](http://victorjiang.github.io/images/2014/apns_prac_5.jpg)<-

选择刚才创建的App ID
<!--添加图片6-->
->![apns_prac_6](http://victorjiang.github.io/images/2014/apns_prac_6.jpg)<-

选择CSR文件生成证书（apns.cer），并下载本地。
<!--添加图片7-->
->![apns_prac_7](http://victorjiang.github.io/images/2014/apns_prac_7.jpg)<-
<!--添加图片8-->
->![apns_prac_8](http://victorjiang.github.io/images/2014/apns_prac_8.jpg)<-

###4.创建Provisioning Profile文件
选择profile文件类型
<!--添加图片9-->
->![apns_prac_9](http://victorjiang.github.io/images/2014/apns_prac_9.png)<-

选择App ID
<!--添加图片10-->
->![apns_prac_10](http://victorjiang.github.io/images/2014/apns_prac_10.png)<-

选择证书、设备后生成文件（apns.mobileprovision），并下载。
<!--添加图片11-->
->![apns_prac_11](http://victorjiang.github.io/images/2014/apns_prac_11.jpg)<-
<!--添加图片12-->
->![apns_prac_12](http://victorjiang.github.io/images/2014/apns_prac_12.jpg)<-

###5.导出密钥
在Keychain Access中导出之前创建的证书的密钥（apns.p12），这里需要对密钥文件进行加密。
<!--添加图片13-->
->![apns_prac_13](http://victorjiang.github.io/images/2014/apns_prac_13.png)<-

###6.证书处理
现在我们有下面几个文件：

1. CertificateSigningRequest.certSigningRequest
2. apns.cer
3. apns.mobileprovision
4. apns.p12

处理证书是为了让我们的服务端能够可以给APNs发送通知，对于不同的服务端，证书的处理也不同。

1. openssl x509 -in apns.cer -inform der -out PushChatCert.pem
2. openssl pkcs12 -nocerts -out PushChatKey.pem -in apns.p12
3. cat PushChatCert.pem PushChatKey.pem > ck.pem

如果服务端是.net，第三步需要用下面的命令生成证书

	openssl pkcs12 -export -in PushChatCert.pem -inkey PushChatKey.pem -certfile CertificateSigningRequest.certSigningRequest -name "apns_web" -out apns_web.p12

##二、代码操作
<font color='red'>注</font>：项目的Bundle Identify需要与创建的App ID保持一致

###1.注册推送
<pre><code>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{

    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert];

    return YES;
}
</code></pre>

* 注册成功

<pre><code>
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
	//注册成功，去除deviceToken中的空格和<>
    
    NSLog(@"register success:%@", deviceToken);
    NSString *token = [[[deviceToken description] stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"<>"]] stringByReplacingOccurrencesOfString:@" " withString:@""];
    
}
</code></pre>

* 注册失败

<pre><code>
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
    
}
</code></pre>

###2.接收通知
接收通知根据App的状态分为三种情况：

####2.1 App未运行
此时didFinishLaunchingWithOptions方法将被调用
<pre><code>
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	NSDictionary *userInfo = [launchOptions objectForKey: UIApplicationLaunchOptionsRemoteNotificationKey];
	
	//通过userInfo判断程序是通过点击推送通知启动还是正常启动
}
</code></pre>

####2.2 App后台运行
当用户点击推送消息的时候才会调用下面的方法
<pre><code>
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
    // 处理推送消息
    NSLog(@"userinfo:%@",userInfo);
    
}
</code></pre>
####2.3 App前台运行
当App正在运行的时候会自动调用上面的didReceiveRemoteNotification方法，此时App的applicationState值为UIApplicationStateActive。

####iOS 7新增方法
<pre><code>
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler;
</code></pre>

##参考文章

1. [一步一步教你做ios推送](http://blog.csdn.net/showhilllee/article/details/8631734)
2. [net发送apns解决方案（iphone push）](http://blog.sina.com.cn/s/blog_4adf31ea010175wo.html)
3. [获取 APNs 推送内容](http://docs.jpush.cn/pages/viewpage.action?pageId=4259879)