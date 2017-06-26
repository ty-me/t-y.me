---
id: 373
title: 微信小程序三级地址联动选择器的实现
date: 2017-06-02T12:03:21+00:00
author: TY
layout: post
comments: true
permalink: /p/373
categories:
  - WEB
tags:
  - 小程序
  - js
  - 微信
 
---

地址联动选择功能在web开发中是很常见的一个功能，在小程序中可以通过`picker`控件来实现，效果如下：

![](http://7o51e6.com1.z0.glb.clouddn.com/addressPicker.gif)

虽然`picker`空间在具体功能上跟web的`select`有所区别，但是实现联动的原理和思路都是一样的：在上一级所选值改变之后，修改并刷新下一级的可选项。

## step 1

首先需要一个三级行政区的数据库，通过搜索我在github上找到了[这个](https://raw.githubusercontent.com/ewen0930/china-citylist/master/data/行政区划代码.json)。在小程序启动的时候，将地址数据拉取下来并放入本地存储,在app.js中增加如下代码：


```
var app = getApp()

Page({

  data: {

    // 地址数据库
    addressData: [], 
    
    // 选择器数据源
    provinceArray: [],
    cityArray: [],
    districtArray: [],
    
    // 选择器选中结果的索引
    provinceIndex: 0,
    cityIndex: 0,
    districtIndex: 0,
    
    // 所选结果
    province:'',
    city:'',
    district:'',

  },

  loadAddressData: function(){
    // 拉取地址数据库保存到本地
    // TODO 网络异常处理
    wx.request({
      url: 'https://raw.githubusercontent.com/ewen0930/china-citylist/master/data/行政区划代码.json',
      success:function(res){
        console.log(res);
        if(res.statusCode != 200){
          return false
        }
        wx.setStorageSync(addressDataKey, res.data.province)
      }
    })
  },

  onLaunch: function () {
    var addressData = wx.getStorageSync(addressDataKey)
    if(!addressData){
      this.loadAddressData()
    }
  },
```

## step 2
在wxml文件中，放3个picker控件，分别表示省市区三级的选择器

```
        <picker bindchange="provinceChange" value="{{ provinceIndex }}" range="{{ provinceArray }}" style="width:100%; overflow:hidden;">
            <view class="weui-select weui-select_in-select-after" >{{ provinceArray[provinceIndex] }}</view>
        </picker>  

        <picker bindchange="cityChange" value="{{ cityIndex }}" range="{{ cityArray }}" style="width:100%; overflow:hidden;">
            <view class="weui-select weui-select_in-select-after" >{{cityArray[cityIndex]}}</view>
        </picker>

        <picker bindchange="districtChange"  value="{{ districtIndex }}" range="{{ districtArray }}" style="width:100%; overflow:hidden;">
            <view class="weui-select weui-select_in-select-after">{{districtArray[districtIndex]}}</view>
        </picker>
```

3个picker分别绑定了3个事件，当选项改变的时候会被调用

## step 3

在js文件中实现这3个事件处理方法，主要代码如下

```
  provinceChange: function(e){
    var provinceIndex = parseInt(e.detail.value)
    this.setData({
      provinceIndex: provinceIndex,
    })
    this.refreshPicker()

  },

  cityChange: function(e){
    var cityIndex = e.detail.value
    this.setData({
      cityIndex: cityIndex,
    })
    this.refreshPicker()
  },

  districtChange: function(e){
    var districtIndex = e.detail.value
    this.setData({
      districtIndex: districtIndex,
    })
    this.refreshPicker()

  },

  onLoad: function (params) {
    this.initAddressPicker()

  },

  initAddressPicker: function(){
    // 初始化地址选择器数据
    var addressData = wx.getStorageSync(app.globalData.addressDataKey)
    this.setData({
      addressData: addressData,
    })
    this.setData({     
      provinceIndex: 0,
      cityIndex:0,
      districtIndex:0,
    })
    this.refreshPicker();

  },

  getProvinceArray: function(){
    var provinceArray = []
    this.data.addressData.forEach(function(i){
      provinceArray.push(i.name)
    })
    provinceArray.push('其它')  
    return provinceArray  
  },

  getDistrictArray: function(provinceIndex, cityIndex){
    console.log('this.data.addressData', this.data.addressData)
    console.log('get city array, pindex,', provinceIndex)    
    var districtArray = []
    this.data.addressData[provinceIndex].city[cityIndex].county.forEach(function(i){
      districtArray.push(i.name)
    })
    districtArray.push('其它')   
    return districtArray 
  },

  getCityArray: function(provinceIndex){

    var cityArray = []
    this.data.addressData[provinceIndex].city.forEach(function(i){
      cityArray.push(i.name)
    })
    cityArray.push('其它')   
    return cityArray 
  },

  refreshPicker: function(){
    // 根据index的变动，刷新选择器以及下一级array的数据
    
    var provinceIndex = this.data.provinceIndex
    var cityIndex = this.data.cityIndex
    var districtIndex = this.data.districtIndex

    var provinceArray = this.getProvinceArray()
    var cityArray = this.getCityArray(provinceIndex)
    var districtArray = this.getDistrictArray(provinceIndex, cityIndex)

    console.log('provinceArray ', provinceArray)
    console.log('cityArray ', cityArray)
    console.log('districtArray', districtArray)

    console.log('provinceIndex', provinceIndex)
    console.log('cityIndex ', cityIndex)
    console.log('districtIndex ', districtIndex)


    this.setData({
      provinceArray: provinceArray,
      cityArray: cityArray,       
      districtArray: districtArray,

      provinceIndex: provinceIndex,
      cityIndex: cityIndex,
      districtIndex: districtIndex,

      province:  provinceArray[provinceIndex],
      city: cityArray[cityIndex],
      district: districtArray[districtIndex],
    })    
  },

})

```
这里所贴的代码为部分主要代码片段，并不完整，仅仅用来说明思路。我将整个demo的代码放在了github上，有需要的可以在这里下载得到源代码：[source code](https://github.com/ty-me/cityDddressPickerDemo)
