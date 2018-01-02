# 第五章 AV Kit 用法

AV Kit 可以简化基于 AV Foundation 框架且满足默认操作系统视觉效果和体验的视频播放器的创建过程。AV Kit 框架第一次出现在 Mac OS X Mavericks 版本中，从 iOS 8 开始被引入到 iOS 平台。

## 1. 针对 iOS 平台的 AV Kit 框架

iOS 9 之前，iOS 播放视频文件一般使用 MPMoviePlayerController 和 MPMoviePlayerViewController，前者继承 NSObject，后者继承 UIViewController。iOS 9 之后，苹果弃用 MPMoviePlayerController，推荐使用 AVPlayer。两者区别为：MPMoviePlayerController 用法简单，但是功能不够强大；AVPlayer 使用稍微麻烦点，不过功能更加强大。

## 2. 针对 Mac OS X 平台的 AV Kit 框架

这部分所讲的都是 Mac OS X 系统的 AV Kit 框架，略过。