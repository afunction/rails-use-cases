# 留 options 給使用的 class

### A. 把邏輯群組起來

一樣是晚鳥票的例子 `ThsrGroupbuy` `ThsrTicket` 有三個欄位：

`depart_no` - 班次編號

`departure_time` - 發車時間 (不給使用者填寫)

`arrival_time` - 到達時間 (不給使用者填寫)


需求：

1. `depart_no` 需要被驗證
2. 存入資料庫前需要以 `form_id` `to_id` `depart_no` 找到該班次的起訖時間並填入 `departrue_time` 和 `arrival_time` 中


以意義上來說，上述的邏輯應該也要放在 concern `Departs` 裡，但是 `Subscribe` 並不需要這些邏輯，加入反而會造成錯誤。

解法如下：

```ruby

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

    # 設定站台名稱
    SITES = [[1, '高雄'], [2, '台南']] # 其他省略
    SITES_HASH = Hash[SITES]
  end

  module ClassMethods
    def acts_need_to_valid_depart_info
      # validation for departs_info
      validates_presence_of :depart_no

      # callbacks
      before_validation :do_set_departure_arrival_time
    end
  end


  protected
    def valid_from_to_cannot_be_same
      if from_id == to_id
        errors.add(:from_sites, '起訖站不能一樣')
      end
    end

    def do_set_departure_arrival_time
      # (偽扣)
      thsr_depart = ThsrDepart.search_depart(depart_no, from_id, to_id)
      self.departure_time = thsr_depart.departure_time
      self.arrival_time = thsr_depart.arrival_time
    end
end
```

注意到了嗎? 我們多寫了一個 class method 叫 `acts_to_valid_depart_info`，裡面定義了驗證、和寫入發車、抵達時間的 callback，但這些內容並不是寫在 `included` block 裡面，而是 `被動的` 等要使用的 class 來呼叫：


使用方法如下：

```ruby
# app/models/thsr_ticket.rb
class ThsrTicket < ActiveRecord::Base
  include Departs

  acts_need_to_valid_depart_info
end

# app/models/thsr_groupbuy.rb
class ThsrGroupbuy < ActiveRecord::Base
  include Departs

  acts_need_to_valid_depart_info
end

# app/models/subscribe.rb
class Subscribe < ActiveRecord::Base
  include Departs
end
```

> Q: 一定要將 `acts_need_to_valid_depart_info` 寫成 class method 呢?
>
> A. 有注意到在 class 內呼叫 `acts_need_to_valid_depart_info` 和呼叫 `validate`  `callback` 都是寫在 class level 的區塊嗎? 因為 validation 和 callbacks 都屬於 class method 層級的東西，所以當然也要實作在 `ClassMethods` 裡面摟，

### B. 由 class 定義細節

其實上述的例子還少講了一個需求：

1. `ThsrTicket` 是 30 天內的早鳥班次，需要檢查 `depart_date` (僅允許現在 ~ 30 天內)
2. 而 `ThsrGroupbuy` 可能會預開超過 30 - 60 天後的班次，所以必須解除限制

所以除了把功能 group 起來，我們還要留更多 options 給使用的 class 來定義，修改後的 method 如下：

```ruby
# app/models/concerns/departs.rb
module ClassMethods
  def acts_need_to_valid_depart_info(opts={})

    # Validations
    validates_presence_of :depart_no

    if opts[:limit_depart_date]
        validates_date :depart_date, after:  lambda { Date.today - 1.day },
                               before: lambda { Date.today +
                               30.day }
   end

    # callbacks
    before_validation :do_set_departure_arrival_time
  end
end
```




