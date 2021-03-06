# FUKSYLiveDemo 快速接入文档

FUKSYLiveDemo 是集成了 Faceunity 面部跟踪和虚拟道具功能 和 [金山云直播 SDK](https://github.com/ksvc/KSYLive_iOS)  的 Demo。

本文是 FaceUnity SDK 快速对金山云直播的导读说明，关于 `FaceUnity SDK` 的详细说明，请参看 [FULiveDemo](https://github.com/Faceunity/FULiveDemo/tree/dev)



## 快速集成方法

### 一、导入 SDK

将 FaceUnity 文件夹全部拖入工程中，并且添加依赖库 OpenGLES.framework、Accelerate.framework、CoreMedia.framework、AVFoundation.framework、stdc++.tbd

### 二、快速加载道具

在 KSYStreamerVC.m 的  `viewDidLoad` 中调用快速加载道具函数，该函数会创建一个美颜道具及指定的贴纸道具。

```c
[[FUManager shareManager] loadItems];
```

注：FUManager 的 shareManager 函数中会对 SDK 进行初始化，并设置默认的美颜参数。

### 三、图像处理

在  `-(void) setCaptureCfg` 设置图像处理 Block ，并对图像进行处理：

```c
_kit.videoProcessingCallback = ^(CMSampleBufferRef buf){
    selfWeak.ctrlView.lblStat.capFrames += 1; // 统计预览帧率(实际使用时不需要)
    // 在此处添加自定义图像处理, 直接修改buf中的图像数据会传递到观众端
    // 或复制图像数据之后再做其他处理, 则观众端仍然看到处理前的图像

    /*** ------ 加入 FaceUnity 效果 ------ **/
    [selfWeak processFaceUnityWithBuffer:buf];
};

- (void)processFaceUnityWithBuffer:(CMSampleBufferRef)buffer {
    
    CVPixelBufferRef pixelBuffer = CMSampleBufferGetImageBuffer(buffer) ;
    [[FUManager shareManager] renderItemsToPixelBuffer: pixelBuffer];
}
```

### 四、切换道具及调整美颜参数

本例中通过添加 FUAPIDemoBar 来实现切换道具及调整美颜参数的具体实现，FUAPIDemoBar 是快速集成用的UI，客户可自定义UI。

1、在 KSYStreamerVC.m 中添加头文件，并创建 demoBar 属性

```C
#import <FUAPIDemoBar/FUAPIDemoBar.h>

@property (nonatomic, strong) FUAPIDemoBar *demoBar ;
```

2、在 demoBar 的 get 方法中对其进行初始化，并遵循代理  FUAPIDemoBarDelegate，实现代理方法 `demoBarDidSelectedItem:` 和 `demoBarBeautyParamChanged`以进一步实现道具的切换及美颜参数的调整。

初始化

```C
// demobar 初始化
-(FUAPIDemoBar *)demoBar {
    if (!_demoBar) {
        _demoBar = [[FUAPIDemoBar alloc] initWithFrame:CGRectMake(0, self.view.frame.size.height - 220, self.view.frame.size.width, 164)];
        
        _demoBar.itemsDataSource = [FUManager shareManager].itemsDataSource;
        _demoBar.selectedItem = [FUManager shareManager].selectedItem ;
        
        _demoBar.filtersDataSource = [FUManager shareManager].filtersDataSource ;
        _demoBar.beautyFiltersDataSource = [FUManager shareManager].beautyFiltersDataSource ;
        _demoBar.filtersCHName = [FUManager shareManager].filtersCHName ;
        _demoBar.selectedFilter = [FUManager shareManager].selectedFilter ;
        [_demoBar setFilterLevel:[FUManager shareManager].selectedFilterLevel forFilter:[FUManager shareManager].selectedFilter] ;
        
        _demoBar.skinDetectEnable = [FUManager shareManager].skinDetectEnable;
        _demoBar.blurShape = [FUManager shareManager].blurShape ;
        _demoBar.blurLevel = [FUManager shareManager].blurLevel ;
        _demoBar.whiteLevel = [FUManager shareManager].whiteLevel ;
        _demoBar.redLevel = [FUManager shareManager].redLevel;
        _demoBar.eyelightingLevel = [FUManager shareManager].eyelightingLevel ;
        _demoBar.beautyToothLevel = [FUManager shareManager].beautyToothLevel ;
        _demoBar.faceShape = [FUManager shareManager].faceShape ;
        
        _demoBar.enlargingLevel = [FUManager shareManager].enlargingLevel ;
        _demoBar.thinningLevel = [FUManager shareManager].thinningLevel ;
        _demoBar.enlargingLevel_new = [FUManager shareManager].enlargingLevel_new ;
        _demoBar.thinningLevel_new = [FUManager shareManager].thinningLevel_new ;
        _demoBar.jewLevel = [FUManager shareManager].jewLevel ;
        _demoBar.foreheadLevel = [FUManager shareManager].foreheadLevel ;
        _demoBar.noseLevel = [FUManager shareManager].noseLevel ;
        _demoBar.mouthLevel = [FUManager shareManager].mouthLevel ;
        
        _demoBar.delegate = self;
    }
    return _demoBar ;
}
```

切换贴纸代理方法

```C
/**      FUAPIDemoBarDelegate       **/

// 切换贴纸
- (void)demoBarDidSelectedItem:(NSString *)itemName {
    
    [[FUManager shareManager] loadItem:itemName];
}
```

更新美颜参数方法

```C
// 更新美颜参数
- (void)demoBarBeautyParamChanged {
    
    [FUManager shareManager].skinDetectEnable = _demoBar.skinDetectEnable;
    [FUManager shareManager].blurShape = _demoBar.blurShape;
    [FUManager shareManager].blurLevel = _demoBar.blurLevel ;
    [FUManager shareManager].whiteLevel = _demoBar.whiteLevel;
    [FUManager shareManager].redLevel = _demoBar.redLevel;
    [FUManager shareManager].eyelightingLevel = _demoBar.eyelightingLevel;
    [FUManager shareManager].beautyToothLevel = _demoBar.beautyToothLevel;
    [FUManager shareManager].faceShape = _demoBar.faceShape;
    [FUManager shareManager].enlargingLevel = _demoBar.enlargingLevel;
    [FUManager shareManager].thinningLevel = _demoBar.thinningLevel;
    [FUManager shareManager].enlargingLevel_new = _demoBar.enlargingLevel_new;
    [FUManager shareManager].thinningLevel_new = _demoBar.thinningLevel_new;
    [FUManager shareManager].jewLevel = _demoBar.jewLevel;
    [FUManager shareManager].foreheadLevel = _demoBar.foreheadLevel;
    [FUManager shareManager].noseLevel = _demoBar.noseLevel;
    [FUManager shareManager].mouthLevel = _demoBar.mouthLevel;
    
    [FUManager shareManager].selectedFilter = _demoBar.selectedFilter ;
    [FUManager shareManager].selectedFilterLevel = _demoBar.selectedFilterLevel;
}
```

3、在 `viewDidLoad:` 中将 demoBar 添加到页面上

```C
[self.view addSubview:self.demoBar];
```



### **五**、道具销毁

视频录制结束时需要调用 `[[FUManager shareManager] destoryItems]`  销毁道具。

**快速集成完毕，关于 FaceUnity SDK 的更多详细说明，请参看 [FULiveDemo](https://github.com/Faceunity/FULiveDemo/tree/dev)**