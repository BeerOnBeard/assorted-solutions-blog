---
title: No More NPM CLI Packages?!
subtitle: Run NPM Packages From the Command Line Without Installing Them Globally
date: 2017-09-04
---

When using tools like Grunt, Gulp, Webpack, or Hexo.io, the best practice has been to install a CLI (command line interface) package globally using `npm install -g`. If you're working on a bunch of projects with different versions of these tools, you have the potential to run into some sticky situations. For example, what if the build server is updated and your local dev machine is not? That can make for some really hard to find bugs. I can already hear "...but it works on my machine!". NPM comes to the rescue with NPM scripts that allow you to access the packages installed in the project via bindings in package.json.

Let's use this blog as an example of how I've implemented some helpful NPM scripts for my local development and build server. I have the following in my package.json for this project.

```javascript
...
  "scripts": {
    "dev": "hexo server",
    "build": "hexo generate",
    "new-post": "hexo new post"
  },
...
```

I can execute each of these bindings from the command line. NPM will look to my project's installed packages for `hexo`. When I want to generate the site and launch a dev http server for testing, I run the following in my terminal.

```bash
npm run dev
```

I can even pass arguments. For example, to generate the Markdown file for this post, I ran the following command.

```bash
npm run new-post "No More NPM CLI Packages?!"
```

There's a lot more power built into the NPM scripts system that I haven't covered here. Check out [the NPM scripts docs](https://docs.npmjs.com/misc/scripts) for more info on other useful features.
