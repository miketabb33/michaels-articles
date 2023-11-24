Like many, I am a fan of video games! In fact, a major incentive on why I embarked on a career in software was to learn more about game development. Thats why I was delighted to take on the Gilded Rose after my mentor at Pragmint asked me to complete the kata.

The Gilded Rose was created by Terry Huges and may or may not be inspired by one of my beloved games, World Of Warcraft! The premise begins with an Inn named the Gilded Rose. The Gilded Rose sells a few different type of items, each item has a name, sell in, and quality property. The inn keeper, Alison, has a system in place, created by Leeroy, that updates the inventory of each item. Alison has signed a new supplier and will soon be selling a new item type called conjured items. Conjured items behave differently than other items in the system so an update is needed for the system to properly track conjured items.

I don’t know about you, but this challenge is right up my alley. So, I opened the source code to get started.

There were 2 classes. The first class was the Item class with the previously mentioned properties and the second class was the Gilded Rose class. The Gilded Rose class was made up of an item array, a constructor for item array injection, and a single poorly named, 47 lined ‘update quality’ method. This ‘update quality’ method contained a for loop and 16 nested if statements, with 1 if statement going 5 levels deep. Other issues were that the if statements were checking for magic strings, like “Sulfuras, Hand of Ragnaros", to determine item type… or rather to create the illusion of item types. In addition to this mess, there were no tests. Thats when it all made sense, this was yolo code created by Leeroy, LEEROY JENKINS!

Potential solution below: Spoiler Warning!

Right, so I needed to add a conjured item type but the existing structure was very mangled and brittle. I could continue down the train wreck path but adding anything to source code would be a feat in itself. For example, the “Sulfuras, Hand of Ragnaros” was the only legendary item but adding another legendary item will require, what, another magic string check? After some thought, I concluded that I needed to refactor before adding the new conjured item type. But first, I need to write tests.

Adding tests was simple enough. The testing suite featured a coverage tool which ensured that I covered and confirmed all expected behavior. It may not have looked pretty but the ol’ monster worked.

Once the tests were ready, I was able to start refactoring. I wasn’t exactly sure what the refactor was going to look like but my goal was to create a more accommodating system by using types for identification instead of names. I was also very interested in aiming for polymorphism to invoke an ‘update item’ method on each item type, effectively cleaning up a lot of the existing code duplication. Another key point I committed to was to keep the tests green at all times by using incremental refactoring.

My first change was to add 3 different classes, all inheriting from the Item class. These 3 classes were completely empty but together, with the base item class, all pre conjured item behaviors could be defined. I then updated each test to initialize the appropriate item type. Finally, within Gilded Rose’s ‘process end of day and get updated items’ method (formerly known as the ‘update quality’ method), I switch out the magic string checks with type checks. Tests passed!

From this point, I flipped all the negative conditional statements to be positive and partitioned the giant Gilded Rose method into smaller methods (removing duplication). I had 1 big file with many smaller methods and kept tests passing along the way.

I created a new class called Item Mutator, which performed incremental and decremental operations and could be used within all the new item classes. I also realized that, in order to get polymorphism to work, I would need to add another item type for normal items. I then scooped out all of Gilded Roses’s methods except the now named ‘update items’ method and moved them appropriately to the Item classes and Item Mutator class. I also moved the tests from the monolithic Gilded Rose class test file into appropriate Item class test files. Of course, I did these changes slowly while continuously running tests.

The end result was a neatly structured Item type hierarchy and a 7 lined ‘update item’ method, which used to be 47 lines. The 16 nested if statements were reduced to 4 un-nested if statements. Another win was that the codebase is was more readable and descriptive.

After the refactor, adding the conjured item type was very simple. First I added tests for expected behavior. Then, I created a new class that extended the Item class called, you guessed it, Conjured Item. Finally, within the Conjured Item class, I updated the quality incrementer to be 2 instead of 1. This satisfied the new requirements and made the test go from red to green.

If you’d like to give the kata a try, check out Emily Bache’s repository: https://github.com/emilybache/GildedRose-Refactoring-Kata

AND…

Here’s my solution to the Gilded Rose (spoiler warning): https://github.com/miketabb33/gilded-rose

Based on my experiences from my apprenticeship at Pragmint. https://pragmint.com/