```
let frontend_messages = editor(|editor|editor.handle_message(message.into()));
```

A **closure expression** defines an anonymous function and captures the enclosing environment.
so its a anonymous function that handles environment variable. 
```
|args| expression
|args| { block }
```
Closure = (function body) + (captured environment).

