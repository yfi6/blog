---
layout: blog
istop: true
title: "-javaScript跨浏览器事件处理程序"
background-image: 
date:  2017-10-10
category: js
tags:
- 前端
---

最近在阅读javascript高级程序设计，事件这一块还是有很多东西要学的，就把一些思考和总结记录下。
在事件处理，事件对象，阻止事件的传播等方法或对象存在着浏览器兼容性问题，开发过程中最好编写成一个通用的事件处理工具。
(function(){
    var EU = {};
    //...
    //在这里添加一些通用的事件处理方法
    //...
    window.EventUtil = EU;
})();
事件处理程序
事件的绑定主要为IE8以下浏览器做兼容处理：
IE8以下只支持事件冒泡
IE事件处理使用attachEvent detachEvent
绑定事件
EU.addHandler = function(element,type,handler){
    //DOM2级事件处理，IE9也支持
    if(element.addEventListener){
        element.addEventListener(type,handler,false);
    }
    else if(element.attachEvent){
        //type加'on'
        //IE9也可以这样绑定
        element.attachEvent('on' + type,handler);
    }
    //DOM0级事件处理步，事件流也是冒泡
    else{
        element['on' + type] = handler;
    }
};
取消绑定事件
和绑定事件的处理基本一致，有一个注意点：
传入的handler必须与绑定事件时传入的相同（指向同一个函数）
EU.removeHandler = function(element,type,handler){
    if(element.removeEventListener){
        element.removeEventListener(type,handler);
    }
    else if(element.attachEvent){
       element.detachEvent('on' + type,handler);
    }
    else{
        //属性置空就可以
        element['on' + type] = null;
    }
};
事件对象
注意点：
IE下event是全局对象，通过window.event取得
EU.getEvent = function(event){
    return event || window.event;
}
事件的目标
注意点:
IE下通过attachEvent绑定事件，内部this并非触发事件的DOM,而是window;
通过目标对象来获取DOM节点，IE下是srcElement属性，等同于其他浏览器的target属性
EU.addTarget = function(event){
    return event.target || event.srcElement;
}
阻止默认事件
EU.preventDefault = function(event){
    if(event.preventDefault){
        event.preventDefault();
    }
    //IE下处理
    else{
        event.returnValue = false; //默认为true
    }
}
关于事件默认行为
阻止事件传播
EU.stopPropagation = function(event){
    if(event.stopPropagation){
        event.stopPropagation();
    }
    //IE下处理
    else{
        event.cancelBubble = true;//默认为false，注意区分于returnValue
    }
}
注意点：
阻止的是DOM层级间的事件传播
比如：对于一个DOM元素，同时绑定捕获事件与冒泡事件，如果在捕获阶段使用stopPropagation,不会阻断冒泡事件的执行；(事件捕获早于事件冒泡)
Demo地址：http://jsfiddle.net/0sfck15b/
如果对子元素和父元素以冒泡形式都绑定'click'事件，在子元素的事件处理中使用stopPropagation阻止事件传播,父元素绑定的click事件不会执行。
Demo地址：http://jsfiddle.net/av6yebsw/
上面的划掉的地方理解有误，更正下。上面的demo中事件的执行都发生了目标阶段,事件对象event的eventPhase属性用来表示事件处理发生在事件流哪个阶段。
对应关系 1：捕获阶段,2: 处于目标,3: 冒泡阶段
还有一点:
