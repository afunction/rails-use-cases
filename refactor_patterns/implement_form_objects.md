# Basic Form Object

## 開始前先介紹 rails 其他的一些 modules

**ActiveModel::Conversion**

這個 module 不大，只定義了幾個 instance method，[source code](http://api.rubyonrails.org/v4.1.1/classes/ActiveModel/Conversion.html)。

其中比較重要的是 `to_key` 這個 method，上一章說過，form helper 會自動判斷你的 record 是否被 save 加上你的 id 去找到 url helper，而這裡的 to_key 指的就是 id，他的實作是：

```ruby
def to_key
  key = respond_to?(:id) && id
  key ? [key] : nil
end
```

https://github.com/rails/rails/blob/7d196cf360321466c0eefc474bfad1be7e3ea7ab/activemodel/lib/active_model/conversion.rb#L51


如果你的 form 寫的是這樣：

```erb
<%= form_for(@user_form) |f| %>
 ... (略)
<% end %>
```

你希望你的 form object 會自動找到 update or create path  你就必須要使用這個 module，不然你的 form object 丟入時會出現
undefine method `to_key`。

**ActiveModel::Model**

這是一個包含其他 module 的 module，裡面包含了

1. ActiveModel::Naming
2. extend  ActiveModel::Translation
3. ActiveModel::Validations
4. ActiveModel::Conversion

還提供了一個建構子，讓你可以在 new 物件的時候順便帶入參數：

```ruby
@user_form = UserForm.new(username: 'Eddie', age: '27')
```
https://github.com/rails/rails/blob/fe49f432c9a88256de753a3f2263553677bd7136/activemodel/lib/active_model/model.rb#L78


接下來，我們要一樣畫葫蘆，拿這些 rails 的 module 拼湊出一個適合自己使用的 form objects:

1. ActiveModel::Name -  定義 form/model name
2. ActiveModel::Validations - 定義驗證邏輯
3. ActiveModel::Model - 讓建構子擁有更新 attributes 功能
4. ActiveModel::Conversion - 讓 form helper 能夠找到 id


## 範例一：驗證登入表單

假設我們會需要在某個操作前讓使用者輸入密碼才能繼續，可以設計一個 `AuthForm` 來檢驗使用者的密碼是否正確。

**Form:**

```ruby
# app/forms/AuthForm.rb
class AuthForm < BaseForm::Basic
  include ActiveModel::Validations

  # 定義此 form 有什麼欄位
  attr_accessor :password
  attr_reader :user

  def initialize(user=nil)
    @user = if user.is_a?(User)
      user
    else
      User.find_by_username(user)
    end
  end

  def verify(password)
    # verify_password? 是 devise 提供的 method
    if user.verify_password?(password)
      yield if block_given?
      true
    else
      errors.add(:password, :invalid)
      false
    end
  end
end
```

**Controller:**

```ruby
class VerySafeController < ApplicationController

  def edit
    @form = AuthForm.new
  end

  def update
    @form = AuthForm.new(current_user)
    if @form.verify(params[:auth_form][:password])
      # 需要驗證密碼的動作
      redirect_to ...
    else
      render :edit
    end
  end
end
```

**View:**

```ruby
<%= form_for(@form, path: '') do |f| %>
  請輸入密碼進行下一步：
  <%= f.input :password %>
  <%= f.submit %>
<% end %>
```


## 範例二：Create 與 Update 邏輯不同

假設我們有一個功能可以收藏網站內的文章，按下 Add Favorite 時則新增一則 favorate，但事後編輯時必須填寫其他欄位，這時候 validation 則不能寫在 model 內了，而是要拉出來寫成 form。

很多時候，我認為在設計 form object 時，先把 `想要如何使用` 的程式碼先寫出來，會比較容易實作，所以我們寫出我們希望如何在 controller 和 view 中使用這個 form ：

**Controller (PostsController): ** 直接新增

```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  # POST /posts/1/add_favorate
  def add_favorate
    current_user.favorates.create(post: @post)
    redirect_to :back, notice: 'Successful'
  end
end
```

**Controller (FavoratesController): ** 事後編輯需要 validation

```ruby
# app/controllers/favorates_controller.rb
class FavoratesController < ApplicationController
  # GET /favorates/1/edit
  def edit
    @form = FavorateForm::Update.new(@favorate)
  end

  # PUT/PATCH /favorates/1
  def update
    @form = FavorateForm::Update.new(@favorate)
    if @form.update(params[:favorate])
      redirect_to ...
    else
      render :edit
    end
  end
end
```

**View:**

```erb
<%= form_for(@form) |f| %>
  <%= f.input :title %>
  <%= f.input :description %>
  <%= f.input :start %>
  <%= f.submit %>
<% end %>
```


**Form:**

```ruby
class FavorateForm::Update
end
```
