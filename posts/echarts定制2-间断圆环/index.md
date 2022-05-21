# Echarts定制-间断圆环

<!--more-->


# Echarts定制- 间断圆环
## 一、基础学习

B站有超详细的基础教学视频，这里我推荐

黑马程序员pink老师[https://www.bilibili.com/video/BV1v7411R7mp](https://www.bilibili.com/video/BV1v7411R7mp)
非常的详细，风格幽默

charts社区:
[https://www.makeapie.com/explore.html](https://www.makeapie.com/explore.html)
这里有超多的炫酷效果

## 二、间断圆环


```js
//圆环的间断效果就是通过每个数据中间插一个值比较小的数据，
//并且将这个数据的样式改为背景色，这样就实现了间断的效果
var data=[
                 {value: 178,
                 name: '1-20岁',
                 itemStyle:{
                     normal: {
                     borderWidth: 5,
                     shadowBlur: 20,
                     borderColor:'#00ffff',
                     shadowColor: '#00ffff'
                    }
                   }
                 },
                 //实现间断的数据
                 {value: 50,//值根据自己需要的效果调大小
                 name: '',
                 itemStyle: {
                     normal: {
                     label: {
                         show: false
                     },
                     labelLine: {
                         show: false
                     },
                     //设置间断的颜色样式
                     color: 'rgba(0, 0, 0, 0)',
                     borderColor: 'rgba(0, 0, 0, 0)',
                     borderWidth: 0
                     }
                 }
                 },
                  {value: 1335,
                 name: '20-30岁',
                 itemStyle:{
                     normal: {
                     borderWidth: 5,
                     shadowBlur: 20,
                     borderColor:'#006ced',
                     shadowColor: '#006ced'
                    }
                   }
                 },
                 {value: 50,
                 name: '',
                 itemStyle: {
                     normal: {
                     label: {
                         show: false
                     },
                     labelLine: {
                         show: false
                     },
                     color: 'rgba(0, 0, 0, 0)',
                     borderColor: 'rgba(0, 0, 0, 0)',
                     borderWidth: 0
                     }
                 }
                 },
                  {value: 457,
                 name: '30-40岁',
                 itemStyle:{
                     normal: {
                     borderWidth: 5,
                     shadowBlur: 20,
                     borderColor:'#ffa800',
                     shadowColor: '#ffa800'
                    }
                   }
                 },
                 {value: 50,
                 name: '',
                 itemStyle: {
                     normal: {
                     label: {
                         show: false
                     },
                     labelLine: {
                         show: false
                     },
                     color: 'rgba(0, 0, 0, 0)',
                     borderColor: 'rgba(0, 0, 0, 0)',
                     borderWidth: 0
                     }
                 }
                 },
                  {value: 567,
                 name: '40岁以上',
                 itemStyle:{
                     normal: {
                     borderWidth: 5,
                     shadowBlur: 20,
                     borderColor:'#ff5b00',
                     shadowColor: '#ff5b00'
                    }
                   }
                 },
                 {value: 50,
                 name: '',
                 itemStyle: {
                     normal: {
                     label: {
                         show: false
                     },
                     labelLine: {
                         show: false
                     },
                     color: 'rgba(0, 0, 0, 0)',
                     borderColor: 'rgba(0, 0, 0, 0)',
                     borderWidth: 0
                     }
                 }
                 },

             ];

 option = {
     backgroundColor: '#0A2E5D',
     series: [
         {
     name: '',
     type: 'pie',
     clockWise: false,
     radius: [55, 59],//设置圆环的内外半径
     hoverAnimation: true,
     itemStyle: {
         normal: {
             label: {
                 show: true,
                 position: 'outside',
                 color: '#ffffff',
                 fontSize :8,
                 //设置提示信息
                 formatter: function(params) {
                                     var percent = 0;
                                     var total = 0;
                                     for (var i = 0; i < 8; i=i+2) {
                                         total += data[i].value;
                                     }
                                     percent = ((params.value / total) * 100).toFixed(0);
                                     if(params.name !== '') {
                                         return '年龄段：' + params.name + '\n' + '\n' + '占百分比：' + percent + '%';
                                     }else {
                                         return '';
                                     }
                                 },
             },
             labelLine: {
                 length:10,
                 length2:20,
                 show: true,
                 color:'#00ffff'
             }
         }
     },
             data:data
         }
     ]
 };
```


