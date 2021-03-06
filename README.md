# SFCoreUtils

Number of useful beans: DateFormatter, UIView macroses, non-retaining arrays etc


## Utils


### VersionSupportHelper
Say 'yes' to easy maintaining of several iOS versions at the same moment!

Run code only on iOS7 like this:

``` objective-c
    [SFVersionSupportHelper onDeviceVersionGreaterOrEqualToVersion:7.0 do:^{
        self.edgesForExtendedLayout = UIRectEdgeNone;
        self.automaticallyAdjustsScrollViewInsets = NO;
        self.view.tintColor = [UIColor whiteColor];
        self.navigationController.navigationBar.tintColor = [UIColor whiteColor];
        self.navigationController.navigationBar.barTintColor = [UIColor whiteColor];
    }];
```

Or on both iOS7 and iOS6:

``` objective-c
    [VersionSupportHelper onDeviceVersionGreaterOrEqualToVersion:7.0 do:^{
        UIImage * image = [UIImage imageNamed:@"navigation-bar-blue-transparent-ios7"];
        [navigationBar setBackgroundImage:image forBarMetrics:UIBarMetricsDefault];
    } elseDo:^{
        navigationBar.barStyle = UIBarStyleBlackTranslucent;
        UIImage * image = [[UIImage imageNamed:@"navigation-bar-blue-transparent"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 8, 0, 8)];
        [navigationBar setBackgroundImage:image forBarMetrics:UIBarMetricsDefault];
    }];
```

iOS8 example:

``` objective-c
[SFVersionSupportHelper onDeviceVersionGreaterOrEqualToVersion:8.0 do:^{
    UIApplication * sharedApplication = [UIApplication sharedApplication];
    if ([sharedApplication respondsToSelector:@selector(registerUserNotificationSettings:)]) {
        [sharedApplication registerUserNotificationSettings:
            [UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIRemoteNotificationTypeSound)
                                              categories:nil]];
    }
}];
```

Easy to use! 

Supports different conditions:
* if-greater-than-do
* if-less-than-do 
* if-greater-than-do-else-do.

Useful even for minor releases!


### UsefulBlocks

Don't want to open http://fuckingblocksyntax.com/ every time, but want to use block easy? Try these:

``` objective-c
typedef void (^SFSuccessBlock)(id result);

typedef void (^SFErrorBlock)(NSError *error);

typedef void (^SFResponseErrorBlock)(id response, NSError *error);

typedef void (^SFActionBlock)();
```

Usage:

``` objective-c
- (void)signUpUser:(NSString *)name
             email:(NSString *)email
          password:(NSString *)password
      successBlock:(SFSuccessBlock)successBlock
        errorBlock:(SFErrorBlock)errorBlock {

    // send request here
    
    if (SUCCESS) {
    	// handle success
    	if (successBlock) {
    		successBlock(responseObject);
    	}	
    } else {
    	// handle error
        if (errorBlock) {
            errorBlock(responseObject, serverError);
        }
    }
}
```

Or like this:

**SFViewController.h**

``` objective-c
@interface SFViewController : UIViewController

@property (nonatomic, copy) SFActionBlock dismissalBlock;

@end
```

**SFViewController.m**

``` objective-c
- (void)hideController {
 	__weak typeof(self) weakSelf = self;
	[self dismissViewControllerAnimated:YES completion:^(){
            if (weakSelf.dismissalBlock) {
                weakSelf.dismissalBlock();
            }
        }];
}
```

**somewhere**

``` objective-c
SFViewController * someController = [SFViewController new];
someController.dismissalBlock = ^{
	// do here smth
 };
 ```
 
 
### UsefulQueues
 
 Simple macroses to use with gcd code:
 
 ``` objective-c
 dispatch_async(BACKGROUND_QUEUE, ^{
    // do smth heavy in background
    dispatch_async(MAIN_QUEUE, ^{
    	// return to main queue
	});
 });
 ```
 
 
 
## NSObject

### NSObject+Blocks

Useful, but if you use this a lot, usually it means that your code smells.  

``` objective-c
	// disable idle timer (in case user has slept during session - do not waste battery)
 	[self performBlock:^{
     	[[UIApplication sharedApplication] setIdleTimerDisabled:NO];
    } afterDelay:30];
```

Same as dispatch_after, but you can cancel block:

``` objective-c
- (void)syncProductList {
    if (self.syncBlock) {
        [CHSyncManager cancelBlock:self.syncBlock];
        self.syncBlock = nil;
        NSLog(@"%@ cancel block", [self class]);
    }

    self.syncBlock = [CHSyncManager performBlock:^{
        NSLog(@"%@ schedule delayed block", [self class]);
        [self delayedSyncProductList];
        
    } afterDelay:3];
}
```


### NSObject+ClassCast

Useful when working with diffent class types in collection. 

``` objective-c
[notifications enumerateObjectsUsingBlock:^(id object, NSUInteger idx, BOOL * stop) {
	ATNotification * notification = [object castOrNil:[ATNotification class]];

	if (notification) {
		NSNumber * notificationID = notification.ID;
		//...
	}
}];
```


## SFDateFormatterUtils

Creates and caches instance of NSDateFormatter to minimize costs for allocating date formatter instances. Supports locale and date styles.

Usage:

``` objective-c
NSDateFormatter * dateFormatter = [SFDateFormatterUtils dateFormatterWithFormat:@"MMMM dd, yyyy" andLocale:@"en_US_POSIX"];
NSString * dateString = [dateFormatter stringFromDate:self];
```

``` objective-c
// 2013-08-24T13:58:22.222+03:00
+ (NSDate * )dateFromISOString:(NSString * )string {
    NSDateFormatter *formatter = [SFDateFormatterUtils dateFormatterWithFormat:@"yyyy-MM-dd'T'HH:mm:ss.SSSZZZZZ"];
    NSDate * date = [formatter dateFromString:string];
    return date;
}
```

Read more why creating a lof of NSDateFormatter is usually a bad idea
http://stackoverflow.com/a/4442389

