# Template
- erb
- haml
- builder (xml)
- jbuilder (json)

```
<% @products.each do |product| %>
  <%= render partial: "product", locals: { product: product } %>
<% end %>
```

# Partials
```
<%= render "shared/ad_banner" %>

<h1>Products</h1>

<p>Here are a few of our fine products:</p>
<% @products.each do |product| %>
  <%= render partial: "product", locals: { product: product } %>
<% end %>

<%= render "shared/footer" %>
```

# layout
```
# view
<%= render partial: 'article', layout: 'box', locals: { article: @article } %>


# layout articlees/_box.html.erb

<div class='box'>
  <%= yield %>
</div>
```

# haml
## erb => haml
```
# erb
<strong><%= item.title %></strong>

# haml
%strong= item.title
```
## attributes
```
# html
<strong class="code" id="message">Hello, World!</strong>

# haml
%strong{:class => "code", :id => "message"} Hello, World!
```

## 默认标签是div
```
# haml
.content Hello, World!

#html
<div class='content'>Hello, World!</div>
```
## 变量
```
# erb
<div class='item' id='item<%= item.id %>'>
  <%= item.body %>
</div>

# haml
.item{:id => "item#{item.id}"}= item.body
```

## 使用partial
```
# erb
<div id='content'>
  <div class='left column'>
    <h2>Welcome to our site!</h2>
    <p><%= print_information %></p>
  </div>
  <div class="right column">
    <%= render :partial => "sidebar" %>
  </div>
</div>

# haml
<div id='content'>
  <div class='left column'>
    <h2>Welcome to our site!</h2>
    <p><%= print_information %></p>
  </div>
  <div class="right column">
    <%= render :partial => "sidebar" %>
  </div>
</div>
```

# response
```
# controller
class BooksController < ApplicationController
  def index
    @books = Book.all
  end
end

# view
<h1>Listing Books</h1>

<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Content</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @books.each do |book| %>
      <tr>
        <td><%= book.title %></td>
        <td><%= book.content %></td>
        <td><%= link_to "Show", book %></td>
        <td><%= link_to "Edit", edit_book_path(book) %></td>
        <td><%= link_to "Destroy", book, method: :delete, data: { confirm: "Are you sure?" } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to "New book", new_book_path %>
```

## render
```
render "products/show"
```

## render inline
```
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

## render text
```
render plain: "OK"
```

## render json
```
render json: @product
```

## render xml
```
render xml: @product
```

## render javascript
```
render js: "alert('Hello Rails');"
```

## render raw file
```
render file: "#{Rails.root}/public/404.html", layout: false
```

## content type
```
render template: "feed", content_type: "application/rss"
```