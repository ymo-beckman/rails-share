# controller
## 基类 ApplicationController
```
# 查看所有路由项
bin/rails routes
```

---

## 命名约定
The naming convention of controllers in Rails favors pluralization of the last word in the controller's name, although it is not strictly required (e.g. ApplicationController). For example, ClientsController is preferable to ClientController, SiteAdminsController is preferable to SiteAdminController or SitesAdminsController, and so on.
Following this convention will allow you to use the default route generators (e.g. resources, etc) without needing to qualify each :path or :controller, and will keep named route helpers' usage consistent throughout your application.

---

## 配置路由
config/routes.rb

```
Rails.application.routes.draw do
  resources :brands, only: [:index, :show] do
    resources :products, only: [:index, :show]
  end

  resource :basket, only: [:show, :update, :destroy]

  resolve("Basket") { route_for(:basket) }
end
```

---

## Router, Methods and Actions

### 定义路由项
- resources :photos
- resources :photos, :books, :videos
- get 'profile', to: 'users#show'
- match 'photos', to: 'photos#show', via: [:get, :post]
- match 'photos', to: 'photos#show', via: :all
- get 'photos/*other', to: 'photos#unknown'
- root to: 'pages#main'
- root 'pages#main' # shortcut for the above 

---

### 命名路由
```
scope 'admin' do
  resources :photos, as: 'admin_photos'
end

resources :photos

# This will provide route helpers such as admin_photos_path, new_admin_photo_path, etc.

```

---

### 定义photos资源
```
resources :photos
```
会生成一下url与action映射

| HTTP Verb	| Path	| Controller#Action	| Used for |
|---|---|---|---|
| GET	| /photos	| photos#index	| display a list of all photos |
|GET	| /photos/new	| photos#new	| return an HTML form for creating a new photo |
|POST	| /photos	| photos#create	| create a new photo |
|GET	| /photos/:id	| photos#show	| display a specific photo |
|GET	| /photos/:id/edit	| photos#edit	| return an HTML form for editing a photo |
|PATCH/PUT	| /photos/:id	| photos#update	| update a specific photo |
|DELETE	| /photos/:id	| photos#destroy	| delete a specific photo |

helpers:
- photos_path returns /photos
- new_photo_path returns /photos/new
- edit_photo_path(:id) returns /photos/:id/edit (for instance, edit_photo_path(10) returns /photos/10/edit)
- photo_path(:id) returns /photos/:id (for instance, photo_path(10) returns /photos/10)
- Each of these helpers has a corresponding _url helper (such as photos_url) which returns the same path prefixed with the current host, port, and path prefix.

### 单个映射
```
get 'profile', to: 'users#show'
```

---

### 嵌套
```
class Magazine < ApplicationRecord
  has_many :ads
end

class Ad < ApplicationRecord
  belongs_to :magazine
end


resources :magazines do
  resources :ads
end
```

|HTTP Verb	|Path	|Controller#Action	|Used for|
|---|---|---|---|
|GET	|/magazines/:magazine_id/ads	|ads#index	|display a list of all ads for a specific magazine|
|GET	|/magazines/:magazine_id/ads/new	|ads#new	|return an HTML form for |creating a new ad belonging to a specific magazine|
|POST	|/magazines/:magazine_id/ads	|ads#create	|create a new ad belonging to a specific magazine|
|GET	|/magazines/:magazine_id/ads/:id	|ads#show	|display a specific ad belonging to a specific magazine|
|GET	|/magazines/:magazine_id/ads/:id/edit	|ads#edit	|return an HTML form for editing an ad belonging to a specific magazine|
|PATCH/PUT	|/magazines/:magazine_id/ads/:id	|ads#update	|update a specific ad belonging to a specific magazine|
|DELETE	|/magazines/:magazine_id/ads/:id	|ads#destroy	|delete a specific ad belonging to a specific magazine|

---

### routing concerns (内嵌复用)
```
concern :commentable do
  resources :comments
end
concern :image_attachable do
  resources :images, only: :index
end


resources :messages, concerns: :commentable
resources :articles, concerns: [:commentable, :image_attachable]
```

---

## 参数
### params:
- query string
- form
- json
- path_variable
- action

## default url option
```
class ApplicationController < ActionController::Base
  def default_url_options
    { locale: I18n.locale }
  end
end
```

## session
## cookies

## rendering（详细见view部分）
```
class UsersController < ApplicationController
  def index
    @users = User.all
    respond_to do |format|
      format.html # index.html.erb
      format.xml  { render xml: @users }
      format.json { render json: @users }
    end
  end
end
```

---

## Filter
- Filters are methods that are run "before", "after" or "around" a controller action.
- Filters are inherited, so if you set a filter on ApplicationController, it will be run on every controller in your application.

  ### Filter actions:
  - before
  - after
  - around

```
class ApplicationController < ActionController::Base
  before_action :require_login

  private

  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end
```

You can prevent this filter from running before particular actions with skip_before_action:
```
class LoginsController < ApplicationController
  skip_before_action :require_login, only: [:new, :create]
end
```

"after" filters are registered via after_action. They are similar to "before" filters, but because the action has already been run they have access to the response data that's about to be sent to the client. Obviously, "after" filters cannot stop the action from running. Please note that "after" filters are executed only after a successful action, but not when an exception is raised in the request cycle.

"around" filters are registered via around_action. They are responsible for running their associated actions by yielding, similar to how Rack middlewares work.

```
class ChangesController < ApplicationController
  around_action :wrap_in_transaction, only: :show

  private

  def wrap_in_transaction
    ActiveRecord::Base.transaction do
      begin
        yield
      ensure
        raise ActiveRecord::Rollback
      end
    end
  end
end

```

While the most common way to use filters is by creating private methods and using before_action, after_action, or around_action to add them, there are two other ways to do the same thing.

```
class ApplicationController < ActionController::Base
  before_action do |controller|
    unless controller.send(:logged_in?)
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url
    end
  end
end

class ApplicationController < ActionController::Base
  before_action LoginFilter
end

class LoginFilter
  def self.before(controller)
    unless controller.send(:logged_in?)
      controller.flash[:error] = "You must be logged in to access this section"
      controller.redirect_to controller.new_login_url
    end
  end
end

```

---

## The Request and Response Objects
### Controller中调用reuqest返回ActionDispatch::Request对象
params从request对象中来，对应三个方法:
- query_parameters | query string
- request_parameters | form body
- path_parameters | path variable


|Property of request	| Purpose|
|---|-----|
|host	|The hostname used for this request.|
|domain(n=2)	|The hostname's first n segments, starting from the right (the TLD).|
|format	|The content type requested by the client.|
|method	|The HTTP method used for the request.|
|get?, post?, patch?, put?, delete?, head?	|Returns true if the HTTP method is GET/POST/PATCH/PUT/DELETE/HEAD.|
|headers	|Returns a hash containing the headers associated with the request.|
|port	|The port number (integer) used for the request.|
|protocol	|Returns a string containing the protocol used plus "://", for example "http://".|
|query_string	|The query string part of the URL, i.e., everything after "?".|
|remote_ip	|The IP address of the client.|
|url	|The entire URL used for the request.|

---

### 调用response返回ActionDispatch::Response对象
|Property of response	|Purpose|
|---|-----|
|body	|This is the string of data being sent back to the client. This is most often HTML.|
|status	|The HTTP status code for the response, like 200 for a successful request or 404 for file not found.|
|location	|The URL the client is being redirected to, if any.|
|content_type	|The content type of the response.|
|charset	|The character set being used for the response. Default is "utf-8".|
|headers	|Headers used for the response.|

---

### 自定义headers
```
response.headers["Content-Type"] = "application/pdf"

or

response.headers.content_type = "application/pdf"
```