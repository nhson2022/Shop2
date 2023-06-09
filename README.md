# Shop

## Create project
```bash
rails new Shop -c sass -j esbuild -d mysql
```

**Update database**
config/database.yml
```yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: localhost
  port: 3306
  username: root
  password: Son@2023
  socket: /var/run/mysqld/mysqld.sock

development:
  <<: *default
  database: shop_basic_development

test:
  <<: *default
  database: shop_basic_test

```

## Create database
```bash
rails db:create
```
## Setup Bootstrap jquery select2
```bash
yarn add bootstrap jquery @popperjs/core select2
```
**Update app/assets/stylesheets/application.sass.scss**
```scss
@import "bootstrap/dist/css/bootstrap";
@import "select2/dist/css/select2";
```
**Update app/javascript/application.js to add jquery and select2**
```js
// Setup jquery
import jquery from 'jquery';
window.jQuery = jquery;
window.$ = jquery;

import Select2 from "select2"
window.Select2 = Select2
Select2()

// load select2
document.addEventListener('turbo:load', () => {
  // apply to all elements that have class .select2
  $('.select2').select2()
})
```

## Run App
```bash
./bin/dev
```

## Create Address model, controller, view (scaffold)
```bash
rails g scaffold Address town:text phone:string
```

## Setup Devise for User
```bash
bundle add devise
rails generate devise:install
rails g devise:views

rails generate devise User
rails db:migrate
```
**Update config/environments/development.rb**
```rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
**Add first_name, lasname, is_admin for User**
```bash
bin/rails generate migration AddIsAdminToUsers is_admin:boolean first_name:text last_name:text
```
**Update app/views/layouts/application.html.erb**
```html
<div class="container">
  <% if notice.present? %>
    <div class="alert alert-primary mt-4" role="alert">
      <%= notice %>
    </div>
  <% end %>

  <% if alert.present? %>
    <div class="alert alert-danger mt-4" role="alert">
      <%= alert %>
    </div>
  <% end %>

  <%= yield %>
</div>
```

## Setup Active Storage
```bash
bin/rails active_storage:install
bin/rails db:migrate
```

## Setup Action Text
```bash
bin/rails action_text:install
bin/rails db:migrate
```

## Create pages home and about
```bash
rails g controller pages home about
```
**Update routes**
```rb
root "pages#home"
get "/about" => "pages#about", as: :pages_about
```

## Create Category model, controller, view (scaffold)
```bash
rails g scaffold Category name description:text
```

## Create Product model, controller, view (scaffold)
```bash
rails g scaffold Product title active:boolean quantity:integer price:integer category:references
```

**Create category_select_options helper method in app/helpers/application_helper.rb**
```rb
# app/helpers/application_helper.rb
def category_select_options
  Category.all.map { |c| [c.name, c.id] }
end
```

**Migration database**
```bash
rails db:migrate
```

## Setup gem will_paginate for pagination
```bash
# https://github.com/mislav/will_paginate
bundle add will_paginate
```
**Will Paginate link renderer bootstrap styles**
```bash
bundle add will_paginate-bootstrap-style
```

## Create seed data in db/seeds.rb
```rb
admin = User.create(first_name: 'Nguyen', last_name: 'Son', email: 'son@example.com', password: 'demo2023', password_confirmation: 'demo2023', is_admin: true)

```

**Run db:seed**
```bash
rails db:seed
```

## Bootstrap style for all pages

## Custom Turbo Confirm Modals with Hotwire in Rails
**Update layout application.html.erb**
```html
<!-- app/views/layouts/application.html.erb -->
<dialog id="turbo-confirm">
  <form method="dialog">
    <p>Are you sure?</p>
    <div>
      <button value="cancel">Cancel</button>
      <button value="confirm">Confirm</button>
    </div>
  </form>
</dialog>
```

**Update application.js**
```js
// app/javascript/application.js
Turbo.setConfirmMethod((message, element) => {
  console.log(message, element)
  let dialog = document.getElementById("turbo-confirm")
  dialog.querySelector("p").textContent = message
  dialog.showModal()

  return new Promise((resolve, reject) => {
    dialog.addEventListener("close", () => {
      resolve(dialog.returnValue == "confirm")
    }, { once: true })
  })
})
```

## Implement search products
**Create this static method in product model**
```rb
# app/models/product.rb

def self.search(params)
  if params[:q].present?
    return includes(:user).with_rich_text_content_and_embeds
              .where("LOWER(title) LIKE LOWER(?)", "%#{params[:q].to_s.squish}%")
              .order(id: :desc)
              .paginate(page: params[:page] || 1, per_page: 10)
  end

  includes(:user).with_rich_text_content_and_embeds
    .order(id: :desc)
    .paginate(page: params[:page] || 1, per_page: 10)
end
```

## Create Cart model, controller, view (scaffold)
```bash
rails g scaffold Cart quantity:integer money:integer product:references user:references
```

## Create Payment model, controller, view (scaffold)
```bash
rails g scaffold Payment paid:integer cart:references user:references address:references
```

