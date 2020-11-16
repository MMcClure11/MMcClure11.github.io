---
layout: post
title:      "Sinatra Project - 2nd Project for Flatiron"
date:       2020-11-16 22:02:26 +0000
permalink:  sinatra_project_-_2nd_project_for_flatiron
---


The past two weeks have been spent working on my second project for Flatiron’s Software Engineering Bootcamp. It is an MVC app built using Sinatra. For my project I decided to create a recipe catalog that would allow a user to make recipes to save in their catalog, and provide them the option to view all of their recipes in a list or organised by category. This was mostly selfishly motivated as I have recipes written on notecards, in three different notebooks, and scraps of paper stuck to the refrigerator.

One of the hardest parts was getting the category functionality that I wanted as a user. I set up my database so that a category can have many recipes, and a recipe can have many categories. Which reflects the fact that a recipe can be both vegan and breakfast for example. When a user makes a new recipe I set up the form so that they can select from a series of checkboxes the categories they want to add, as well as made a space for them to submit a new category name.

```
<label for="category">Choose a Category:</label><br>
   <% Category.all.sort {|a, b| a.name <=> b.name}.each do |category| %>
     <input type="checkbox" name="categories[]" id="<%= category.id %>" value="<%= category.id %>"><%= category.name %></input>
   <% end %>

 <p>Or Create a Category:</p>
   <label for="new_category">Category Name:</label>
   <input type="text" name="category[name]">

 <input type="submit" value="Create">
 ```
My post controller would then set the category ids for any categories that were submitted via the checklist, and also check to see if the user typed in a category name. First it checks if the params[:category][:name] is empty, and if it’s not it checks that the name does not already belong to the recipe, to prevent users from making duplicates of a category for a recipe.
```
@recipe.category_ids = params[:categories]

if !params[:category][:name].empty? && !@recipe.categories.include?(Category.find_by(name: sanitize(params[:category][:name]).downcase.capitalize))
     @category = Category.find_or_create_by(name: sanitize(params[:category][:name]).downcase.capitalize)
   end
	 ```
This gave me most of the functionality that I wanted. However when I was testing out my app I discovered that my sanitize method for javascript attacks such as
```
<script>alert('malicious popup alert')</script>
```
would clear out the entry entirely and a new category would be created with no name at all! Which was definitely not what I wanted. This then led me to test out putting in a bunch of whitespaces. This gave the same result, so now a user was able to create virtually an infinite number of categories that were nameless.

My initial fix was to change my sanitize method, and instead of using the Sanitize Gem, simply gsub out the carrots and white spaces. Which worked, but I wanted to be able to use the gem for all of my params. After struggling for a day trying different variations of conditionals, I reached out to my cousin, a Ruby software engineer for some help.

We added a validation to the Category class to validate the presence of name. I hadn’t done this initially, thinking that this would mean that the field would always be required. Before validating the name, we added a method #trim_whitespace to remove any spaces before or after the name entered, or all whitespaces if that was the only input.

I learned that in ruby you can use &. which is a Safe Navigation Operator, meaning that the operator returns null if the first argument is null, otherwise it will perform the operation. This avoids the undefined method for nil:NilClass error that would otherwise crop up. Which is what we used to strip the name of the category of whitespace.
```
class Category < ActiveRecord::Base

 has_many :recipe_categories
 has_many :recipes, through: :recipe_categories

 before_validation :trim_whitespace
 validates :name, presence: true

 def trim_whitespace
   name&.strip
 end

end
```

Then we added a nested if statement to check to see if the created category persisted, saving it to the recipe’s categories if successful, and rendering the form again if it was not persisted.
```
if !params[:category][:name].empty? && !@recipe.categories.include?(Category.find_by(name: sanitize(params[:category][:name]).downcase.capitalize))
     @category = Category.find_or_create_by(name: sanitize(params[:category][:name]).downcase.capitalize)
     if @category.persisted?
       @recipe.categories << @category
     else
       return(erb :'/recipes/new')
     end
   end
if @recipe.save
     redirect "/recipes/#{@recipe.id}"
 else
     erb :'/recipes/new'
 end
 ```
By adding the error messages given to us by the validation for Category name, when the form is rerendered the error with the full message will be displayed to the user, telling them that the “Category name cannot be blank”.
```
<% if @category && @category.errors.any? %>
<ul>
 <% @category.errors.full_messages.each do |error|%>
   <li>Category <%= error.downcase %></li>
 <% end %>
</ul>
<% end %>
```
After all this, from the user perspective, my program was functioning as I wanted. If a user entered in a script or a bunch of spaces, the recipe's new form was rerendered with the error message. And if the user filled out all the fields properly but left the Create a Category form blank, it successfully saved the recipe and redirected the user to the recipe’s show page.

However, I was still confused as to why it was working. My primary stumping point was that, if the Category model has a validation that requires that the category have a name, why was the recipe able to successfully save when there is no input at all?

It turns out that it depends on how you set up the post request in your controller and understanding it requires some knowledge on how blank form fields get submitted. The important piece of code is this:
```
if !params[:category][:name].empty?
```
If we inspect the params that come in to the controller when the form is submitted with the category field unfilled, they look like this:
```
Params[:category][:name] 
#=> “”
```
Which is an empty string.
“”.empty? is true, and so the statement evaluates to !true, ie. false. Because the first part of the condition fails, ruby short-circuits the rest of the code inside the if statement. And the next line of code that is run is the if @recipe.save. So the validation on Category name is never run at all! It only gets run if the field is not empty, so when a user enters a valid name, a bunch of white spaces, or a script of some kind. And because there is a method that removes whitespace before the validation, malicious input would fail the validation, giving us the error messages.

It took some playing around with the params and seeing exactly what inputs will return true and false for the #empty? method, but once it clicked, I could see the elegance in how this code was working.

There were lots of bumps along the way to completing this project, and I learned a lot. I also found that making a web app is a lot like making art. I keep thinking of things I can add, changes to make, features to create, edits and little bugs to fix, but at some point you stop working on it, because there are others to be made. It’s not truly finished, just abandoned, for the moment. I’ll probably come back to it, when feeling inspired or wanting more practice, just like I do for my paintings and drawings.

Shoutout to my cousin Kori for reviewing this blog post, and helping me understand why the validation was never called.
Kori is a web developer at JOGL.io. Follow him on twitter @koriroys .
