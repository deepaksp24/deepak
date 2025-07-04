```
imple Dispacter{
pub fn handle_message<T: Into<Message>>(&mut self, message: T, process_after_all_current: bool) {
        let message = message.into();
        // Add all additional messages to the buffer if it exists (except from the end buffer message)

        if !matches!(message, Message::EndBuffer(_)) {
            if let Some(buffered_queue) = &mut self.buffered_queue {
                Self::schedule_execution(buffered_queue, true, [message]);
                return;
            }
        }
      // If we are not maintaining the buffer, simply add to the current queue
        Self::schedule_execution(&mut self.message_queues, process_after_all_current, [message]);
        while let Some(message) = self.message_queues.last_mut().and_then(VecDeque::pop_front) {
            // Skip processing of this message if it will be processed later (at the end of the shallowest level queue)

            if SIDE_EFFECT_FREE_MESSAGES.contains(&message.to_discriminant()) {

                let already_in_queue = self.message_queues.first().filter(|queue| queue.contains(&message)).is_some();

                if already_in_queue {

                    self.log_deferred_message(&message, &self.message_queues, self.message_handlers.debug_message_handler.message_logging_verbosity);

                    self.cleanup_queues(false);

                    continue;

                } else if self.message_queues.len() > 1 {
                    self.log_deferred_message(&message, &self.message_queues, self.message_handlers.debug_message_handler.message_logging_verbosity);
                    self.cleanup_queues(true);
                    self.message_queues[0].add(message);
                    continue;
                }
            }

            // Print the message at a verbosity level of `info`
            self.log_message(&message, &self.message_queues, self.message_handlers.debug_message_handler.message_logging_verbosity);

            // Create a new queue for the child messages
            let mut queue = VecDeque::new();

            // Process the action by forwarding it to the relevant message handler, or saving the FrontendMessage to be sent to the frontend

            match message {
                Message::StartBuffer => {
                    self.buffered_queue = Some(std::mem::take(&mut self.message_queues));

                }
                Message::EndBuffer(render_metadata) => {
                    // Assign the message queue to the currently buffered queue

                    if let Some(buffered_queue) = self.buffered_queue.take() {

                        self.cleanup_queues(false);

                        assert!(self.message_queues.is_empty(), "message queues are always empty when ending a buffer");

                        self.message_queues = buffered_queue;
                    };

                    let graphene_std::renderer::RenderMetadata {
                        upstream_footprints: footprints,
                        local_transforms,
                        click_targets,
                        clip_targets,
                    } = render_metadata;

  

                    // Run these update state messages immediately
                    let messages = [
                        DocumentMessage::UpdateUpstreamTransforms {
                            upstream_footprints: footprints,
                            local_transforms,
                        },
                        DocumentMessage::UpdateClickTargets { click_targets },
                        DocumentMessage::UpdateClipTargets { clip_targets },
                    ];

                    Self::schedule_execution(&mut self.message_queues, false, messages.map(Message::from));

                }

                Message::NoOp => {}
                Message::Init => {
                    // Load persistent data from the browser database
                    queue.add(FrontendMessage::TriggerLoadFirstAutoSaveDocument);
                    queue.add(FrontendMessage::TriggerLoadPreferences);
                    // Display the menu bar at the top of the window
                    queue.add(MenuBarMessage::SendLayout);
                    // Send the information for tooltips and categories for each node/input.
                    queue.add(FrontendMessage::SendUIMetadata {
                        node_descriptions: document_node_definitions::collect_node_descriptions(),
                        node_types: document_node_definitions::collect_node_types(),
                    });

  

                    // Finish loading persistent data from the browser database
                    queue.add(FrontendMessage::TriggerLoadRestAutoSaveDocuments);
                }
                Message::Animation(message) => {
              self.message_handlers.animation_message_handler.process_message(message, &mut queue, ());
                }
                Message::Batched(messages) => {
                    messages.iter().for_each(|message| self.handle_message(message.to_owned(), false));
                }
                Message::Broadcast(message) => self.message_handlers.broadcast_message_handler.process_message(message, &mut queue, ()),
                Message::Debug(message) => {                    self.message_handlers.debug_message_handler.process_message(message, &mut queue, ());
                }
                Message::Dialog(message) => {
                    let data = DialogMessageData {
                        portfolio: &self.message_handlers.portfolio_message_handler,
                        preferences: &self.message_handlers.preferences_message_handler,
                    };

                    self.message_handlers.dialog_message_handler.process_message(message, &mut queue, data);

                }
                Message::Frontend(message) => {
                    // Handle these messages immediately by returning early
                    if let FrontendMessage::TriggerFontLoad { .. } = message {
                        self.responses.push(message);
                        self.cleanup_queues(false);
                          // Return early to avoid running the code after the match block
                        return;
                    } else {

                        // `FrontendMessage`s are saved and will be sent to the frontend after the message queue is done being processed

                        self.responses.push(message);

                    }

                }

                Message::Globals(message) => {
            self.message_handlers.globals_message_handler.process_message(message, &mut queue, ());
                }

                Message::InputPreprocessor(message) => {

                    let keyboard_platform = GLOBAL_PLATFORM.get().copied().unwrap_or_default().as_keyboard_platform_layout();

  

                    self.message_handlers

                        .input_preprocessor_message_handler

                        .process_message(message, &mut queue, InputPreprocessorMessageData { keyboard_platform });

                }

                Message::KeyMapping(message) => {

                    let input = &self.message_handlers.input_preprocessor_message_handler;

                    let actions = self.collect_actions();

  

                    self.message_handlers

                        .key_mapping_message_handler

                        .process_message(message, &mut queue, KeyMappingMessageData { input, actions });

                }

                Message::Layout(message) => {

                    let action_input_mapping = &|action_to_find: &MessageDiscriminant| self.message_handlers.key_mapping_message_handler.action_input_mapping(action_to_find);

  

                    self.message_handlers.layout_message_handler.process_message(message, &mut queue, action_input_mapping);

                }

                Message::Portfolio(message) => {

                    let ipp = &self.message_handlers.input_preprocessor_message_handler;

                    let preferences = &self.message_handlers.preferences_message_handler;

                    let current_tool = &self.message_handlers.tool_message_handler.tool_state.tool_data.active_tool_type;

                    let message_logging_verbosity = self.message_handlers.debug_message_handler.message_logging_verbosity;

                    let reset_node_definitions_on_open = self.message_handlers.portfolio_message_handler.reset_node_definitions_on_open;

                    let timing_information = self.message_handlers.animation_message_handler.timing_information();

                    let animation = &self.message_handlers.animation_message_handler;
                    self.message_handlers.portfolio_message_handler.process_message(
                        message,
                        &mut queue,
                        PortfolioMessageData {
                            ipp,
                            preferences,
                            current_tool,
                            message_logging_verbosity,
                            reset_node_definitions_on_open,
                            timing_information,
                            animation,
                        },
                    );
                }

                Message::Preferences(message) => {                    self.message_handlers.preferences_message_handler.process_message(message, &mut queue, ());
                }

                Message::Tool(message) => {
                    let document_id = self.message_handlers.portfolio_message_handler.active_document_id().unwrap();
                    let Some(document) = self.message_handlers.portfolio_message_handler.documents.get_mut(&document_id) else {

                     warn!("Called ToolMessage without an active document.\nGot {message:?}");

                        return;

                    };
                    let data = ToolMessageData {
                        document_id,
                        document,
                        input: &self.message_handlers.input_preprocessor_message_handler,
                        persistent_data: &self.message_handlers.portfolio_message_handler.persistent_data,

                        node_graph: &self.message_handlers.portfolio_message_handler.executor,

                        preferences: &self.message_handlers.preferences_message_handler,

                    };                    self.message_handlers.tool_message_handler.process_message(message, &mut queue, data);
                }
                Message::Workspace(message) => {
self.message_handlers.workspace_message_handler.process_message(message, &mut queue, ());
                }
            }

            // If there are child messages, append the queue to the list of queues
            if !queue.is_empty() {
                self.message_queues.push(queue);
            }
            self.cleanup_queues(false);
        }
    }
}
```

### 🔁 **Flow of Execution**

- ✅ Check if message should be **buffered** → If yes, add to buffer and return.
- 📥 If not, **schedule it for execution** by adding to the queue.
- 🔁 While queue has messages:
    - 🧼 De-duplicate trivial messages.
    - 🔍 Log and process each message.
    - ➕ Child messages go into a temporary `queue`.
    - ➕ If queue has child messages, push them onto the queue stack.
    - 🧹 Cleanup after each iteration.
### 🧩 **Core Responsibilities**

1. **Convert input to a `Message`** using `.into()`.
    
2. **Buffer messages** if buffering is active and message is not `EndBuffer`.
    
3. **Directly queue message** if buffering is not active or it's an `EndBuffer`.
    
4. **Process messages in queue** using a `while` loop.
    
5. **Avoid duplicate or redundant message processing** using `SIDE_EFFECT_FREE_MESSAGES` logic.
    
6. **Handle each message type** using a `match` block and forward to the appropriate message handler.
    
7. **Allow messages to generate child messages** and add them to the queue for further processing.
    
8. **Flush or restore buffers** when encountering `StartBuffer` or `EndBuffer`.
    
9. **Clean up empty queues** after each message is processed.
    
10. **Log messages** for debug visibility.

#### Does are message are sent to frontend ?
No ,different message type are push to different things [[Message Match Handling Explained]]