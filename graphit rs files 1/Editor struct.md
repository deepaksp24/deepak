
```
pub struct Editor {
    pub dispatcher: Dispatcher,
}
impl Editor {
    /// Construct the editor.
    /// Remember to provide a random seed with `editor::set_uuid_seed(seed)` before any editors can be used.
    pub fn new() -> Self {
        Self { dispatcher: Dispatcher::new() }
    }
    #[cfg(test)]
    pub(crate) fn new_local_executor() -> (Self, crate::node_graph_executor::NodeRuntime) {
        let (runtime, executor) = crate::node_graph_executor::NodeGraphExecutor::new_with_local_runtime();
        let dispatcher = Dispatcher::with_executor(executor);
        (Self { dispatcher }, runtime)
    }

    pub fn handle_message<T: Into<Message>>(&mut self, message: T) -> Vec<FrontendMessage> {
        self.dispatcher.handle_message(message, true);
        std::mem::take(&mut self.dispatcher.responses)
    }

}
```

here you can see Editor  it’s a **struct type** and it has field named `dispatcher`  and it has function called handle_message . and here we go again it calling another function looks like function chaining  [[handle message form dispatcher struct]]. this handles all the message 
