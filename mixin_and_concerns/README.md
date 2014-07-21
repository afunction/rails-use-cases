# Mixin & Concerns

###  為什麼把這麼複雜的東西擺第一章?

包括後續的 topic `controllers` `models` `helpers` `views`  都會大量地使用 mixin 和 ruby 的物件導向特性。

**P.S. 強烈建議看此篇前先讀完：[Understanding the Object Model](http://www.sitepoint.com/understanding-object-model/?utm_source=rubyweekly&utm_medium=email)**


# mixin (混入)

簡單來說你可以定義一個 module 然後透過 mixin 混入一或多個 class 中， 以達到把相似、或共通邏輯拆出的目的，也許你會說：那不是繼承的功能嗎?

但事情永遠不是那麼簡單，舉個例子，假設你的 A, B, C class 分別需要 D class 的部分功能，而傳統由上而下的繼承方式則會顯得不合用、或無法實作。

mixin 雖然不是新玩意兒，但感覺 ruby 社群滿強調這個項目的，很多的 gem 也大量使用 mixin 來達到各種不同的設計目的。

在不影響原本的繼承下，透過 mixin 分別在 class 內來 `include` 或 `extend` 指定的 module 達到類似 `多重繼承` 的目的。另外 mixin 也可以拿來擴充已存在的物件如 `ActiveRecord::Base` `routes` 等等、甚至是 ruby 基礎物件 `String` `Fixnum`。

> 社群友人建議，你可以想像成：
>
> 繼承 => `你的血統`
>
> mixin => `後天整形`

