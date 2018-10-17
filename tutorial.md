## Cours Rails

[medium article](https://medium.com/@deallen7/how-to-build-a-todo-app-in-rails-e6571fcccac3)
### Initial
Create folder toto : `bundle init` 
Add  `gem 'rails', '~> 5.1', '>= 5.1.4'`
Execute `bundle exec rails new .`
---> conflit with Gemfile ---> override yes

Launch `bundle exec rails s`
—> Congrats first App rails
---> First commit 
  
### Generate model : todo_list
Create a model todo_list contains: title and description
`bundle exec rails g scaffold todo_list title:string description:text`

_“_**_Scaffolding_** _in_ **_Ruby on Rails_** _refers to the auto generation of a simple set of a model, views and controller usually for a single table. Would create a full CRUD (create, read, update, delete) web interface for the Users table”_

 The image demonstrates the interaction of a person with a webpage:
  ![
](https://cdn-images-1.medium.com/max/2000/1*kB9LJPI-dWuuo1mUJWtVUA.png)


Update database (apply model todo_list) : `bundle exec rake db:migrate`

create a note

In `config/routes.rb`: `root "todo_lists#index»`. (Define /todo_lists as /)
launch `bundle exec rails s` (show the current todo added before)

### Generate model : todo_item  
Create a model todo_item contains : content and a reference to a todo_list:
`bundle exec rails g model todo_item content:string todo_list:references`
Update database (apply todo_item): `bundle exec rake db:migrate`

Open migration `db/migrate/..create_todo_items.rb`
And we added a todo_list model, with an association to the todo_item model, that says the model “belongs_to :todo_list”

Open `app/models/todo_list.rb` and add: `has_many :todo_items`
(a toto_item refers to a toto_list, and a todo_list can have many todo_item)

Add in `config/routes.rb`:
```
resources :todo_lists do
	resources :todo_items
end
```

launch `bundle exec rake routes` (explains all routes)

create controller for this model: `bundle exec rails g controller todo_items`
open `app/controllers/todo_items_controller.rb` and add:
```
before_action :set_todo_list

  def create
    @todo_item = @todo_list.todo_items.create(todo_item_params)
    redirect_to @todo_list
  end

  private

  def set_todo_list
    @todo_list = TodoList.find(params[:todo_list_id])
  end

  def todo_item_params
    params[:todo_item].permit(:content)
  end
```
(explain before_action, set_todo_list, todo_item_params)

we’ve created an action for creating new todo’s, so now we need a form that will let us collect this data from the user in the views.

create two “partials” in our `app/views/todo_items` folder: `_todo_item.html.erb` and 
`_form.html.erb`

In `_form.html.erb`, add:
```
<%= form_for([@todo_list, @todo_list.todo_items.build]) do |f| %>  
	<%= f.text_field :content, placeholder: "New Todo" %>  
	<%= f.submit %>  
<% end %>
```
(explain @todo_list, @todo_list.todo_items.build)

In `_todo_item.html.erb`, add:
```
<p><%= todo_item.content %></p>
```

In `app/views/todo_lists/show.htm.erb`,  add before the 2 last line :
```
<div id="todo_items_wrapper">
  <%= render @todo_list.todo_items %>
  <div id="form">
    <%= render "todo_items/form" %>
  </div>
</div>
```
Update `app/views/todo_items/_todo_item.html.erb` :
`
<%= link_to todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
`
we want to replace the link: `/todo_lists/1/todo_items/1` by `Delete`, add 
` "Delete", ` after `link_to`

Refresh and click on Delete 
---> See error, need to create action destroy

Add action `destroy` in `app/controllers/todo_items_controller.rb`:
```
def destroy
  @todo_item = @todo_list.todo_items.find(params[:id])
  if @todo_item.destroy
    flash[:success] = "Todo List item was deleted."
  else
    flash[:error] = "Todo List item could not be deleted."
  end
  redirect_to @todo_list 
end
```
(explain flash and redirect)

### Mark an item as “complete”
create a migration: `bundle exec rails g migration add_completed_at_to_todo_items completed_at:datetime`
(explain name, completed)

Update database (apply change todo_item) : `bundle exec rake db:migrate`
In `config/routes.rb`, update `resources :todo_items`
```
resources :todo_items do
  member do
    patch :complete
  end
end
```
 In `app/views/todo_items/_todo_item.html.erb`,  add at the end :
`<%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>`

In `todo_items_controller.rb`, add :
after `private`: 
```
def set_todo_item  
  @todo_item = @todo_list.todo_items.find(params[:id])  
end
```
after the first `before_action`:
`before_action :set_todo_item, except: [:create]` (explain)
and before `private`:
```
def complete  
  @todo_item.update_attribute(:completed_at, Time.now)  
  redirect_to @todo_list, notice: "Todo item completed"  
end
```
Refresh and click on `Mark as complete`, we see `“Todo item completed”`
We need to update the view

Replace the content of `app/views/todo_items/_todo_item.html.erb`: 
```
<div class="row clearfix">  
 <% if todo_item.completed? %>  
  <div class="complete">  
   <%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>  
  </div>  
  <div class="todo_item">  
   <p style="opacity: 0.4;"><strike><%= todo_item.content %></strike></p>  
  </div>  
  <div class="trash">  
   <%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>  
  </div>  
 <% else %>  
  <div class="complete">  
   <%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>  
  </div>  
  <div class="todo_item">  
   <p><%= todo_item.content %></p>  
  </div>  
  <div class="trash">  
   <%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>  
  </div>  
 <% end %>  
</div>
```
(explain content)

Define `completed?` in `todo_item.rb` model:
```
def completed?  
  !completed_at.blank?  
end
```

### Add Style
rename the file in `app/assets/stylesheets/` -> `application.css` in `application.scss`
copy `https://github.com/mackenziechild/Todo-App` application.scss

In `app/views/layouts/application.html.erb`, we put our `<%= yield %>` inside a container.
```
<div class="container">
	<%= yield %>
</div>
```

In `app/views/layouts/todo_lists/index.html.erb`, replace the content by:
```
<% @todo_lists.each do |todo_list| %>
  <div class="index_row clearfix">
    <h2 class="todo_list_title"><%= link_to todo_list.title, todo_list %></h2>
    <p class="todo_list_sub_title"><%= todo_list.description %></p>
  </div>
<% end %>
<div class="links">
  <%= link_to "New Todo List", new_todo_list_path %>
</div>
```

At the end of `app/assets/stylesheets/application.scss`, add:
```
.todo_list_title {
 text-align: center;
 font-weight: 700;
 font-size: 2.5rem;
 text-transform: uppercase;
 color: white;
 margin: 0;
 a {
  text-decoration: none;
  color: white;
  transition: all .4s ease-in-out;
  &:hover {
   opacity: 0.4;
  }
 }
}
	
.todo_list_title {
 text-align: center;
 font-weight: 700;
 font-size: 1.0rem;
 text-transform: uppercase;
 color: white;
 margin: 0;
 a {
  text-decoration: none;
  color: white;
  transition: all .4s ease-in-out;
  &:hover {
   opacity: 0.4;
  }
 }
}
```

In `app/views/layouts/application.html.erb`, before `<\head>`, add:
```
<link href='http://fonts.googleapis.com/css?family=Lato:300,400,700' rel='stylesheet' type='text/css'>
<link href="//maxcdn.bootstrapcdn.com/font-awesome/4.2.0/css/font-awesome.min.css" rel="stylesheet">
```

In `app/views/todo_items/_todo_item.html.erb`, replace all content:
```
<div class="row clearfix">
  <% if todo_item.completed? %>
    <div class="complete">
      <%= link_to complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch do %>
        <i style="opacity: 0.4;" class="fa fa-check"></i>
      <% end %>
    </div>
    <div class="todo_item">
      <p style="opacity: 0.4;"><strike><%= todo_item.content %></strike></p>
    </div>
    <div class="trash">
      <%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
    </div>
  <% else %>
    <div class="complete">
      <%= link_to "Mark as Complete", complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch %>
    </div>
    <div class="todo_item">
      <p><%= todo_item.content %></p>
    </div>
    <div class="trash">
      <%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
    </div>
  <% end %>
</div>
```

updte the content between the `else` and the `end`
```
<div class="complete">
	<%= link_to complete_todo_list_todo_item_path(@todo_list, todo_item.id), method: :patch do %>
		<i class="fa fa-circle-thin"></i>
	<% end %>
</div>
<div class="todo_item">
	<p><%= todo_item.content %></p>
</div>
<div class="trash">
	<%= link_to "Delete", todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } %>
</div>
```

replace the content in `<div class="trash">`
```
<%= link_to todo_list_todo_item_path(@todo_list, todo_item.id), method: :delete, data: { confirm: "Are you sure?" } do %>
	<i class="fa fa-trash"></i>
<% end %>
```

In `app/views/todo_lists/show.html.erb`, replace the content by:
```
<p id="notice"><%= notice %></p>
<h2 class="todo_list_title"><%= @todo_list.title %></h2>
<p class="todo_list_sub_title"><%= @todo_list.description %></p>
<div id="todo_items_wrapper">
<%= render @todo_list.todo_items %>
  <div id="form">
    <%= render "todo_items/form" %>
  </div>
</div>
<%= link_to 'Edit', edit_todo_list_path(@todo_list) %> |
<%= link_to 'Back', todo_lists_path %>
```

encapsuled the two lonk a the end of the file by `<div class="links">` and `<\div>`
add this between `edit` and `back`:
`<%= link_to 'Delete', todo_list_path(@todo_list), method: :delete, data: { confirm: "Are you sure?" } %> |`


in `app/controller/todo_lists_controller.rb`, we nee to update the `destroy` method:
```
@todo_list.destroy
respond_to do |format|
	format.html { redirect_to root_url, notice: 'Todo list was successfully destroyed.' }
	format.json { head :no_content }
end
```
In `app/views/todo_lists/new.html.erb`, replace the content by:
```
<h1 class="todo_list_title">New Todo List</h1>
<div class="forms">
  <%= render 'form', todo_list: @todo_list %>
</div>
<div class="links">
  <%= link_to 'Back', todo_lists_path %>
</div>
```
In `app/views/todo_lists/edit.html.erb`, replace the content by:
```
<h1 class="todo_list_title">Editing Todo List</h1>
<div class="forms">
  <%= render 'form', todo_list: @todo_list %>
</div>
<div class="links">
  <%= link_to 'Cancel', todo_lists_path %>
</div>
```