
```
#[inline(always)]
    pub fn to_linear_srgb(&self) -> Self {
        Self {
            red: Self::srgb_to_linear(self.red),
            green: Self::srgb_to_linear(self.green),
            blue: Self::srgb_to_linear(self.blue),
            alpha: self.alpha,
        }
    }
```
#### **inline function** 
if the function is so small its best to replace then calling  ( this concept is there in c and c++)
This boosts the performance 
https://media.geeksforgeeks.org/wp-content/uploads/20221229112934/Inline-Function-in-Cpp.png

Notice there is no return statement because the rust method with last line without semicolon is a return statement .

```
red: Self::srgb_to_linear(self.red),
```
this call the another method
```
#[inline(always)]
    pub fn srgb_to_linear(channel: f32) -> f32 {
        if channel <= 0.04045 { channel / 12.92 } else { ((channel + 0.055) / 1.055).powf(2.4) }
    }
```
It does this because:
- sRGB stores colors in a **non-linear** way to match human vision.
- But to do **correct color math (blending, gradients, lighting)**, you need **linear light**.