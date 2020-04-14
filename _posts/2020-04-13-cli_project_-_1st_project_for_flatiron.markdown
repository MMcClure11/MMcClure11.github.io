---
layout: post
title:      "CLI Project - 1st Project for Flatiron"
date:       2020-04-14 00:46:16 +0000
permalink:  cli_project_-_1st_project_for_flatiron
---


Creating an entire project from scratch was one of the most stressful and intellectually challenging activities I have undergone in a very long time. Of course when I say from scratch, that is with the knowledge that there is a wealth of code that goes into my program before I even open a file tree. Prior to this all of my labs for the Flatiron curriculum had tests written so we could learn how to write code relating to the topic at hand, the file tree was set up with what needed to be required, and everything we could possibly need from Pry to Httparty was already installed for us. By doing all of this for myself, I developed a much deeper understanding for how gems, applications, and programs in Ruby work and how interdependent all code is on pre-existing code. 

For my project I built a CLI program that uses the DND 5th Edition API to get data about the different races to help a user quickly see information in order to help with character creation. Of course DND Beyond has made a much more in depth version of this that includes selecting classes and other attributes, but that requires a user to have an account and has a lot of images to download which doesn’t always fair-well with poor wifi reception. I wanted something more accessible on my computer for when my wifi connection is poor to allow me to quickly flip through the information for my character design.

--------------------------------------------------------------------------

One major challenges was deciding how to transform the data from the API into a data structure that I could more easily work with. The data came in as an array of hashes with both the keys and values as strings.    
```ruby
{"count"=>3,
"results"=>
 [{"index"=>"dragonborn", "name"=>"Dragonborn", "url"=>"/api/races/dragonborn"},
  {"index"=>"dwarf", "name"=>"Dwarf", "url"=>"/api/races/dwarf"},
  {"index"=>"elf", "name"=>"Elf", "url"=>"/api/races/elf"}]}
```
When initialising new objects as instances of race, I didn't want order to matter, and be able to pass in named symbols. To do this I made a series of methods to get the data and transform it.

In my API class I had my base url and my method to get_races from the API
```ruby
BASE_URL = "https://www.dnd5eapi.co"

def self.get_races
      res = HTTParty.get(BASE_URL + "/api/races")
      DndRaces::Race.create_from_api(res["results"])
end
```
Which used methods from my class of Race to transform my data: 
```ruby
def self.create_from_api(array_of_hashes)
       array_of_hashes.each do |racehash|
           self.create(self.format_hash(racehash))
       end
   end
 
   def self.format_hash(hsh)
       hsh.each_with_object({}) do |(k,v), mem|
           mem[k.to_sym] = v
       end
   end
 
   def self.create(index: nil, name:, url:)
       race = new(name: name, url: url)
       race.save
       race
   end
```
The “.create_from_api” method takes in the array of hashes from the API and iterates over each hash in that array. For each of those hashes they are formatted in the “.format_hash” method. This method uses “.each_with_object({})” which iterates over a collection (in this case, our racehash from the API), and passes the current element and the “mem” into the block which starts as an empty hash. So what gets stored and returned is the new hash with the key/value pairs and all of the keys are transformed from strings to  symbols. 

This formatted hash is then passed into “.create” which instantiates the race as an instance of an object and those objects are what are called on in the CLI portion of my project. The API provides an extra index which is the name of the race, which is the same as the “name” => “Elf” pair but not capitalized. This was extra information that I wasn’t going to use in my CLI but I had to pass that into my “.create” method since the formatted hash included this key/value pair. By setting the default value to nil, I did not have to instantiate a new object with the index information. 

Going through this process of converting data into symbols and seeing the result of that in my fully functioning CLI was an incredibly rewarding part of this project. It also really helped me understand the importance and power of knowing what kind of data structure you have and how you can manipulate that data.

--------------------------------------------------------------------------

There have been many rough spots and challenges along the way to building this project. There is the ever present itch, knowing that even though it is ready for submission, there is more I could refactor, improvements that could be made, flow that could be improved. 
Sometimes I look at the end product and marvel at how much time, effort and resources were put into this very simple CLI. This has been a very humbling experience. Not only have I gained an appreciation for how much I have learned in six weeks (before beginning Flatiron, I didn’t know what API or CLI stood for, much less what they were), but also a deeper realisation of how much more I have to learn.

