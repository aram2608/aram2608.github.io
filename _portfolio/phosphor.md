---
title: "Phosphor"
excerpt: "A toy text editor written in Raylib to help learn C++<br/><img src='/images/phosphor/phosphor.png'>"
collection: portfolio
---

Overview
---

Phosphor is a small text editor I have been working on to try and learn some more
advanced C++ concepts. It is written with the Raylib graphics library and has an
embedded Lua API using the sol3 binding library.

Most of it was written by me, with the big exception being the gap buffer. It
ended up being a bit too much out of my comfort zone so I had to rely on our good
friends ChatGPT and Gemini to get it in a functioning state.

Originally, I was handling raw strings under the hood but inserting text into a string
is quite an arduous task. To improve, I was between a rope, gap buffer, 
and piece table, and after reading some stack overflow discussions and watching a 
video from Computerphile I ultimately landed on the gap buffer.

With all that said, the gap movement isn't entirely efficient. The data
structure I chose to represent my buffer is a `std::vector<char>`, and the only method
that seemed easy to implement for gap movement was to simply make a new 
temporary vector all together. I could then transfer the contents from the original
buffer while extending the gap at the same time. So rather than a text editor,
its more of a fancy notepad at this point.

Even so, I am quite proud of the state of it so far, and am especially happy with
the Lua scripting API. It is primarily for configuration and customization, as
every modifer key is capable of binding methods available to the editor such as
text insertion and toggling between different color palettes. While not revolutionary
features, I did learn a whole lot about embedding Lua and got some fun functionality
out of it, so I am quite pleased with the end result to say the least.

The current plan is to continue making small improvements, currently text can only
be pasted and not copied, it also can't be selected, the up and down arrows don't
change the cursor position, and mouse clicks don't change the cursor much either.
It turns out calculating the logical position from the pixels on screen was a bit
harder than I imagined so that will hopefully come in the near future.

Features
---

```lua
-- Registering a command with a key binding
-- Command + H on macOS
register_command(keys.KEY_H, Mod.SUPER, function(ed)
  ed:insert_text("hello world!")
end)

-- Registering a command for toggling between palettes
register_command(keys.KEY_T, Mod.SUPER, function (ed)
  ed:toggle_palette()
end)

--[[
  Available functions:
    ed:insert_text("string")
    ed:toggle_palette()
    ed:backspace()
    ed:paste_text()
    ed:new_line()
    ed:tab()
]]

-- Overriding default palette at loadup
pick_palette(Palette.Blue)

--[[
    Palette options:
        Green (default)
        Blue
        Amber
        White
        Cyan
        Magenta
        Red
]]
```

Themes
---

![Green](/images/phosphor/green.png)

![Blue](/images/phosphor/blue.png)

![White](/images/phosphor/white.png)

![Amber](/images/phosphor/amber.png)

![Cyan](/images/phosphor/cyan.png)

![Red](/images/phosphor/red.png)

![Magenta](/images/phosphor/magenta.png)