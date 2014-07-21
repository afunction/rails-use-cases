# Concerns

concerns 是 `ActiveSupport` 基於 mixin，並解決相依問題實作的 feature，範例可參考 [這篇](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)，另外使用 concerns 實作 mixin 也會讓 code 變得比較簡潔。

如果剛剛的 case 改用 concerns 實作，必須需額外在 module 內 extend `ActiveSupport::Concern` 才能使用，完整範例：

```ruby
# app/models/concerns/gas_handler.rb
module GasHandler
  extend ActiveSupport::Concern

  included do
    # 可以在這裡放當 include 時要執行的東西
    # 這裡你可以存取所有 class level 的東西
  end

  module ClassMethods
    def fill_all_gas!
      puts "gas filled all ..."
    end
  end

  def fill_gas!
    puts "gas filled all ..."
  end
end
```

有別於之前的例子，不需要在 `self.included` 裡 class eval，直接在 module 內宣告一個 `ClassMethods` module，裡面的 methods 全部會變成 class level method，是不是很方便呢?

注意：
> 1. included 的寫法有點不一樣
> 2. `ClassMethods` 大小寫不要打錯了


# 相依性

你也可以由多個 concerns 組合成另一新的 concerns。

例如把上傳 `Uploadable` 和編輯 `Editable`、軟刪除 `SoftDeletable`，組合出一個新功能 `Documentation`：

```ruby
# app/models/concerns/documentation.rb
module Documentation
  extend ActiveSupport::Concern

  included do
    include Uploadable
    include Editable
    include SoftDeletable
  end

  # ... (略)
end
```

而你只需要在需要的 class include `Documentation` 則包含了所有 Documentation 會用到的 concerns，讓這些相依性決定在使用到的 `module` 決定。


```ruby
# app/models/photo.rb
class Photo < ActiveRecord::Base
  include Documentation
end

# app/models/attach_file.rb
class AttachFile < ActiveRecord::Base
  include Documentation
end
```
