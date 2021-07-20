# Active Record
- ORM
- 约定
- CRUD
- Associations
- Query interface
- Callbacks
- Validations

## ORM
- 定义数据结构
- 定义数据关联
- 定义层级关系
- 数据持久化前校验
- 以面向对象方式完成数据库操作

## 约定
### 命名约定
- model名: 单数形式，每个单词首字母大写
- 数据库表名: 最后一个单词复数，单词全小写，下划线分割单词

|Model / Class| Table / Schema|
|---|-----|
|Article	| articles |
|LineItem	| line_items|
|Deer	|deers|
|Mouse	|mice|
|Person	|people|

### 列
- Foreign keys: 单数表表名_id (例子：item_id, order_id)
- Primary keys: 默认id, 执行migration时候自动创建


### 使用已经数据库表
```
class Product < ApplicationRecord
    # 指定表明
    self.table_name = "my_products"
    # 指定主键
    self.primary_key = "product_id"
end
```

---

## Migration
- Creating standalone Migration
- Model Generators
- Writing a Migration
- Running Migrations
- Changing Existing Migrations

```
bin/rails generate model Author
bin/rails generate model Book

bin/rails db:migrate

bin/rails generate migration AddAuthorRefToBook author:references
bin/rails generate migration AddNameToBook 'name:string{20}'

bin/rails db:migrate

bin/rails generate migration RemoveNameFromBook name:string
bin/rails db:migrate
```

---

## CRUD

### Create
```
user = User.create(name: "Foo", title: "Bar")

# or

user = User.new
user.name = "Foo"
user.title = "Bar"

user.save
```

### Read
```
users = User.all

user = User.first

foo = User.find_by(name: "Foo")

users = User.where(name: "Foo", title: "bar").order(created_at: :desc)

users = User.find_by_sql("select * from users where name = 'Foo'")
```

### Update
```
user = User.find_by(name: "Foo")
user.name = "Bar"
user.save
```

### Delete
```
user = User.find_by(name: "Foo")
user.destroy

# or

User.destroy_by(name: 'Foo')

# or

User.destroy_all
```


---
## Associations
```
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end

class Book < ApplicationRecord
  belongs_to :author
end
```

### Types：
- belongs_to
- has_one
- has_many
- has_many :through
- has_one :through
- has_and_belongs_to_many

#### belongs_to:
- 使用单数关联
- 单向一对一
- 要设置

---
## Query interface

---

## Callbacks

### Life cycle
- created
- updated
- destroyed

### callbacks are called
- created
- saved
- updated
- deleted
- validated
- loaded from database

### Available callbacks:

#### creating an object:
- before_validation
- after_validation
- before_save
- around_save
- before_create
- around_create
- after_create
- after_save
- after_commit / after_rollbak

#### updating an object:
- before_validation
- after_validation
- before_save
- around_save
- before_update
- around_update
- after_update
- after_save
- after_commit / after_rollback

#### destroying an object
- before_destroy
- around_destroy
- after_destroy
- after_commit / after_rollback


#### initialize
- after_initialize ActiveRecord.new 或者 loaded from database之后调用
- after_find loaded from database之后调用

#### touch
- after_touch 调用touch方法之后，会遍历touch所有belongs_to

### 触发callback
以下方法触发callbacks:
- create
- create!
- destroy
- destroy!
- destroy_all
- destroy_by
- save
- save!
- save(validate: false)
- toggle!
- touch
- update_attribute
- update
- update!
- valid?

### after_find被以下方法触发:
- all
- first
- find
- find_by
- find_by_*
- find_by_*!
- find_by_sql
- last


### 停止执行
model的执行队列包括validations, 注册的callback和数据库操作。如果callback抛出异常，并触发ROLLBACK


### Callback类
封装复用
```
class PictureFileCallbacks
  def self.after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end


class PictureFile < ApplicationRecord
  after_destroy PictureFileCallbacks
end
```

### Transaction callbacks
- after_commit
- after_rollback
与active record model需要与非数据库事务的外部系统保持一致性（调接口，删文件，清缓存等等）


---

## Validations
```
class User < ApplicationRecord
    validates :name, presence: true
end

user = User.new

user.valid?
=> false

user.save
=> false

user.save!
ActiveRecord::RecordInvalid: Validation failed: Name can't be blank

puts user.errors
```

### Validation helpers
- acceptance: 校验表单中的checkout
- validates_associated: 循环调用每一个关联对象的校验
- confirmation: 校验两个text field值是否完全相同， 比如密码
- exclusion: 校验字段不包含集合
- format: 用给定的正则表达式校验
- inclusion: 校验值从给定的集合中取
- length: 校验长度
    - minimum
    - maximum
    - in
    - is
- numericality: 校验是数字
    - only_integer: 可选，校验是否整数，如出现有一下可选项
        - :greater_than
        - :greater_than_or_equal_to
        - :equal_to
        - :less_than
        - :less_than_or_equal_to
        - :other_than
        - :odd
        - :even
- presence: 非空 （nil或者空值）
- absence: 空
- uniqueness: 校验唯一性， 不加db唯一性约束，并发操作可能生成相同值
- validates_with: 传入另一个类做校验
    ```
    class GoodnessValidator < ActiveModel::Validator
        def validate(record)
            if record.first_name == "Evil"
            record.errors.add :base, "This person is evil"
            end
        end
    end

    class Person < ApplicationRecord
        validates_with GoodnessValidator
    end

    ```
- validates_each: 传入一个block，传入所有属性进行校验

### Validation可选项
- :allow_nil 值为nil时跳过校验
    ```
    class Coffee < ApplicationRecord
    validates :size, inclusion: { in: %w(small medium large),
        message: "%{value} is not a valid size" }, allow_nil: true
    end

    ```
- :allow_blank 空值可以通过校验
- :message 自定义错误信息
    ```
    class Person < ApplicationRecord
        # Hard-coded message
        validates :name, presence: { message: "must be given please" }

        # Message with dynamic attribute value. %{value} will be replaced
        # with the actual value of the attribute. %{attribute} and %{model}
        # are also available.
        validates :age, numericality: { message: "%{value} seems wrong" }

        # Proc
        validates :username,
            uniqueness: {
            # object = person object being validated
            # data = { model: "Person", attribute: "Username", value: <username> }
            message: ->(object, data) do
                "Hey #{object.name}, #{data[:value]} is already taken."
            end
            }
    end

    ``
- :on 定义触发校验时机, 例如（:create, :update）

### strict 校验不通过抛出ActiveModel::StrictValidationFailed异常

### 条件校验

### 自定义Validator
继承ActiveModel::Validator类， 实现validate方法
```
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors.add attribute, (options[:message] || "is not an email")
    end
  end
end

class Person < ApplicationRecord
  validates :email, presence: true, email: true
end
```

### 校验错误

### View中显示校验错误
```
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:</h2>

    <ul>
      <% @article.errors.each do |error| %>
        <li><%= error.full_message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```

---