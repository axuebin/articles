## 现状

iPhone X 底部是需要预留 34px 的安全距离，需要在代码中进行兼容。

现状对于 iPhone X 的判断基本是这样的：

```javascript
// h5
export const isIphonex = () => /iphone/gi.test(navigator.userAgent) && window.screen && (window.screen.height === 812 && window.screen.width === 375);
```

这在之前是没问题的，新的 iPhone X Series 设备发布之后，这个就会兼容就有问题。

## iPhone X Series 参数

| 机型 | 倍率	| 分辨率 |	pt |
| :--- | :--- | :--- | :--- |
| iPhone X |	3 | 2436 × 1125 | 812 × 375 |
| iPhone XS | 3 | 2436 × 1125 | 812 × 375 |
| iPhone XS Max | 3	| 2688 × 1242	 | 896 × 414 |
| iPhone XR | 2 | 1792 × 828 | 896 × 414 |

width === 375 && height === 812 只能识别出 iPhone X 和 iPhone XS，对于 iPhone XS Max 和 iPhone XR 就无能为力了。

## 解决方法

### 对每个机型进行判断
```javascript
const isIphonex = () => {
  // X XS, XS Max, XR
  const xSeriesConfig = [
    {
      devicePixelRatio: 3,
      width: 375,
      height: 812,
    },
    {
      devicePixelRatio: 3,
      width: 414,
      height: 896,
    },
    {
      devicePixelRatio: 2,
      width: 414,
      height: 896,
    },
  ];
  // h5
  if (typeof window !== 'undefined' && window) {
    const isIOS = /iphone/gi.test(window.navigator.userAgent);
    if (!isIOS) return false;
    const { devicePixelRatio, screen } = window;
    const { width, height } = screen;
    return xSeriesConfig.some(item => item.devicePixelRatio === devicePixelRatio && item.width === width && item.height === height);
  }
  return false;
}
```
### 统一处理方法
因为现在 iPhone 在 iPhone X 之后的机型都需要适配，所以可以对 X 以后的机型统一处理，我们可以认为这系列手机的特征是 `ios` + `长脸`。

在 H5 上可以简单处理。

```javascript
const isIphonex = () => {
  if (typeof window !== 'undefined' && window) {
    return /iphone/gi.test(window.navigator.userAgent) && window.screen.height >= 812;
  }
  return false;
};
```

### 媒体查询

```css
@media only screen and (device-width: 375px) and (device-height: 812px) and (-webkit-device-pixel-ratio: 3) {
}
@media only screen and (device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 3) {
}
@media only screen and (device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 2) {
}
```

媒体查询无法识别是不是 iOS，还得加一层 JS 判断，否则可能会误判一些安卓机。