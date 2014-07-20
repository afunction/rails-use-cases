# Mixin & Concerns

了解 concerns 前，應該先講講 ruby mixin，因為 concerns 的設計加強了 `ruby mixin` 技巧，並額外解決 `module 間的相依相問題`，範例可參考 [這篇](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)，而使用 concerns 也讓 code 變得更簡潔了。

# mixin (混入)

mixin 雖然不是新玩意兒，但感覺 ruby 滿強調這個項目的，很多的 gem 也大量使用 mixin 來達到各種不同的設計目的。

簡單來說你可以定義一個 module 然後透過 mixin 混入一或多個 class 中， 以達到把相似、或共通邏輯拆出的目的，也許你會說：那不是繼承的功能嗎?

但事情永遠不是那麼簡單，舉個例子，假設你的 A, B, C class 分別需要 D class 的部分功能，而傳統由上而下的繼承方式則會顯得不合用、或無法實作。

在不影響原本的繼承下，透過 mixin 分別在 class 內來 `include` 或 `extend` 指定的 module 達到類似 `多重繼承` 的目的。另外 mixin 也可以拿來擴充已存在的物件如 `ActiveRecord::Base` `routes` 等等、甚至是 ruby 基礎物件 `String` `Boolean`。


