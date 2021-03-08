js-event-more is a js event library that is similar to the once famous YUI3 event framework, it is written to fulfill ex event pub-sub requirements with the following exclusive features:

1. Behave like dom events with default event handler and event bubbling
2. Event firing pipeline with <strong>before</strong>, <strong>on</strong>, <strong>after</strong> phases, provides strong controls over event  ordering. Event subscribers are called in the order: <strong>before -> on -> default -> after -> bubble up -> before -> on -> default -> after -> bubble and so on</strong>
3.  Subscription priority, event subscripitions in the same phase can be ordered by priority



Suppose you are writing a page with a complex form, you are separating your domain model and your view.  

The domain model is responsible for things like validation logics, submit, loading initial form data, and data processing. Upon submitting, the form is validated, if validation fails, it does not submit and the view layer should display error messages. 



~~~
import EventTarget from 'js-event-more'

// initlaize form model
class FormModel {

   constructor(){
   	let eventTarget = new EventTarget();
   	
   	// specify the _doSubmit() method to be the default handler for submit event
   	eventTarget.publish('submit',{
       defaultFn : model._doSubmit,
       context: model     
	});
	
    // validate the form before submit
    eventTarget.before('submit', model.validate, model);
	this.eventTarget = eventTarget;
   }
   
	submit(){
		this.eventTarget.fire('submit');
	}
	
	validate(e){
		  let valid = true, error = ''
       	  //do your validation here

          if(!valid){
          // if validation fails, prevent the default handler to be called.
          // in this example, default handler is _doSubmit()
          	e.preventDefault();
          	
          	// fire the invalidated event, the view layer should 
          	// catch the event and display error messages
          	this.eventTarget.fire('invalidated', {
          	  error : error
          	});
          }
			
	}
	
	_doSubmit(){
		// do the actual submit logic here
	
	}
	
	destroy(){
	   // detach all listeners
	   this.eventTarget.off();
	}
}

~~~



Here is an example for event bubbling. Suppose you are  maintaining a list of books, you have a BookList.js and Book.js, where BookList contains a list of Books. Book.js has a delete() method which sends an ajax request to delete the book from server, then BookList should delete the book from the list. 

One way to do this is to fire a deleted event from Book, BookList subscribes to the event for every Book. 

With event bubbling, BookList only needs to subscribe to deleted event once. That's pretty much like event bubbling in DOM event.

```
class Book {

     constrcutor(model){
        this.model = model;
        this.eventTarget = new EventTarget();
     }
	
     delete(){
        // delete from server
       let promise = new Promise().then(()=>{
       		this.eventTarget.fire('deleted')
       });
        
     }

}  

class BookList {
      
      constructor(){
        this.books = [];
        this.eventTarget = new EventTarget();
        // remove book on deleted event
        this.eventTarget.on('deleted', this.removeBook,this);
      
      }
      addBook(model){
       let book = new Book(model);
       this.books.push(book);
       // tell book to bubble events to book list
       book.addTarget(this);
       
      }
      
      removeBook(e){
      		let book = e.target.model;
      		let index = this.books.findIndex(b=>b.model.id === book.model.id)
      		if(index !== -1){
      			this.books.splice(index,1);
      		}
      
      }	
}
```



EventTarget is the only class you may interact with, it has only a few methods:



<strong>publish(eventName, config)</strong> - configure the event, it is not required to call publish to fire and subscribe to events

config.defautFn - default event handler

config.context - default context for event handlers, context means where "this" points to in event handlers



<strong>on(eventName, callback, context, priority, once)</strong> -  subscribe to the event, called in the "on" phase

<strong>once(eventName, callback, context, priority)</strong> -  subscribe to the event, only called once

<strong>before(eventName, callback, context, priority,once)</strong> -  subscribe to the event, called in the "before" phase

<strong>after(eventName, callback, context, priority,)</strong> -  subscribe to the event, called in the "after" phase

<strong>fire(name,data)</strong> - fire event with data

<strong>addTarget(target) </strong> - add a bubble target, must be an instance of EventTarget

<strong>removeTarget(target)</strong> - remove a bubble target

<strong>off(eventName,callback)</strong> - remove a subscriber, if no callback specified, remove all subscribers for the event; if no eventName specified, all subscribers for all events are removed



