### include & extend 規則如下：

1. class `include` module 會把該 module 的 methods 混入成 `instance method`
2. class `extend` module 會把該 module 的 methods 混入成 `class method`


### [範例] include

需求：

我有兩個 class 叫做 `Airplane` 和 `Bus`

```ruby
class Airplane < FlyMachine
end

class Bus < Car
end
```

假設他們的加油方式都一樣，我希望為這兩個 class 的 `instance` 擴充同一種加油方法 `fill_gas!`。

例如:

```ruby
@my_airplane = Airplane.new
@my_airplane.fill_gas!

@my_car = Car.new
@my_car.fill_gas!
```

問題是他們分別繼承了不同的 `superclass`，而且不一定所有繼承 `FlyMachine` 或 `Car` 的其他 class 都需要或適用 `fill_gas!` 這個方法。即使我實作了，變成要貼兩次一樣的程式碼在不同的 class 裡。

所以這時候我們可以建立一個 `module` 叫做 `GasHandler`

```ruby
module GasHandler
  def fill_gas!
    puts "gas filled ..."
  end
end
```

> module 可以是 class / methods / modules 的集合，用來組織程式架構、或是當 package namespace 的用途。

然後我只需要在 `Airplane` 和 `Car` 裡面 include `GasHandler` 就可以了：

```ruby
class Airplane < FlyMachine
  include GasHandler
end

class Bus < Car
  include GasHandler
end

@my_airplane = Airplane.new
@my_airplane.fill_gas! # "gas filled ..."

@my_car = Car.new
@my_car.fill_gas! # "gas filled ..."

```


### [範例] extend

以剛剛的例子，我想為 `Airplane` 和 `Car` 擴充一樣的搜尋功能，並放在 `class level method` 裡，像是。

```ruby
my_airplane = Airplane.search_by_name('my_airplane')
my_car = Car.search_by_name("eddie's car")
```

和剛剛的做法一樣，但這次我要使用 `extend` 來讓 module 裡的 method 變成 `class level method`，範例：

```ruby
module Searchable
  def search_by_name(name)
    # your search logic
  end
end
```

再到 model 去 extend `Searchable`:

```ruby
class Airplane < FlyMachine
  include GasHandler
  extend Searchable
end

class Bus < Car
  include GasHandler
  extend Searchable
end
```

使用 extend 混入後，就可以直接使用 `Airplane.search_by_name` 和 `Car.search_by_name` method 了。

### [範例] 同時需要混入 class & instance methods

拿 `GasHandler` module 的例子，如果我希望還可以加一個 class method `fill_all_gas!` 一次把所有 car 或 airplane 加滿油，也可以單獨為 instance 加油的情況，我的 module 必須修改如下：

```ruby
module GasHandler
 def self.included(base)
    base.class_eval do
      def self.fill_all_gas!
        puts "gas filled all ..."
      end
    end
  end

  def fill_gas!
    puts "gas filled ..."
  end
end
```
以上的 code 只要任何 class include 這支 module 則會執行這裡的 `self.included`，並傳入是誰 include 了這支 module 到第一個參數 `base`，然後再用 class level eval 去創建該 class 的 method。

以上是基本的 mixin 範例，另外還可以搭配各種設計方式來達到不同的需求
