---
layout: blog
istop: true
title: "ie 下 placeholder默认值问题"
background-image: 
date:  2017-06-10
category: js
tags:
- 前端
---

$(document).ready(function(){
    // 判断浏览器是否支持placeholder属性
    function isSupportPlaceholder() {
        var input = document.createElement('input');
        return 'placeholder' in input;
    }
    // jQuery替换placeholder的处理
    function input(obj, val) {
        var v = obj.attr('placeholder');
        if(v === val){
            obj.attr('placeholder','');
        }
    }

    if(isSupportPlaceholder() == false){
        input($('#stime'), $('#stime').attr("placeholder"));
        input($('#etime'), $('#etime').attr("placeholder"));
        input($('#keywords'), $('#keywords').attr("placeholder"));
        input($('#fuwushang'), $('#fuwushang').attr("placeholder"));
    }

});

过滤掉 默认为空
