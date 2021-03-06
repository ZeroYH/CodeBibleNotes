# iOS开发之iOS中的动画实现,通过CAShapeLayer、CAShapeLayer,CAReplicatorLayer实现炫酷的动画、雷达效果，波纹效果，咻一咻效果。iOS核心动画实现。



##### 先简单说一下CALayer

------

- iOS 的动画都是基于 CALayer 的，每个UIView都对应有一个CALayer。

  所以修改UIView的属性所呈现的动画都是CALayer实现的。CALayer不能响应事件。下图是UIView和CALyer的关系

  ![img](http://upload-images.jianshu.io/upload_images/292993-6df78925a2156668.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


------

##### iOS中可以实现动画的方式，有不对的地方敬请指正

1. UIView动画

   ```objective-c
   //Duration 动画持续时间
    //animations  动画过程
    [UIView animateWithDuration:<#(NSTimeInterval)#> animations:<#^(void)animations#>];
    
    //block里是动画完成的回调
    [UIView animateWithDuration:<#(NSTimeInterval)#> animations:<#^(void)animations#> completion:<#^(BOOL finished)completion#>];
    
    //delay 等待时间
    //options 动画类型
    [UIView animateWithDuration:<#(NSTimeInterval)#> delay:<#(NSTimeInterval)#> options:<#(UIViewAnimationOptions)#> animations:<#^(void)animations#> completion:<#^(BOOL finished)completion#>];
    
    //弹性动画
    //damping:阻尼，范围0-1，阻尼越接近于0，弹性效果越明显
    //velocity:弹性复位的速度
    [UIView animateWithDuration:<#(NSTimeInterval)#> delay:<#(NSTimeInterval)#> usingSpringWithDamping:<#(CGFloat)#> initialSpringVelocity:<#(CGFloat)#> options:<#(UIViewAnimationOptions)#> animations:<#^(void)animations#> completion:<#^(BOOL finished)completion#>];
    
    //关键帧动画
    [UIView animateKeyframesWithDuration:<#(NSTimeInterval)#> delay:<#(NSTimeInterval)#> options:<#(UIViewKeyframeAnimationOptions)#> animations:<#^(void)animations#> completion:<#^(BOOL finished)completion#>];
   ```

2. CoreAnimation

   核心动画

   ![img](http://upload-images.jianshu.io/upload_images/292993-2e9b3c01ec9da922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


CAAnimation：核心动画的基础类，不能直接使用，负责动画运行时间、速度的控制，本身实现了CAMediaTiming协议。
CAPropertyAnimation：属性动画的基类（通过属性进行动画设置，注意是可动画属性），不能直接使用。
CAAnimationGroup：动画组，动画组是一种组合模式设计，可以通过动画组来进行所有动画行为的统一控制，组中所有动画效果可以并发执行。
CATransition：转场动画，主要通过滤镜进行动画效果设置。
CABasicAnimation：基础动画，通过属性修改进行动画参数控制，只有初始状态和结束状态。
CAKeyframeAnimation：关键帧动画，同样是通过属性进行动画参数控制，但是同基础动画不同的是它可以有多个状态控制。

基础动画、关键帧动画都属于属性动画，就是通过修改属性值产生动画效果，开发人员只需要设置初始值和结束值，中间的过程动画（又叫“补间动画”）由系统自动计算产生。和基础动画不同的是关键帧动画可以设置多个属性值，每两个属性中间的补间动画由系统自动完成，因此从这个角度而言基础动画又可以看成是有两个关键帧的关键帧动画。

------

- 下面利用CABasicAnimation实现一个简单的缩放动画

  ```objective-c
  CABasicAnimation *scaleAnimation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
  scaleAnimation.duration=.3;
  scaleAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
  scaleAnimation.fromValue = @(1);
  scaleAnimation.toValue = @(1.5);
  scaleAnimation.fillMode = kCAFillModeForwards;
  scaleAnimation.removedOnCompletion = NO;        
  [view.layer addAnimation:scaleAnimation forKey:@"transformscale"];
  ```

  **注意点** 如果fillMode=kCAFillModeForwards和removedOnComletion=NO，那么在动画执行完毕后，图层会保持显示动画执行后的状态。但在实质上，图层的属性值还是动画执行前的初始值，并没有真正被改变。

- CAKeyframeAnimation用法和CABasicAnimation类似，多了一个values属性

- CAAnimationGroup动画组可以把基础动画和关键帧动画加入进animations属性

------

在实际项目中很多需求使用UIView动画或者基本动画就可以实现，比如下面比较常用的缩放和弹性动画效果

![img](http://upload-images.jianshu.io/upload_images/292993-3d1f6209124a76eb.gif?imageMogr2/auto-orient/strip)

AnimationdemoGif1.gif

![img](http://upload-images.jianshu.io/upload_images/292993-e23badf4fdb830f9.gif?imageMogr2/auto-orient/strip)

AnimationdemoGif.gif

------

#### 炫酷的动画

如果想实现更炫酷一点的动画，如雷达，波纹，咻一咻效果，时钟，带动画的指示器等就需要用到UIBezierPath和CAShapeLayer,CAReplicatorLayer等知识。
下面简单的说一下CAShapeLayer和CAReplicatorLayer

- CAShapeLayer
  普通CALayer在被初始化时是需要给一个frame值的,这个frame值一般都与给定view的bounds值一致,它本身是有形状的,而且是矩形.CAShapeLayer在初始化时也需要给一个frame值,但是,它本身没有形状,它的形状来源于你给定的一个path,然后它去取CGPath值,它与CALayer有着很大的区别

- CAShapeLayer有着几点很重要:

  1. 它依附于一个给定的path,必须给与path,而且,即使path不完整也会自动首尾相接
  2. strokeStart以及strokeEnd代表着在这个path中所占用的百分比
  3. CAShapeLayer动画仅仅限于沿着边缘的动画效果,它实现不了填充效果

  ```objective-c
  // **创建一个shapeLayer**   
  CAShapeLayer *layer = [CAShapeLayer layer];   
  layer.frame         = showView.bounds;                // 与showView的frame一致    
  layer.strokeColor   = [UIColor greenColor].CGColor;   // 边缘线的颜色    
  layer.fillColor     = [UIColor clearColor].CGColor;   // 闭环填充的颜色   
  layer.lineCap       = kCALineCapSquare;               // 边缘线的类型 
  layer.path          = path.CGPath;                    // 从贝塞尔曲线获取到形状   
  layer.lineWidth     = 4.0f;                           // 线条宽度
  layer.strokeStart   = 0.0f;   
  layer.strokeEnd     = 0.1f;
   
  // **将layer添加进图层**    
  [showView.layer addSublayer:layer];
  ```

- CAReplicatorLayer
  CAReplicatorLayer可以复制自己子层的layer,并且复制的出来的layer和原来的子layer拥有相同的动效

------

#### 贝塞尔曲线UIBezierPath

```objective-c
UIBezierPath主要是用来绘制路径的，分为一阶、二阶.....n阶。一阶是直线，二阶以上才是曲线。而最终路径的显示还是得依靠CALayer
     初始化方法一共7种
     //方法1：构造bezierPath对象，一般用于自定义路径。
     [UIBezierPath bezierPath];
 
     //方法2：根据某个CGRect绘制路径。
     [UIBezierPath bezierPathWithRect:<#(CGRect)#>];
 
     //方法3：根据某个CGRect绘制内切圆或椭圆（CGRect是正方形即为圆，为长方形则为椭圆）。
     [UIBezierPath bezierPathWithOvalInRect:<#(CGRect)#>];
 
     //方法4：根据某个路径绘制路径。
     [UIBezierPath bezierPathWithCGPath:<#(nonnull CGPathRef)#>];
 
     //方法5：绘制每个角都是圆角的矩形，参数2是半径。
     [UIBezierPath bezierPathWithRoundedRect:<#(CGRect)#> cornerRadius:<#(CGFloat)#>];
 
     //方法6：绘制带圆角的矩形路径，参数2哪个角，参数3，横、纵向半径。
     [UIBezierPath bezierPathWithRoundedRect:<#(CGRect)#> byRoundingCorners:<#(UIRectCorner)#> cornerRadii:<#(CGSize)#>];
 
     //方法7：绘制圆弧路径，参数1是中心点位置，参数2是半径，参数3是开始的弧度值，参数4是结束的弧度值，参数5是是否顺时针(YES是顺时针方向，NO逆时针)。
     [UIBezierPath bezierPathWithArcCenter:<#(CGPoint)#> radius:<#(CGFloat)#> startAngle:<#(CGFloat)#> endAngle:<#(CGFloat)#> clockwise:<#(BOOL)#>];
```

```objective-c
自定义路径常用api
     - (void)moveToPoint:(CGPoint)point; // 移到某个点
     - (void)addLineToPoint:(CGPoint)point; // 绘制直线
     - (void)addCurveToPoint:(CGPoint)endPoint controlPoint1:(CGPoint)controlPoint1 controlPoint2:(CGPoint)controlPoint2; //绘制贝塞尔曲线
     - (void)addQuadCurveToPoint:(CGPoint)endPoint controlPoint:(CGPoint)controlPoint; // 绘制规则的贝塞尔曲线
     - (void)addArcWithCenter:(CGPoint)center radius:(CGFloat)radius startAngle:(CGFloat)startAngle endAngle:(CGFloat)endAngle clockwise:(BOOL)clockwise
     // 绘制圆形曲线
     - (void)appendPath:(UIBezierPath *)bezierPath; // 拼接曲线
```



------

![img](http://upload-images.jianshu.io/upload_images/292993-199ead8881b0c3e1.gif?imageMogr2/auto-orient/strip)

AnimationdemoGif2.gif

- 实现代码

```objective-c
//波纹，咻一咻，雷达效果
- (void)setup
{
    _testView=[[UIView alloc] initWithFrame:CGRectMake(30, 300, 100, 100)];
    [self.view addSubview:_testView];
 
    _testView.layer.backgroundColor = [UIColor clearColor].CGColor;
    CAShapeLayer *pulseLayer = [CAShapeLayer layer];
    pulseLayer.frame = _testView.layer.bounds;
    pulseLayer.path = [UIBezierPath bezierPathWithOvalInRect:pulseLayer.bounds].CGPath;
    pulseLayer.fillColor = [UIColor redColor].CGColor;//填充色
    pulseLayer.opacity = 0.0;
 
    CAReplicatorLayer *replicatorLayer = [CAReplicatorLayer layer];
    replicatorLayer.frame = _testView.bounds;
    replicatorLayer.instanceCount = 4;//创建副本的数量,包括源对象。
    replicatorLayer.instanceDelay = 1;//复制副本之间的延迟
    [replicatorLayer addSublayer:pulseLayer];
    [_testView.layer addSublayer:replicatorLayer];
 
    CABasicAnimation *opacityAnima = [CABasicAnimation animationWithKeyPath:@"opacity"];
    opacityAnima.fromValue = @(0.3);
    opacityAnima.toValue = @(0.0);
 
    CABasicAnimation *scaleAnima = [CABasicAnimation animationWithKeyPath:@"transform"];
    scaleAnima.fromValue = [NSValue valueWithCATransform3D:CATransform3DScale(CATransform3DIdentity, 0.0, 0.0, 0.0)];
    scaleAnima.toValue = [NSValue valueWithCATransform3D:CATransform3DScale(CATransform3DIdentity, 1.0, 1.0, 0.0)];
 
    CAAnimationGroup *groupAnima = [CAAnimationGroup animation];
    groupAnima.animations = @[opacityAnima, scaleAnima];
    groupAnima.duration = 4.0;
    groupAnima.autoreverses = NO;
    groupAnima.repeatCount = HUGE;
    [pulseLayer addAnimation:groupAnima forKey:@"groupAnimation"];
}
```



```objective-c
//利用UIBezierPath和CAShapeLayer实现不规则的图形，并带有动画效果，可以在折线图中使用
-(void)myTest{
    UIView *line=[[UIView alloc] initWithFrame:CGRectMake(0, 100, 400, 1)];
    line.backgroundColor=[UIColor grayColor];
    [self.view addSubview:line];
 
    _testView1=[[UIImageView alloc] initWithFrame:CGRectMake(0, 100, 300, 200)];
    _testView1.userInteractionEnabled=YES;
    [self.view addSubview:_testView1];
 
 
    //贝塞尔曲线,以下是4个角的位置，相对于_testView1
    CGPoint point1= CGPointMake(10, 80);
    CGPoint point2= CGPointMake(10, 200);
    CGPoint point3= CGPointMake(300, 200);
    CGPoint point4= CGPointMake(300, 80);
 
    _path=[UIBezierPath bezierPath];
    [_path moveToPoint:point1];//移动到某个点，也就是起始点
    [_path addLineToPoint:point2];
    [_path addLineToPoint:point3];
    [_path addLineToPoint:point4];
    [_path addQuadCurveToPoint:point1 controlPoint:CGPointMake(150, -30)];
 
    CAShapeLayer *shapeLayer=[CAShapeLayer layer];
    shapeLayer.path=_path.CGPath;
    shapeLayer.fillColor=[UIColor clearColor].CGColor;//填充颜色
    shapeLayer.strokeColor=[UIColor orangeColor].CGColor;//边框颜色
    [_testView1.layer addSublayer:shapeLayer];
 
    //动画
    CABasicAnimation *pathAniamtion = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
    pathAniamtion.duration = 3;
    pathAniamtion.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    pathAniamtion.fromValue = [NSNumber numberWithFloat:0.0f];
    pathAniamtion.toValue = [NSNumber numberWithFloat:1.0];
    pathAniamtion.autoreverses = NO;
    [shapeLayer addAnimation:pathAniamtion forKey:nil];
 
 
    UITapGestureRecognizer *tap=[[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(clickk:)];
    [_testView1 addGestureRecognizer:tap];
 
}
 
-(void)clickk:(UITapGestureRecognizer *)tap{
    CGPoint point=[tap locationInView:_testView1];
    if ([_path containsPoint:point]) {
       NSLog(@"点击不规则图形");
    }
 
}
```

---

**补充一--基本动画设置参数**
在动画方法中有一个option参数，UIViewAnimationOptions类型，它是一个枚举类型，动画参数分为三类，可以组合使用：

1.常规动画属性设置（可以同时选择多个进行设置）
UIViewAnimationOptionLayoutSubviews：动画过程中保证子视图跟随运动。UIViewAnimationOptionAllowUserInteraction：动画过程中允许用户交互。UIViewAnimationOptionBeginFromCurrentState：所有视图从当前状态开始运行。UIViewAnimationOptionRepeat：重复运行动画。UIViewAnimationOptionAutoreverse ：动画运行到结束点后仍然以动画方式回到初始点。UIViewAnimationOptionOverrideInheritedDuration：忽略嵌套动画时间设置。UIViewAnimationOptionOverrideInheritedCurve：忽略嵌套动画速度设置。UIViewAnimationOptionAllowAnimatedContent：动画过程中重绘视图（注意仅仅适用于转场动画）。 UIViewAnimationOptionShowHideTransitionViews：视图切换时直接隐藏旧视图、显示新视图，而不是将旧视图从父视图移除（仅仅适用于转场动画）UIViewAnimationOptionOverrideInheritedOptions ：不继承父动画设置或动画类型。
2.动画速度控制（可从其中选择一个设置）
UIViewAnimationOptionCurveEaseInOut：动画先缓慢，然后逐渐加速。UIViewAnimationOptionCurveEaseIn ：动画逐渐变慢。UIViewAnimationOptionCurveEaseOut：动画逐渐加速。UIViewAnimationOptionCurveLinear ：动画匀速执行，默认值。
3.转场类型（仅适用于转场动画设置，可以从中选择一个进行设置，基本动画、关键帧动画不需要设置）
UIViewAnimationOptionTransitionNone：没有转场动画效果。UIViewAnimationOptionTransitionFlipFromLeft ：从左侧翻转效果。UIViewAnimationOptionTransitionFlipFromRight：从右侧翻转效果。UIViewAnimationOptionTransitionCurlUp：向后翻页的动画过渡效果。 UIViewAnimationOptionTransitionCurlDown ：向前翻页的动画过渡效果。 UIViewAnimationOptionTransitionCrossDissolve：旧视图溶解消失显示下一个新视图的效果。 UIViewAnimationOptionTransitionFlipFromTop ：从上方翻转效果。 UIViewAnimationOptionTransitionFlipFromBottom：从底部翻转效果。

------

**补充二--keyPath**
layer有很多属性，通过animationWithKeyPath修改，列举一下常用的属性:

![img](http://upload-images.jianshu.io/upload_images/292993-81c5817606549368.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么怎么知道这些属性都有哪些呢。
1.[apple官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Key-ValueCodingExtensions/Key-ValueCodingExtensions.html)
2.通过runtime
3.多看大神博客，注意收集

#### [附上以上动画的Demo，需要的朋友可以下载看一下](https://github.com/xinge1/LXAnimationDemo)

------

推荐几篇关于动画的博客

- [iOS开发系列--让你的应用“动”起来](http://www.cnblogs.com/kenshincui/p/3972100.html#overview)
- [放肆的使用UIBezierPath和CAShapeLayer画各种图形](http://www.jianshu.com/p/c5cbb5e05075)
- [iOS动画（补充）--特殊Layer动画](http://www.jianshu.com/p/4b6d60755dd3)

