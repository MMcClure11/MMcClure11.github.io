---
layout: post
title:      "Fourth Project: JS and Rails API"
date:       2020-11-17 02:39:35 +0000
permalink:  fourth_project_js_and_rails_api
---


Just today I wrapped up my fourth project for flatiron using Ruby on Rails as an API for the backend and with a vanilla JavaScript frontend. Witchy Wardrobe is a closet organising SPA that allows you to create items of clothing and outfits. It is intended to help people realise how much they spend on clothes, how often do they use clothing items, and create outfits. It would be a great supplemental tool for someone trying out Project 333, the 4x4 capsule, or any other minimalist closet.

There were many challenges with this project that ranged from the very basic of organising my code so that is is both readable and reusable, to the more intense creating a functioning patch for editing items and outfits. But for me the biggest challenge by far was within that already difficult patch, creating and preselecting checkboxes of items that already belonged to that outfit and passing that data into the backend. This process hit on many key elements to understanding JavaScript including what Flatiron calls the Three Pillars: Recognise Events, Manipulate the Dom, and communicating with the server.

The first step for editing an outfit was adding an edit button in the Outfit class that would sit in the card for an outfit. I then passed that button along with the form and the attributes of the outfit to an outfitEditHandler.

witchy-wardrobe-frontend/src/Outfit.js

Inside creating a card for an outfit:
```
  const editOutfitForm = document.createElement('form')
    OutfitForm.outfitEditHandler(editBtn, editOutfitForm, name, likes, this.outfit.items)
		```

I made another class to handle the creation of the forms for making new and editing outfits. Here an event listener was added to the editBtn that would display the block and invoke another function to render the content of the form.

src/OutfitForm.js
```
static outfitEditHandler(editBtn, editOutfitForm, name, likes, items){
    editBtn.addEventListener("click", () => {
      modal.style.display = "block"
      modalContent.append(editOutfitForm)
      OutfitForm.renderFormContent(editOutfitForm, name, likes, items)
    })
  } . . .}


static renderFormContent(editOutfitForm, name, likes, selectedItems, outfitForm){

...

 const itemsCheckContainer = document.createElement('div')
    const itemsCheck = document.createElement('div')
    itemsCheck.className = "form-check-container"
    const checkboxLabel = document.createElement('label')
    checkboxLabel.innerText = "Pick your clothes for your Outfit:"

    ApiService.getAllItems(selectedItems)
      .then(items => {
        items.forEach(item => {
          let inputLabelDiv = document.createElement('div')
          inputLabelDiv.className = 'form-check'
          let checkbox = document.createElement('input')
          checkbox.className = "checks form-check-input"
          checkbox.type = "checkbox"
          checkbox.id = item.id
          checkbox.name = item.name
          let checkLabel = document.createElement('label')
          checkLabel.className = 'form-check-label'
          checkLabel.innerText = item.name
          if(selectedItems){
            selectedItems.forEach( item => {
              if(item.name === checkbox.name){
                checkbox.checked = true
              }
            })
          }
          inputLabelDiv.append(checkbox, checkLabel)
          itemsCheck.appendChild(inputLabelDiv)
        })
      })

      itemsCheckContainer.append(checkboxLabel, itemsCheck)

    const submitBtn = document.createElement('button')
    submitBtn.className = 'btn'
    submitBtn.innerText = "Submit"

    if(editOutfitForm){
      editOutfitForm.append(outfitNameDiv, outfitLikesDiv, itemsCheckContainer, submitBtn)
    } else if (outfitForm) {
      outfitForm.append(outfitNameDiv, outfitLikesDiv, itemsCheckContainer, submitBtn)
    }
  }
	```
Inside the render form content method I made a div to hold all the items and their checkboxes and labels. In order to make it dynamic I then sent a request to my adapter class called ApiService.js to get all the items in the database. This hits the backend with a fetch request to get all the items. (As an aside, this was a valuable lesson for me in the asynchrony of fetch, I initially made a fetch request to the ApiService to get all the items and pushed each item into an array to try and access it outside the request. Turns out it was always empty because the rest of the function was being run first. This was solved by creating and appending all the elements for checkboxes inside the method that invokes the fetch request.)I then iterated over each item with a forEach to create the labels and checkboxes for each item. And within that iteration, in order to preselect the values, I had a second iteration:
```
if(selectedItems){
            selectedItems.forEach( item => {
              if(item.name === checkbox.name){
                checkbox.checked = true
              }
            })
						```
The selected items were passed from the card as this.outfit.items which granted be access to the item ids and names. So I could evaluate if an item.name matched the checkbox.name that was created and mark it checked if true.
Then it was a matter of appending the Dom elements to the form, to do this I also had to check if an editForm or outfitForm was passed in since I reused the form content for both creating and editing an outfit.

Once everything was created and appended to the modal and the form, in Outfit.js I added an event listener to the form on submit and prevented the default action so that it would not refresh the page with a GET request. The trickiest part here was getting access to all the items that were checked. This took several hours of debugging and help from my cohort lead. Each item checkbox was given a class name of "checks" so I was able to make a new const checks that was an array of all the checked items. I then made a new array of checkedItems by filtering through the checks array based on if they were checked. After that I mapped the checkedItems to a new array of the items ids so I could pass the data to server.

witchy-wardrobe-frontend/src/Outfit.js
in CardContent function
```
    editOutfitForm.addEventListener("submit", (e) => {
      e.preventDefault();
      const checks = Array.from(e.target.querySelectorAll(".checks"))
      const checkedItems = checks.filter( item => item.checked )
      let itemIdsArray = checkedItems.map( item => parseInt(item.id))
      const editedOutfit = {
        name: e.target.name.value,
        likes: e.target.likes.value,
        item_ids: itemIdsArray
      }
      this.updateOutfitHandler(editedOutfit, card)
			```
I then passed the object with the values of the edited outfit to my updateOutfitHandler which took in the outfit id and outfit object and passed it to the updateOutfit function in my ApiService class.
witchy-wardrobe-frontend/src/Outfit.js
```
  updateOutfitHandler(editedOutfit, card){
    ApiService.updateOutfit(this.outfit.id, editedOutfit)
    .then(. . . )}
witchy-wardrobe-frontend/src/ApiService.js
  static updateOutfit(outfitId, outfit){
    return fetch(`${OUTFITS_URL}/${outfitId}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(outfit)
    })
    .then(res => res.json())
  }
	```
The fetch request sent the data to the update action as a string which is necessary because JSON is a text based data format and data sent via HTTP requests requires a text based communications protocol.

witchy-wardrobe-backend/app/controllers/outfits_controller.rb
```
  def update
      outfit = Outfit.find_by(id: params[:id])
      if outfit.update(outfit_params)
        render json: OutfitSerializer.new(outfit).to_serialized_json
      else
        render json: {errors: outfit.errors.full_messages.to_sentence}, status: :unprocessable_entity
      end
  end

private

  def outfit_params
    params.permit(:name, :likes, :item_ids => [])
  end 
	```
Here, it took digging deep into the recesses of my rails mind to remember that the outfit_params needed item_ids to point to an empty array to accept multiple items.

witchy-wardrobe-frontend/src/Outfit.js
```
  updateOutfitHandler(editedOutfit, card){
    ApiService.updateOutfit(this.outfit.id, editedOutfit)
    .then(updatedOutfit => {
      if (updatedOutfit.errors){
        alert(updatedOutfit.errors)
      } else {
        this.outfit = updatedOutfit
        card.innerHTML = ""
        this.cardContent(card)
        modal.style.display = "none"
        modal.querySelector("form").remove()
      }
    })
    .catch(error => alert(error))
}
```
Once it was all successfully updated in the server, I had to handle manipulating the Dom to reflect that change without doing a refresh to the page. First I checked for validation errors and alerted the user if there was a failure for some reason. If it was successful it set this.outfit to the updated data, which changes the data for the particular outfit in the constructor. Then I needed to clear the innerHTML of the card so that it wouldn't be rendered twice. Then I invoked the method that creates the card content of the particular card that was edited. It also closes the modal and removes the form from the modal.

Throughout this whole process, I learned a lot about code organisation. It is so crucial even when building a project in order to successfully troubleshoot, especially when passing around a lot of data and invoking lots of functions.

I have many more features I want to eventually build into this app including an outfit randomiser and creating a system for rating ones wardrobe based on its environmental and social impact as well as users with authentication. But that's for another day!


