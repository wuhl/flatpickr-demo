# Flatpickr-Demo - Rails + webpacker + bootstrap + stimulus + flatpickr
Steps to create a rails app based on Ruby 2.5.1, Rails 5.2.0, Webpacker, Bootstrap, Stimulus and Flatpickr

Based on https://andyyou.github.io/2018/05/02/rails-5-webpacker-stimulus-bootstrap-starter/
and https://github.com/adrienpoly/stimulus-flatpickr

## 1. Check Ruby and Rails version
```console
$ ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-darwin17]
$ rails -v
Rails 5.2.0
```
## 2. Create new app with webpack and stimulus
```console
$ rails new flatpickr-demo --webpack=stimulus --skip-coffee --skip-test
...
$ cd flatpickr-demo
$ rails db:create
...
```
## 3. Configure scss architecture controller
```console
$ mkdir app/javascript/stylesheets
$ touch app/javascript/stylesheets/application.scss
$ touch app/javascript/stylesheets/_variables.scss
$ touch app/javascript/stylesheets/_base.scss
```
```scss
// app/javascript/stylesheets/application.scss
@import 'variables';
@import 'base';
```
```scss
// app/javascript/stylesheets/_variables.scss
$colors: (
  major: #00D252,
  minor: #2F3B59
);
```
```scss
// app/javascript/stylesheets/_base.scss
h1 {
  color: map-get($colors, major);
}
```
```js
// app/javascript/packs/application.js
import 'stylesheets/application'
```
## 5. Add 1. Stimulus controller root
```js
// app/javascript/controllers/clipboard_controller.js
import { Controller } from 'stimulus'
export default class extends Controller {
  static targets = ['source']
  initialize() {
    console.log('clipboard initialize')
  }
  connect() {
    console.log('clipboard connect')
    if (document.queryCommandSupported('copy')) {
      this.element.classList.add('clipboard--supported')
    }
  }
  copy(e) {
    e.preventDefault()
    this.sourceTarget.select()
    document.execCommand('copy')
  }
}
```
## 6. Create Controller with first page
```console
$ rails g controller pages example
...
```
```erb
<%# /app/views/pages/example.html.erb %>
<h1>Hello, World</h1>
<hr>
<div data-controller="clipboard members dashboard">
  PIN
  <input type="text" data-target="clipboard.source" value="1234" readonly>
  <button data-action="clipboard#copy" class="clipboard-button">
    Copy to Clipboard
  </button>
</div>
```
## 7. Add pack to layout
```erb
<%# /app/views/layout/application.html.erb %>
<!DOCTYPE html>
<html>
  <head>
    <title>FlatpickrDemo</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_pack_tag 'application' %>
    <%= javascript_pack_tag 'application' %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```
## 8. Add route
```ruby
# config/routes.rb
# tip: at the top to be matched first
...
Rails.application.routes.draw do
  root to: 'pages#example'
end
```
## 9. Install bootstrap 
```console
$ yarn add jquery popper.js bootstrap
...
```
## 8. Import bootstrap stylesheets
```scss
// app/javascript/stylesheets/application.scss
@import '~bootstrap/scss/bootstrap';
@import 'variables';
@import 'base';
```
## 9. Import bootstrap javascript
```js
// app/javascript/packs/application.js
import 'bootstrap'
```
## 10. Configure webpacker
Add configuration to config/webpack/environment.js. If you do not setup this step, the abilities related to Popper.js such as tooltip will not working.
```js
// config/webpack/environment.js
const { environment } = require('@rails/webpacker')
const webpack = require('webpack')
/**
 * Automatically load modules instead of having to import or require them everywhere.
 * Support by webpack. To get more information:
 *
 * https://webpack.js.org/plugins/provide-plugin/
 * http://j.mp/2JzG1Dm
 */
environment.plugins.prepend(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    jquery: 'jquery',
    'window.jQuery': 'jquery',
    Popper: ['popper.js', 'default']
  })
)
module.exports = environment
```
## 11. Expose jQuery to global views
Sometimes you may like to use jQuery in views you should expose jQuery to global.
```console
# https://webpack.js.org/loaders/expose-loader/
$ yarn add expose-loader -D
```
Add configuration to config/webpack/environment.js
```js
// config/webpack/environment.js
const { environment } = require('@rails/webpacker')
const webpack = require('webpack')
/**
 * Automatically load modules instead of having to import or require them everywhere.
 * Support by webpack. To get more information:
 *
 * https://webpack.js.org/plugins/provide-plugin/
 * http://j.mp/2JzG1Dm
 */
environment.plugins.prepend(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    jquery: 'jquery',
    'window.jQuery': 'jquery',
    Popper: ['popper.js', 'default']
  })
)

/**
 * To use jQuery in views
 */
environment.loaders.append('expose', {
  test: require.resolve('jquery'),
  use: [{
    loader: 'expose-loader',
    options: '$'
  }]
})

module.exports = environment
```
## 12. Stimulas is installed and running now let's add flatpickr
```console
$ yarn add stimulus-flatpickr flatpickr
```
## 13. Basic usage
If you only need to convert an input field in a DateTime picker, you just need to register a standard Stimulus controller and add some markup to your input field.

Register a Flatpickr Controller

manually register a new stimulus controller in your main js entry point in app/javascript/packs/application.js:
```js
/* eslint no-console:0 */
// This file is automatically compiled by Webpack, along with any other files
// present in this directory. You're encouraged to place your actual application logic in
// a relevant structure within app/javascript and only use these pack files to reference
// that code so it'll be compiled.
//
// To reference this file, add <%= javascript_pack_tag 'application' %> to the appropriate
// layout file, like app/views/layouts/application.html.erb

console.log('Hello World from Webpacker')

import 'bootstrap'
import 'stylesheets/application'

import { Application } from "stimulus"
import { definitionsFromContext } from "stimulus/webpack-helpers"

const application = Application.start()
const context = require.context("controllers", true, /.js$/)
application.load(definitionsFromContext(context))

// Manually register Flatpickr as a stimulus controller
application.register("flatpickr", Flatpickr);
```
## 15. Using it with rails
```erb
<%# /app/views/pages/example.html.erb %>
<h1>Hello, World</h1>
<hr>
<div data-controller="clipboard members dashboard">
  PIN
  <input type="text" data-target="clipboard.source" value="1234" readonly>
  <button data-action="clipboard#copy" class="clipboard-button">
    Copy to Clipboard
  </button>
</div>

<%= form_tag(root_path, method: :get) do %>
  <%= text_field :start_time, params[:start_time], 
    data: {
      controller: "flatpickr",
      flatpickr_enable_time: false,
      flatpickr_max_date: Time.zone.now + 3.days
    } %>
    <%= submit_tag 'Search' %>
<% end %>
```
## 16. Advanced usage
If you need more than just displaying the standard DateTime picker, then you can extend the stimulus-flatpickrwrapper controller. This is necessary when you need to:

set a custom language
create customs callbacks
perform JS business logic
Skip basics installation steps from above!

Extends the controller

create a new Stimulus controller that will inherit from stimulus-flatpickr
```js
// ./controllers/flatpickr_controller.js
// import stimulus-flatpickr wrapper controller to extend it
import Flatpickr from "stimulus-flatpickr";

// you can also import a translation file
import { German } from "flatpickr/dist/l10n/de.js";

// import a theme (could be in your main CSS entry too...)
import "flatpickr/dist/themes/dark.css";

// create a new Stimulus controller by extending stimulus-flatpickr wrapper controller
export default class extends Flatpickr {
  initialize() {
    // sets your language (you can also set some global setting for all time pickers)
    this.config = {
      locale: German
    };
  }

  // all flatpickr hooks are available as callbacks in your Stimulus controller
  change(selectedDates, dateStr, instance) {
    console.log("the callback returns the selected dates", selectedDates);
    console.log("but returns it also as a string", dateStr);
    console.log("and the flatpickr instance", instance);
  }
}
```