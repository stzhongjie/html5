
前言：最近参加公司的H5页面创意竞赛，又遇到不少页面在不同系统上的坑。踩坑之余，觉得很多之前遇到的知识点都忘了，索性开一篇博文，把这些坑都统一归纳起来，持续收集更新，于己利人，抛砖引玉。

## 1. ios系统手机无法自动播放BGM
这个是苹果系统限制，默认不允许自动播放音频，往往需要点一下触发play()事件才能播放。
那么我们在页面onload后触发播放事件：
```
document.getElementById('music').play();
```

到这里一般都可以播放音乐了，如果还不行，很有可能是微信的限制。这时需要调用微信接口。
页面先引入：
```
<script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
```
然后JS写入微信事件：
```
document.addEventListener("WeixinJSBridgeReady", function() {
    document.getElementById('music').play();
  }, false);
```
这样利用微信接口调用play()事件，可以完美解决ios音频无法autoplay的问题。

## 2. ios系统摇一摇播放音效事件无效
在实现摇晃（引用了封装好的shake.js）手机触发某一音效这个需求时，发现在微信中，音效没有被触发。后面找到原因：在ios里并没有把自定义摇晃事件shake当成交互动作。而要播放音效，需要用户有交互动作。没有交互，音效就没被加载，那么我们先加载音效，结合上面的微信接口：

```
document.addEventListener("WeixinJSBridgeReady", function () {
  shakeMusic.load();
}, false);
```

*load()*过之后，再调用play()即可听到音效。

## 3. ios系统不支持动画暂停样式（*animation-play-state*）
H5页面一般都会有BGM，也会提供一个旋转的音乐图标供用户开启关闭音乐。我们希望当用户点击音乐按钮时图标停止旋转，再点图标顺着之前停止的位置继续跑动画。*animation-play-state*是最简便的方式，然而，ios不支持。

目前的解决方案是：音乐图标负责跑动画，图标父级元素负责记录停止时的转动值。

####html
```
<div class="music">
    <img class="musicImg" src="/images/music.png">
</div>
```

####sass
```
.music {
  position: absolute;
  width: rem(64px);
  height: rem(64px);
  top: rem(66px);
  left: rem(15px);
  z-index: 1000;

  img {
    width: 100%;
  }
}

.musicRun {
  -webkit-animation: music 2.5s infinite linear 0.5s;
  animation: music 2.5s infinite linear 0.5s;
}

@-webkit-keyframes music {
  0% {}
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}

@keyframes music {
  0% {}
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
```
####JS
```
var $img = $('.musicImg')
  var music = document.getElementById('music');
  var isPlaying = false
  running()
  $img.on('click', function() {
    !isPlaying ? running() : paused()
})

  function running() {
    music.play();
    $img.addClass('musicRun')
    isPlaying = true
  }

  function paused() {
    music.pause();
    var siteImg = $img.css('transform') //获取当前元素的动画改变，transform的值
    var siteWp = $('.music').css('transform')
    $('.music').css('transform', siteWp === 'none' ? siteImg : siteImg.concat('', siteWp))
    //由于父元素没有动画，所以每次赋值的时候，需要将上次父元素的状态加上
    $img.removeClass('musicRun')
    isPlaying = false
  }
```



## 4.安卓微信打开页面时动画静止
H5页面动画很重要。当我布好了动画样式，用安卓微信打开发现页面静止不动，动画没有生效。新进入页面没效果，刷新一下就恢复。

最直接了当的解决方案：把动画提取出来，例如提到一个*running*样式中，然后在页面load完了再把这个 动画值赋上去。

```
window.onload = function() {
  $('.index').addClass('running')
};
```
