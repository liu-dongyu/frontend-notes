```
var getDevicePixelRatio = function() {
  ratio = 1
  if(window.screen.systemXDPI != undefined and window.screen.logicalXDPI != undefined and window.screen.systemXDPI > window.screen.logicalXDPI) {
    ratio = window.screen.systemXDPI / window.screen.logicalXDPI;
  }else if(window.devicePixelRatio != undefined) {
    ratio = window.devicePixelRatio;
  }
  return ratio;
}
```
