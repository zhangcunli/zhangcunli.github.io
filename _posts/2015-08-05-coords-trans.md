---
layout: post
title: Coords transform
tags: Coords transform
categories: Work
---

<div class="toc"></div>


国内各地图使用的坐标系标准及距离计算公式及相互转换方法


##1.坐标系标准
坐标系的国际标准是 GPS 使用的 WGS84，以经纬度表示地球表面上的一个点。
但是在我国，政府了出于安全考虑，国内的所有导航电子地图必须使用国家测绘局制定的加密坐标系统，也就是将一个真实的经纬度加密成一个不正确的经纬度坐标，国际标准我们称为地球坐标，中国标准我们称为火星坐标。
* 国际标准： WGS-84
* 中国标准： GCJ-02

##2.国内各地图使用的坐标系
国内常见地图，如google中国地图、搜搜地图等使用的是中国标准 GCJ-02，百度在GCJ-02的基础上又进行了二次加密，形成了BD-09坐标系。
其它地图同百度类似，都是在GCJ-02进行加密处理，参考下表：

|   API 坐标系       | 坐标类型  |
| ------------------|----------:|
| 百度地图API        | 百度坐标  |
| 腾讯搜搜地图API    | 火星坐标   |
| 搜狐搜狗地图API    | 搜狗坐标*  |
| 阿里云地图API      | 火星坐标   |
| 图吧MapBar地图API  | 图吧坐标   |
| 高德MapABC地图API  | 火星坐标   |
| 灵图51ditu地图API  | 火星坐标   |


##3.坐标系转换工具
python lib： https://github.com/zxteloiv/pycoordtrans
~~~
./coordtrans gcj02 bd09ll 116.306411 39.981839
~~~
将坐标从中国标准转换成百度坐标
~~~
./coordtrans bd09ll gcj02 112.406411 39.781839
~~~

将坐标从百度坐标转换成中国标准
要和 WGS-84 转换，只需要将参数换成 wgs84 即可，该工具是 python 包，可导入python脚本直接使用

##4.距离计算公式
各种坐标系都可使用同一公式计算球面距离，误差非常小，只要坐标系相同即可。

~~~
#define PI  3.14159265
#define EARTH_RADIUS  6378137
#define RAD  (PI/180.0)
int get_distance(
    double lat1, 
    double lng1, 
    double lat2, 
    double lng2) 
{
        double radLat1 = lat1 * RAD;
        double radLat2 = lat2 * RAD;
        double a = radLat1 - radLat2;
        double b = (lng1 - lng2) * RAD;
        double s = 2  * asin(  sqrt( pow(sin(a / 2), 2) + cos(radLat1) * cos(radLat2) * pow(sin(b / 2), 2))); 
        s = s * EARTH_RADIUS;
        return static_cast<int>(s);
}
~~~

//另一种优化后的公式：
~~~
#define EARTH_RADIUS_IN_METERS  6378137.0
#define RATIO_0 0.9970707597894937
#define RATIO_1 0.0004532759255277588
#define RATIO_2  -0.00017587656744607181
#define RATIO_3 0.0000005028600490023173
double radian(double d)
{
    return d * PI / 180.0; 
}

double get_distance_lbs(
    double lat1, 
    double lng1, 
    double lat2, 
    double lng2)
{
    //1) 
    double dx = lng1 - lng2; 
    double dy = lat1 - lat2; 
    double b = (lat1 + lat2) / 2.0; 
    
    //2) 
    double Lx = (RATIO_3 * b * b * b + RATIO_2 * b * b  + RATIO_1 * b + RATIO_0 ) * radian(dx) *EARTH_RADIUS_IN_METERS; 
    double Ly = EARTH_RADIUS_IN_METERS * radian(dy);
    
    //3)
    return sqrt(Lx * Lx + Ly * Ly);
}
~~~
