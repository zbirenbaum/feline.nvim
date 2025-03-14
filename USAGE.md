# Usage

This is a full guide to customizing Feline to build your own statusline configuration from scratch. While this may look daunting at first, be assured that this is very easy once you understand it. But before you can customize Feline, you have to understand how Feline works.

## Components

Feline has a statusline generator that takes a `components` value, which is a Lua table that needs to contain the statusline components. Inside the `components` table, there needs to be two tables, `active` and `inactive`, which will dictate whether the component is a part of the statusline when it's in an active window or an inactive window. Inside each of the `active` and `inactive` tables, you can put any amount of tables, each of which will indicate a section of the statusline.

For example, if you want two sections (left and right), you can put two tables inside each of the `active` and `inactive` tables. If you want three sections (left, mid and right), you can put three tables inside each of the `active` and `inactive` tables. There is no limit to the amount of sections you can have. It's also possible to have a different amount of sections for the `active` and `inactive` statuslines.

So first, in your init.lua file, you have to initialize the components table like this:

```lua
-- Initialize the components table
local components = {
    active = {},
    inactive = {}
}
```

You can then add new sections to the statusline by adding an element to the `active` or `inactive` tables. For example:

```lua
-- Insert three sections (left, mid and right) for the active statusline
table.insert(components.active, {})
table.insert(components.active, {})
table.insert(components.active, {})

-- Insert two sections (left and right) for the inactive statusline
table.insert(components.inactive, {})
table.insert(components.inactive, {})
```

Now you can add statusline components to each of those sections by adding elements to the sections.
For example:

```lua
-- Insert a component that will be on the left side of the statusline
-- when the window is active:
table.insert(components.active[1], {
    -- Component info here
})

-- Insert another component that will be on the left side of the statusline
-- when the window is active:
table.insert(components.active[1], {
    -- Component info here
})

-- Insert a component that will be on the middle of the statusline
-- when the window is active:
table.insert(components.active[2], {
    -- Component info here
})

-- Insert a component that will be on the right side of the statusline
-- when the window is active:
table.insert(components.active[3], {
    -- Component info here
})

-- Insert a component that will be on the left side of the statusline
-- when the window is inactive:
table.insert(components.inactive[1], {
    -- Component info here
})

-- Insert a component that will be on the right side of the statusline
-- when the window is inactive:
table.insert(components.inactive[2], {
    -- Component info here
})
```

Alternatively you can also use Lua table indices instead of table.insert, like:

```lua
-- Insert a component that will be on the right side of the statusline
-- when the window is active:
components.active[3][1] = {
    -- Component info here
}

-- Insert another component that will be on the right side of the statusline
-- when the window is active:
components.active[3][2] = {
    -- Component info here
}
```

**NOTE:** If you use the index instead of table.insert, remember to put the correct index. Also keep in mind that unlike most other programming languages, Lua indices start at `1` instead of `0`.

### Component values

You can use component values to customize each component to your liking. Providing the values isn't necessary and you can omit all of the component values, in which case the defaults would be used instead. The component values can either be set to a fixed value or a function that generates the value everytime the statusline is being generated.

Though you should keep in mind that if a component value is set to a function, the function can take no arguments. The [`provider`](#component-providers) value is an exception to this rule (more on that below). The return type of the function must also be the same as the type of value required by the component. For example, since [`enabled`](#conditionally-enable-components) requires a boolean value, if you set it to a function, the function must also return a boolean value.

The different kinds of component values are discussed below.

#### Component providers

The `provider` value of a component can be a table, string or function.

If it's a string, it represents the text to show.

```lua
-- Providers can simply just contain a string, such as:
provider = 'some text here'
```

A string value can also refer to a [default provider](#default-providers). When the name of a default provider is used as the value of `provider`, it uses the default provider for the component.

```lua
provider = 'position' -- This will use the default file position provider
```

Some of these providers can also take some special options, in which case the value of `provider` can be a table containing two values: `name` which represents the name of the provider, and `opts` which represents the options passed to the provider.

```lua
provider = {
    name = 'file_info',
    opts = {
        type = 'unique',
        file_modified_icon = 'M'
    }
}
```

Note that you can also use your [manually added providers](#setup-function) the same way as the default providers.

The value of `provider` can also be set to a function. The function must return a string when called. The function may also optionally return an [`icon`](#component-icon) value alongside the string, which would represent the provider's default icon. The provider functions can take two arguments: `component`, which represents the component itself and can be used to access the component values from within the provider, and `opts`, which represents the provider options discussed above.

Here are a few examples of setting the provider to a function:

```lua
-- Here's an example of a basic provider with no arguments
provider = function()
    return tostring(#vim.api.nvim_list_wins())
end

-- Providers can take the component itself as an argument to access the component values using the
-- first argument passed to the provider function
provider = function(component)
    if component.icon then
        return component.icon
    else
        return ''
    end
end
```

Functions that are added as [custom providers](#setup-function) can also take a second argument, `opts`, which represents the provider options given to the provider (if any). For example:

```lua
provider = function(_, opts)
    if opts.return_two then
        return 2
    else
        return 3
    end
end
```

If you omit the provider value, it will be set to an empty string. A component with no provider or an empty provider may be useful for things like [applying a highlight to section gaps](#highlight-section-gaps) or just having an icon or separator as a component.

#### Truncation

Feline has an automatic smart truncation system where components can be automatically truncated if the statusline doesn't fit within the window. It can be useful if you want to make better use of screen space. It also allows you to better manage which providers are truncated, how they are truncated and in which order they are truncated.

**NOTE:** Due to some internal limitations, truncation does not work on Windows. As a result, all configurations related to truncation will be ignored on Windows and components won't be truncated.

There are a few component values associated with truncation which are described below.

##### Component short provider

`short_provider` is an optional component value that allows you to take advantage of Feline's truncation system. Note that this should only be defined if you want to enable truncation for the component, otherwise it's absolutely fine to omit it.

`short_provider` works just like the `provider` value, but is activated only when the component is being truncated due to the statusline not fitting within the window. `short_provider` is independent from the `provider` value so it can be a different provider altogether, or it can be a shortened version of the same provider or the same provider but with a different `opts` value. For example:

```lua
-- In this component, short provider uses same provider but with different opts
local file_info_component = {
    provider = {
        name = 'file_info',
        opts = {
            type = 'full-path'
        }
    },
    short_provider = {
        name = 'file_info',
        opts = {
            type = 'short-path'
        }
    }
}

-- Short provider can also be an independent value / function
local my_component = {
    provider = 'loooooooooooooooong',
    short_provider = 'short'
}
```

Feline doesn't set `short_provider` to any component by default, so it must be provided manually.

##### Hide components during truncation

If you wish to allow Feline to hide a component entirely if necessary during truncation, you may set the `truncate_hide` component value to `true`. By default, `truncate_hide` is `false` for every component.

##### Component priority

When components are being truncated by Feline, you can choose to give some components a higher priority over the other components. The `priority` component value just takes a number. By default, the priority of a component is `0`. Components are truncated in ascending order of priority. So components with lower priority are truncated first, while components with higher priority are truncated later on. For example:

```lua
-- This component has the default priority
local my_component = {
    provider = 'loooooooooooooooong',
    short_provider = 'short'
}
-- This component has a higher priority, so it will be truncated after the previous component
local high_priority_component = {
    provider = 'long provider with high priority',
    short_provider = 'short',
    priority = 1
}
```

Priority can also be set to a negative number, which can be used to make a component be truncated earlier than the ones with default priority.

#### Conditionally enable components

The `enabled` value of a component can be a boolean or function. This value determines if the component is enabled or not. If false, the component is not shown in the statusline. For example:

```lua
-- Enable if opened file has a valid size
enabled = function()
    return vim.fn.getfsize(vim.fn.expand('%:p')) > 0
end

-- Enable if current window width is higher than 80
enabled = function()
    return vim.api.nvim_win_get_width(0) > 80
end
```

#### Component icon

Some inbuilt providers such as `git_branch` provide default icons. If you either don't have a patched font or don't like the default icon that Feline provides, or if you want an icon for a component that doesn't have any default icons, you may set this value to use any icon you want instead.

The component's icon can be a table, string or function. By default, the icon inherits the component's highlight, but you can also change the highlight specifically for the icon. To do this, you need to pass a table containing `str` and `hl`, where `str` would represent the icon and `hl` would represent the icon highlight. The icon's highlight works just like the `hl` component's values.

There's also another value you can set if the value of `icon` is a table, which is `always_visible`. By default, the icon is not shown if the value returned by the provider is empty. If you want the icon to be shown even when the provider string is empty, you need to set `always_visible` to `true`.

```lua
-- Setting icon to a string
icon = ' + '

-- Setting icon to a function
icon = function() return ' - ' end

-- Setting icon to a table
icon = {
    str = ' ~ ',
    hl = { fg = 'orange' }
}

-- Making icon always visible
icon = {
    str = '',
    hl = {
        fg = require('feline.providers.vi_mode').get_mode_color(),
        bg = 'black',
        style = 'bold'
    },
    always_visible = true
}
```

#### Component highlight

The `hl` component value represents the component highlight. It can be a table, string or function.

If a string, it'll use the given string as the name of the component highlight group. In that case, this highlight group must be defined elsewhere (i.e. in your colorscheme or your Neovim configuration).

If it's a table, it'll automatically generate a highlight group for you based on the given values. The hl table can have four values: `fg`, `bg`, `style` and `name`.

The `fg` and `bg` values are strings that represent the RGB hex or [name](#value-presets) of the foreground and background color of the highlight, respectively. (eg: `'#FFFFFF'`, `'white'`). If `fg` or `bg` is not provided, it uses the default foreground or background color provided in the `setup()` function, respectively.

The `style` value is a string that determines the formatting style of the component's text (do `:help attr-list` in Neovim for more info). By default it is set to `'NONE'`

The `name` value is a string that determines the name of highlight group created by Feline (eg: `'StatusComponentVimInsert'`). If a name is not provided, Feline automatically generates a unique name for the highlight group based on the other values, so you can also just omit the `name` and Feline will create new highlights for you when required. However, setting `name` may provide a performance improvement since Feline caches highlight names and doesn't take the time to generate a name if the name is already provided by the user.

Note that if `hl` is a function that can return different values, the highlight is not redefined if the name stays the same. Feline only creates highlights when they don't exist, it never redefines existing highlights. So if `hl` is a function that can return different values for `fg`, `bg` or `style`, make sure to return a different value for `name` as well if you want the highlight to actually change.

Here are a few examples using the `hl` value:

```lua
-- As a table
hl = {
    fg = 'skyblue'
}

-- As a string
hl = "MyStatuslineHLGroup"

-- As a function returning a table
hl = function()
    return {
        name = require('feline.providers.vi_mode').get_mode_highlight_name(),
        fg = require('feline.providers.vi_mode').get_mode_color(),
        style = 'bold'
    }
end

-- As a function returning a string
hl = function()
    if require("feline.providers.vi_mode").get_vim_mode() == "NORMAL" then
        return "MyStatuslineNormal"
    else
        return "MyStatuslineOther"
    end
end
```

#### Component separators

There are two types of separator values that you can put in a component, which are `left_sep` and `right_sep`, which represent the separator on the left and the right side of the component, respectively.

The value of `left_sep` and `right_sep` can just be set to a string that's displayed. You can use a function that returns a string just like the other component values. The value can also be equal to the name of one of the [separator presets](#value-presets).

The value of `left_sep` and `right_sep` can also be a table or a function returning a table. Inside the table there can be three values, `str`, `hl` and `always_visible`. `str` represents the separator string and `hl` represents the separator highlight. The separator's highlight works just like the component's `hl` value. The only difference is that the separator's `hl` by default uses the parent's background color as its foreground color.

By default, Feline doesn't show a separator if the value returned by the provider is empty. If you want the separator to be shown even when the component string is empty, you can set the `always_visible` value in the separator table to `true`. If unset or set to `false`, the separator is not shown if the component string is empty.

You can also set `left_sep` and `right_sep` to be a `table` containing multiple separator elements. It's useful if you want to have different highlights for different parts of the left/right separator of the same component, or if you want to always show certain parts of the separator regardless of whether the component string is empty, or if you just want to better organize the component's separator.

For example:

```lua
-- Setting sep to a string
left_sep = ' '

-- Setting sep to a separator preset
left_sep = 'slant_right'

-- Setting sep to a table with highlight
left_sep = {
    str = 'slant_left',
    hl = {
        fg = 'oceanblue',
        bg = 'bg'
    }
}

-- Making sep always visible
right_sep = {
    str = ' ',
    always_visible = true
}

-- Setting sep to a function
right_sep = function()
    local values = { 'right_rounded', 'right_filled', 'right' }
    return values[math.random(#values)]
end

-- Setting sep to a list separator elements
right_sep = {
    {
        str = ' ',
        hl = {
            fg = 'NONE',
            bg = 'oceanblue'
        },
        always_visible = true
    },
    -- The line below is equivalent to { str = 'slant_right' }
    'slant_right'
}
```

### Component Examples

Now that we know of the possible values you can set in a component, let's make some actual components to show you how it all looks like together:

<details>
<summary>Component example</summary>

```lua
-- Component that shows Vi mode with highlight
components.active[1][1] = {
    provider = 'vi_mode',
    hl = function()
        return {
            name = require('feline.providers.vi_mode').get_mode_highlight_name(),
            fg = require('feline.providers.vi_mode').get_mode_color(),
            style = 'bold'
        }
    end,
    right_sep = ' '
}

-- Component that shows file info
components.active[1][2] = {
    provider = 'file_info',
    hl = {
        fg = 'white',
        bg = 'oceanblue',
        style = 'bold'
    },
    left_sep = {' ', 'slant_left_2'},
    right_sep = {'slant_right_2', ' '},
    -- Uncomment the next line to disable file icons
    -- icon = ''
}

-- Component that shows current file size
components.active[1][3] = {
    provider = 'file_size',
    right_sep = {
        ' ',
        {
            str = 'slant_left_2_thin',
            hl = {
                fg = 'fg',
                bg = 'bg'
            }
        },
        ' '
    }
}

-- Component that shows file encoding
components.active[2][1] = {
    provider = 'file_encoding'
}

-- Component that shows current git branch
components.active[3][1] = {
    provider = 'git_branch',
    hl = {
        fg = 'white',
        bg = 'black',
        style = 'bold'
    },
    right_sep = {
        str = ' ',
        hl = {
            fg = 'NONE',
            bg = 'black'
        }
    }
}
```

</details>

[**NOTE:** Remember to initialize the components table before assigning anything to it]

## Setup function

Now that you know about the components table and how Feline components work, you can learn about Feline's `setup()` function. The `setup()` function initializes Feline with your provided configuration. The configuration can be passed to the function through a table. The available configuration options are listed below:

- `preset` - Set it to use a preconfigured statusline. Currently it can be equal to either `default` for the default statusline or `noicon` for the default statusline without icons. You don't have to put any of the other values if you use a preset, but if you do, your settings will override the preset's settings. To see more info such as how to modify a preset to build a statusline, see: [Modifying an existing preset](#3.-modifying-an-existing-preset)
- `components` - The [components table](#components).
- `custom_providers` - A table containing user-defined [provider functions](#component-providers). For example:

```lua
custom_providers = {
    window_number = function()
        return tostring(vim.api.nvim_win_get_number(0))
    end
}
```

- `colors` - A table containing custom [color value presets](#value-presets). The value of `colors.fg` and `colors.bg` also represent the default foreground and background colors, respectively.
- `separators` - A table containing custom [separator value presets](#value-presets).
- `force_inactive` - A table that determines which buffers should always have the inactive statusline, even when they are active. It can have 3 values inside of it, `filetypes`, `buftypes` and `bufnames`, all three of them are tables which contain Lua patterns to match against file type, buffer type and buffer name respectively.<br><br>
  Default:

```lua
{
    filetypes = {
        '^NvimTree$',
        '^packer$',
        '^startify$',
        '^fugitive$',
        '^fugitiveblame$',
        '^qf$',
        '^help$'
    },
    buftypes = {
        '^terminal$'
    },
    bufnames = {}
}
```

- `disable` - Similar to `force_inactive`, except the statusline is disabled completely. Configured the same way as `force_inactive`. Feline doesn't disable the statusline on anything by default.
- `vi_mode_colors` - A table containing colors associated with Vi modes. It can later be used to get the color associated with the current Vim mode using `require('feline.providers.vi_mode').get_mode_color()`. For more info on it see the [Vi-mode](#vi-mode) section.<br><br>
Here is a list of all possible vi_mode names used with the default color associated with them:

| Mode        | Description           | Value       |
| ----------- | --------------------- | ----------- |
| `NORMAL`    | Normal mode           | `'green'`   |
| `OP`        | Operator pending mode | `'green'`   |
| `INSERT`    | Insert mode           | `'red'`     |
| `VISUAL`    | Visual mode           | `'skyblue'` |
| `LINES`     | Visual lines mode     | `'skyblue'` |
| `BLOCK`     | Visual block mode     | `'skyblue'` |
| `REPLACE`   | Replace mode          | `'violet'`  |
| `V-REPLACE` | Virtual Replace mode  | `'violet'`  |
| `ENTER`     | Enter mode            | `'cyan'`    |
| `MORE`      | More mode             | `'cyan'`    |
| `SELECT`    | Select mode           | `'orange'`  |
| `COMMAND`   | Command mode          | `'green'`   |
| `SHELL`     | Shell mode            | `'green'`   |
| `TERM`      | Terminal mode         | `'green'`   |
| `NONE`      | None                  | `'yellow'`  |

- `highlight_reset_triggers` - Feline automatically resets its cached highlights on certain autocommands to prevent the statusline colors from getting messed up. The value of `highlight_reset_triggers` can be set to a table containing a list of autocommands that'll trigger a highlight reset.<br><br>
  Default: `{'SessionLoadPost', 'ColorScheme'}`

## Example configuration

You can check out the code in the [default preset](lua/feline/presets/default.lua) to see how the components in it are set up so you can get a good practical idea of how to use the tools that Feline gives you to create all kinds of different statusline components.

## Modifying an existing preset

If you like the defaults for the most part but there's some things you want to change, then you'd be glad to know that it's easy to just modify an existing preset to get the statusline configuration you want. Just do:

```lua
-- Substitute preset_name with the name of the preset you want to modify.
-- eg: "default" or "noicon"
local components = require('feline.presets')[preset_name]
```

After that, you can just modify the components and call the [setup function](#setup-function) with the preset as you normally would.

## Default providers

Feline by default has some built-in providers to make your life easy. They are:

| Name                                  | Description                                    |
| ------------------------------------- | ---------------------------------------------- |
| [`vi_mode`](#vi-mode)                 | Current vim mode                               |
| `position`                            | Get line and column number of cursor           |
| `line_percentage`                     | Current line percentage                        |
| `scroll_bar`                          | Scroll bar that shows file progress            |
| [`file_info`](#file-info)             | Get file icon, name and modified status        |
| `file_size`                           | Get file size                                  |
| `file_type`                           | Get file type                                  |
| `file_encoding`                       | Get file encoding                              |
| [`git_branch`](#git)                  | Shows current git branch                       |
| [`git_diff_added`](#git)              | Git diff added count                           |
| [`git_diff_removed`](#git)            | Git diff removed count                         |
| [`git_diff_changed`](#git)            | Git diff changed count                         |
| `lsp_client_names`                    | Name of LSP clients attached to current buffer |
| [`diagnostic_errors`](#diagnostics)   | Diagnostics errors count                       |
| [`diagnostic_warnings`](#diagnostics) | Diagnostics warnings count                     |
| [`diagnostic_hints`](#diagnostics)    | Diagnostics hints count                        |
| [`diagnostic_info`](#diagnostics)     | Diagnostics info count                         |

### Vi-mode

The vi-mode provider by itself only shows an icon. To actually indicate the current Vim mode, you have to use `require('feline.providers.vi_mode').get_mode_color()` for the component's `hl.fg`.

Note that this is different if you set the `icon` value of the component to `''`, in that case it'll use the name of the mode instead of an icon, which is what the `noicon` preset uses.

Here is the simplest method to make a component with proper Vi-mode indication:

```lua
-- Remember to change "components.active[1][1]" according to the rest of your config
components.active[1][1] = {
    provider = 'vi_mode',
    hl = function()
        return {
            name = require('feline.providers.vi_mode').get_mode_highlight_name(),
            fg = require('feline.providers.vi_mode').get_mode_color(),
            style = 'bold'
        }
    end,
    right_sep = ' ',
    -- Uncomment the next line to disable icons for this component and use the mode name instead
    -- icon = ''
}
```

The Vi-mode provider also provides a helper function `get_mode_highlight_name()` which can be used through `require('feline.providers.vi_mode').get_mode_highlight_name()`, it returns the highlight name for the current mode, which you can then use for the provider's `hl.name` to give its highlight groups meaningful names.

### File Info

The `file_info` provider has some special provider options that can be passed through the provider `opts`:

- `colored_icon` (boolean): Determines whether file icon should use color inherited from `nvim-web-devicons`.<br>
  Default: `true`
- `file_modified_icon` (string): The icon that is shown when a file is modified.<br>
  Default:`'●'`
- `file_readonly_icon` (string): The icon that is shown when a file is read-only.<br>
  Default:`'🔒'`
- `type` (string): Determines which parts of the filename are shown. Its value can be one of:

  - `'full-path'`: Full path of the file (eg: `'/home/user/.config/nvim/init.lua'`)
  - `'short-path'`: Shortened path of the file (eg: `'/h/u/.c/n/init.lua'`)
  - `'base-only'`: Show only base filename and extension (eg: `'init.lua'`)
  - `'relative'`: File path relative to the current directory.
  - `'relative-short'`: Combination of `'relative'` and `'short-path'`.
  - `'unique'`: Unique substring of the full path.<br>
    For example: If you have three buffers with the paths `'/home/user/file.lua'`, `'/home/user/dir1/file.lua'` and `'/home/user/dir2/file.lua'`, Feline will show the names `'user/file.lua'`, `'dir1/file.lua'` and `'dir2/file.lua'` for them, respectively.<br>
    If there's no files that share the same name, it behaves the same as `'base-only'`.
  - `'unique-short'`: Combination of `'unique'` and `'short-path'`.

  <br>Default: `'base-only'`

### Git

The git providers all require [gitsigns.nvim](https://github.com/lewis6991/gitsigns.nvim/), make sure you have it installed when you use those providers, otherwise they'll have no output.

The git provider also provides a utility function `require('feline.providers.git').git_info_exists()` for checking if any git information exists.

### Diagnostics

The diagnostics and LSP providers all require the Neovim built-in LSP to be configured and at least one LSP client to be attached to the current buffer, else they'll have no output.

The diagnostics provider also provides a utility function `require('feline.providers.lsp').diagnostics_exist(type)` for checking if any diagnostics of the provided type exists. The values of `type` must be one of `'Error'`, `'Warning'`, `'Hint'` or `'Information'`.

## Value presets

Value presets are names for colors and separators that you can use instead of the hex code or separator string, respectively.

For your ease of use, Feline has some default color and separator values set. You can manually access them through `require('feline.defaults').colors` and `require('feline.defaults').separators` respectively. But there's a much easier way to use them, which is to just directly assign the name of the color or separator to the value, eg:

```lua
hl = {bg = 'oceanblue'},
right_sep = 'slant_right'
```

Not only that, you can add your own custom colors and separators through the [setup function](#setup-function) which allows you to just use the name of the color or separator to refer to it.

Below is a list of all the default value names and their values:

### Default colors

| Name        | Value       |
| ----------- | ----------- |
| `fg`        | `'#D0D0D0'` |
| `bg`        | `'#1F1F23'` |
| `black`     | `'#1B1B1B'` |
| `skyblue`   | `'#50B0F0'` |
| `cyan`      | `'#009090'` |
| `green`     | `'#60A040'` |
| `oceanblue` | `'#0066cc'` |
| `magenta`   | `'#C26BDB'` |
| `orange`    | `'#FF9000'` |
| `red`       | `'#D10000'` |
| `violet`    | `'#9E93E8'` |
| `white`     | `'#FFFFFF'` |
| `yellow`    | `'#E1E120'` |

### Default Separators

| Name                 | Value |
| -------------------- | ----- |
| `vertical_bar`       | `'┃'` |
| `vertical_bar_thin`  | `'│'` |
| `left`               | `''` |
| `right`              | `''` |
| `block`              | `'█'` |
| `left_filled`        | `''` |
| `right_filled`       | `''` |
| `slant_left`         | `''` |
| `slant_left_thin`    | `''` |
| `slant_right`        | `''` |
| `slant_right_thin`   | `''` |
| `slant_left_2`       | `''` |
| `slant_left_2_thin`  | `''` |
| `slant_right_2`      | `''` |
| `slant_right_2_thin` | `''` |
| `left_rounded`       | `''` |
| `left_rounded_thin`  | `''` |
| `right_rounded`      | `''` |
| `right_rounded_thin` | `''` |
| `circle`             | `'●'` |

## Tips and tricks

### Get current window or buffer

When the statusline for a window is being generated, Neovim temporarily sets the current window and buffer to the window and buffer for which the statusline is being generated.

This is important to note when you set a component value to a function. Inside a component value that's set to a function, functions like `vim.api.nvim_get_current_win()` and `vim.api.nvim_get_current_buf()` will return the statusline window and buffer instead of the actual current window and buffer number. In order to access the actual current window or buffer, you have to use `vim.g.actual_curbuf` or `vim.g.actual_curwin` (respectively) inside the function instead. For example:

```
-- Provider function that shows current window number
provider = function()
    return vim.g.actual_curwin
end

-- Provider function that shows name of current buffer
provider = function()
    return vim.api.nvim_buf_get_name(tonumber(vim.g.actual_curbuf))
end
```

Note that the values of both `vim.g.actual_curwin` and `vim.g.actual_curbuf` are strings, not numbers. So if you want to use them as a number, use `tonumber()` to convert the string to a number first, as shown in the second example.

### Reset highlight

If, for some reason, you want to clear all highlights that Feline sets (useful if you want to reload your entire Neovim config which may mess up highlights), you can do:

```lua
require('feline').reset_highlights()
```

And then Feline will automatically regenerate those highlights when it needs them, so you don't have to worry about setting the highlights yourself.

### Highlight section gaps

By default, gaps between two sections inherit the highlight of the last element of the section. If you wish to customize the highlight of the gap between two sections, you can just add a component with only an `hl` value to the end of the first section. 

For example, if you had two sections in the active statusline and wanted the gap between the first and second section to use a certain background color, you could do this:

```lua
components.active[1] = {
    {
        -- Insert all components of first section here

        -- Component for customizing highlight for the gap between section 1 and 2
        {
            hl = {
                -- Replace 'oceanblue' with whatever color you want the gap to be.
                bg = 'oceanblue'
            }
        }
    },
    {
        -- Insert all components of second section here
    }
}
```

It's even simpler if you want to use the default `bg` color for the gap between sections. In that case, you can just put an empty component at the end of the first section. You don't even have the define the `hl` manually since `hl` by default uses the default `bg` as its background.
