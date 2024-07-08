# photogram-ajax

Target: https://pg-ajax-1.matchthetarget.com/

Notes:

### A. The _comments.html.erb file.

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

### B. Ajaxify destroy

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
  ii. Add the appropriate function to the comments_controller.rb:
      
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

4. Add the following and make sure that the app still works:

```
<!-- app/views/comments/_comment.html.erb -->

<%= link_to comment,
      data: { turbo_method: :delete },
      class: "btn btn-link btn-sm text-muted",
      remote: true do %>
  
  <i class="fas fa-trash fa-fw"></i>
<% end %>
<!-- ... -->
<script>
  // bind the click handler
  $("#comment_<%= comment.id %>_delete_link").on("click", function() { 
    
    $.ajax({
      url: "/comments/<%= comment.id %>", // what URL to submit a request to
      type: "DELETE",                     // make it a DELETE request
      dataType: "script"                  // what is the format of the request
    });

    return false;                         // break the link
  });
</script>
```
