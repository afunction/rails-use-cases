# Concerns

concerns 主要是 rails `ActiveSupport` 為了解決相依問題實作的功能，另外使用 concerns 實作 mixin 也會讓 code 變得比較簡潔。

如果剛剛的 case 改用 concerns 實作，必須需額外在 module 內 extend `ActiveSupport::Concern` 才能使用，完整範例：

```ruby
# app/models/concerns/departs.rb
module Departs
  extend ActiveSupport::Concern

  included do
    # 可以在這裡放當 include 時要執行的東西
    # 你可以存取所有 class level 的東西
    # ex1: 宣告 shared scope
    # ex2: 可寫 shared validation
  end

  module ClassMethods
    def sites_list
      I18n.t('depsrts')
    end
  end

  def from_site
    I18n.t("departs.#{from_id}")
  end

  def to_site
    I18n.t("departs.#{to_id}")
  end
end
```

不需要 class eval 直接在 module 內宣告一個 `ClassMethods` module，裡面的 methods 全部會變成 class level method，是不是很方便呢?

(記得 `ClassMethods` 大小寫不要打錯了)

# 相依性

如果你的 concerns 間有相依關係，例如有一個 concerns 叫做 `HightSpeedRail` 相依 `Departs` 和付款 `Payable`、訂位 `Reservable`，並組合出一個新功能，則可以這樣使用：

```ruby
# app/models/concerns/high_speed_rail.rb
module HighSpeedRail
  extend ActiveSupport::Concern
  include Departs
  include Payable
  include Reservable


  # ... (略)
end
```
