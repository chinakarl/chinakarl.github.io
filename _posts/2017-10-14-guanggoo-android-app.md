---
layout: post
title: html5 canvas 简单的那些事儿
categories: html
description: html5
keywords: html5, canvas
---

var canvas = document.getElementById("canvas");
var cxt = canvas.getContext("2d");
//画线(用线画三角型)
cxt.strokeStyle = "red";//线作色
cxt.lineTo(59, 59);//线的起点
cxt.lineTo(20, 20);
cxt.lineTo(59, 20);
cxt.closePath();//可以用closePath()将你画的图形封闭起来
cxt.stroke();
    
//画圆
//如果是rgba(1,2,3,4)第四个参数是不透明度
//绘制实心圆
cxt.beginPath();
cxt.arc(100, 100, 50, 0, Math.PI * 2, false); //arc(起始x，起始y，半径，起始角度，终止角度，是否是逆时针)
cxt.closePath();
cxt.fillStyle = "rgb(250,0,0)";
cxt.fill();
//绘制空心圆
cxt.beginPath();
cxt.arc(200,100, 50, Math.PI * 2, false); //arc(x,y,半径,)
cxt.closePath();
cxt.strokeStyle = "rgb(250,0,0)";
cxt.stroke();
//画圆弧(画空心圆弧和空心圆一样同上)
cxt.beginPath();
cxt.strokeStyle = colors[1];
cxt.beginPath();
cxt.moveTo(200,200);//画圆弧的时候必须先给个起点，不然会找不到起点,会画不出圆来。用lineTo(x,y)也一样
cxt.arc(200, 200, 50, Math.PI*0, Math.PI/3, false);//arc(起始x，起始y，半径，起始角度，终止角度，是否是逆时针)
cxt.closePath();
cxt.fill();
//设置阴影部分
cxt.beginPath();
cxt.fillStyle = "rgb(0,255,255)";
cxt.shadowOffsetX = 30; //设置水平位移,就是阴影部分的宽度
cxt.shadowOffsetY = 30; // 设置垂直位移 ,就是阴影部分的长度
cxt.shadowBlur = 10; // 设置模糊度,值越大模糊越厉害。
cxt.shadowColor = "rgba(0,0,0,0.5)"; //设置阴影部分颜色
cxt.fillRect(300, 300, 100, 100);
cxt.closePath();
cxt.fill();
//设置字体，并设置艺术字效果
cxt.beginPath();
cxt.font = "Bold 40px Arial";//设置字体样式
cxt.textAlign = "left";// 设置对齐方式
cxt.fillStyle = "#008600"; // 设置填充颜色
cxt.shadowOffsetX =4; //设置水平位移,就是阴影部分的宽度
cxt.shadowOffsetY =5.5; // 设置垂直位移 ,就是阴影部分的长度
cxt.shadowBlur = 7; // 设置模糊度,值越大模糊越厉害。
cxt.shadowColor = "rgba(0,0,0,0.5)"; //设置阴影部分颜色
cxt.fillText("Hello!", 600, 400); // 设置字体内容，以及在画布上的位置fillText(字体,x坐标,y坐标)
cxt.strokeText("Hello!", 740, 400); // 绘制空心字
cxt.closePath();
//设置渐变
var myGradient = cxt.createLinearGradient(500, 100, 600, 250);//createLinearGradient(起始x,起始y,终止x,终止y)
myGradient.addColorStop(1, "#ffffff");//addColorStop(0是起始1是终止颜色,颜色)
myGradient.addColorStop(0, "#000000");
cxt.fillStyle = myGradient;//用myGradient变量来填充矩形颜色
cxt.fillRect(500, 100, 100, 150);//fillRect()绘制矩形
cxt.closePath();
cxt.fill();
