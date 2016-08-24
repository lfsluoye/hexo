---
layout: post
title: "iOS 清理缓存"
date: 2016-6-23 
categories: iOS
comments: false
tags: OC 
---
大家伙都知道，随着手机使用时间的越来越长，产生的垃圾也就会越来越多，从而会影响手机系统性能和手机运行的流畅度。这时，我们就需要清理手机里的垃圾，而这些垃圾大部分都是一些缓存的一些数据。所谓缓存就是系统在运行应用软件时把一些暂时不需要调用的数据写进缓存区，当应用软件被关闭后这些被写进缓存区的数据可能不会被清理，它们仍然会驻留在缓存区中，此时为了将存储区空出来就需要清除缓存。
　　以上是本人对于缓存的一些基本理解，接下来才是我今天所要说的缓存清理——如何在代码中清理沙盒里的缓存。首先，介绍下沙盒内各目录如何获取；其次，介绍如何获取缓存文件大小以及缓存清理。
<!-- more -->
###一、各目录获取：
:    (1)获取程序的Home目录：

```OC
NSString *homeDirectory = NSHomeDirectory();
　　NSLog(@"path:%@", homeDirectory);
```
�
:    (2)获取 document目录

```
NSArray *paths = NSSearchPathForDirectorieslnDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *path = [paths objectAtIndex:0];
NSLog(@"path :%@", path); //只有用户生成的文件、其他数据及其他程序不能重新创建的文件 ，应该保存在<Application_Home>/Documents目录下面，并将通过iCloud自动备份。
```
�
:    (3)获取cache目录：

```OC
NSArray *paths = NSSearchPathForDirectorieslnDomains(NSCachesDirectory, NSUserDomainMask, YES);
NSString *path = [paths objectAtIndex:0];
NSLog(@"path:%@", path); // 缓存目录
```
�
:   (4)获取Library目录：

```
NSArray *paths = NSSearchPathForDirectorieslnDomains(NSLibraryDirectory, NSUserDomainMask, YES)；
NSString *path = [paths objectAtIndex:0];
NSLog(@"path: %@", path); //可以重新下载或者重新生成的数据应该保存在<Application_Home>/Library/Caches目录下面 。比如新闻、地图使用的数据缓存文件和可下载的内容保存到这个文件夹中。
```
�
:   (5)获取tmp目录：
```
NSString *tmpStr = NSTempararyDirectory();
NSLog(@"path: %@", path); // 只是临时使用的数据应该保存到<Application_Home>/tmp文件夹，但是iCloud不会备份这些文件。
```
###二、缓存清理：
:   (1)计算缓存文件大小：

```
//单个文件的大小
- (long long) fileSizeAtPath:(NSString*) filePath{
    NSFileManager* manager = [NSFileManager defaultManager];
    if([manager fileExistsAtPath:filePath]){
        return[[manager attributesOfItemAtPath:filePath error:nil] fileSize];
    }
    return0;
}

//遍历文件夹获得文件夹大小，返回多少M
//设置folderPath为cache路径。
- (float) folderSizeAtPath:(NSString*) folderPath{
    NSFileManager* manager = [NSFileManager defaultManager];
    if(![manager fileExistsAtPath:folderPath]) {
　　　　　return0;
　　}
    NSEnumerator *childFilesEnumerator = [[manager subpathsAtPath:folderPath] objectEnumerator];
    NSString* fileName;
    longlong folderSize = 0;
    while((fileName = [childFilesEnumerator nextObject]) != nil){
        NSString* fileAbsolutePath = [folderPath stringByAppendingPathComponent:fileName];
        folderSize += [self fileSizeAtPath:fileAbsolutePath];
    }
    return　folderSize/(1024.0*1024.0); //得到缓存大小M
}
```
�
:   (2)删除文件达到缓存清理的目的：
```
-(void)clearCacheFlies{
　dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSString *cachPath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory,NSUserDomainMask, YES) objectAtIndex:0];
                    NSArray *files = [[NSFileManager defaultManager] subpathsAtPath:cachPath];
                    NSLog(@"files :%d",[files count]); //文件夹的数量
                    for (NSString *p in files) {
                        NSError *error;
                        NSString *path = [cachPath stringByAppendingPathComponent:p];
                        if ([[NSFileManager defaultManager] fileExistsAtPath:path]) {
                            [[NSFileManager defaultManager] removeItemAtPath:path error:&error];
                        }
　　　　　　　　　　　　NSLog(@"清理成功！");
                  　 }

}
```