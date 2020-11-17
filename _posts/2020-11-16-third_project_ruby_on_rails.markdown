---
layout: post
title:      "Third Project: Ruby on Rails"
date:       2020-11-17 02:36:31 +0000
permalink:  third_project_ruby_on_rails
---


Just today I wrapped up my third project for flatiron using Ruby on Rails. I enjoy reading and so decided to create the “Book Nook”, a website that allows users to add books to the Book Nook’s collection, create reading lists and add books to their lists, and also write reviews of books. The website also keeps track of the top five reviewers and the top ten highest ranked books based on their average rating. It has typical sign up, login, and logout functionality plus the additional option to sign up/login using a Github account through Ominauth.

There were many challenges with this project that ranged from setting up Omniauth to incorporating the GoogleBooks API so that users can search for a book and have it automatically created and added into the Book Nook database without having to manually enter in all the information. But for me the biggest challenge by far was adding a book to a user’s list. When I first started, I thought this would be a simple task, and once I found the solution of course it seemed simple, but it took several days of mulling the problem over before I got the user functionality I wanted.

app/views/books/show.html.erb
```
<%= form_with model: @book, local: true do |f| %>
        <%= f.collection_collection_select :list_id, current_user.lists, :id, :name, :include_hidden => false  %>
     <%= f.submit "Add Book to List"%>
 <% end %>
app/controllers/books_controller.rb
def update
     if @book.can_edit_and_delete?
       @book.update(book_params)
       if @book.save
         flash[:success] = "The book was successfully updated."
         redirect_to @book
       else
         render :edit
       end
     else
       flash[:notice] = "The book #{@book.title} cannot be edited."
       redirect_to @book
     end
   end
 end

private

def book_params
   params.require(:book).permit(:title, :author, :year_published, :page_count, :description, :list_id)
 end
 ```
Originally I added a form_with to the books show page that would allow a user to select one of their lists and add it to that list. By passing in the :list_id to the book_params in the BooksController whenever a book was updated, it would update the :list_id associated with the book to the list_id passed in by the user. At first glance I thought it was doing exactly as I wanted, I navigated between different lists that belonged to the user I was logged in as and I could add the same book to different lists with no issues. Thinking I was done with that, I moved on to the next piece of my project.

It wasn’t until a couple of days later that a classmate of mine who was reviewing my project realised that when I added a book to the list of one user, if a different user also had that same book on one of their lists, the application would remove the book from the other user’s list(s). Well that was a bit of a shock and certainly a terrible user experience, so I set about fixing the issue. While troubleshooting I started to get a grasp of why it was functioning that way, under the hood rails was resetting the list_id and in so doing clearing out the ids of lists that belonged to other users.

My initial response was to take out the list_id from the book params and push the found id into that book’s lists:
app/controllers/books_controller.rb
```
def update
 list = List.find(params[:book][:list_id])
 @book.update(book_params)
 if @book.save
   @book.lists << list
   flash[:success] = "Your book was successfully updated."
   redirect_to @book
 else
   render :edit
 end
end
```
This solved the problem of removing books from other users’ lists, because instead of resetting the lists that are associated with that book, it was adding a new list_id. Great! Problem solved! But, then as I continued testing it out in the browser I realised that now users could add the same book to a list multiple times and that was also not how I wanted the user to experience the app.

I spent quite a bit of time trying to set up a conditional in the BooksController update action so that it would only push in a list_id if it didn’t already exist for that book. I couldn’t get it to work quite right, and after a day of frustration I walked away from the issue. A few days later, as I was doing some menial task it dawned on me, how about I go at it from the other direction and only allow the user to select from lists that don’t already contain that book? Once I had the idea, it took about 15 minutes to implement.

The first iteration had a lot of logic in the view and in the controller, but once I got it working I abstracted some of that away into the User model to check if a user could add a book to a list:

app/models/user.rb
```
def can_add_book_to_list?(book)
   lists = []
   self.lists.each do |list|
     lists << list if !list.books.include?(book)
   end
   lists
 end
 ```
Then I used that method in the collection_select to indicate which lists should appear in the dropdown on the book show page. Instead of current_user.lists, it only shows lists that don’t already have that book on it. I also added a conditional to check if the user has any lists that they can add that book too, because if they didn’t, I did not want the submit button and an empty dropdown to appear on the page.

app/views/books/show.html.erb
```
<%= form_with model: @book, local: true do |f| %>
   <% if !current_user.can_add_book_to_list?(@book).empty? %>
     <%= f.collection_select :list_id, current_user.can_add_book_to_list?(@book), :id, :name, :include_hidden => false  %>
     <%= f.submit "Add Book to List"%>
   <% end %>
 <% end %>
 ```
Then in the BooksController I check to see if the params of params[:book][:list_id] are present and if they are, use the add_book_to_list action which I put in the ApplicationController to help thin up the update action for better readability.

app/controllers/application_controller.rb
```
def add_book_to_list(list_params, book)
   list = List.find(list_params)
   authorize(list)
   book.lists << list
 end
app/controllers/books_controller.rb
def update
   if List.find_by_id(params[:book][:list_id]).present?
     add_book_to_list(params[:book][:list_id], @book)
     flash[:success] = "Book was successfully added to your list."
     redirect_to @book
   else
     if @book.can_edit_and_delete?
       @book.update(book_params)
       if @book.save
         flash[:success] = "The book was successfully updated."
         redirect_to @book
       else
         render :edit
       end
     else
       flash[:notice] = "The book #{@book.title} cannot be edited."
       redirect_to @book
     end
   end
 end

private

def book_params
   params.require(:book).permit(:title, :author, :year_published, :page_count, :description)
 end
 ```
It took quite a bit of mental power on my part to get the lists working as I wanted them to. I imagine there are other ways to do so, but I was proud of myself for working through to a conclusion on this one. I’ve been told by multiple software engineers that some of their best solutions come to them when they aren’t even at their computer, and in this case that was certainly true for me.

While I learned a lot building out this project, one of my biggest takeaways was walking away from a problem, especially if you’re frustrated. It wasn’t until I was in a calmer state of mind that I made substantial progress. While I had read about this approach to coding and been told about too, it wasn’t until I experienced it myself that I really became a believer in the power of a mind at ease.
