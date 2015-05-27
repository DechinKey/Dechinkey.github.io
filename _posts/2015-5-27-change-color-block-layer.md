---
date: 2015-05-27
layout:     post
comments:   yes
code:        yes
title:      利用JS实现四个色块的层级设置和透明度设置
category:   blog
tags: [JavaScript,html,css,jQyery,frontend]
---
>**问题描述:利用JS实现四个色块的层级设置和透明度设置，如下图所示**

![问题描述](http://media-clark.qiniudn.com/color-layer.png)

**主要原理:在文档靠后的元素会覆盖前面的,剩下的就是insertBefore()和appendChild()的运用了。insertBefore()和appendChild()方法配合可以把节点方便地插入到DOM树的任意位置**


>**成品链接:** [点这里](http://notes.dengck.com/projects/change-color-block-layer.html)

**代码实现:**

**原生JS版**

    <!DOCTYPE html>
	<html lang="en">
	<head>
    <meta charset="UTF-8">
    <title>用JS设置色块的层级</title>
	</head>
	<body>
    <div class="main">
        <div class="abox box"></div>
        <div class="bbox box"></div>
        <div class="cbox box"></div>
        <div class="dbox box"></div>
    </div>
    <lable>透明度</lable>
    <input type="number" min="0" max="100" step="1">
    <input type="button" value="设置">
    <br/>
    <input type="button" value="上移一层">
    <input type="button" value="下移一层">
    <input type="button" value="切换到顶部">
    <input type="button" value="切换到底部">

    <script>
        window.onload = function() {
            var box = document.querySelectorAll('.box')

            //选中的元素添加selected类
            for (var i = 0;i < box.length;i ++) {
                box[i].addEventListener('click', function() {
                    for (var i = 0;i < box.length;i ++) {
                        box[i].classList.remove('selected')  //先全部清除，再单独为选中的元素添加类
                    }

                    this.classList.add('selected')
                }, false)
            }

            //设置透明度
            var set = document.querySelector('[value=设置]')
            set.onclick = function() {
                var opacity = (document.querySelector('[type=Number]').value)/100
                document.querySelector('.selected').style.opacity = opacity
            }

            //上移
            var up = document.querySelector('[value=上移一层]')
            up.onclick = function() {
                var selected = document.querySelector('.selected')

                if (selected.nextElementSibling != null) {
                    selected.parentNode.insertBefore(selected.nextElementSibling, selected)
                }
            }

            //下移
            var down = document.querySelector('[value=下移一层]')
            down.onclick = function() {
                var selected = document.querySelector('.selected')

                if (selected.previousElementSibling != null) {
                    selected.parentNode.insertBefore(selected, selected.previousElementSibling)
                }
            }

            //置顶
            var top = document.querySelector('[value=切换到顶部]')
            top.onclick = function() {
                var selected = document.querySelector('.selected')

                selected.parentNode.appendChild(selected)
            }

            //置底
            var bottom = document.querySelector('[value=切换到底部]')
            bottom.onclick = function() {
                var selected = document.querySelector('.selected')

                selected.parentNode.insertBefore(selected, selected.parentNode.firstElementChild)
            }
        }
        //alert(ok);
    </script>

    <style>
        .main {
            position: relative;
            overflow: hidden;
            width: 400px;
            height: 400px;
        }
        .abox {
            position: absolute;
            width: 100px;
            height: 200px;
            left: 40%;
            top: 40%;
            background-color: #9c47a3;
        }
        .bbox {
            position: absolute;
            width: 200px;
            height: 100px;
            left: 45%;
            top: 25%;
            background-color: #25b3ff;
        }
        .cbox {
            position: absolute;
            width: 100px;
            height: 200px;
            left: 35%;
            top: 5%;
            background-color: #edf51b;
        }
        .dbox {
            position: absolute;
            width: 200px;
            height: 100px;
            left: 6%;
            top: 35%;
            background-color: #20d658;
        }
        .box{
        cursor: pointer;
        }
        .selected::before {
            content: 'selected!';
            position: absolute;
            left: 0;
            top: 0;
        }
    </style>
	</body>
	</html>

**jQuery版**

	<script>
    $(document).ready(function(){
        //选中的元素添加selected类
        $(".box").bind("click",function(){
            
                    $(".box").removeClass("selected");
                    //alert("ok");
                
            $(this).addClass("selected");
        });
        
         //设置透明度
         $("#b1").click(function(){
            var opacity = parseInt($("#alphalnp").val())/100;
            $(".selected").css("opacity",opacity);

         });

         
         //上移
         $("#b2").click(function(){
            if($(".selected").next().length){
                //alert("ok");
                $(".selected").next().insertBefore($(".selected"));
            }
         });

         //下移
         $("#b3").click(function(){
            if($(".selected").prev().length){
                //alert("ok");
                $(".selected").insertBefore($(".selected").prev());
            }
         });

         //切换到顶部
         $("#b4").click(function(){
            $(".selected").parent().append($(".selected"));
         });

         //切换到底部
         $("#b5").click(function(){
            $(".selected").insertBefore($(".selected").parent().children(":first"));
         }); 
    }); 
 	</script>

