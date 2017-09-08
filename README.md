# problem-summary
平时遇到的古怪问题的总结

- 功能键在非Firefox浏览器不触发keypress事件的问题
	- 比如，按下backspace键在Firefox下会触发keydown、keypress事件，而在chrome下只会触发keydown事件
	- 参考[键盘事件keydown、keypress、keyup随笔整理总结](http://www.cnblogs.com/xcsn/p/3413074.html)
- 如果canvas中绘制了跨域图片，canvas.toDataURL会报错，可以用img.setAttribute('crossO0rigin', 'Anonymous')解决，但前提是服务器允许跨域请求图片
	
	
