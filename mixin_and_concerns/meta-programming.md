# 搭配 meta-programming

像 [[Gem] simple_enum](https://github.com/lwe/simple_enum) 這類的 gem 只要在 model 內使用 `as_enum` method：

```ruby
class User < ActiveRecord::Base
  as_enum :gender, [:male, :female]
end
```

宣告完成後，奇怪了，就可以這樣使用耶：

```ruby
@user = User.find(1)
@user.male? # true
@user.female? # false
@user.human_gender # 男性
```

還可以再 form 這樣使用

```erb
<%= simple_form_for(@user) do |f| %>
  <%= f.input :gender, collection: User.genders_for_select %>
<% end %>
```

一開始很好奇為什麼會多出 `male?` `female?` 和 `genders_for_select` 我只是多寫一行程式，怎麼跑出這麼多商業邏輯名稱的 method 出來?

去翻了 [source code](https://github.com/lwe/simple_enum/blob/90bd98e6f25484d227958ad684966956a115e76a/lib/simple_enum/attribute.rb#L45) 後發現，原來是透過 `動態的定義` 的方式來達成。google 了相關資料找到一個名詞 `meta-programming`，目前找到最容易理解的解釋是 [ihower blog](http://ihower.tw/rails3/ruby.html) 說的：「用程式來寫程式」。

### 舉個例子：

這是我們常用的操作，取得使用者的 email `(Getter)` 和設定 email `(Setter)`：

```ruby
@user = User.find(1)
@user.email
=> xxx@domain.com
@user.email = 'username@domain.com'
```

這是 `ActiveRecord::Base` 已經幫我們做好的事，所以我們並不會在 class `User` 內看到相關的程式碼。

但如果我們想自己產生 `Getter` 和 `Setter` 該怎麼做呢?

```ruby
class Dog
  def name
    @name
  end

  def name=(value)
    @name = value
  end
end
```

這樣一來我們就可以使用如下：

```ruby
@mydog = Dog.new
@mydog.name
=> nil
@mydog.name = 'Leo'
@mydog.name
=> 'Leo'
```

但如果一次有多個 attributes 呢? 類似的程式碼不是寫到死 ... 所以這時候 meta-programming 的用程式寫程式就派上很大用場了。

```ruby
class Dog
  # 快速定義 getter, setter 的 method
  def self.define_setter_and_getters(*attribues)
    attribues.each do |attribute|
      # 定義 Getter
      define_method attribute do
        # 取得 instance variable
        instance_variable_get("@#{attribute}")
      end

      # 定義 Setter
      define_method "#{attributes}=" do |value|
        # 設定 instance variable
        instance_variable_set("@#{attribute}", value)
      end
    end
  end


  # 一次定義多個 methods
  define_setter_and_getters :id, :name, :age, :type, :owner, :home_address
end
```

這樣子只要寫一次 `define_setter_and_getters` 就不用在手動一個一個定義了。

```ruby
@dog = Dog.new
@dog.id = 1
@dog.id
=> 1
@dog.name = 'Leo'
@dog.name
=> Leo
@dog.age
=> nil
```

當然這麼好用的方法不會只有一個 class 需要，所以你應該會想到它可以拆到 concerns 讓需要的 class 直接 include 對吧！

不過其實 ruby 語言已經內建了完全一樣的功能，使用方法也一樣，而且還有更多選擇喔：

1. `attr_accessor` 同時建立 Getter / Setter
2. `attr_reader` 只建立 Getter
3. `attr_writer` 只建立 Setter

以上 `attr_*` 是 for instance，但是在 rails 提供了更多 ，如 `cattr_*` 是給 class level, `mattr_*` 是給 module，想了解更多的人可以看一下 [source code](https://github.com/autotelik/adam/blob/master/lib/ruby/activesupport/lib/active_support/core_ext/class/attribute_accessors.rb)。

### 開始寫一個程式、讓它自動寫程式吧 (yo)

這個地方要想一個情境很難，因為好的應用案例不是被 rails 就是已經被其他 gem 做走了，所以這裡舉的例子可能不是很好 (有更好的例子請讓我知道)

#### 情境：

我的管理界面 /admin/* 底下有有三個 resources `users` `thsr_tickets` `thsr_groupbuys` 都需要有 enable, disable 的功能。

我先實作了 `Admin::UsersController` 這隻檔案：

```ruby
class Admin::UsersController < Admin::BaseController
  # ... (略)

  # POST /admin/users/1/enable
  def enable
    @user.update(enable: true)
    ActionLog.create(target: @user, operator: current_admin, ip: remote.ip, action_message: 'enable')
    redirect_to :back, notice: 'User Enabled'
  end

  # POST /admin/users/1/disable
  def disable
    @user.update(enable: false)
    ActionLog.create(target: @user, operator: current_admin, ip: remote.ip, action_message: 'disable')
    redirect_to :back, notice: 'User Enabled'
  end
end
```

**Tips**

> 以上的 `ActionLog` 是把所有操作記錄下來，增加 enable/disable 的程式碼份量是為了突顯等等的例子，實際情況可能會更複雜。

如果有其他的 controller 也需要類似的邏輯，拆到 `Admin::BaseController` 是一種做法，但問題是：

1. 如果非 admin/* 也需要用呢?
2. 終在 routes.rb 的設定還是需要 `def enable` `def disable`。

```ruby
# config/routes.rb
namespace :admin do
  resources :users, :thsr_tickets, :thsr_groupbuys do
    post :enable
    post :disable
  end
end
```

觀察需求可能不同的項目、變數有:

1. operator - 操作的人，可能是 `current_admin` `current_user`
2. target - 被操作的對象，可能是 `@user` `@thsr_ticket` .. etc
3. notice - 導頁後的 message (enable, disable)

並且我希望可以在需要的地方這樣宣告。

```ruby
class Admin::ThsrTicketController < Admin::BaseController
  before_action :set_thsr_ticket, only: [:edit, :update, :enable, :disable] # 加入 enable, disable

  # ... (略)
  acts_enable_disable_flow_for operator: :current_admin, target: :thsr_ticket, enabled_message: 'Ticket Enabled', disable_message: 'Ticket Disabled'
end
```

我的 concerns 則會長這樣：

```ruby
module EnableControlFlow
  def acts_enable_disable_flow_for(opts)
    define_method :enable do
      target = instance_variable_get("@#{opts[:target]}")
      operator = send(operator)

      target.update(enable: true)
      ActionLog.create(target: target, operator: operator, ip: remote.ip, action_message: 'enable')
      redirect_to :back, notice: opts[:enabled_message]
    end

    define_method :disable do
      target = instance_variable_get("@#{opts[:target]}")
      operator = send(operator)

      target.update(enable: false)
      ActionLog.create(target: target, operator: operator, ip: remote.ip, action_message: 'disable')
      redirect_to :back, notice: opts[:disabled_message]
    end
  end
end
```

在接下來需要的地方如：

**一般使用者的 thsr_tickets: **

```ruby
# app/controllers/thsr_ticket.rb
class ThsrTicketController < ApplicationController
  # 混合成 class method
  extend EnableControlFlow

  # 加入 enable, disable
  before_action :set_thsr_ticket, only: [:edit, :update, :enable, :disable]

  acts_enable_disable_flow_for operator: :current_user, target: :thsr_ticket, enabled_message: '已啟用', disable_message: '已停用'
end
```

**管理者的 thsr_groupbuys: **

```ruby
# app/controllers/admin/thsr_groupbuys.rb
class Admin::ThsrGroupbuysController < Admin::BaseController
  # 混合成 class method
  extend EnableControlFlow

  # 加入 enable, disable
  before_action :set_thsr_groupbuys, only: [:edit, :update, :enable, :disable]

  acts_enable_disable_flow_for operator: :current_admin, target: :thsr_groupbuy, enabled_message: '啟用團購', disable_message: '關閉團購'
end
```

當然我們還可以為 `acts_enable_disable_flow_for` 設計更多 options，來達成不同需求，例如：設計 callbacks 讓 controller 決定更多流程、或在 controller 決定要更新的欄位名稱等等。

不過儘量減少 options，大家遵守一個 convention 感覺比較接近 rails way，如何進行設計還是要爭酌情況。

而就我的觀察，其實很少機會、甚至完全不會用到 meta-programming 寫 business logic 的東西。

而大多的應用像是 `activerecord` `simple_enum` `devise` `soft_deletion` 這種必須提供給使用者彈性的 library 工具才會大量用到。


> 這樣說起來，如果不寫 gem 好像不容易碰到 meta-programming，但事情不是這樣的，很多時候使用別人的 gem 出現狀況，是要去翻該工具的 source code 才能解決，如果不知道你用的東西怎麼被設計的，怎麼能夠故障排除呢?
