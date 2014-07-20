### include & extend 規則如下：

1. class `include` module 會把該 module 的 methods 混入成 `instance method`
2. class `extend` module 會把該 module 的 methods 混入成 `class method`


### [範例] include

```ruby
# app/models/concerns/departs.rb
module Departs
  def from_site
    I18n.t("departs.#{from_id}")
  end

  def to_site
    I18n.t("departs.#{to_id}")
  end
end

# app/models/thsr_groupbuys.rb
class ThsrGroupbuys < ActiveRecord::Base
  # 假設這個 model 有 from_id & to_id column
  include Depsrts
end

# app/models/thsr_groupbuys.rb
class ThsrTickets < ActiveRecord::Base
  # 假設這個 model 有 from_id & to_id column
  include Depsrts
end
```

這樣一來把 `ThsrTickets` 和 `ThsrGroupbuys` 都擁有了 `from_site` `to_site` 的 instance method，而不用分別重寫在兩個 model 裡，所以你可以這樣操作。

```ruby
@group = ThsrGroupbuys.find(1)
@group.from_site # 左營
@group.to_site # 台北

@ticket = ThsrTicket.find(1)
@ticket.from_site # 左營
@ticket.to_site # 台北
```

### [範例] extend

```ruby
module TranslateModelName
  def model_name
    I18n.t("models_name.#{self.to_s.downcase}")
  end
end

class ThsrGroupbuy < ActiveRecord::Base
  extend TranslateModelName
end
```

這樣一來 `model_name` 就會變成 `ThsrGroupbuy` 的 class level method，所以你就可以這樣用：

```ruby
ThsrGroupbuy.model_name
```

(以上這個例子 ActiveRecord 已經有類似的 method)

### [範例] 同時需要 mix class & instance methods

拿 departs module 的例子，如果我希望可以使用 `ThsrGroupbuy.sites_list` `ThsrTicket.sites_list` 可以得到車站的陣列，則必須修改成以下：

```ruby
# app/models/concerns/departs.rb
module Departs

  def self.included(base)
    base.class_eval do
      def self.sites_list
        %w(左營 南台 嘉義 台中 桃園 新竹 板橋 台北)
      end
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
以上的 code 只要任何 class include 這支 module 則會執行這裡的 `self.included`，並傳入是誰 include 了這支 module 到第一個參數 `base`，然後再用 class level eval 去創建該 class 的 method。

以上是基本的 mixin 範例，另外還可以搭配各種設計方式來達到不同的需求
