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

            <%= link_to comment, method: :delete, class: "btn btn-link btn-sm text-muted", remote: true do %>
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

- Why doesn't the page get refreshed automatically to show the effect? How to add timer?

Added a refresh command:

```
// app/views/comments/create.js.erb


var added_comment = $("<%= j(render @comment) %>");

//hide comment
added_comment.hide();

$("#<%= dom_id(@comment.photo) %>_new_comment_form").before(added_comment);

//reveal comment with effect
added_comment.slideDown();

//delete the previous comments in the comments field.
$("#<%= dom_id(@comment.photo) %>_new_comment_form #comment_body").val("");

/*
//refresh page: https://tecadmin.net/refresh-page-with-jquery/
$(document).ready(function(){
   location.reload(true);
});

*/

//display message in console.
console.log("<%= @comment.body %>")
console.log("Comment created successfully")
```

TIPS: check the order of the parameters. If incorrect, the comment won't refresh correctly. If correct, the refresh page comment is no longer needed. 

### IV. Abbreviations 

Abbreviations:

1. escape_javascript() helper — j():

```
var added_comment = $("<%= j(render 'comments/comment', comment: @comment) %>");
```

2. 
```
render 'comments/comment', comment: @comment
```
as

```
render @comment
```

3. Using both:

```
var added_comment = $("<%= j(render @comment) %>");
```

### V. Ajaxify edit

1. Objective: when the edit icon is clicked, let’s replace the comment with an edit form, right in place.

2. Follow the three-step process:
  - Switch the link/form from HTML to JS with remote: true and (method: :delete if it is a DELETE request) on link_to, or local: false on form_with.
    
    ```
    <!-- app/views/comments/_comment.html.erb -->

    <% if current_user == comment.author %>
    <%= link_to edit_comment_path(comment), class: "btn btn-link btn-sm text-muted", remote: true do %>
      <i class="fas fa-edit fa-fw"></i>
    <% end %>
    ```

  - Add format.js to the appropriate respond_to block within comments_controller.rb.

    ```
    //comments_controller.rb

    def edit
      respond_to do |format|
        format.html
        format.js
      end
    end
    ```

  - Write a JS response template. This will usually involve:
    - Using or creating partials to represent the components being rendered via Ajax.
    - Adding top-level elements with id="" attributes to the partials, if they don’t already have them.
    - Writing some jQuery to select an existing element in the DOM and insert near it, replace it, etc.

    ```
    //app/views/comments/edit.js.erb

    $("#<%= dom_id(@comment) %>").replaceWith("<%= j(render "comments/form", comment: @comment) %>");
    ```

### V. Ajaxify update

1. Objective: when that form is submitted, let’s replace it with the updated comment, right in place.

2. Follow the three-step process:
  - Switch the link/form from HTML to JS with remote: true and (method: :delete if it is a DELETE request) on link_to, or local: false on form_with.
    ```   
    <!--app/views/comments/_form.html.erb-->

    <li id="<%= dom_id(comment.photo) %>_<%= dom_id(comment) %>_form" class="list-group-item">
    ```

  - Add format.js to the appropriate respond_to block within comments_controller.rb.

    ```
    //app/controllers/comments_controller.rb

    def update
      respond_to do |format|
        if @comment.update(comment_params)
          format.html { redirect_to root_url, notice: "Comment was successfully updated." }
          format.json { render :show, status: :ok, location: @comment }
          format.js
        else
          format.html { render :edit, status: :unprocessable_entity }
          format.json { render json: @comment.errors, status: :unprocessable_entity }
        end
      end
    end
    ```

  - Write a JS response template. This will usually involve:
    - Using or creating partials to represent the components being rendered via Ajax.
    - Adding top-level elements with id="" attributes to the partials, if they don’t already have them.
    - Writing some jQuery to select an existing element in the DOM and insert near it, replace it, etc.

    ```
    //app/views/comments/update.js.erb

    $("#<%= dom_id(@comment.photo) %>_<%= dom_id(@comment) %>_form").replaceWith("<%= j(render @comment) %>");
    ```

### VI. Other challenges
Explore the target to find other things to practice Ajax on:

- Like/unlike

- Follow/unfollow

Lesson: https://learn.firstdraft.com/lessons/204-rails-unobtrusive-ajax

Solutions:
  - comments#destroy: https://github.com/appdev-projects/photogram-ajax/commit/ad9b6de74d6048b6f1c4b7dde6d3b5fd0f967194
  - comments#create: https://github.com/appdev-projects/photogram-ajax/commit/b6cf072aa42a8a5e043218c4e02a92c65e6446dc
  - comments#edit: https://github.com/appdev-projects/photogram-ajax/commit/8aab3e621e7a47030ce2dec916866ff8b141d16a
  - comments#update: https://github.com/appdev-projects/photogram-ajax/commit/c18294a800e01a240bd2339a9c708d1cb64cf4dc

<hr>

### VII. Ajaxify like/unlike button

1. Follow the three-step process:
  i. Switch the link/form from HTML to JS with remote: true and (method: :delete if it is a DELETE request) on link_to, or local: false on form_with.
      - just like in the tutorial, we have to modify views/likes/_form.html for create. Add a specific tag and add local: false.
        ```
        <li id="<%= dom_id(like.photo) %>_new_like_form" class="list-group-item">

        <%= form_with(model: like, local: false,class: "d-inline-block") do |form| %>
        ```

      - just like in the tutorial, we have to modify views/likes/_like.html.erb for destroy
        ```
        NEED TO DO
        ```

  ii. Add format.js to the appropriate respond_to block within `likes_controller.rb`. Observation: The liking and unliking a photo results in a creation and a destruction of likes.
      - add format.js within def create and def destroy.
      ```
      def create
        @like = Like.new(like_params)

        respond_to do |format|
          if @like.save
            format.html { redirect_back fallback_location: root_url, notice: "Like was successfully created." }
            format.json { render :show, status: :created, location: @like }

            format.js
            
          else
        ...
      ```

      and

      ```
      def destroy
        @like.destroy
        respond_to do |format|
          format.html { redirect_back fallback_location: root_url, notice: "Like was successfully destroyed." }
          format.json { head :no_content }

          format.js
          
        end
      end
      ...
      ```

      - create views/likes/create.js.erb and views/likes/destroy.js.erb. 

      views.likes/create.js.erb
      ```
      console.log("Howdy from create.")
      ```

      views.likes/destroy.js.erb
      ```
      console.log("Howdy from destroy.")
      ```

NEED TO DO:
  iii. Write a JS response template. This will usually involve:
    - Using or creating partials to represent the components being rendered via Ajax.
    - Adding top-level elements with id="" attributes to the partials, if they don’t already have them.
    - Writing some jQuery to select an existing element in the DOM and insert near it, replace it, etc.

 
-> Let's tackle one feature at a time. We will start out with ajaxing the like/create method first, we will worry about the unlike/destroy method second.

2. Final solution and bottom line.
    - You need to modify three files to ajaxify the CRUDE actions. Two of the modifications (one with the controller.rb file and another with the .html.erb file to re-route to js.erb) are straightforward.
    - The third file is the creation of the .js.erb file and inserting the appropriate contents. For this modification, you need recognize the dom-id. You can determine that by inspecting the html page.

    Here are the solutions for the js.eb files for each of the CRUDE actions for comments. You need to recognize the shorthand-one-line commands to appreciate the code:

    C- CREATE
    - create comment

      ```
      //app/views/comments/create.js.erb

      var added_comment = $("<%= j(render @comment) %>");

      added_comment.hide();

      $("#<%= dom_id(@comment.photo) %>_new_comment_form").before(added_comment);

      added_comment.slideDown();

      $("#<%= dom_id(@comment.photo) %>_new_comment_form #comment_body").val("");
      ```

    R - READ
      ```
      None
      ```

    U - UPDATE
    - update comment
      ```
      app/views/comments/update.js.erb

      $("#<%= dom_id(@comment.photo) %>_<%= dom_id(@comment) %>_form").replaceWith("<%= j(render @comment) %>");
      ```

    D - DELETE 

    - destroy comment

      ```
      //app/views/comments/destroy.js.erb

      $("#<%= dom_id(@comment) %>").fadeOut(function() {
        $(this).remove();
      });
      ```

    <hr>

    E - EDIT

    ```
    //app/views/comments/edit.js.erb

      $("#<%= dom_id(@comment) %>").replaceWith("<%= j(render "comments/form", comment: @comment) %>");
    ```

3. Here are the CRUDE solutions for the likes icon.

    C - CREATE
    ```
    //app/views/likes/create.js.erb

    var existingLikes = $("#<%= dom_id(@like.photo) %>_likes");

    var updatedLikes = $("<%= j(render "photos/likes", photo: @like.photo) %>");

    existingLikes.replaceWith(updatedLikes)

    var icon = $("#<%=dom_id(@like.photo)%>_likes_icon")

    icon.addClass("fa-beat")

    icon.css("--fa-animation-iteration-count",1)

    console.log("Like created successfully.")
    ```

    D- DESTROY
    ```
    //app/views/likes/destroy.js.erb

    var existingLikes = $("#<%= dom_id(@like.photo) %>_likes");

    var updatedLikes = $("<%= j(render "photos/likes", photo: @like.photo) %>");

    existingLikes.replaceWith(updatedLikes)

    var icon = $("#<%=dom_id(@like.photo)%>_likes_icon")

    console.log("Like destroyed successfully.")
    ```

### VIII. Follow Request

1. The routine steps include applying the same three files :
  - add parameter to the _form.html.erb file.
  - create .js.erb within the views/follow_requests folder.
  - add format.js to the follow_request_controller.rb file within the appropriate functions.

2. First, let's observe what happens when the the follow/unfollow button is toggled without ajax.
  - When the follow button is pressed, it says "created". 
  - When the Un-request button is pressed, it says "destroyed".

3. Therefore, let's start working on the create function. Go to follow_requests_controller.rb and add format.js.
- modify follow_requests_controller.rb
```
//controller/follow_requests_controller.rb

respond_to do |format|
  if @follow_request.save
    format.html { redirect_back fallback_location: root_url, notice: "Follow request was successfully created." }
    format.json { render :show, status: :created, location: @follow_request }

    format.js
        respond_to do |format|
  if @follow_request.save
    format.html { redirect_back fallback_location: root_url, notice: "Follow request was successfully created." }
    format.json { render :show, status: :created, location: @follow_request }

    format.js
```

- Second, modify views/follow_requests/_form.html.erb, not the follow_requests.html.erb. Add the parameter local: false.

```
<%= form_with(model: follow_request, class: "d-inline-block", local: false) do |form| %>
  <% if follow_request.errors.any? %>
```

- Third, make the file views/follow_rquests/create.js.erb.

```
console.log("Howdy from follow.")
```

4. Found the matching button as in the follow_unfollow html page:

```
<!-- views/follow_requests/_follow_unfollow.html.erb. -->

<% if follow_request.pending? %>
  <%= link_to follow_request,  data: { turbo_method: :delete }, class: "btn btn-outline-secondary" do %>
    Un-request
  <% end %>
```

To determine the parameter associated with follow_request, go to the follow_requests_controller.rb. You will see @follow_requests being mentioned. To know the method, go to follow_requests/index.html.erb. In this case, it is .reciepient. To know the dom_id, for likes, I found it in views/photos/_likes.html.erb. go to...

You can find the dom_id by inspecting the html page. and looking at the lines above the button.

Essentially, you are making the changes solely within the partial html file (views/follow_requests/_follow_unfollow.html.erb).

You can trace this partial being called in the views/photos/_photo.html.erb. Note the line:
  ```
  <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: photo.owner %>
  ```
In the above, to render the page, it requires you to specify sender and recipient.

TIPS: look at likes example for directions.

5. To determine the class id, look at the target, pull up the inspect page, and select "Network" > click on the element, and click on the name that pops up. Click on the "Headers" tab to view the general information, and click on the "Preview" tab to know the class name.

6. You also have to assign a div class name to the follow_requests partial to be able to point to the follow/un-request button. Note that, unlike the "like" button, where you point to the element via a specific id, in this case, you point to it as a class. Below shows the difference between assigning a class and an id in css.

```
.class_name {
  background-color: red;
 }

 #id_name {
  background-attachment: green;
 }
``` 

### Appendix A
- To delete unmerged or merged branch, type `git branch -D <branchName>`.
- If you forget to create a branch and you have been making changes on main:
  - create a new branch from the latest changes with `git checkout -b <new-branch-name>
`.
  - Important! Publish the newly created branch. 
  - Move the main to the earliest version (f683840) with:
    
    ```
    git checkout main
    git reset --hard f683840
    ```
  
<hr>
