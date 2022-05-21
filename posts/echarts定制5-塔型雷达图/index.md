# Echarts定制-塔形雷达图

<!--more-->



```javascript
//主要的思路就是，一个八边形的雷达图取最下面两份
//然后将雷达图的分割线等隐藏
option = {
         backgroundColor: '#0A2E5D',
 		color: [  '#ff5b00', '#ffa800', '#006ced','#11A6D0','#00ffff'],
 		radar: {
 			name: {
 				show: false
 			},
 			indicator: [
 			{
 					name: 'A',
 					max: 100
 				},
 				{
 					name: 'B',
 					max: 100
 				},
 				{
 					name: 'C',
 					max: 100
 				},
 				{
 					name: 'D',
 					max: 100
 				},
 				{
 					name: 'E',
 					max: 100
 				},
 				{
 					name: 'F',
 					max: 100
 				},
 				{
 					name: 'G',
 					max: 100
 				},
 				{
 					name: 'H',
 					max: 100
 				}
 			],
 			center: ['50%', '10%'],
 			radius: 300,
 			axisLine: {
 				show: false
 			},
 			splitLine: {
 				show: false
 			},
 			splitArea: {
 				show: false
 			}
 		},
 		series: [
 		    {
 			type: 'radar',
 			areaStyle: {
 				opacity: 1,
 				shadowBlur: 1,
 				shadowColor: 'rgba(0,0,0,.5)',
 			},

 			silent: true,
 			data: [
 			    {
 					value: [0, 0, 0, 100, 100, 99, 0, 0],
 					name: '8k以下',
 					symbol: 'circle',
 					symbolSize: 1,
 					label: {
 						show: true,
 						position: [-5, -10],
 						formatter: function(point) {
 							if(point.value == 99)
 								return "—— " + point.name
 							else
 								return ''
 						},
 					}

 				},
 				{
 					value: [0, 0, 0, 79, 80, 80, 0, 0],
 					name: '8k-10k',
 					symbol: 'circle',
 					symbolSize: 1,
 					label: {
 						show: true,
 						position: [-55, -10],
 						formatter: function(point) {
 							if(point.value == 79)
 								return "" + point.name + " ——"
 							else
 								return ''
 						},
 					}
 				},
 				{
 					value: [0, 0, 0, 60, 60, 59, 0, 0],
 					name: '10k-20k',
 					symbol: 'circle',
 					symbolSize: 1,
 					label: {
 						show: true,
 						position: [-5, -10],
 						formatter: function(point) {
 							if(point.value == 59)
 								return "—— " + point.name
 							else
 								return ''
 						},
 					}
 				},
 				{
 					value: [0, 0, 0, 39, 40, 40, 0, 0],
 					name: '20k-30k',
 					symbol: 'circle',
 					symbolSize: 1,
 					label: {
 						show: true,
 						position: [-55, -10],
 						formatter: function(point) {
 							if(point.value == 39)
 								return "" + point.name + " ——"
 							else
 								return ''
 						},
 					}
 				},
 				{
 					value: [0, 0, 0, 20, 20, 19, 0, 0],
 					name: '30k以上',
 					symbol: 'circle',
 					symbolSize: 1,
 					label: {
 						show: true,
 						position: [-5, -10],
 						formatter: function(point) {
 							if(point.value == 19)
 								return "—— " + point.name
 							else
 								return ''
 						},
 					}
 				}
 			]
 		}],
 		itemStyle: {
 			emphasis: {
 				show: false,
 				shadowBlur: 10,
 				shadowOffsetX: 0,
 				shadowColor: 'rgba(0, 0, 0, 0.5)'
 			}
 		}
 	};
```

