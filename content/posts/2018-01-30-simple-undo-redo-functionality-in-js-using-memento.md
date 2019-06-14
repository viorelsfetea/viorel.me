---
layout: post
title: "Simple Undo/Redo system in Javascript based on the Memento pattern"
summary: "There comes a time in every developer's life when they have to implement a undo/redo functionality."
date: 2019-01-30 12:00:18
comments: false
author: Viorel
category: techy
slug: simple-undo-redo-functionality-in-js-using-memento
tags:
- javascript
---
There comes a time in every developer's life when they *have* to implement a undo/redo functionality. Kind of like a coming of age. And just like any coming of age, a myriad of people have gone through the same struggles and have come victorious on the other side. Thus are rituals established and methods perfected to take the young ones safely through.

In the case of the undo/redo functionality, there are two OOP patterns that most people use and consider suitable: the Command pattern and the lesser used Memento pattern. A main difference that *I* find between the two is that you can use the Command pattern for more complex data structures, collections or actions, but it's overkill for remembering the state of one single type of simple object. On the other side, Memento works perfectly for those simple objects, but becomes cumbersome as complexity grows. Memento is simple and there's a good chance you're going to find in the *Others* section of some patterns books.

I'm going to show the pattern below in ES6 syntax. There's not going to be a huge theoretical introduction, but, if you're into that, feel free to check the [Wikipedia page](https://en.wikipedia.org/wiki/Memento_pattern). Also, I'm a big fan of analogies, so we're going to be talking about a Baker (Caretaker) baking a really awesome Cake (Originator). This was a special order for the king's birthday so the cake needs to be absolutely perfect. And you guessed it, we're going to use Memento to give the stressed Baker the possibility of experimenting with undoing and redoing parts of the cake so he can get it just right.

***It's like Memento, but in Javascript***

The main idea with Memento is that you save different states of an object to be able to come back to them in the future. In our specific example, the Baker starts baking the Cake adding ingredients to it. Let's work a little bit on the implementation. See below how our main character starts baking, adds ingredients and saves the progress from time to time, but only when he's not sure he added the correct ingredient. Then he does some undoing/redoing:

{{< highlight js >}}
    class Memento {
      constructor(state) {
        this._state = state;
      }
    
      get state() {
        return this._state;
      }
    }
    
    class Cake {
      constructor() {
        this.ingredients = [];
      }
    
      addIngredient(ingredient) {
        this.ingredients.push(ingredient);
      }
    
      getState() {
        return new Memento(this.ingredients);
      }
    
      restoreState(memento) {
        this.ingredients = memento.state;
      }
    }
    
    class Baker {
      constructor() {
        this.cake = new Cake();
        this.states = [];
        this.currentState = 0;
      }
    
      bake() {
        this.cake.addIngredient('cream');
        this.cake.addIngredient('sugar');
        // Hmm, unsure of ingredients, better save from now on
        this.saveState(this.cake.getState());
        this.cake.addIngredient('eggs');
        this.saveState(this.cake.getState());
        this.cake.addIngredient('milk');
        this.saveState(this.cake.getState());
        // Oops, too much milk
        this.undo();
        // Or was it OK? Maaaan, I'm a terrible baker
        this.redo();
      }
    
      saveState(state) {
        this.states.push(state);
        this.currentState = this.states.length - 1;
      }
    
      undo() {
        if(this.currentState === 0) {
          console.error("That's it, bowl's empty");
          return;
        }
        
        this.currentState -= 1;
        this.cake.restoreState(this.currentState);
      }
    
      redo() {
        if(this.currentState === this.states.length - 1) {
          console.error("Nothing to do more than think of another career");
          return;
        }
    
        this.currentState += 1;
        this.cake.restoreState(this.currentState);
      }
    }
    
    const baker = new Baker();
    baker.bake(); // Like your life depends on it
{{< /highlight >}}

Looks cool, no? It's implemented by the book, but there's a slight slight problem with it: **it doesn't work**. I mean, the code does what it needs to do, but the Baker is extremely surprised to see that nothing happens to his Cake mix when he frantically pushes the Undo button. Uh-oh.

**But why?**

I was saying that the implementation of Memento in Javascript differs a little bit from others due to the internals of the language. There are two things that need to be taken into consideration: 
1. (almost) everything in Javascript is an object 
2. objects (that are not primitives) are always assigned by reference in Javascript. This, of course happens in many other languages, but, combining with the first rule, arrays, functions and, uhm, objects are objects, therefore, they are always assigned by reference.

Now looking back at the code, we see that we added the ingredients to an array. This was done for a simpler manipulation. When the Baker adds an ingredient, it's added to that array. When the Baker wants to save the state, he gets a Memento with the ingredients added up to that point and saves it in an array of his own. Then he adds whatever other ingredients he needs, confident that he can always go back. When he *does* want to go back, he just goes to his array of Mementos and picks the one he wants from there.

Now, back to undoing not working. The Baker extracts the correct Memento and tells the Cake to replace its ingredients with the ones from the Memento. But that doesn't visibly happen. And that happens for the reasons laid above: when creating a Memento object, the Cake never gives it a *copy* of its ingredients list. It gives it the ingredients list. The array with the ingredients is actually an object and the Memento only receives a reference to it. Then the Baker takes the reference and saves it to his Mementos array. *Meaning that, when the list of ingredients in the Cake object is changed, that's what the Baker will have in his states list. Because what he has is a list of references to the same object - the ingredients array*. So, when a Memento is restored, the ingredients list does get replaced, just by it's replaced by itself.

**The solution?** Well, make sure the Memento object saves and returns a copy of the ingredients array not a reference to it... *obviously*. 

It would be great if the Memento class had no idea of the internals of the class using it, so we'd need to create the copy in the Cake class. On the other hand, the Cake class should not need to know that it needs to prepare the data in a certain way - it should only send the ingredients array and expect to receive it back as it was at the moment the Memento was created.

But what if we would save some primitive instead of the array? Primitives aren't assigned by reference. The best primitive for the job would be a String (I promise). I'm hell-bent on the Cake class not having to prepare the data in a certain way, so I will need to somehow modify the Memento, but still in a way in which Memento doesn't become aware of Cake's implementation. 

*Enter serialization.* If I serialize the data in a way in which it doesn't care if Memento received an array, boolean, string or object and then deserialize it everything should be fine. Easiest way is to use the JSON class:

{{< highlight js >}}
    class Memento {
      constructor(state) {
        this._state = JSON.stringify(state);
      }
    
      get state() {
        return JSON.parse(this._state);
      }
    }
{{< /highlight >}}

(the end)

*If you got it this far, please note that the Baker class is not 100% complete. You'd still have to implement some functionality that resets the Redo action, if the Undo action was used and a state was overwritten by adding another ingredient.*
