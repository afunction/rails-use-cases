# Form Helper 的角色

由於 Form object 大部分時候是設計給 `form helper` 使用，所以開始實作 form object 前則必須了解 from helper 的職責。


### 基本職責

不論是 `simple_form_for` `bootstrap_form_for` 和最基礎的 `form_for` 他們基本職責是：

1. generate input, input value
2. 顯示 error message
3. 和準備丟到哪裏 (form action="...")
4. 和 csrf protected (這裡略過、有興趣可自行 google `rails 4 form authenticity token`)


### 1. Input & Value

```erb
<% form_for(@order) do |f| %>
  <%= f.input :address %>
<% end %>
```

rails 範例幾乎沒有對 form helper 做 detail 的說明，其實以上程式碼的 `@order` 不一定要丟入 model。

而 `f.input :address` 只是去建立一個 input html 並把 value 填上 `@order.address` 而已。

既然是這樣，只要我有一個物件有 Getter 可以取值，那也就可以使用如下：

```ruby
class Animal
  def name
    "I'm Dog"
  end
end

@dog = Animal.new
```

```erb
<% form_for(@dog, url: 'some path') do |f| %>
  <%= f.input :name %>
<% end %>
```

### 2. Error Messages

Form helper 的第二個任務就是顯示 error messages，如果你對一個 model 操作 `save` 回傳 `false` 代表驗證沒有通過：

```ruby
@user = User.new(nickname: 'Eddie Li')
@user.save # false
@user.errors
 => #<ActiveModel::Errors:0x0000010bcaeea8 @base=#<User id: nil, email: "", encrypted_password: "", reset_password_token: nil, reset_password_sent_at: nil, remember_created_at: nil, sign_in_count: 0, current_sign_in_at: nil, last_sign_in_at: nil, current_sign_in_ip: nil, last_sign_in_ip: nil, confirmation_token: nil, confirmed_at: nil, confirmation_sent_at: nil, unconfirmed_email: nil, is_admin: false, is_manager: false, created_at: nil, updated_at: nil, nickname: "", location: "", avatar: "", tel: "", mobi: "", extra_data: nil, birthday: nil, shipping_city: "", shipping_area: "", shipping_zip: "", shipping_address: "", shipping_recipient: "", shipping_gender_cd: 0, shipping_contact_mobile: "", pay_notification: nil, lock_login: false, authentication_token: nil>, @messages={:email=>["不能是空白字元"], :password=>["不能是空白字元"]}>
```

這時候 `@user.errors` 回傳的 `ActiveModel::Errors` 物件，裡面記載了你這個 record 的 instance，還包含 columns 對應的 error messages，如果你輸入 `@user.errors.messages` 則會出現：

```ruby
@user.errors.messages
 => {:email=>["不能是空白字元"], :password=>["不能是空白字元"]}
```

所以除了上一小節談到的 input & value 外，他還會依照你的 attribute name 去找到是否有 error message，實作出，如果這個欄位有錯誤，就顯示紅色的功能。


### 3. form helpers 的差異

**form_for**

form_for 的職責只是很簡單的負責 input & values 並沒有 form wrapper 的功能，所以用 form_for 你需要在裡面輸入非常多的 html，如：

```erb
<%= form_for(@user) do |f| %>
  <div class="form_control">
    <%= f.text_field :nickname %>
  </div>
  <div class="form_control">
    <%= f.password_field :password %>
  </div>
  <div class="form_control">
    <%= f.password_field :password_confirmation %>
  </div>
<% end %>
```

**simple_form**

simple_form 解決了 form_for 需要不斷寫重複的 html 的問題，使用 simple_form 的樣子如下：

```erb
<%= simple_form_for(@user) do |f| %>
  <%= f.input :nickname %>
  <%= f.input :password %>
  <%= f.input :password_confirmation %>
<% end %>
```

為了能夠 DRY 又能夠支援不同的 form html 結構，所以 simple_form 實作了一個抽象概念 form wrapper，並把一個 form 的元素歸納於以下幾個項目：

1. label, input
2. error messages
3. help or hints。

並且提供了一組 customize form wrapper 的 [API](https://github.com/plataformatec/simple_form#the-wrappers-api) 讓用的人可以 custom made 自己的 form 結構。

當然還有其他更多功能，有興趣的可以至  [simple_form github](https://github.com/plataformatec/simple_form) 查看。

**bootstrap_form**

[[Gem] bootstrap_form](https://github.com/bootstrap-ruby/rails-bootstrap-forms) 本身則是一套專屬 bootstrap 表單結構的 form helper，所以若是專案已經使用 bootstrap 我比較傾向使用這套。


### 4. Form Action

這是我認為比較不好理解，但理解後會覺得 rails 的 RESTful 設計非常了不起的地方。

##### A. method:

了解 RESTful 運作的人會知道，通常新增資料時要 `post` 更新則是 `put` 或 `patch`，但通常我們在 form 裡只寫了：

```
form_for(@user)
```

他是怎麼知道我們開始用 `post` 還是 `patch` 的呢? 原因是其實 ActiveRecord 有提供內建兩個 method `persisted?` 和 `new_record?`。

```
@user = User.new
@user.new_record?
=> true
@user.persisted?
=> false

@user = User.find(1)
@user.new_record?
=> false
@user.persisted?
=> true
```

沒錯，這兩個 method 指的是，這筆 record 是不是已經存入 database 中了，而 form helper 也會依照這個值去決定是要 `post` 還是 `put`。

##### B. action path:

但他是怎麼知道要丟到哪個網址呢? 其實 form helper 非常聰明，他會拿你丟入的變數 `@user` 找到該變數的 `model_name` 是 `User` 進而找到你的 user resource 的 `url helpers`

1. 如果是 `new_record?` 則丟到 users_path 中
2. 如果是 `persisted?` 則丟到 user_path(@user) 中

所以如果我們 follow RESTful 的設計在做網站的 CRUD 會非常快速，不過這種擁有非常多慣例的設計的確會讓剛接觸的人覺得非常困惑。

##### C. namespace & resource member

另外，我們也會遇到 namespace 的情況，比如：

```ruby
namespace :admin do
  resources :user do
    post :dispatch_credit
  end
end
```

管理者 admin 底下的 user resources，這時候的 form 千萬不要寫成

```erb
<% form_for(@user, url: admin_user_path(@user)) do |f| %>
```

有更 clean 的方式：

```erb
<%= form_for([:admin, @user]) do |f| %>
```

另外，我們為 resource 擴充的 member `dispatch_credit` 也是透過這種方式：

```erb
<%= form_for([:dispatch_credit, :admin, @user]) do |f| %>
```

是不是很簡單呢?


##### D. Model Name (非常重要)

剛剛的 action path 小節有說到 form helper 會以你丟入的 `@user` 找到 `model_name`，其實這裡的 model_name 並不是指 `User` 這個 model 的名字。

而是 `ActiveRecord::Base` 引用了 [ActiveModel::Naming](http://api.rubyonrails.org/classes/ActiveModel/Naming.html) 這個 module 所提供的功能，來定義這張 model 的名字。

**i. 為什麼不直接使用 model 的名字呢?**

當然是為了彈性啊，User model 預設的情況是拿 User 當作 model_name，但不一定所有情況都是 model name 來當作 resource 的名字。

**ii. model_name 的其他的用途**

除了他會拿你的 model_name 去自動找到 action 該丟到哪個 resource 去之外，使用 form helper 建出來的 input name 也會依照你的 model_name 做命名：

```erb
<% form_for(@user) do |f| %>
  <%= f.input :nickname %>
  <!-- <input type="text" name="user[nickname]"> -->
<% end %>
```

所以你在 controller 內取得的 `params[:user]` 的 `:user` 也是因為 model_name 的關係。

**iii. 做個小實驗**

如果我們在 app/models/user.rb 中複寫 model_name：

```ruby
class User < ActiveRecord::Base
  def self.model_name
    ActiveModel::Name.new(self, nil, :student)
  end
end
```

打開原本你寫好了 form 會發現找不到 `student_path` 或 `students_path`。

我們在 `routes.rb` 加入 resource：

```ruby
resources :students
```

再打開 form，檢視原始碼，有沒有發現你的 input name 都變成

```html
<input type="text" name="student[nickname]" value="">
```

有沒有突然通了的感覺? 然後開始覺得 rails 的設計既彈性、又方便呢?
