# photogram-ajax

Target: https://pg-ajax-1.matchthetarget.com/

Notes:

### A. The _comments.html.erb file.

1. Start with typing `rake sample_data` to populate tables. You can verify that the rake file exists by going to `lib/tasks/dev.rake`. After populating the tables, visit the ..rails/db page to check.
2. Since we are modifying the file called `app/views/comments/_comment.html.erb`, we need to know how to inspect the html page.
3. Looking at config/routes.rb, I can see that comments routes exist:

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
