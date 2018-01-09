# problem-summary
平时遇到的古怪问题的总结

- 功能键在非Firefox浏览器不触发keypress事件的问题
	- 比如，按下backspace键在Firefox下会触发keydown、keypress事件，而在chrome下只会触发keydown事件
	- 参考[键盘事件keydown、keypress、keyup随笔整理总结](http://www.cnblogs.com/xcsn/p/3413074.html)
	- 可以使用DOM 3级的KeyboardEvent来模拟keypress事件，但有部分兼容性问题，参考[KeyboardEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent)
- 如果canvas中绘制了跨域图片，canvas.toDataURL会报错，可以用img.setAttribute('cross0rigin', 'Anonymous')解决，但前提是服务器允许跨域请求图片
- 呼出键盘后，页面元素被遮挡，可以设置页面高度为document.getClientHeight来解决
- 在ios下，呼出键盘后，页面元素的位置会调整，但绝对定位元素中的输入框的焦点却还在原来的位置。可以在呼出键盘后，手动使绝对定位元素中的input获得一次焦点来解决（focus方法）
- ios下，不会执行keyup事件回调函数中的异步请求
- 安卓下用vh设置高度，呼出键盘时内容会被压缩，同样的，可以设置页面的高度为document.getClientHeight来解决
- 在vue-cli的webpack模板中使用postcss，无论如何配置，display:flex的前缀都添加不对，导致在ios8下显示异常，但单独使用postcss处理工程中的css或者在postcss的网页工具中处理，却能正确添加前缀。猜测是因为vue 2.3之后会自动为display:flex这样的属性添加前缀，但根据文档，vue只是针对v-bind:style添加到元素上的样式做处理，详见多[重值](https://cn.vuejs.org/v2/guide/class-and-style.html#多重值)<br>
这里暂时强行解决，在webpack打包完成后，再用postcss为打包后的css添加前缀。在vue-cli工程中的处理为，在build/build.js中，在webpack打包完成的回调函数中添加如下代码：
  ```javascript
        var fs = require("fs");
        var postcss = require('postcss');
        var autoprefixer = require('autoprefixer');
        var zlib = require('zlib');
        // dist中的css路径
        var path = './dist/static/css';
        // 读取css文件夹中的文件
        fs.readdir(path, function (err, files) {
          // 第一个文件即为输出的css
            var result_file = path + '/' + files[0];
            // 读取css
            fs.readFile(result_file, function (err, data) {
              // 添加前缀
                postcss([autoprefixer({ browsers: ['Last 20 versions'] })])
                    .process(data.toString()).then(result => {
                      // 输出添加前缀后的css
                    fs.writeFile(result_file, result.css);
                    // gzip压缩css
                    zlib.gzip(result.css, (error, gzip)=>{
                        fs.writeFile(result_file + '.gz', gzip, ()=>{
                            console.log(chalk.cyan('  autoprefix css complete.\n'))
                            console.log(chalk.cyan('  gzip css complete.\n'))
                            console.log(chalk.cyan('  Build complete.\n'))
                            console.log(chalk.yellow(
                                '  Tip: built files are meant to be served over an HTTP server.\n' +
                                '  Opening index.html over file:// won\'t work.\n'
                            ))
                        });
                    });
                });
            });
        });  
  ```
- ios下上传图片，本地预览是正确显示的，上传到服务器后，页面再把图片请求回来显示，可能因为图片orientation值的不同导致图片是倒置的。解决代码[将图片旋转到正确的角度](https://github.com/Youjingyu/Code-Collection/blob/master/image-processing/resetImgOrientation.js)
- outline不能单独设置某一边，可以用css3的box-shadow模拟，也可以用下面代码hack：
```css
    .element:before {
           content: "\a0";
           display: block;
           padding: 2px 0;
           line-height: 1px;
           border-top: 1px dashed #000; 
         }
```
- 只有a、button、input类标签才有blur、focus时间，希望其他元素有这两个事件，需要在元素上添加tabindex（有兼容性问题），其值在0到32767之间，数字代表被tab键遍历到的顺序，0代表不会被表遍历到。
- 输入框opacity为0时，ie仍然可以可见光标，设置输入框的color: transparent，无效，设置text-indent: -999em，能够达到效果。
- chrome和firefox支持outline: #ccc auto 1px 的写法，但在ie和微信浏览器下无效，必须制定外边框线类型，如outline: #ccc solid 1px
- for in 会遍历原型链，Object.keys()与for in的区别是不会遍历原型链，两者都不可遍历不可枚举属性；Object.getOwnPropertyNames可以获取对象自身的可枚举和不可枚举属性。
- video不能播放以链接返回的视频，如https://v.qq.com/iframe/player.html?vid=p0553hnmh8g&tiny=0&auto=0，可以使用flash播放，或者嵌入iframe：
```html
<iframe frameborder="0" width="640" height="498" src="https://v.qq.com/iframe/player.html?vid=p0553hnmh8g&tiny=0&auto=0" allowfullscreen></iframe>></iframe>
```
- 可以使用如下方式实现垂直居中：
```css
    .parent{
        position: relative;
    }
    .child{
        position: absolute;
        top: 0;
        bottom: 0;
        margin: auto 0;
    }
```
- element ui的Select组件，手动设置值时，数组类型必须和option的value的数据类型一致，否则会显示为设置的值而不是值对应的label
- 使用display: table; display: table-cell;vertical-align: bottom;可以实现某个子元素高度增加，但所有子元素仍然基线对齐。
- 使用rem和border-radius: 50%实现圆形，在小分辨率下会变成椭圆。可以在小分辨率下使用固定像素解决。
- 移动端webview很多都有导航栏（如微信和qq），设计整屏页面时，需要扣除导航高度（如qq内置浏览器，顶部占用150px，底部占用260px，750\*1334的设计稿最终为750*1074）；一种设计是，页面主体内容不要太高，剩余部分用纯色填充，或者用绝对定位元素填充，实现自适应。
- 二维码要用img引入，背景图长按不能识别为二维码。
- qq中设置分享链接预览信息，参考文档[手机QQ接口文档：setShareInfo](http://open.mobile.qq.com/api/mqq/index#api:setShareInfo)：
    ```html
    <title>QQ中链接的标题由此处获取</title>
    <meta name="description" content="QQ中链接的描述由此处获取">
    <!-- QQ默认获取的图片有可能出现缩放问题，效果不佳，可以通过如下方法进行设置 -->
    <meta itemprop="image" content="http://*.*.com/static/images/share.png" />
    ```
    如果预览中没有正确显示预览图片，尝试将页面链接从somedomain/ 或者 somedomain/index，修改为 somedomain/index.html。
- 微信中需要保证视区内只有一个二维码，否则只能识别出第一个二维码。
- 微信中，使用meta缩放页面后，不能识别二维码:
    ```html
    <!--同一张二维码图片-->
    <!--下面这张 opacity 为 0，隐藏起来，但是实际存在，并且宽为 100%，屏幕有多大就多大-->
    < img style="right:0; top:0; height: auto;width: 100%;opacity: 0;position: absolute;" src="二维码图片地址">
    <!--下面这张是呈现给用户看的-->
    < img src="二维码图片地址" title="qrcode" alt="qrcode">
    <!--PS: img 标签前面的空格记得去掉，这里加上空格是因为简书有 bug，针对 img 标签代码渲染会出错-->
    ```
- 使用meta标签缩放页面
    ```javascript
    <!-- 以下代码默认设计稿尺寸为 640 x 1134 -->
    <meta id="viewport" content="width=device-width, user-scalable=yes,initial-scale=1" name="viewport" />
    <script>
        var detectBrowser = function(name) {
            if(navigator.userAgent.toLowerCase().indexOf(name) > -1) {
                return true;
            } else {
                return false;
            }
        };
        var width = parseInt(window.screen.width);
        var scale = width/640;  // 根据设计稿尺寸进行相应修改：640=>?
        var userScalable = 'no';
        if(detectBrowser("qq/")) userScalable = 'yes';
        document.getElementById('viewport').setAttribute(
                'content', 'target-densitydpi=device-dpi,width=640,user-scalable='+userScalable+',initial-scale=' + scale); // 这里也别忘了改：640=>?
    </script>
    ```
- vue+vux的项目中，如果使用了未定义的事件方法会报错
```TypeError: Cannot read property '_withTask' of undefined```
让人非常迷茫，代码如下：
```vue
<template>
    <div class="report-content">
        <div class="ger-loading" v-show="isError" @click="REPORT_REGET">加载失败，点击重试</div>
    </div>
</template>
<script type="text/javascript">
  import vuex from 'vuex';
  const mapActions = vuex.mapActions;
  export default {
    methods:{
      ...mapActions([
        'RENDER_CHARTS'
      ])
    }
  }
</script>
```
- es6中，如果用如下方式使用class，可能会报错```Class constructor Config$1 cannot be invoked without 'new'```
```javascript
import superReport from './superReport'
let Report = ( supperclass ) => class extends supperclass {
  constructor( options ) {
    super(options)
  }
}
new (Report(superReport))();
```
如果superReport来自node_modules，会报错，否则不会报错。  
这是由es6 class的运行机制导致，可以修改babel的配置为```"presets": [ "es2015-node5" ]```解决  
参考https://github.com/babel/babel/issues/4269
- vmware安装centos，选择配置系统iso后，开机依然system not found：在设置 -> CD/DVD -> 开启启动时连接。
- vmware安装centos过程中，f12为确定并到下一屏，相当于next
- 安装为了在虚拟机与主机间复制粘贴，需要安装vmware tools，安装方法http://blog.csdn.net/programmer_sir/article/details/46626409