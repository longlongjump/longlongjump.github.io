---
layout: post
title:  "Swipeable buttons inside UITableViewCell"
date:   2016-03-09 12:00:00 +0200
categories: ios
tags: [UIKit, UITableView, UITableViewCell, UITableViewRowAction, SWTableViewCell, ios, swipe, Xcode]
---

![SWTableViewCell](/assets/posts/2016-03-09-swipe-cell/cell-swipe.gif){: .img-responsive }

In my current project, I found that `UITableViewRowAction` does not match my needs anymore. I need much more than simple button with title. Github has tons of libraries that can add swipe options for cell:

`SWTableViewCell` — popular, old framework and maybe most buggy framework of any. Customizing it to my needs and fixing all bugs at previous project was not happy experience at all.
MGSwipeTableCell — New popular library, it has all this fancy animations 3K of GitHub stars and nice api.
In most cases `MGSwipeTableCell` is sufficient. But not in mine. Library uses snapshotting of cell contentView which totally preventing any UI update while cell swiped.

I was not happy to return to `SWTableViewCell` so I decided to code it from scratch.

## Swipe Cell Layout

I started researching SWTableViewCell code. SWTableViewCell replaces contentView with `UISrollView` and then put `contentView` itself in `UISrollView`. Because `UITableViewCell` has its own way to work contentView this approach produces nasty layout bugs especially when scrolling. `SWTableViewCell` has a significant amount of code to just fix such problems.

My solution is not to touch `contentView` at all . I created new distinct view to hold cell elements and assign it to `slideContentView` variable. Then I place `slideContentView` with all option buttons inside `UISrollView` which is subview of contentView. This approach is more clean in sense of implementation and keeps layout safe from unexpected frame changes.

![SWTableViewCell](/assets/posts/2016-03-09-swipe-cell/ib.png){: .img-responsive }

After cell initialization, layout looks like this.

![SWTableViewCell Layout](/assets/posts/2016-03-09-swipe-cell/layout.png){: .img-responsive }

Dashed lines show visible part of cell
## Cell Selection

Big problem with swipe options was handling tap events, `UIScrollView` needs forward all user touches to contentView to make cell selection and highlighting work properly. The best solution I found so far is to create Overlay View. And override its touchesBegan/Moved/Ended/Canceled methods to forward all touches to contentView

Scrolling

Inside scrollViewWillEndDragging method I handle high-speed scroll to make `UIScrollView` decelerating animation smooth.

## Results

I ended with ~450 lines of swift code. I uploaded code and Demo application to https://github.com/longlongjump/LLSwipeCell. It has podspec file for easy intergation.

LLSwipeCell still has some drawbacks:

The main drawback is that is need to create one more contentView to wrap cell elements. But I think this is an affordable inconvenience for not writing tons of code to handle all layout glitches.
It does not support UITableViewCell system disclosure indicator, textLabel, and imageView because these components located outside contentView.
Lacks of many fancy animation. The one is currently available is Border Transition.
