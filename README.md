# Example Sinatra, Bootstrap, SASS and Compass site

https://github.com/bergcloud/example-sinatra-site

I recently made two new websites which had to be done fairly quickly (but still look good), be simple technically, be responsive, and be dynamic. There are many ways to do this but I settled on using [Sinatra](http://www.sinatrarb.com/), [Bootstrap](http://getbootstrap.com/), [SASS](http://sass-lang.com/) and [Compass](http://compass-style.org/), and this is how I stitched that together. This isn't groundbreaking but might be useful for someone who wants to do something similar without piecing together parts and conventions.

You can see a skeleton of the project [on GitHub](https://github.com/bergcloud/demo-sinatra-site). Once you've got a copy locally, `cd` into `example-sinatra-site/` and then for development you'll want to run these in separate windows:

    $ compass watch .
    $ rackup

The first makes Compass watch the SASS files (in `public/sass/`) for changes, at which point it compiles new CSS files (into `public/css/`). The second runs the Sinatra webserver, so visiting [http://0.0.0.0:9292/](http://0.0.0.0:9292) should get you "Hello World".

Let's look at each part.

## Sinatra

If I was making a quick static site, I'd probably use [Jekyll](http://jekyllrb.com/) but usually there's at least one thing that needs some server-side processing. So, this being a Ruby place, I used [Sinatra](http://www.sinatrarb.com/).

Some notes on the structure and how it works:

  * There are two layouts in `views/_layouts/` as examples of using different general page structures.

  * Each layout uses the same header and footer, both in `views/_partials/`.

  * Any changes made to views or `app.rb` should automatically take effect when you reload a page, thanks to [Sinatra::Reloader](http://www.sinatrarb.com/contrib/reloader.html).

  * `app.rb` contains the config, helper functions and routes for this example site. Delete or expand these as you need.

    * There's a commented-out section that will let you add username/password authentication for a particular environment.

    * Add a Google Analytics ID and the `views/_partials/footer.erb` template will render the required JavaScript.

    * There are two routes set up for `/` and `/special/`. The second is an example of a page that has some unique (stupidly simple) server-side stuff happening.

    * There's a catch-all route for URLs that do nothing but render a view. Add new paths and page details to the hash, and then add matching views, to get them working. For example, if you want `/projects/demo/` to work as a URL, add this to the `pages` hash:

            'projects/demo' => {
              :page_name => 'projects_demo',
              :title => 'Demo project',
              :layout => 'with_sidebar',
            },

And create a new view at `views/projects_demo.erb` containing the main content for that page.

The `:layout` is optional. If not set, the `views/_layouts/_with_sidebar.erb` layout will be used.


## Bootstrap

I feel defensive using Bootstrap's CSS [because it's so common](http://notes.gross.is/post/43508972396/please-stop-using-twitter-bootstrap) these days. I used it because: the responsive grid structure seems to work well; the default styles for forms, tables etc are nicer than the usual browser defaults; we'd be restyling it from the default look; and we've used it for sites before so there were people here already familiar with it.

We don't want to include all of Bootstrap's CSS and JS files, but only the bits we need. One way of doing this is to use the [customise](http://getbootstrap.com/customize/) tool to download your own version, although that becomes a pain if you want to keep tweaking things.

(Top tip if you do use that tool though: when you download your files you'll get a `config.json` file. Paste the contents into a [Gist](https://gist.github.com/). Take the ID number from the Gist's URL and add it to the URL of the Bootstrap customizer. eg, from `https://gist.github.com/anonymous/9325216` you get `http://getbootstrap.com/customize/?id=9325216`. The customizer then loads your settings. Nice.)

### Bootstrap's CSS

We're using [Bootstrap-SASS](https://github.com/twbs/bootstrap-sass) (a mirror of the standard version which is based on [Less](http://lesscss.org/)). I haven't taken full advantage of [SASS's](http://sass-lang.com/) powers, but it lets us:

  * Easily add or remove Bootstrap components.
  * Override some of the default Bootstrap colours, sizes, etc.
  * Re-use common bits of CSS and set variables.

Edit CSS in the SASS files (in `public/sass/` with `.scss` extensions). These are then compiled by Compass into one or more CSS files in `public/css/`. Any SASS file whose name starts with an underscore is _not_ compiled to a CSS file, so this is used to name included files.

Here's our SASS file structure:

#### `styles.scss`

This will be compiled to `styles.css` and used on the site. Styles in here should be very specific to our site. You could, instead, have several files like this, for different sections of the site, containing section-specific CSS. This file imports…

#### `_base.scss`

This sets any common, generic styles and variables and imports…

#### `_bootstrap_custom.scss`

This is a copy of the official [`bootstrap.scss`](https://github.com/twbs/bootstrap-sass/blob/master/vendor/assets/stylesheets/bootstrap/bootstrap.scss) with the `@import` paths tweaked and any parts of Bootstrap we don't need commented out. SASS then only includes the parts we need in the generated CSS file(s), keeping it smaller. This file also imports…

#### `_bootstrap_variables.scss`

The official [`_variables.scss`](https://github.com/twbs/bootstrap-sass/blob/master/vendor/assets/stylesheets/bootstrap/_variables.scss) file contains all the default Bootstrap values. In our local version we set any values that we want to change.

For example, the original `_variables.scss` has `$body-bg: #fff !default;`. If we want a different background color, we'd put something like `$body-bg: #eee;` in our local `_bootstrap_variables.scss`.

.

The upshot of all those files is that we can choose which components of Bootstrap to use (minimising our CSS file size), set default values that Bootstrap uses to build its CSS, and set our own styles afterwards. This lets us override any Bootstrap styles we want in `_base.scss` or `styles.scss`.

Alternatively, if you want to add a Bootstrap style to a different element, SASS makes that easy. eg, to make WordPress's comment `textarea` look like a Bootstrap form element you need to add the `form-control` class to it. If you don't want to touch the HTML you can add the `form-control` class's styles to it:

    .comment-form textarea {
      @extend .form-control;
    }

This applies all of `.form-control`'s styles to that `textarea`.

### Bootstrap's JavaScript

If you need any of Bootstrap's JavaScript you'll either need to use the [customizer](http://getbootstrap.com/customize/) to download what you need, or (as in this example) just use the complete JavaScript file. Either way, it goes in `public/js/`. Don't forget to enable the bits of Bootstrap CSS (in `_bootstrap_custom.scss`) each JS component requires.

## Compass

I find [Compass's documentation](http://compass-style.org/) baffling; I can never find anything I need. But we're only using Compass for its ability to watch some SASS files and compile them to CSS when something changes.

`config.rb` contains the configuration for Compass, telling it where to find and put things, and what format to generate CSS in. You might not need to touch this, but [here's the Compass configuration reference](http://compass-style.org/help/tutorials/configuration-reference/).

## HTML

`views/` contains the HTML templates. You'll need to change stuff here, but it's enough to display something for our quick example.

