# Echarts定制-滑动条

<!--more-->




```js
//主要是配置dataZoom
option = {
    backgroundColor: '#0A2E5D',
    tooltip: {
        trigger: "axis",
        axisPointer: {
            lineStyle: {
                color: "#dddc6b"
            }
        }
    },
    legend: {
        top: "10%",
        textStyle: {
            color: "rgba(255,255,255,.5)",
            fontSize: "12"
        }
    },
    grid: {
        left: "50",
        top: "100",
        right: "50",
        bottom: "100",
        containLabel: true
    },

    xAxis: [{
            type: "category",
            boundaryGap: false,
            axisLabel: {
                textStyle: {
                    color: "rgba(255,255,255,.6)",
                    fontSize: 12
                }
            },
            axisLine: {
                lineStyle: {
                    color: "rgba(255,255,255,.2)"
                }
            },

            data: [
                "01",
                "02",
                "03",
                "04",
                "05",
                "06",
                "07",
                "08",
                "09",
                "11",
                "12",
                "13",
                "14",
                "15",
                "16",
                "17",
                "18",
                "19",
                "20",
                "21",
                "22",
                "23",
                "24",
                "25",
                "26",
                "27",
                "28",
                "29",
                "30"
            ]
        },
        {
            axisPointer: {
                show: false
            },
            axisLine: {
                show: false
            },
            position: "bottom",
            offset: 20
        }
    ],

    yAxis: [{
        type: "value",
        axisTick: {
            show: false
        },
        axisLine: {
            lineStyle: {
                color: "rgba(255,255,255,.1)"
            }
        },
        axisLabel: {
            textStyle: {
                color: "rgba(255,255,255,.6)",
                fontSize: 12
            }
        },

        splitLine: {
            lineStyle: {
                color: "rgba(255,255,255,.1)"
            }
        }
    }],
    //缩放组件
    dataZoom: [
        //滑动条
        {
            show: true,
            height: 20,
            xAxisIndex: [0],
            //调整缩放组件位置
            //top bottom left right
            bottom: 75,
            type: "slider",
            "start": 20,
            "end": 80,
            handleIcon: 'path://M306.1,413c0,2.2-1.8,4-4,4h-59.8c-2.2,0-4-1.8-4-4V200.8c0-2.2,1.8-4,4-4h59.8c2.2,0,4,1.8,4,4V413z',
            handleSize: '110%',
            handleStyle: {
                color: "#5B3AAE",
            },
            textStyle: {
                color: "rgba(204,187,225,0.5)",
            },
            fillerColor: "rgba(67,55,160,0.4)",
            borderColor: "rgba(204,187,225,0.5)",

        },
        //内置于坐标系中，使用户可以在坐标系上
        //通过鼠标拖拽、鼠标滚轮、手指滑动（触屏上）来缩放或漫游坐标系。
        {
            type: "inside",
            show: true,
            height: 15,
            start: 1,
            end: 35
        }
    ],
    series: [{
            name: "播放量",
            type: "line",
            smooth: true,
            symbol: "circle",
            symbolSize: 5,
            showSymbol: false,
            lineStyle: {
                normal: {
                    color: "#0184d5",
                    width: 2
                }
            },
            areaStyle: {
                normal: {
                    color: new echarts.graphic.LinearGradient(
                        0,
                        0,
                        0,
                        1,
                        [{
                                offset: 0,
                                color: "rgba(1, 132, 213, 0.4)"
                            },
                            {
                                offset: 0.8,
                                color: "rgba(1, 132, 213, 0.1)"
                            }
                        ],
                        false
                    ),
                    shadowColor: "rgba(0, 0, 0, 0.1)"
                }
            },
            itemStyle: {
                normal: {
                    color: "#0184d5",
                    borderColor: "rgba(221, 220, 107, .1)",
                    borderWidth: 12
                }
            },
            data: [
                30,
                40,
                30,
                40,
                30,
                40,
                30,
                60,
                20,
                40,
                20,
                40,
                30,
                40,
                30,
                40,
                30,
                40,
                30,
                60,
                20,
                40,
                20,
                40,
                30,
                60,
                20,
                40,
                20,
                40
            ]
        },
        {
            name: "转发量",
            type: "line",
            smooth: true,
            symbol: "circle",
            symbolSize: 5,
            showSymbol: false,
            lineStyle: {
                normal: {
                    color: "#00d887",
                    width: 2
                }
            },
            areaStyle: {
                normal: {
                    color: new echarts.graphic.LinearGradient(
                        0,
                        0,
                        0,
                        1,
                        [{
                                offset: 0,
                                color: "rgba(0, 216, 135, 0.4)"
                            },
                            {
                                offset: 0.8,
                                color: "rgba(0, 216, 135, 0.1)"
                            }
                        ],
                        false
                    ),
                    shadowColor: "rgba(0, 0, 0, 0.1)"
                }
            },
            itemStyle: {
                normal: {
                    color: "#00d887",
                    borderColor: "rgba(221, 220, 107, .1)",
                    borderWidth: 12
                }
            },
            data: [
                50,
                30,
                50,
                60,
                10,
                50,
                30,
                50,
                60,
                40,
                60,
                40,
                80,
                30,
                50,
                60,
                10,
                50,
                30,
                70,
                20,
                50,
                10,
                40,
                50,
                30,
                70,
                20,
                50,
                10,
                40
            ]
        }
    ]
};
```


