Apple的机制是对于LaunchScreen.storyboard启动的app，部分机型会缓存一份启动图放在/沙盒/Library/SplashBoard 中，
每次启动会优先读取缓存，造成启动图无法及时更新。所以我们要做的是每次启动时读取并清除缓存，重新加载启动图：
- (void)removeLaunchScreenCacheIfNeeded {
   NSString *filePath = [NSString stringWithFormat:@"%@/Library/SplashBoard", NSHomeDirectory()];
   
   if ([[NSFileManager defaultManager] fileExistsAtPath:filePath]) {
    NSError *error = nil;
    [[NSFileManager defaultManager] removeItemAtPath:filePath error:&error];

    if (error) {
         NSLog(@"清除LaunchScreen缓存失败");
       } else {
         NSLog(@"清除LaunchScreen缓存成功");
       }
    }
}