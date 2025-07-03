```
#[derive(Debug, Default)]
pub struct Dispatcher {
    buffered_queue: Option<Vec<VecDeque<Message>>>,
    message_queues: Vec<VecDeque<Message>>,
    pub responses: Vec<FrontendMessage>,
    pub message_handlers: DispatcherMessageHandlers,
}


/// Provides access to the `Editor` by calling the given closure with it as an argument.
fn editor<T: Default>(callback: impl FnOnce(&mut editor::application::Editor) -> T) -> T {
    EDITOR.with(|editor| {
        let mut guard = editor.try_lock();
        let Ok(Some(editor)) = guard.as_deref_mut() else { return T::default() };
        callback(editor)
    })
}
```
The `editor` function provides a safe, convenient way to access the global `Editor` by running your code (a closure) with a mutable reference to it — or returns a default if it can’t