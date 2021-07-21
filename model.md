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
# 创建
bin/rails generate model Author
bin/rails generate model Book

bin/rails db:migrate

# 更改
bin/rails generate migration AddAuthorRefToBook author:references
bin/rails generate migration AddNameToBook 'name:string{20}'

bin/rails db:migrate

# 删除列
bin/rails generate migration RemoveNameFromBook name:string

bin/rails db:migrate

# migration创建表
bin/rails generate migration CreateTemp2 name:string{20} count:integer
rake db:migrate
```

### Migration methods:
- add_column
- add_foreign_key
- add_index
- add_reference
- add_timestamps
- change_column_default (must supply a :from and :to option)
- change_column_null
- create_join_table
- create_table
- disable_extension
- drop_join_table
- drop_table (must supply a block)
- enable_extension
- remove_column (must supply a type)
- remove_foreign_key (must supply a second table)
- remove_index
- remove_reference
- remove_timestamps
- rename_column
- rename_index
- rename_table

如果不够可以些SQL


### 运行Migration
```
# rollback
bin/rails db:rollback
bin/rails db:rollback STEP=3

# reset
bin/rails db:drop
bin/rails db:reset

# running specific migration
bin/rails db:migrate VERSION=20210720060447

# in different env
bin/rails db:migrate RAILS_ENV=test
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

#### belongs_to（存其他表id）, has_one
```
# book belongs to author
create table authors (
    id integer auto increment primary key,
    name varchar(20)
);

create table books (
    id integer auto increment primary key,
    author_id integer
);
```

```
# supplier has one account
create table suppliers (
    id integer auto increment primary key,
    name varchar(20)
);

create table accounts (
    id integer auto increment primary key,
    supplier_id integer
);
```

#### has_many
```
class Author < ApplicationRecord
    has_many :books
end

class Book < ApplicationRecord
    belongs_to :author
end
```

#### has_many :through
customers <==> orders <==> products
```
class Customer < ApplicationRecord
    has_many :orders
    has_many :products, through: :orders
end

class Product < ApplicationRecord
    has_many :orders
    has_many :customers, through: :orders
end

class Order < ApplicationRecord
    belongs_to :customer
    belongs_to :product
end

bin/rails generate model product
bin/rails generate model customer
bin/rails generate model order product:references customer:references

bin/rails db:migrate
```

#### has_one :through
suppliers <= accounts <= account_histories
```
class Supplier < ApplicationRecord
  has_one :account
  has_one :account_history, through: :account
end

class Account < ApplicationRecord
  belongs_to :supplier
  has_one :account_history
end

class AccountHistory < ApplicationRecord
  belongs_to :account
end


bin/rails g model Supplier
bin/rails g model Account supplier:references
bin/rails g model AccountHistory credit_rating:integer account:references

bin/rails db:migrate
```

#### has_and_belongs_to_many (中间表不需要model对象)
assemblies <= assemblies_parts => parts
```
# models
class Assembly < ApplicationRecord
  has_and_belongs_to_many :parts
end

class Part < ApplicationRecord
  has_and_belongs_to_many :assemblies
end

# migrations
class CreateAssembliesAndParts < ActiveRecord::Migration[6.0]
  def change
    create_table :assemblies do |t|
      t.string :name
      t.timestamps
    end

    create_table :parts do |t|
      t.string :part_number
      t.timestamps
    end

    create_table :assemblies_parts, id: false do |t|
      t.belongs_to :assembly
      t.belongs_to :part
    end
  end
end
```

#### Self Joins
```
class Employee < ApplicationRecord
  has_many :subordinates, class_name: "Employee",
                          foreign_key: "manager_id"
  belongs_to :manager, class_name: "Employee", optional: true
end
```

#### 其他
- 关联默认作用域是同module
- 双向关联需要在两个model对象上定义
- 关联会增加getter, setter, build_*, create_*, create_*!, reload_*六个方法
- 可以通过where或者select等option 改sql等行为（具体见文档）


---
## Query interface
```
Customer.find(1)

select * from customers where (customers.id = 10) limit 1

Customer.take
select * from customers limit 1

Customer.take(2)
select * from customers limit 2

Customer.first
select * from customers order by customer.id asc limit 1

Customer.first(3)
select * from customers order by customer.id asc limit 3

Customer.order(:first_name).first
select * from customers order by customers.fist_name asc limit 1

Customer.last
select * from customers order by customer.id desc limit 1

Customer.find_by first_name: 'Foo'
Customer.where(first_name: 'Foo').take
select * from customers where (cusotmers.first_name = 'Foo') limit 1

Customer= Customer.find([1, 10])
select * from customers where (customers.id in (1, 10))

Customer.all.each do |customer|
    # do something
end

Customer.find_each do |customer|
    # do something
end

# not
Customer.where.not(orders_count: [1, 3, 5])
select * from customers where customers.orders_count not in (1, 3, 5)

# or
Customer.where(last_name: 'Foo').or(Customer.where(orders_count: [1, 3, 5]))
select * from customers where (customers.last_name = 'Foo' or customers.orders_count in (1, 3, 5))

# ordering
Customer.order(:created_at)
# OR
Customer.order("created_at")

Customer.order(orders_count: :asc, created_at: :desc)

# fields
Customer.select(:isbn, :out_of_print)
select isbn, out_of_print from customers

# distinct
Customer.select(:last_name).distinct
select distinct last_name from customers

# limit and offset
Customer.limit(5).offset(30)
select * from customers limit 5 offset 30

# group
Customer.select(:created_at).group(:created_at)
select created_at from customers group by created_by

# count
Customer.group(:status).count
select count(*) as count_all, status from customers group by status

# having
Customer.select("created_at, sum(total) as total_price").group(:created_at).having("sum(total) > ?", 200)
select created_at, sum(total) as total_price from orders group by created_at having sum(total) > 200

# start: 2000, finish: 10000
Customer.where(name: 'Foo').find_each(start: 2000, batch_size: 1000) do |customer|
    # do something
end

Customer.find_in_batches(batch_size: 2500, start: 5000) do |customer|
    # do something
end
```

### 悲观锁
```
Customer.transaction do
    customer = Customer.lock.first
    customer.total = 100
    customer.save!
end

Customer.transaction do
    customer = Customer.lock().find(1)
    ...
end

customer = Customer.first
customer.with_lock do
    ...
end

```

### join
#### join with sql
```
Author.joins("INNER JOIN books ON books.author_id = authors.id AND books.out_of_print = FALSE")

SELECT authors.* FROM authors INNER JOIN books ON books.author_id = authors.id AND books.out_of_print = FALSE
```

#### join with associations
```
Book.joins(:reviews)
SELECT books.* FROM books INNER JOIN reviews ON reviews.book_id = books.id

Book.joins(:author, :reviews)
SELECT books.* FROM books
  INNER JOIN authors ON authors.id = books.author_id
  INNER JOIN reviews ON reviews.book_id = books.id

# 嵌套
Book.joins(reviews: :customer)

SELECT books.* FROM books
  INNER JOIN reviews ON reviews.book_id = books.id
  INNER JOIN customers ON customers.id = reviews.customer_id

```

#### Enums
```
class Order < ApplicationRecord
  enum status: [:shipped, :being_packaged, :complete, :cancelled]
end
```

#### Calculation
- count
- average
- minimum
- maximum
- sum

#### 执行计划
```
Customer.where(id: 1).joins(:orders).explain
```



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