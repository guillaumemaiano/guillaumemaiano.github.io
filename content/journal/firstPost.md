+++
title = 'Setup a Github Page'
type = "post"
date = 2023-10-27T18:55:39+02:00
draft = false
tags =  ["hugo", "Setup", "techblog", "Github Pages"]
summary = "Experimenting with Github Pages"
+++

## Experimental page to test out Github Pages

### Potentialities

I'm going, if I can find time, to turn this into a techblog.

### Process

This is a quick reminder built with Vim and basic html.

- Create a repository, public, with SSL enabled, and the {username}.github.io name.
- Turn Github Actions on, because CI/CD is good and cool.
- Turn it into a Hugo Site - because Hugo is nice and great.
        Read this: https://gohugo.io/hosting-and-deployment/hosting-on-github/
- Enable Github submodules - Because HugoWebsites tend to run with themes which use Submodules.
        I believe I only need to change the .gitmodules to pull on https?
        Read this:
        https://docs.github.com/fr/pages/getting-started-with-github-pages/using-submodules-with-github-pages
- Added a footer at the bottom, using the method demonstrated by Walton:
        https://philipwalton.github.io/solved-by-flexbox/demos/sticky-footer/
- Dark mode (many tutorials out there, but would be nice to just pull a nice efficient library?)
- Code in a TODO effect using text-decoration: linethrough and ui checkbox / glyphs (circle
        outline + check circle sounds good)
        Ideally, build that into the CSS for future usage
- Theme Semantic appropriately using the recommanded method rather than quick-and-dirty edits of
        the CSS I wrote
- Theme Hugo using the recommanded method and a self-made Hugo Theme - likely building on
        something basic from the Hugo Theme Library
- Turn this HTML into MD
- Make sure tabs work for posts

### More ideas

||
|-----|
| Make the whole footer a green gradient|
| Make sure the footer is visible when content isn't overlarge|

### Code test

```swift
let thereBe = "code"
```

### Other notes

To run locally even though the HUGO TOML states the URL is guillaumemaiano.github.io, just `hugo server -b http://localhost:1313`