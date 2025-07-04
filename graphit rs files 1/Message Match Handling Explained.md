Does all message coming are executed , sent where will it go?
### ✅ The answer is:

> **It depends on the message type.**

This function is a **central dispatcher**. Some messages are:

- ✅ **Executed internally** (by running logic in Rust)
- 📤 **Sent to the frontend** (JavaScript/browser UI)
- 🧵 **Buffered** for later execution
- 🔁 **Recursively dispatched** if batched

|`Message` Variant|What It Does|What It Pushes|Where It Goes|
|---|---|---|---|
|`StartBuffer`|Begins buffering mode|None|Moves `self.message_queues` → `self.buffered_queue`|
|`EndBuffer(render_metadata)`|Ends buffering, flushes buffer, and pushes 3 internal messages|`DocumentMessage::UpdateUpstreamTransforms`, `UpdateClickTargets`, `UpdateClipTargets`|Added to `self.message_queues` via `schedule_execution`|
|`NoOp`|Does nothing|None|N/A|
|`Init`|Queues frontend setup messages|Multiple `FrontendMessage`s & `MenuBarMessage::SendLayout`|Pushed to `queue` (then to `self.message_queues`)|
|`Animation(message)`|Delegates to `animation_message_handler`|May push more messages|Appended to `queue`|
|`Batched(messages)`|Recursively processes messages|Each sub-message|Dispatched via recursive `handle_message`|
|`Broadcast(message)`|Handled via `broadcast_message_handler`|May push more|Appended to `queue`|
|`Debug(message)`|Handled via `debug_message_handler`|May push more|Appended to `queue`|
|`Dialog(message)`|Handled via `dialog_message_handler`|May push more|Appended to `queue`|
|`Frontend(message)`|Either: pushes to `self.responses` (→ frontend) or returns early|`FrontendMessage`|`self.responses` (not the message queue!)|
|`Globals(message)`|Handled via `globals_message_handler`|May push more|Appended to `queue`|
|`InputPreprocessor(message)`|Uses platform info, processes via `input_preprocessor_message_handler`|May push more|Appended to `queue`|
|`KeyMapping(message)`|Handles input-to-action logic|May push more|Appended to `queue`|
|`Layout(message)`|Processes layout update logic|May push more|Appended to `queue`|
|`Portfolio(message)`|Core document/tool/preference logic|May push more|Appended to `queue`|
|`Preferences(message)`|Updates user preferences|May push more|Appended to `queue`|
|`Tool(message)`|Tool interaction logic on active document|May push more|Appended to `queue`|
|`Workspace(message)`|Handles workspace state updates|May push more|Appended to `queue`|