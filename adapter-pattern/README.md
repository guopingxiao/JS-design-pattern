# 适配器模式（Adapter）

> 适配器模式主要用来解决两个已有接口之间不匹配的问题，它不考虑这些接口是怎样实现的，也不考虑它们将来可能会如何演化。适配器模式不需要改变已有的接口，就能够使它们协同作用。

适配器的别名是包装器（`wrapper`），这是一个相对简单的模式。在程序开发中有许多这样的场景：当我们试图调用模块或者对象的某个接口时，却发现这个接口的格式并不符合目前的需求。这时候有两种解决办法，第一种是修改原来的接口实现，但如果原来的模块很复杂，或者我们拿到的模块是一段别人编写的经过压缩的代码，修改原接口就显得不太现实了。第二种办法是创建一个适配器，将原接口转换为客户希望的另一个接口，客户只需要和适配器打交道。

现实中也有很多适配器的例子：电源插座，usb适配器等等；例如iPhone7以后的耳机接口从3.5mm圆孔接口更改成为了苹果专属的 lightning接口。许多人以前的圆孔耳机就需要下面的一个适配器，才能够在自个儿新买的iPhone上面听歌。


## 2. 适配器模式使用场景

### 2.1 接口适配

当我们向`googleMap 和`baiduMap` 都发出“显示”请求时，`googleMap`和`baiduMap` 分别以各自的方式在页面中展现了地图
```javascript
const googleMap = {
  show: function () {
    console.log('开始渲染谷歌地图');
  }
};

const baiduMap = {
  show: function () {
    console.log('开始渲染百度地图');
  }
};

const renderMap = function (map) {
  if (map.show instanceof Function) {
    map.show();
  }
};

renderMap(googleMap); // 输出：开始渲染谷歌地图
renderMap(baiduMap);  // 输出：开始渲染百度地图
```
这段程序得以顺利运行的关键是`googleMap` 和`baiduMap` 提供了一致的show 方法，但第三方的接口方法并不在我们自己的控制范围之内，假如baiduMap 提供的显示地图的方法不叫show 而叫display 呢？

baiduMap 这个对象来源于第三方，正常情况下我们都不应该去改动它,而且有时候我们也改不了它。此时我们可以通过增加baiduMapAdapter 来解决问题：

```javascript
const googleMap = {
  show: function () {
    console.log('开始渲染谷歌地图');
  }
};

const baiduMap = {
  display: function () {
    console.log('开始渲染百度地图');
  }
};

const baiduMapAdapter = {
  show: function(){
    return baiduMap.display();
  }
};

renderMap(googleMap);       // 输出：开始渲染谷歌地图
renderMap(baiduMapAdapter); // 输出：开始渲染百度地图
```

### 2.2 参数的适配

有的情况下一个方法可能需要传入多个参数，例如在做一些监控上报的SDK时，可能采集的数据比较多，这个类中有一个systemInfo，需要传入五个参数用于接收手机的相关信息：

```javascript
class SDK {
  systemInf(brand, os, carrier, language, network) {

    //dosomething.....
  }
}
```

通常在传入的参数大于3的时候，我们就可以考虑将参数合并为一个对象的形式，就像我们$.ajax的做法一样。下面我们可以将systemInf的参数接口定义如下（String代表参数类型，?: 代表可选项）

```javascript
{
  brand: String
  os: String
  carrier:? String
  language:? String
  network:? String
}
```

可以看出，carrier、language，network这三个属性不是必须传入的，它们在方法内部可能被设置一些默认值。所以这个时候我们就可以在方法内部采用适配器来适配这个参数对象。这种方式在我们的插件或者npm包中是常见的；
```javascript
class SDK {
  systemInf(config) {

    let defaultConfig = {
      brand: null,  //手机品牌
      os: null, //系统类型： Andoird或 iOS
      carrier: 'china-mobile', //运营商，默认中国移动
      language: 'zh', //语言类型，默认中文
      network: 'wifi', //网络类型，默认wifi
    }

    //参数适配
    for( let i in config) {
      defaultConfig[i] = config[i] || defaultConfig[i];
    }

    //dosomething.....
  }
}
```

### 2.3 数据的适配
数据的适配在前端中是最为常见的场景，这时适配器在解决前后端的数据依赖上有着重要的意义。通常服务器端传递的数据和我们前端需要使用的数据格式是不一致的，特别是在在使用一些UI框架时，框架所规定的数据有着固定的格式。所以，这个时候我们就需要对后端的数据格式进行适配。

例如网页中有一个使用Echarts折线图对网站每周的uv，通常后端返回的数据格式如下所示：
```javascript

[
  {
    "day": "周一",
    "uv": 6300
  },
  {
    "day": "周二",
    "uv": 7100
  },  {
    "day": "周三",
    "uv": 4300
  },  {
    "day": "周四",
    "uv": 3300
  },  {
    "day": "周五",
    "uv": 8300
  },  {
    "day": "周六",
    "uv": 9300
  }, {
    "day": "周日",
    "uv": 11300
  }
]
```
但是Echarts需要的x轴的数据格式和坐标点的数据是长下面这样的:
```javascript

["周二", "周二", "周三"， "周四"， "周五"， "周六"， "周日"] //x轴的数据

[6300. 7100, 4300, 3300, 8300, 9300, 11300] //坐标点的数据
```
所以这是我们就可以使用一个适配器，将后端的返回数据做适配：

```javascript
//x轴适配器
function echartXAxisAdapter(res) {
  return res.map(item => item.day);
}

//坐标点适配器
function echartDataAdapter(res) {
  return res.map(item => item.uv);
}
```

# 3 总结

适配器模式是一对相对简单的模式。但适配器模式在JS中的使用场景很多，在参数的适配上，有许多库和框架都使用适配器模式；数据的适配在解决前后端数据依赖上十分重要。我们要认识到的是适配器模式本质上是一个"亡羊补牢"的模式，它解决的是现存的两个接口之间不兼容的问题，你不应该在软件的初期开发阶段就使用该模式；如果在设计之初我们就能够统筹的规划好接口的一致性，那么适配器就应该尽量减少使用。

在JavaScript中的适配器更多应用于在对象之间，为了使对象可用，我们通常会将对象拆分并重新组装，这样就必须了解适配对象的内部结构，这也是和外观模式的区别所在，当然是配资模式同样解决了对象之间的耦合度，包装的适配代码增加了一些资源消耗，但这是微乎其微的。

