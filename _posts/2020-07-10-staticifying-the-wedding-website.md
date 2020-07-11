---
title: Static-ifying the Wedding Website
---

I got married in 2016. It was great.

One of the things I did in the lead up to the event was build a website as a companion for the invitations; it had more info for guests, a form to RSVP, a page to offer or request a lift to the event (which was held ~3 hours' drive from the city in which most of the guests lived) and a gift registry.

I built the site in a technology stack that I was comfortable with at the time based on my experience at work: a Knockout frontend, a Node.js/Express backend and a MongoDB database to support the liftshare and gift registry functionality. The backend was hosted on a single free Heroku dyno, while the database was provided by free add-on. It all worked perfectly adequately.

Fast-forward a few years, and code inevitably rots. In this case, the provider of the free database is being acquired by another company and will shortly (as of this writing) terminate my database. What to do? Well, the site has served its purpose, so I could just take it down. It has sentimental value, so instead I'm archiving it into a format that will hopefully last a little bit longer before rotting by removing as much of the code as possible.

Let's walk through the process.

- toc
{:toc}

## Recon

Upon opening the repository, I am greeted by a deafening lack of documentation - no README, no CHANGELOG. That would be too easy. Instead we have:

- `.gitignore` It looks suspiciously like I did the initial development for this site in Visual Studio, which makes sense as I only moved to Visual Studio Code in late 2016 or perhaps early 2017.
- `.bowerrc` Ah, I used Bower - like NPM for frontend packages, from before NPM was like NPM for frontend packages. I should be able to remove Bower completely in this migration.
- `bower.json` Here we see a convenient list of frontend packages I'm using:
  - `requirejs` Module loader and bundler, supplanted by Webpack.
  - `knockout` JavaScript framework, supplanted by React/Vue/a million others.
  - `jquery` Needs no introduction. I used it for both DOM manipulation and as an XHR client to make API request to the backend.
  - `lodash` Array utilities gone wild. I used to be a big fan of Lodash but its relevance has somewhat faded as JavaScript's native features have improved.
  - `domReady` A tiny little RequireJS plugin that waits for the DOMLoaded event before starting the app.
  - `text` A RequireJS plugin to load non-JavaScript files as text. Why does that require a plugin? Nevermind.
  - `bootstrap` The ubiquitous CSS framework - this is one of the few parts I might want to keep.
  - `pagerjs` A Single-Page Application framework/router built for Knockout.
  - `assert` A port of Node's built-in assert utility; this is primarily a development aide to make assumptions about parameters concrete (no static types around here).
- `package.json` Another convenient list of dependencies, this time those installed by NPM for the backend and the build/dev process:
  - `body-parser` Express middleware to (shock) parse the request body from JSON or URL-encoded (i.e. from HTML forms) text to JS objects.
  - `bower` Of course we use one package manager to install the other one.
  - `bunyan` Logging library which (by default) exposes the stream of log messages as newline-delimited JSON with a companion command-line tool to pretty-print that stream.
  - `compression` Express middleware to (shock) compress responses. You shouldn't really need this - either performance isn't a concern and you don't need compression, or performance is a concern and your separate webserver or proxy should be doing the compression.
  - `express` Web application framework.
  - `flat` Tiny utility to de-nest a JS object, in this case for compatibility with Heroku's log parser.
  - `http-auth` Express middleware to implement HTTP Basic authentication.
  - `kerberos` If I was using HTTP Basic auth, why did I need Kerberos? If I remember correctly, the MongoDB driver would fail to install without it. Fun.
  - `lodash` Array utilities gone wild, the sequel.
  - `logfmt` A library provided by Heroku to transform log messages into their supported format.
  - `mongoose` A popular Object-Document Mapper (ODM) for use with MongoDB.
  - `node-uuid` Generates Universally-Unique IDentifiers.
  - `q` I believe one of the first implementations for Promises for JavaScript, supplanted by native Promises.
  - `sendgrid` Client library for SendGrid's email service (another free Heroku add-on if you stay below the usage limits).
  - `through2-map` A stream-oriented map utility - applies your function to each object in the stream, then passes it on.
  - `grunt` A task runner, used here to coordinate the build/dev process.
  - `grunt-contrib-clean` A Grunt task to delete files from a folder.
  - `grunt-contrib-jshint` A Grunt task to run the JSHint linter. Apprently this project was before I discovered ESLint.
  - `grunt-contrib-less` A Grunt task to run the Less compiler and generate plain CSS files.
  - `grunt-contrib-requirejs` A Grunt task to run the RequireJS bundler.
- `Gruntfile.js` Configures the build process.
- `Procfile` This tells Heroku (and now me!) how to start the application.
- `web.config` Apprently I set this up to also run in IIS using the IISNode handler.
- `server/` The backend Express application, which I'd like to remove.
- `client/` The frontend Knockout application, which I'd like to massively simplify.

With that information, I have a rough plan of attack:

1. Remove the backend completely.
1. Remove anything from the frontend app that relies on API calls to the backend, starting with the liftshare and gift registry functionality.
1. Flatten what's left of the frontend into a static HTML and CSS site with as little JavaScript as possible and preferably no external dependencies.
1. Deploy as a GitHub pages site.
1. Clean up by deleting the Heroku app and DNS entry.

## Delete the Backend

Looking through the backend code, I find absolutely nothing worth keeping. That was easy.

## Trim the Frontend

The site is structured as 5 pages with a navigation bar shared across all of them. Let's go through them in turn:

- Home: Nothing dynamic here, but we have an image carousel that uses Bootstrap's jQuery plugin. Leave this for now.
- More Details: Nothing dynamic, but even more Bootstrap-jQuery goodness - a side navigation pane that uses Affix for positioning, ScrollSpy to highlight the link for the section currently on screen, and jQuery's animate to smoothly scroll between sections on click. I had forgotten about all this, it looks really quite nice.
- RSVP: This page is essentially just a form to provide info to an API that sends an email to me and a confirmation email to the user, which relies on the backend. There's nothing of sentimental value here, so I'll just delete the page.
- Liftshare: Built around an embedded Google Maps map, this lets users drop a pin with their location (and fill out a form with a few details) to either request or offer a lift to the event. Uses the backend, nothing sentimental here, delete.
- Gifts: Another API-centric page, this one directs people to not give us gifts if they can help it - and also lets them record what gift they are getting us so that other people can check and avoid duplicates. No one used this page. Delete.

Next, I can remove the code used to display the Google Maps map, which was implemented as a Knockout custom binding.

The CSS for the site was originally compiled from Less files. Fortunately, I have a copy of the compiled CSS, so I don't need to install the Less compiler. I'll trim down the CSS to remove styles only needed by the pages I've deleted and remove the Less files.

## Flatten the Frontend

I'm left with just two pages. While they do share some code in the form of the navbar and `<head>` tag, I'm willing to duplicate those shared elements in the name of simplicity - it's not like this site needs to be actively maintained.

Each of the pages gets its own HTML file containing a copy of the shared layout, which I now need to populate with the deconstructed Knockout component templates. This is basically the reverse of the process I would sometimes use to build Knockout components from wireframes; in this case I'm taking Knockout bindings like `if`, `foreach`, `text` and `attr` and simulating them manually to produce the resulting HTML.

To set up the Bootstrap/jQuery plugins for the carousel and side navigation on the details page, I'm taking essentially the code from the Knockout component viewmodel modules, trimming off the RequireJS module wrapper and Knockout-specific setup code, and including the result in a `<script>` tag at the bottom of each page.

While the custom JavaScript has been taken care of, I still need to ensure that jQuery and Bootstrap (both JS and CSS) are loaded and ready for use by that custom code. I'll download those files from a CDN and include them in the codebase to protect myself from the CDN archiving those old versions in the future.

With that, I can delete the remaining Knockout code, as well as all of the files to set up NPM, Bower, Grunt, Heroku, IIS and even the `.gitignore` - there's nothing left to ignore! I'm left with just two HTML files, one CSS file and a folder of images.

## Deploy as GitHub Pages site

Not much to say here - I already have GitHub pages set up for my user site and one other repository site, so it's a matter of a few clicks in the repository settings to add another repository site. In this case, the repository is nothing but the site, so I choose to configure Pages to look at the master branch.

## Clean up

- Delete the Heroku application (including the add-ons such as the MongoDB database).
- Remove the DNS entries pointing at the Heroku application.
- Remove the DNS entries used as domain ownership verification by SendGrid.

## Conclusion

For a simple application that had only a few dynamic features, the process to convert to a static site is straight forward. If the dynamic features had been a central part of the site's purpose, this would be a pointless process, but this particular site had enough static content with enough sentimental value that the conversion was worthwhile.
