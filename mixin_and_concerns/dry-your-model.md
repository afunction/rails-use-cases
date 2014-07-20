# DRY 你的 model

以 [晚鳥票](https://www.latebird.co) 的例子，這個 case 讓我省了非常多時間刻同樣的程式碼。

晚鳥票有三張 model，都擁有 `form_id` 和 `to_id` 都是 `integer` 欄位儲存起訖站:

1. `Subscribe` 儲存有票通知我的訂閱資料
2. `ThsrTicket` 儲存使用者的轉讓班次資訊
3. `ThsrGroupbuy` 儲存團購班次

需求是：

1. 我希望以上三張 model 能夠這樣操作
    ```ruby
    @subscribe = Subscribe.find(params[:id])
    @subscribe.from_site # 左營
    @subscribe.to_site # 臺北

    @ticket = ThsrTicket.find(params[:id])
    @ticket.from_site # 左營
    @ticket.to_site # 左營

    @group = ThsrGroupbuy.find(params[:id])
    @group.from_site # 左營
    @group.to_site # 左營
    ```
2. 並且都會檢驗 `to_id` `from_id` 是否等於數字 1 - 8，並且不能一樣
3. 都可以定義好一樣的 `scope`，如：
    ```ruby
    ThsrTicket.from_to_list(1, 8) # 回傳左營 => 臺北的轉讓票
    Subscribe.from_to_list(8, 1) # 回傳臺北 => 左營的訂閱訊息
    ThsrGroupbuy.from_to_list(7, 2) # 回傳板橋 => 台南的團購班次
    ```

# 相同邏輯使用 concerns 拆出來

```ruby
# app/models/concerns/departs.rb
module Departs
  extend ActiveSupport::Concern

  included do
    # 設定 validations
    validates_inclusion_of :from_id, :to_id, in: (1..8), message: '請選擇站台'
    validate :valid_from_to_cannot_be_same

    # 設定 scopes
    scope :from_to_list, -> (from_id, to_id) {
      where(from_id: from_id, to_id: to_id)
    }
  end

  # 顯示中文的起站
  def from_site
    SITES_HASH[from_id]
  end

  # 顯示中文的訖站
  def to_site
    SITES_HASH[to_id]
  end

  protected
    def valid_from_to_cannot_be_same
      if from_id == to_id
        errors.add(:from_sites, '起訖站不能一樣')
      end
    end
end
```


注意：
> 寫在 `included` blcok 內的程式碼，必須等 module `被 include` 了以後才執行，才不會找不到 `validate` `validates_inclusion_of` `scope` 這些屬於 `ActiveRecord::Base` 的 class methods


# Tips

以上案例為了方便說明而簡化過，實作起訖站欄位 `from_id` 和 `to_id` 更好的方法是使用 [[Gem] simple_enum](https://github.com/lwe/simple_enum) 有 `i18n`、`validate` 支援，也考慮到使用 `form select` 會用到的 `collection`，例如：

```erb
<% simple_form_for(@subscribe) do |f| %>
  <%= f.from_id, collection: Subscribe.sites_collections %>
  <%= f.to_id, collection: Subscribe.sites_collections %>
<% end %>
```

另外 rails 4.1.x 也開始支援 `enum` 功能了，可參考 [這篇文件](http://edgeapi.rubyonrails.org/classes/ActiveRecord/Enum.html) ，但目前還沒有 `i18n` `collection helper`。
