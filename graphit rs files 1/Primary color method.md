

**method that updates the primary color** 
```
    #[wasm_bindgen(js_name = updatePrimaryColor)]
    pub fn update_primary_color(&self, red: f32, green: f32, blue: f32, alpha: f32) -> Result<(), JsValue> {
        let Some(primary_color) = Color::from_rgbaf32(red, green, blue, alpha) else {
            return Err(Error::new("Invalid color").into());
        };
        let message = ToolMessage::SelectPrimaryColor {
            color: primary_color.to_linear_srgb(),
        };
        self.dispatch(message);
        Ok(())
    }
```
==input== : takes 4 f32 as input `red`, `green`, `blue`, alpha
```
let Some(primary_color) = Color::from_rgbaf32(red, green, blue, alpha) else {
            return Err(Error::new("Invalid color").into());
        };
```
tries to create a object of color by passing 4 values if it fails else will be executed. if the objected created successfully then its stored in primary color  
```
let message = ToolMessage::SelectPrimaryColor {
            color: primary_color.to_linear_srgb(),
        };
```
what does primary_color.to_linear_srgb() do ?
[[primary_color.to_linear_srgb(),]]

```
self.dispatch(message);
        Ok(())
```
dispatch take the input message and updates the app's state[[dispatch fn]]