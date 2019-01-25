## iPhone X Series 参数
| 机型 | 倍率	| 分辨率 |	pt |
| :--- | :--- | :--- | :--- |
| iPhone X |	3 | 2436 × 1125 | 812 × 375 |
| iPhone XS | 3 | 2436 × 1125 | 812 × 375 |
| iPhone XS Max | 3	| 2688 × 1242	 | 896 × 414 |
| iPhone XR | 2 | 1792 × 828 | 896 × 414 |

## 代码
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
};
```