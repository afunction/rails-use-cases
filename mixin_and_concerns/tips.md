# Tips

1. `mixin & concerns` 不是由上而下的繼承方式、特性也很動態，所以清楚了解你的程式碼會 run 在 class 或 instance 變得很重要。
2. 越 DRY 的 code 則有可能出現維護的問題，一些教學中指出 concerns 的問題是，只單看 concerns 內容無法得知上下文 (哪些 class 使用了)，如果不妥善使用反而可能造成更難維護的情況。
