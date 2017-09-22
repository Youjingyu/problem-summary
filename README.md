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
