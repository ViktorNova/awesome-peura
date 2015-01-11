# awesome-peura
A theme & configuration for Awesome WM. All image files are from Zenburn theme.

Wallpaper is made by [Jen Bartel](http://www.jenbartel.com/).

The theme itself may not be so interesting, but diagonal wiboxes are a unique implementation. 
Rc.lua contains only clock and systray widgets by default. Layouts are from [Lain](https://github.com/copycat-killer/lain) – which is included in this repository.

![alt text](https://raw.githubusercontent.com/olzraiti/awesome-peura/master/screenshot.png)

#Installing
1. Clone this repository to your computer and move `rc.lua` and `themes/peura` to `~/.config/awesome`.

2. Add your widgets to the `widgets` array.

If you wish to use different theme, remember to add theme.bg_widgets array to the theme.

`theme.bg_systray` must be pointed to `theme.bg_widgets` by it's index in widgets, i.e. `theme.bg_systray = theme.bg_widgets[1]`

If you wish to copy the diagonal wiboxes functionality to your rc.lua without using this rc.lua, replace this line

`mytasklist[s] = awful.widget.tasklist(s, awful.widget.tasklist.filter.currenttags, mytasklist.buttons)`

with this:

```
function update_function(w, buttons, label, data, objects)
	-- update the widgets, creating them if needed
	w:reset()
	updateFirstSeparator(beautiful.bg_normal)
	local len = tablelength(objects)
	for i, o in ipairs(objects) do
		local cache = data[o]
		local ib, tb, bgb, l
		if cache then
			ib = cache.ib
			tb = cache.tb
			bgb = cache.bgb
		else
			ib = wibox.widget.imagebox()
			tb = wibox.widget.textbox()
			bgb = wibox.widget.background()
			l = wibox.layout.fixed.horizontal()

			-- All of this is added in a fixed widget
			l:fill_space(true)
			l:add(ib)
			l:add(tb)

			-- And all of this gets a background
			bgb:set_widget(l)

			bgb:buttons(common.create_buttons(buttons, o))

			data[o] = {
				ib = ib,
				tb = tb,
				bgb = bgb,
			}
		end

		local text, bg, bg_image, icon = label(o)
		-- The text might be invalid, so use pcall
		if not pcall(tb.set_markup, tb, text) then
			tb:set_markup("<i>&lt;Invalid text&gt;</i>")
		end
		bgb:set_bg(bg)
		if type(bg_image) == "function" then
			bg_image = bg_image(tb,o,m,objects,i)
		end
		bgb:set_bgimage(bg_image)
		ib:set_image(icon)

		local prevtext, prevBg = nil
		if i > 1 then
			prevtext, prevBg = label(objects[i - 1])
		end

		local nextText, nextBg = nil
		if i < len then
			nextText, nextBg = label(objects[i + 1])
		end

		local bgbContainer = wibox.layout.align.horizontal()
		if prevBg == nil then 
			local lDecoration = wibox.widget.background()
			lDecoration:set_widget(separator_text)
			lDecoration:set_bg(beautiful.bg_normal)
			lDecoration:set_fg(bg)
			bgbContainer:set_left(lDecoration)
		end

		bgbContainer:set_middle(bgb)

		local rDecoration = wibox.widget.background()
		rDecoration:set_widget(separator_text)
		if nextBg ~= beautiful.bg_normal then
			rDecoration:set_bg(bg)
			rDecoration:set_fg(nextBg)
		else
			rDecoration:set_fg(beautiful.bg_normal)
		end
		if nextBg ~= nil then
			bgbContainer:set_right(rDecoration)
			updateFirstSeparator(beautiful.bg_normal)
		else
			updateFirstSeparator(bg)
		end

		local bgbWrapper = wibox.widget.background()
		bgbWrapper:set_widget(bgbContainer)
		bgbWrapper:set_bg(bg)
		bgbWrapper:set_fg(beautiful.bg_normal)
		w:add(bgbWrapper)
	end
end

mytasklist[s] = awful.widget.tasklist(s, awful.widget.tasklist.filter.currenttags, mytasklist.buttons, nil, update_function, tasklist_layout)
```
