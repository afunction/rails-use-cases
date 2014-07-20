# 擴充已存在物件

### 不一定需要在 class 內做 mixin 動作


記得剛接觸 rails [[Gem] devise](https://github.com/plataformatec/devise) 的時候，一直很困惑為什麼只是再 Gemfile 加入一個 gem，在完全沒動到 model 繼承關係的情況下，
就冒出來一個 `devise` method，如下：


```ruby
class User < ActiveRecrod::Base
    devise :database_authenticatable, :registerable, :confirmable, :recoverable, stretches: 20
end
```

看了 devise 的 [source code](https://github.com/plataformatec/devise/blob/c67de7e91cc3ea201c7cedc0ed71e1f6c76b2d36/lib/devise/orm/active_record.rb) 學到一招可以不需要在 class 去做 `include` 或 `extend` 混入的動作。

寫法如下：

```ruby
module TestModule
  def test_method
    "Helloworld, I'm string instance method"
  end
end

String.send(:include, TestModule)
```


> `send` 是 ruby 所有 object 都有，用來執行 method 的 method，其特性可以跳過 `private` 和 `protected` 保護進而執行 method。
>
> http://ruby-doc.org/core-2.1.2/Object.html#method-i-send


### 例子

1. [[Gem] stamp](https://github.com/jeremyw/stamp) 擴充了 `Date` 和 `Datetime` 物件、方便設定時間轉字串的 format。
2. [[Gem] super_accessors](https://github.com/afunction/super_accessors) 將 ActiveRecord 的 [store](http://api.rubyonrails.org/classes/ActiveRecord/Store.html) 擴充成支援 `integer` `boolean` .. etc (原本全部都是 string)


### 在專案內擴充原有物件

如果你有類似需求，又不打算把你寫的 code 就打包成 gem，你可以把你寫的 `module` 放在 lib 裡面，並在 `initializer/` 裡面去做 mixin 的動作。

> 即使在開發環境中 `initializer/*` 裡的資料要是有變動，則需要重新啟動 web server 才會生效。
