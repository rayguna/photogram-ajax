# photogram-ajax

Target: https://pg-ajax-1.matchthetarget.com/

Notes:

### I. The _comments.html.erb file.

1. Start with typing `rake sample_data` to populate tables. You can verify that the rake file exists by going to `lib/tasks/dev.rake`. After populating the tables, visit the ..rails/db page to check.
2. Looking at config/routes.rb, I can see that comments routes exist:

```
Rails.application.routes.draw do
  root "users#feed"

  devise_for :users
  
  resources :comments
  resources :follow_requests
  resources :likes
  resources :photos

  get ":username" => "users#show", as: :user
  get ":username/liked" => "users#liked", as: :liked
  get ":username/feed" => "users#feed", as: :feed
  get ":username/discover" => "users#discover", as: :discover
end
```
3. we can also explore the routes by visiting:
`https://super-xylophone-6r9qwqx45vr2r6ww-3000.app.github.dev/rails/info/routes`. In the textbox, you can type "fields".

### II. Ajaxify destroy

1. To implement comment destroy via ajax.
2.  You need to modify three files:
  i. app/views/comments/_comment.html.erb: to the link_to tag, add method: :delete and remote: true do

      ```
    <li class="list-group-item">
      <div class="d-flex">
        <div class="flex-shrink-0">
          <%= image_tag comment.author.avatar_image, class: "rounded-circle mr-2", width: 36 %>
        </div>
        <div class="flex-grow-1 ms-3">
          <h5 class="mt-0">
            <%= link_to comment.author.username, user_path(comment.author.username), class: "text-dark link-dark link-underline-opacity-0 link-underline-opacity-100-hover" %>
          </h5>
          <p><%= comment.body %></p>
        </div>
        <div>
          <% if current_user == comment.author %>
            <%= link_to edit_comment_path(comment), class: "btn btn-link btn-sm text-muted" do %>
              <i class="fas fa-edit fa-fw"></i>
            <% end %>

            <%= link_to comment, method: :delete, remote: true, class: "btn btn-link btn-sm text-muted" do %>
              <i class="fas fa-trash fa-fw"></i>

            <% end %>
          <% end %>
        </div>
      </div>
    </li>
      ```
  ii. Within the appropriate function in comments_controller.rb, add format.js do block to link to the js.erb file:
      
      ```
      # app/controllers/comments_controller.rb

      # ...
      def destroy
        @comment.destroy
        respond_to do |format|
          format.html { redirect_back fallback_location: root_url, notice: "Comment was successfully destroyed." }
          format.json { head :no_content }
          
          format.js do
            render template: "comments/destroy"
          end
        end
      end
      # ...
      ```

  iii. create a .js.erb file within the views folder.

      ```
      // app/views/comments/destroy.js.erb

      console.log("bye comment!")
      ```

3. Remove the comment
  -  Add unique ID to the li tag:

  ```
  <!-- app/views/comments/_comment.html.erb -->

  <li id="comment_<%= comment.id %>" class="list-group-item">
    <div class="d-flex">
      <div class="flex-shrink-0">
      <!-- ... -->
  ```

- Alternatively, you may pass the id with rails as follows:

  ```
  <!-- app/views/comments/_comment.html.erb -->

  <li id="<%= dom_id(comment) %>" class="list-group-item">
  ```

- In this case, we will pass the with ajax.

-  Remove the comment with:

  ```
  // app/views/comments/destroy.js.erb

  $("#<%= dom_id(@comment) %>").remove();
  ```

- Add a fading effect:

  ```
  // app/views/comments/destroy.js.erb

  $("#<%= dom_id(@comment) %>").fadeOut(5000, function() {
    $(this).remove();
  });
  ```

  ### III. Ajaxify create

  1. We will follow the same pattern as deleting in the above:

  i. Switch the request (in this case, a form instead of a link) from HTML to JS. Add the keyword local: false.
    
    ```
    <!-- app/views/comments/_form.html.erb -->

    <li class="list-group-item">
      <%= form_with(model: comment, local: false) do |form| %>
        <% if comment.errors.any? %>
        <!-- ... -->
    ```

  ii. Update the respond_to block to handle requests for JS. add the block:
  ```
  format.js do
    render template: "comments/create.js.erb"
  end
  ```

  or, to be precise, you can replace the whole block with:

  ```
  format.js
  ```

  into the if block. The above short for mis possible because: The view folder name matches the controller name; The template name matches the action name; and The request format matches the file extension.

  ```
  # app/controllers/comments_controller.rb

  # ...
  def create
    @comment = Comment.new(comment_params)
    @comment.author = current_user

    respond_to do |format|
      if @comment.save
        format.html { redirect_back fallback_location: root_path, notice: "Comment was successfully created." }
        format.json { render :show, status: :created, location: @comment }
        
        format.js do
          render template: "comments/create"
        end
      else
        # ...
  ```

  iii. Write a JS response template within the app/views/comments:

  ```
  // app/views/comments/create.js.erb

  console.log("<%= @comment.body %>")
  console.log("Comment created successfully")
  ```

- Add some effects: https://learn.firstdraft.com/lessons/203-minimal-js#frequently-used-jquery-methods

2. Format input by adding a dom_id as in the destroy function above. Note the specific selector called .photo:

```
<!--  app/views/comments/_form.html.erb -->

<li id="<%= dom_id(comment.photo) %>_new_comment_form" class="list-group-item">
```

3. Rendering element with partials: Update the js template accordingly using the same selector. Note the render path points to the partial, _form.html.erb.

```
// app/views/comments/create.js.erb

var added_comment = $("<%= j(render @comment) %>");

added_comment.hide();

$("#<%= dom_id(@comment.photo) %>_new_comment_form").before(added_comment);

added_comment.slideDown();

//delete the previous comments in the comments field.
$("#<%= dom_id(@comment.photo) %>_new_comment_form #comment_body").val("");

//console.log("Comment created successfully")
```

- Why doesn't the page get refreshed automatically to show the effect? How toadd timer?

### IV. Abbreviations 
