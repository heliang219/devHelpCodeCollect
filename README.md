# 单例
```
+ (MyClass *)sharedInstance
{
    static MyClass *sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[MyClass alloc] init];
        // Do any other initialisation stuff here
    });
    return sharedInstance;
}

/** 宏的单例创建 */
#undef    AS_SINGLETON
#define AS_SINGLETON( __class ) \
+ (__class *)sharedInstance;

#undef    DEF_SINGLETON
#define DEF_SINGLETON( __class ) \
+ (__class *)sharedInstance \
{ \
static dispatch_once_t once; \
static __class * __singleton__; \
dispatch_once( &once, ^{ __singleton__ = [[__class alloc] init]; } ); \
return __singleton__; \
}
```

# 宏定义

## 空判断
```
static inline BOOL IsEmpty(id thing) {
    return thing == nil || [thing isEqual:[NSNull null]]
        || ([thing respondsToSelector:@selector(length)]
        && [(NSData *)thing length] == 0)
        || ([thing respondsToSelector:@selector(count)]
        && [(NSArray *)thing count] == 0);
}


/** 判断数据是否为空 */

// 字符串是否为空
#define kISNullString(str) ([str isKindOfClass:[NSNull class]] || str == nil || [str length] < 1 ? YES : NO )
// 数组是否为空
#define kISNullArray(array) (array == nil || [array isKindOfClass:[NSNull class]] || array.count == 0 ||[array isEqual:[NSNull null]])
// 字典是否为空
#define kISNullDict(dic) (dic == nil || [dic isKindOfClass:[NSNull class]] || dic.allKeys == 0 || [dic isEqual:[NSNull null]])
// 是否是空对象
#define kISNullObject(_object) (_object == nil \
|| [_object isKindOfClass:[NSNull class]] \
|| ([_object respondsToSelector:@selector(length)] && [(NSData *)_object length] == 0) \
|| ([_object respondsToSelector:@selector(count)] && [(NSArray *)_object count] == 0))

```
## Object转String
```
static inline NSString *StringFromObject(id object) {
	if (object == nil || [object isEqual:[NSNull null]]) {
		return @"";
	} else if ([object isKindOfClass:[NSString class]]) {
		return object;
	} else if ([object respondsToSelector:@selector(stringValue)]){
		return [object stringValue];
	} else {
		return [object description];
	}
}
```

## 颜色
```
// example usage: UIColorFromHex(0x9daa76)
#define UIColorFromHexWithAlpha(hexValue,a) [UIColor colorWithRed:((float)((hexValue & 0xFF0000) >> 16))/255.0 green:((float)((hexValue & 0xFF00) >> 8))/255.0 blue:((float)(hexValue & 0xFF))/255.0 alpha:a]
#define UIColorFromHex(hexValue)            UIColorFromHexWithAlpha(hexValue,1.0)
#define UIColorFromRGBA(r,g,b,a)            [UIColor colorWithRed:r/255.0 green:g/255.0 blue:b/255.0 alpha:a]
#define UIColorFromRGB(r,g,b)               UIColorFromRGBA(r,g,b,1.0)
```

## 设备相关
```
/** 主屏幕的宽度 */
#define kScreenWidth [[UIScreen mainScreen] bounds].size.width
/** 主屏幕的高度 */
#define kScreenHeight [[UIScreen mainScreen] bounds].size.height

/**适配 iPhone X TabBar和导航区别
 * Top区别:iPhone X 为例:导航(44 points)+状态栏(44 points)= 88 points
 *        Iphone 6s为例:导航(44 points)+状态栏(20 points)= 64 points
 * Bottom区别:iPhone X 为例: 83 points高度(TabBar) = Danger Area(34 points) + 原来的49 points
 *           Iphone 6s为例:49 points高度(TabBar) = 49 points
 */
 
//iPhoneX / iPhoneXS
#define  isIphoneX_XS     (kScreenWidth == 375.f && kScreenHeight == 812.f ? YES : NO)
//iPhoneXR / iPhoneXSMax
#define  isIphoneXR_XSMax    (kScreenWidth == 414.f && kScreenHeight == 896.f ? YES : NO)
#define  isFullScreen    (isIphoneX_XS || isIphoneXR_XSMax)
#define  StatusBarHeight     (isFullScreen ? 44.f : 20.f)
#define  NavigationBarHeight  44.f
#define  TabbarHeight         (isFullScreen ? (49.f+34.f) : 49.f)
#define  TabbarSafeBottomHeight         (isFullScreen ? 34.f : 0.f)
#define  StatusBarAndNavigationBarHeight  (isFullScreen ? 88.f : 64.f)

/** 设备判断 */
#define IS_IPHONE [[UIDevice currentDevice] userInterfaceIdiom] == UIUserInterfaceIdiomPhone
#define IS_PAD (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad)
/** App主窗口 */
#define KeyWindow [UIApplication sharedApplication].keyWindow
#define appDelegate ((AppDelegate*)[UIApplication sharedApplication].delegate)
```

## App相关
```
/** 系统的版本号 */
#define SYSTEM_VERSION [[[UIDevice currentDevice] systemVersion] floatValue]

/** APP版本号 */
#define APP_VERSION  [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleShortVersionString"]

/** APP BUILD 版本号 */
#define APP_BUILD_VERSION  [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleVersion"]

/** APP名字 */
#define APP_DISPLAY_NAME  [[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleDisplayName"]

/** 当前语言 */
#define LOCAL_LANGUAGE [[NSLocale currentLocale] objectForKey:NSLocaleLanguageCode]

/** 当前国家 */
#define LOCAL_COUNTRY [[NSLocale currentLocale] objectForKey:NSLocaleCountryCode]

//获取temp
#define kPathTemp            NSTemporaryDirectory()
//获取沙盒 Document
#define kPathDocument        [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
//获取沙盒 Cache
#define kPathCache           [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject]
```

## View设置圆角
```
// View 圆角和加边框
#define ViewBorderRadius(View, Radius, Width, Color)\
[View.layer setCornerRadius:(Radius)];\
[View.layer setMasksToBounds:YES];\
[View.layer setBorderWidth:(Width)];\
[View.layer setBorderColor:[Color CGColor]]

```

## 获取图片
```
// 加载图片
#define kGetImage(imageName)   [UIImage imageNamed:[NSString stringWithFormat:@"%@",imageName]]
// 读取本地图片 （文件名，后缀名）
#define kGetBundleImage(__FILENAME__,__EXTENSION__) [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:__FILENAME__ ofType:__EXTENSION__]]

```

## 不同屏幕尺寸字体适配
```
// 字体大小(常规/粗体)
#define BOLDSYSTEMFONT(FONTSIZE)[UIFont boldSystemFontOfSize:FONTSIZE]
#define SYSTEMFONT(FONTSIZE)    [UIFont systemFontOfSize:FONTSIZE]
#define FONT(NAME, FONTSIZE)    [UIFont fontWithName:(NAME) size:(FONTSIZE)]

//中文字体
#define CHINESE_FONT_NAME @"Heiti SC"
#define CHINESE_SYSTEM(x) [UIFont fontWithName:CHINESE_FONT_NAME size:x]

//不同屏幕尺寸字体适配（320，568是因为效果图为IPHONE5 如果不是则根据实际情况修改）
#define kScreenWidthRatio  (kScreenWidth / 375.f)
#define kScreenHeightRatio (kScreenHeight / 667.f)
#define AdaptedWidth(x)  ceilf((x) * kScreenWidthRatio)
#define AdaptedHeight(x) ceilf((x) * kScreenHeightRatio)
#define kFontSize(R)     CHINESE_SYSTEM(AdaptedWidth(R))

```

## 空格相关
```
/** 去掉首尾空格和换行符 */
#define FirstAndLastSpace(str) [str stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]]
/**去掉所有空格*/
#define RemoveAllSpaces(str) [str stringByReplacingOccurrencesOfString:@" " withString:@""]

```
## 打印 NSlog
```
#ifdef DEBUG
#   define DLog(fmt, ...) NSLog((@"%s [Line %d] " fmt), __PRETTY_FUNCTION__, __LINE__, ##__VA_ARGS__);
#else
#   define DLog(...)
#endif

```
## 强弱引用
```
#define kWeakSelf(type)  __weak typeof(type) weak##type = type;
#define kStrongSelf(type)  __strong typeof(type) type = weak##type;

```