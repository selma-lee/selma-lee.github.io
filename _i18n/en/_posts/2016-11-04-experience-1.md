---
layout: post
title: The Pit of Mobile Refactoring (Long Term Update)
date: 2016-11-04 00:47:34
tags: Refactor
categories:
 - UX
---

This article is used to record the usual mobile refactoring encountered pitfalls, to remember, long-term update.

<!-- more -->

## Keyboard masking affects layout

Pit: Fixed property fails under ios. When you type in the edit box, the soft keyboard will pop up, and the limited screen area of the phone will often cover the input interface.
As the following screenshot, the image is from [%Phantom#Shadow% - Bloggernacle](https://www.cnblogs.com/cmblogs/p/4448336.html)
! [Keyboard occlusion affects layout](/assets/img/2016/11/experience-1-2.jpg)
fix:

* :: Method 1: css set flex:1

``` css
.container { /* 父类 */
    display: flex;
}
.container-content { /* 不想被遮挡的内容所在类 */
    flex: 1;
}
```

* Method 2: js listen to the input box focus event, focus on the page to move up, out of focus to restore the original state

``` js
$('input').on('focus', function() {
    $('.container-content').css('transform','translateY(-50%)');
})
$('.container-content').on('click', function() {
    $('.container-content').css('transform','translateY(0)');
})
```

## Unable to modify document.title in wechat and other webviews

Pit: will encounter the need to dynamically modify the needs of document.title, this will encounter in WeChat and other webviews can not modify the pit of document.title
fix:

``` js
// hack在微信等webview中无法修改document.title的情况
var $iframe = $('<iframe src="#"></iframe>'),
    $body = $('body');
$iframe.on('load',function() {
    setTimeout(function() {
        $iframe.off('load').remove();
    }, 0);
}).appendTo($body);
```

``` css
iframe {
  visibility: hidden;
  width: 1px;
  height: 1px;
}
```

## Picture edges appear truncated

Pitfall: image edge truncation on plus machine. The icon pixels are odd, and my predecessor said that mobile icons are required to be even to not have problems.
! [image edge truncation](/assets/img/2016/11/experience-1-1.jpeg)
FIX: Edit the image size to an even number!
P.S. It could also be due to the use of rem. Don't use a specific rem value for the background-size of smaller background images (like some icons), you'll lose the edges after cropping. You should use an element-size cutout and set background-size: contain|cover to scale it.

## Androids drop the rem decimal part ##

Pit: IOS is sensitive to decimal pixels but Android is not, some androids will drop the rem decimal part
fix:

* The root element is set to 50px, to 50px, i.e. it's good math, and there will be less decimal places.
* The more brutal ones don't use rem, use px, and think of other adaptive solutions.

## The change event for input[type="file"] is called only once

After calling it once, I realize that the input box is not bound to the change event, so I bind the change event again inside the callback function.

``` js
$('input[type="file"]').on('change', function(e) {
  e.preventDefault();
  e.stopPropagation();
  var $target = $(e.target);
  var fileList = e.target.files||e.dataTransfer.files;
  // do something
  $target.replaceWith('<input type="file" class="fd-file" data-fd="file" id="file"'+Math.random()+'>');
  $('input[type="file"').on('change', function(e) {
    var $target = $(e.target);
    var fileList = e.target.files||e.dataTransfer.files;
    // do something
  })
})
```

## Reference

* [Dynamically change title with document.title="xxx", not effective under WeChat in ios](https://segmentfault.com/q/1010000002926291)
