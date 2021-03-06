---
title: Routing
---

As we've seen, there are two types of route in Sapper — pages, and server routes.


### Pages

Pages are Svelte components written in `.html` files. When a user first visits the application, they will be served a server-rendered version of the route in question, plus some JavaScript that 'hydrates' the page and initialises a client-side router. From that point forward, navigating to other pages is handled entirely on the client for a fast, app-like feel.

The filename determines the route. For example, `routes/index.html` is the root of your site:

```html
<!-- routes/index.html -->
<svelte:head>
	<title>Welcome</title>
</svelte:head>

<h1>Hello and welcome to my site!</h1>
```

A file called either `routes/about.html` or `routes/about/index.html` would correspond to the `/about` route:

```html
<!-- routes/about.html -->
<svelte:head>
	<title>About</title>
</svelte:head>

<h1>About this site</h1>
<p>TODO...</p>
```

Dynamic parameters are encoded using `[brackets]`. For example, here's how you could create a page that renders a blog post:

```html
<!-- routes/blog/[slug].html -->
<svelte:head>
	<title>{post.title}</title>
</svelte:head>

<h1>{post.title}</h1>

<div class='content'>
	{@html post.html}
</div>

<script>
	export default {
		// the (optional) preload function takes a
		// `{ params, query }` object and turns it into
		// the data we need to render the page
		preload({ params, query }) {
			// the `slug` parameter is available because this file
			// is called [slug].html
			const { slug } = params;

			return this.fetch(`blog/${slug}.json`).then(r => r.json()).then(post => {
				return { post };
			});
		}
	};
</script>
```

> See the section on [preloading](guide#preloading) for more info about `preload` and `this.fetch`


### Server routes

Server routes are modules written in `.js` files that export functions corresponding to HTTP methods. Each function receives Express `request` and `response` objects as arguments, plus a `next` function. This is useful for creating a JSON API. For example, here's how you could create an endpoint that served the blog page above:

```js
// routes/blog/[slug].json.js
import db from './_database.js'; // the underscore tells Sapper this isn't a route

export async function get(req, res, next) {
	// the `slug` parameter is available because this file
	// is called [slug].json.js
	const { slug } = req.params;

	const post = await db.get(slug);

	if (post !== null) {
		res.set('Content-Type', 'application/json');
		res.end(JSON.stringify(post));
	} else {
		next();
	}
}
```

> `delete` is a reserved word in JavaScript. To handle DELETE requests, export a function called `del` instead.

There are three simple rules for naming the files that define your routes:

* A file called `routes/about.html` corresponds to the `/about` route. A file called `routes/blog/[slug].html` corresponds to the `/blog/:slug` route, in which case `params.slug` is available to `preload`
* The file `routes/index.html` (or `routes/index.js`) corresponds to the root of your app. `routes/about/index.html` is treated the same as `routes/about.html`.
* Files and directories with a leading underscore do *not* create routes. This allows you to colocate helper modules and components with the routes that depend on them — for example you could have a file called `routes/_helpers/datetime.js` and it would *not* create a `/_helpers/datetime` route



### Subroutes

Suppose you have a route called `/settings` and a series of subroutes such as `/settings/profile` and `/settings/notifications`.

You could do this with two separate pages — `routes/settings.html` (or `routes/settings/index.html`) and `routes/settings/[submenu].html`, but it's likely that they'll share a lot of code. If there's no `routes/settings.html` file, then `routes/settings/[submenu].html` will match `/settings` as well as subroutes like `/settings/profile`, but the value of `params.submenu` will be `undefined` in the first case.



### Error page

In addition to regular pages, there is a 'special' page that Sapper expects to find — `routes/_error.html`. This will be shown when an error occurs while rendering a page.

The `error` object is made available to the template along with the HTTP `status` code.



### Regexes in routes

You can use a subset of regular expressions to qualify route parameters, by placing them in parentheses after the parameter name.

For example, `routes/items/[id([0-9]+)].html` would only match numeric IDs — `/items/123` would match, but `/items/xyz` would not.

Because of technical limitations, the following characters cannot be used: `/`, `\`, `?`, `:`, `(` and `)`.