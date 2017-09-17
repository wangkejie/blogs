---
title: scrollView左右上下单方向滑动下问题
date: 2017-09-15 11:26:39
tags: scrollView
categories: iOS
---

# 解决iOS的scrollView属性directionalLockEnabled的问题修正 

苹果官方文档说明
> "If this property is NO, scrolling is permitted in both horizontal and vertical directions. If this property is YES and the user begins dragging in one general direction (horizontally or vertically), the scroll view disables scrolling in the other direction. If the drag direction is diagonal, then scrolling will not be locked and the user can drag in any direction 
until the drag completes. The default value is NO"

问题出现了，当你斜着滑动的时候，这个属性就失效了，就会出现斜着滑动的情况

## 解决办法如下：

<!--more-->

```
@property (nonatomic) CGPoint scrollViewStartPosPoint;
@property (nonatomic) int     scrollDirection;

@synthesize scrollViewStartPosPoint = _scrollViewStartPosPoint;
@synthesize scrollDirection = _scrollDirection;

- (id)initWithFrame:(CGRect)frame
{
     _scrollDirection = 0;
}
- (void)scrollViewDidScroll:(UIScrollView *)scrollView 
{ 
    if (self.scrollDirection == 0){//we need to determine direction
        //use the difference between positions to determine the direction.
        if (abs(self.scrollViewStartPosPoint.x-scrollView.contentOffset.x)<
            abs(self.scrollViewStartPosPoint.y-scrollView.contentOffset.y)){
            //Vertical Scrolling
            self.scrollDirection = 1;
        } else {
            //Horitonzal Scrolling
            self.scrollDirection = 2;
        }
    }
    //Update scroll position of the scrollview according to detected direction.
    if (self.scrollDirection == 1) {
        scrollView.contentOffset = CGPointMake(self.scrollViewStartPosPoint.x,scrollView.contentOffset.y);
    } else if (self.scrollDirection == 2){
        scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x,self.scrollViewStartPosPoint.y);
    }
}

- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView 
{
    self.scrollViewStartPosPoint = scrollView.contentOffset;
    self.scrollDirection = 0;
}

- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate 
{
    if (decelerate) {
        self.scrollDirection =0;
    }
}

- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView 
{
    self.scrollDirection =0;
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    [super touchesBegan:touches withEvent:event]; // always forward touchesBegan -- there's no way to forward it later
//    if (_isHorizontalScroll)
//        return; // UIScrollView is in charge now
//    if ([touches count] == [[event touchesForView:self] count]) { // initial touch
//        _originalPoint = [[touches anyObject] locationInView:self];
//        _currentChild = [self honestHitTest:_originalPoint withEvent:event];
//        _isMultitouch = NO;
//    }
//    _isMultitouch |= ([[event touchesForView:self] count] > 1);
//    [_currentChild touchesBegan:touches withEvent:event];
}

- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
     [super touchesMoved:touches withEvent:event];
//    if (!_isHorizontalScroll && !_isMultitouch) {
//        CGPoint point = [[touches anyObject] locationInView:self];
//        if (fabsf(_originalPoint.x - point.x) > kThresholdX && fabsf(_originalPoint.y - point.y) < kThresholdY) {
//            _isHorizontalScroll = YES;
//            [_currentChild touchesCancelled:[event touchesForView:self] withEvent:event]
//        }
//    }
//    if (_isHorizontalScroll)
//        [super touchesMoved:touches withEvent:event]; // UIScrollView only kicks in on horizontal scroll
//    else
//        [_currentChild touchesMoved:touches withEvent:event];
}
```
