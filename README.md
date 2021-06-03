# vue-viser-vue 单省份热力图

main.js中需要进行引入
```javascript
import Viser form 'viser-vue'
Vue.user(Viser)
```

非必要不建议使用。
简单的热力图echarts可以通过visualMap实现

echarts中关键配置
```javascript
visualMap: {
    calculable: true,
    min: min,
    max: max,
    inRange: {
        color: [
            '#BAE7FF',
            '#1890FF',
            '#0050B3',
        ]
    }
},

series: [
    {
        type: 'map',
        ...
    }
]
```
