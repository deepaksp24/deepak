```
    // Sends a message to the dispatcher in the Editor Backend
    fn dispatch<T: Into<Message>>(&self, message: T) {
        // Process no further messages after a crash to avoid spamming the console
        if EDITOR_HAS_CRASHED.load(Ordering::SeqCst) {
            return;
        }


        // Get the editor, dispatch the message, and store the `FrontendMessage` queue response

        let frontend_messages = editor(|editor| editor.handle_message(message.into()));

        // Send each `FrontendMessage` to the JavaScript frontend
        for message in frontend_messages.into_iter() {
            self.send_frontend_message_to_js(message);
        }
    }
```

first we will take 
```
<T: Into<Message>>
```
this tell 'T' is generic  that implements trait Into<message> 
This overall tells that input type should have implement certain trait.

```
if EDITOR_HAS_CRASHED.load(Ordering::SeqCst) {
            return;
        }
```
this check if the app has crashed , if yes it return else continue with excution

```
let frontend_messages = editor(|editor| editor.handle_message(message.into()));
```

editor(|editor| editor.handle_message(message.into())); 
this looks cryptic lets see, that is closure . to know about closur go to this [[closure]].

Ok in the code we going to see is some what confusing because there are editor 3 time. and there is function in body .

ok the first editor is a function  that take another function as input  and its **type is editor**.
so where is there function editor [[Editor function]]

and  second `editor` is a struct type editor
[[Editor struct]]
