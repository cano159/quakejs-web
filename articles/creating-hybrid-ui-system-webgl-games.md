Title: Creating a hybrid UI system for WebGL games
Published: December 27, 2012

UI work is never easy. Layout engines are full of kludges, custom styling of elements is overly complex, and targetting different screen resolutions is often completely uncharted territory.

With that said, I decided to take advantage of the DOM / CSS for our UI layout and styling without a moment's hesiation.

**Pros**

 * I didn't have to write yet-another-bad layout system. I can't stress this enough. While the CSS box model has it's fair share of issues, there is a huge amount of available skill that already knows them and it's been used to create some of the richest UIs of the past decade.
 * HTML / CSS enforces a good separation of layout and styling.
 * CSS3 provides great out-of-the-box rendering capabilities such as rounded corners, drop shadows and gradients.
 * Targetting different resolutions has many well-known approaches.

**Cons**

 * Games often expect the UI to be rendered each frame. Unfortunately, rendering DOM is slow meaning our UI must be programmed very defensively. Using proper models to represent the data for your views can help out with this (we're using [backbone.js](http://www.backbonejs.org)).
 * We won't receive the normal DOM events since we're trapping user input in our game (e.g. through requestPointerLock). A layer must be created that emulates DOM events such as `click` from the trapped input in order to provide the same familiar ecosystem.
 * It may not be possible to directly access your image assets through <img> tags and background styles.

Again, having worked with enough difficult UI systems, the first pro was enough to sway me. After some initial tests showed that overlaying my UI on my canvas wouldn't impose any significant performance penalty, I was ready to get started.

For non-interactive menus, getting started is very easy. Basically, you're just throwing HTML in a div that's absolutely positioned and stretched across the top of your canvas element. Brandon Jones put together an excellent sample project [showing how to do this](http://media.tojicode.com/webgl-samples/hud-test.html).

The biggest hurdle in this approach comes when you need to interact with the UI. Once your WebGL canvas has requested a pointer lock, your input events are going to be contextual to it, having nothing to do with any UI you've absolutely positioned on top of it. The easy out here is to expect users to always break the pointer lock (which means pressing the escape key) to use your menus, but in the interest of providing a good user experience, I opted to emulate the basic DOM events for our views.

## Emulating the DOM events

Frontend web engineers are familiar with DOM events such as `mouseenter`, `mouseleave`, `click`, etc. for handling mouse input. In addition, they're accustomed to pseudo-selectors in CSS such as `:hover` for styling hovered elements.

While emulating the DOM events is pretty straight-forward, emulating pseudo selectors such as the hover one can be tricky.

In our case, I chose to adopt using concrete classes such as `.hover`. This means manually finding *all* of the currently hovered elements and managing the adding / removing of this class to each element on each mouse movement event.

However, since browsers only expose the ability to find the topmost element at any given point, you'll end up having to write some code similar to this in order to find the elements for a given point inside a view:

	var el = document.elementFromPoint(x, y);
	if (!el) {
		return null;
	}

	// Does the view even contain the matched element?
	if (!$.contains(view.el, el)) {
		return [el];
	}

	// Find all of the child elements that are also at this point.
	var $matches = $(el).parentsUntil(view.el, function () {
		var $parent = $(this);
		var offset = $parent.offset();
		var range = {
			x: [offset.left, offset.left + $parent.outerWidth()],
			y: [offset.top, offset.top + $parent.outerHeight()]
		};
		return (x >= range.x[0] && x <= range.x[1]) && (y >= range.y[0] && y <= range.y[1]);
	});
{: class="prettyprint linenums"}

## Writing defensive views

The DOM is slow. Don't expect to be able to render your UI each frame, and program defensively when doing so.

For example, take advantage of backbone's models if you're using it and feel free to update the model each frame. However, use [the model's change events](http://backbonejs.org/#Model) and partially update your view from there.

Also, be very careful using a libraries such as jQuery. While it provides great abstractions for things such as setting CSS properties, you can quickly find that it's sucking up a significant portion of your render loop if you use it intensely.

Finally, cache your DOM elements for subsequent partial updates after a complete render - don't query the DOM for them each time they need to be updated - that can be *very* slow!

## Referencing external assets

Another issue I encountered was referencing assets such as images from within views. We abstract away file loading in the game, so we don't want HTML / CSS referencing specific URIs for their content.

In the end, I ended up using data attributes to describe the desired resource inside a view like so:

	...
	<div class="fullscreen" data-image="ui/main_menu_bg.jpg"></div>
{: class="prettyprint linenums"}

After rendering each view, we check for those data attributes and use our filesystem abstractions to load up and swap in an image asset as [base64-encoded background image](http://en.wikipedia.org/wiki/Data_URI_scheme).

## Summary

While emulating some of these events is a bit frustrating up front, it pays off in the long run. Creating web-based UI is, even with all of its issues, the easiest experience I've had in my years of Win32 / MFC / .NET UI work. Emulating the DOM events to persist this experience to the game world has been a huge overall win.

The HTML / CSS / DOM stack has been one of the most battle tested UI platforms of the past decade and with it, has some of the best resources available for working with it to create solid user interfaces - take advantage of it!

![quakejs UI](/articles/creating-hybrid-ui-system-webgl-games/ui.jpg)