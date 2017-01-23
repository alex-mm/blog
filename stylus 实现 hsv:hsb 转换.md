# stylus 实现 hsv / hsb 转换
## 背景
* 组件做换肤方案，less 有 hsv 函数，stylus 没有

## 换肤方案
* 定义品牌色、中立色等。

```stylus
// 品牌色
$brand-primary = #ff7700; // B1_1
$brand-secondary = #333130; // B1_2
// 中立色
$basic-100 = #fff; // N1_1
$basic-200 = #fafafa; // N1_2
...
```
每套皮肤只需要改变品牌色即可。

```stylus
.t-button-primary
  background $brand-primary
  color $basic-100
  &:active
    background hsvTransform($brand-primary, 0, 4, -1)
```

## 色值模型（色彩空间）介绍
![image](https://zos.alipayobjects.com/skylark/e0df8665-92ce-45f0-b9bb-7074e6ed67c7/attach/2012/d9ccf92c7d0c657f/image.png)
### rgb (Red Green Blue)
* 给机器使用的

![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/03bfcd77028933cfffa307460404891d/image.png)

### hsv / hsb (Hue, Saturation, Value / Brightness)
* 设计使用

![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/1d20aaf5c57838135be5ea733d41adab/image.png)
### hsl / hsi (Hue, Saturation Lightness / Intensity)
* 设计使用

![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/766c7402b85f0db54c093ea536ddb258/image.png)

### CMYK(Cyan Magenta Yellow Black)
* 印刷


![image](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/027335b3c23822bb73b7f4ff375b9fb3/image.png)

## Lab（L表示亮度（Luminosity），a表示从洋红色至绿色的范围，b表示从黄色至蓝色的范围）
* 不常用


![image](https://img.alicdn.com/tps/TB1fB1IOFXXXXbxaXXXXXXXXXXX-480-480.png)

## YPbPr、Xyz and so on.

## why hsv ?
> HSB colors are also more readily available for use than HSL if you choose/get your colors from graphics software. Programs such as GIMP or Adobe’s Photoshop and Illustrator all prefer the HSB color model to HSL.

## 方案
### hsv to hsl to hex
* [参考](https://www.sitepoint.com/hsb-colors-with-sass/)
* stylus 支持 hsl 转换

```stylus
hsv(h-hsv, s-hsv, v-hsv, a = 1)
    if v-hsv == 0
        return hsla(0, 0, 0, a)
    else
        s-hsv = unit(s-hsv, '')
        v-hsv = unit(v-hsv, '')
        l-hsl = (v-hsv / 2) * (2 - (s-hsv / 100))
        s-hsl = (v-hsv * s-hsv) / (l-hsl < 50 ? l-hsl * 2 : 200 - l-hsl * 2)
        return hsla(h-hsv, s-hsl, l-hsl, a)
```
### hex to hsv
* stylus 支持 rgb 转换


#### rgb 与 hsv / hsl 转换

![Hsl-and-hsv.svg](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/e46a95f9ba50e722877eaf32ce010505/Hsl-and-hsv.svg)
![image](https://zos.alipayobjects.com/skylark/e3af9c54-9a2f-4d0e-afe0-293cf94d3031/attach/2012/e0287189d6cdaffb/image.png)


### hex to rgb to hsv

![20150315145318868](http://git.cn-hangzhou.oss.aliyun-inc.com/uploads/vision/render-engine/080030a18459ff787423763dc31e3504/20150315145318868)
![image](https://zos.alipayobjects.com/skylark/07583117-962c-4bd4-b006-4e01f8736439/attach/2012/dec2b06b0fd21df3/image.png)

```
// 伪代码
diff = max-min
// h
h = 0                        if max = min
h / 60 = (g - b) / diff + 0  if max = r && g >= b
h / 60 = (g - b) / diff + 6  if max = r && g < b
h / 60 = (b - r) / diff + 2  if max = g
h / 60 = (r - g) / diff + 4  if max = b

// s
s = 0                        if max = 0
s = diff / max               else

// v
v = max
```


```stylus
// stylus 实现
  r = red(color);
  g = green(color);
  b = blue(color);
  
rgbToHsv(r, g, b)
    r = (r / 255);
    g = (g / 255);
    b = (b / 255);
    max = max(max(r, g), b);
    min = min(min(r, g), b);
    h = s = v = max;
    d = max - min;
    s = (max == 0) ? 0 : (d / max);
    if max == min
        h = 0;
    else
        if max == r
            h = ((g - b) / d) + (g < b ? 6 : 0);
        else if max == g
            h = ((b - r) / d) + 2;
        else if max == b
            h = ((r - g) / d) + 4;

    h = (h / 6);

    return round(h * 360) round(s * 100) round(v * 100)
```

### 完整代码
* https://github.com/alex-mm/hsv-for-stylus/blob/master/src/hsv.styl

## 参考地址
* http://colorizer.org/
* https://en.wikipedia.org/wiki/HSL_and_HSV
* http://blog.csdn.net/tonyshengtan/article/details/44277191
* https://www.sitepoint.com/hsb-colors-with-sass/


