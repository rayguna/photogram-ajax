# photogram-ajax

Target: https://pg-ajax-1.matchthetarget.com/

Notes:

### A. The _comments.html.erb file.

1. Since we are modifying the file called `app/views/comments/_comment.html.erb`, we need to know how to inspect the html page.
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


