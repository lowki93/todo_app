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
