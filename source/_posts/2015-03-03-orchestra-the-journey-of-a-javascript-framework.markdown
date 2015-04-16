---
layout: post
title: "Orchestra - The Journey of a Javascript Framework"
date: 2015-04-16 15:00:00 +0100
comments: true
author: Andrew Humphreys
categories: javascript
---


A lot of my colleagues are probably sick and tired of hearing me talk about this super awesome framework we have built in the Bede Games Studio, so I thought I'd give them all a break a write a blog post about it instead!

<!-- more -->

## What is Orchestra?

The chances are you're probably reading this wondering what on Earth Orchestra actually is... in a nutshell - a purpose built Javascript framework for large scale applications. Harnessing the flexibility of [MarionetteJS](http://marionettejs.com/) framework to give us the tools we need and enable us to implement some battle tested patterns for large scale Javascript applications (probably one for a different blog post!).

It was built with the modularity and flexibility of the Bede platform in mind, apps are split into smaller reuseable components: Chat, Tickets, Countdown etc for Bingo, what this means is that should we decide we want a chat client in another game - we just include the orchestra-chat component and provide its config and 'it will just work'. It has built in support for theming and localisation allowing us to easily skin up new designs based on existing components, our Bingo client is a great example of this in action. All of bingo clients use the same core codebase, with a different theme and language file.

Even in its early days it was clear Orchestra was going to be a great fit for Bingo but now we have taken this a step further with a series of changes and have built out our Slots Client using the same framework, the aim of this post is to explain the changes we made and the rational behind them.

## The battle of the Javascript Build Tools

If I had a £ for every article I've read with this heading as the title I'd be an extremely rich man and would probably be writing this post from my yacht in the Bahamas - unfortunately that is not the case. So, originally [Grunt](http://gruntjs.com) was the build tool used for our JS projects at Bede, it was amazing - everyone loved it but then something changed... [Gulp](http://gulpjs.com) came along.

Now before you all think this is another case of simply jumping on a bandwagon, let me provide some justification for converting our build from Grunt to Gulp. We know that Gulp provides two main benefits:

1. Code over configuration
2. Speed

But what difference do these make to our apps? Code over configuration is something that is quite a difficult sell, however take a look at this snippet from our old Gruntfile:

```js
less: {
  	development: {
	    options: {
	      	sourceMap: true,
	      	ieCompat: true,
	      	customFunctions: {
	        	'assets-path': function (less, path) {
		          	return '/assets/media/' + path.value;
		        }
	      	}
	    },
	    files: {
	      	'assets/css/bingostars/desktop.css': ['assets/less/themes/bingostars/desktop/main.less'],
	      	'assets/css/bingostars/mobile.css': ['assets/less/themes/bingostars/mobile/main.less'],
	      	'assets/css/healthbingo/desktop.css': ['assets/less/themes/healthbingo/desktop/main.less'],
	      	'assets/css/healthbingo/mobile.css': ['assets/less/themes/healthbingo/mobile/main.less'],
	      	'assets/css/stv/desktop.css': ['assets/less/themes/stv/desktop/main.less'],
	      	'assets/css/stv/mobile.css': ['assets/less/themes/stv/mobile/main.less'],
	      	'assets/css/betvictor/desktop.css': ['assets/less/themes/betvictor/desktop/main.less'],
	      	'assets/css/betvictor/mobile.css': ['assets/less/themes/betvictor/mobile/main.less'],
	      	'assets/css/lovebingo/desktop.css': ['assets/less/themes/lovebingo/desktop/main.less'],
	      	'assets/css/lovebingo/mobile.css': ['assets/less/themes/lovebingo/mobile/main.less']
	    }
  	}
}
```

and compare it to:

```js
var lessTaskNames = ['cleanCSS'].concat(themeTaskGenerator('less', function (theme, platform) {
  return gulp.src('assets/less/themes/' + theme.name + '/' + platform + '/main.less')
    .pipe(less({
      modifyVars: {
        'asset-path': '"/assets"'
      }
    }))
    .pipe(rename(platform + '.css'))
    .pipe(gulp.dest('assets/css/' + theme.name + '/'));
}));

```

from our new Gulp task, straight away you can see the reduced lines of code, but perhaps hidden in here is what I would consider a bigger benefit. Notice the list of output files in the Grunt config? This grows with every new theme, but because Gulp is code based, we can simply iterate over a `themes` array. 

Something that is not such a difficult sell is the time this move shaved off our build task. Before Gulp we built 6 themes in a total time of 8 minutes, now after the conversion we are building 12 themes in 6 minutes - I think we can class that as a win. There are 2 main reasons for this speed benefit, the first being tasks are ran concurrently by default whereas Grunt will only run tasks asynchronously. The second being that Gulp makes use of `pipe` which is native in node and most popular OS's this means that whereas Grunt is required to create temporary files for its tasks, Gulp can pipe a file into its next task.

## The battle of the Javascript Module Loaders

I'm guessing you can see a theme appearing here, yet another well trodden path in the plethora of JavaScript blogs/articles is the comparison of the different module loaders available. For me there are 3 main contenders: [RequireJS](http://requirejs.org) which will load in AMD modules, [Browserify](http://browserify.org) which will load in CommonJS modules and [Webpack](http://webpack.github.io) which will load anything and everything. It should be noted that ES6 brings a new module pattern, which the community hope will bring some unity. It is possible to start using these new ES6 features now with the help of [Babel](http://babeljs.io/) which will transpile ES6 modules to either AMD or CommonJS until they are widely available. I've dabbled with Babel a bit and have some friends that work on the Babel project, it looks amazing, but we're not switching just yet! :smile:

Originally Orchestra was setup to build using AMD modules with the RequireJS tool, this was something that hindered us for a couple of reasons:

1. The modules are loaded at the same time during development vs concatenated during a production build.
2. The build time was around 60seconds per theme using RequireJS.

Gulp was also unable to run these tasks concurrently as that isn't supported by the RequireJS compiler, we needed to look at the alternatives. Browserify came up trumps - it allowed us to use the same module pattern across browser apps and Node.js apps and also solved the 2 problems I mentioned above: Your development version is the same as the production build, it compiles and provides sourcemaps (the overhead for the compile step when using [watchify](https://github.com/substack/watchify) is milliseconds) and the build time per theme was reduced to 18seconds - when you add the benefit of building all themes concurrently its easy to see why this was a wise move!

Thats it for part 1, in Part 2 I'll cover changes that were made as more apps began to use the Orchestra framework.




