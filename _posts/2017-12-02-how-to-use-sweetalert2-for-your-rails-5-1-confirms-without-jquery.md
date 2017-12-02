---
date: 2017-12-02
categories:
- Rails
- SweetAlert2
title: How to use SweetAlert2 for your Rails +5.1 (rails-ujs) confirms without jQuery
author_staff_member: 01_peter
medium_link: https://medium.com/store2be-tech/how-to-use-sweetalert2-for-your-rails-5-1-rails-ujs-confirms-without-jquery-8a5b516b2a1
practical_dev_link: https://dev.to/peterfication/how-to-use-sweetalert2-for-your-rails-51-rails-ujs-confirms-withoutjquery-67h
---

_TL;DR see [this demo project](https://github.com/store2be/rails-sweetalert) for the final solution._

_Please be aware that the examples here are provided with ES6 syntax. [There are different ways to get ES6 to work in Rails](https://stackoverflow.com/a/42464363). The demo project has an [ES5 branch](https://github.com/store2be/rails-sweetalert/tree/master-es5) for reference._

---

If you are reading this I assume you are familiar with [Ruby on Rails](http://rubyonrails.org/) and [SweetAlert2](https://limonte.github.io/sweetalert2/). With Rails before version 5.1, [when rails-ujs was still jquery-ujs](https://blog.bigbinary.com/2017/06/20/rails-5-1-has-dropped-dependency-on-jquery-from-the-default-stack.html), there was an easy way [to hook up SweetAlert (or SweetAlert2)](http://alexzerbach.com/use-sweet-alert-rails/) with [the Rails confirm functionality](https://stackoverflow.com/questions/16668949/how-to-add-confirm-message-with-link-to-ruby-on-rails).

One way to achieve it was to overwrite the confirm handler of Rails:

```javascript
// Override the default confirm dialog of Rails
$.rails.handleConfirm = link => {
  if (link.data('confirm') === undefined){
    return true
  }
  showSweetAlertConfirmationDialog(link)
  return false
}
```

We had this solution in one of our apps and I wanted to use the new `rails-ujs`. My first thought was, that it should be an easy task to adapt. Just change `$.rails.` into `Rails.` and we’re good:

```javascript
// Override the default confirm dialog of Rails
Rails.handleConfirm = link => {
  if (link.data('confirm') === undefined){
    return true
  }
  showSweetAlertConfirmationDialog(link)
  return false
}
```

As it turns out some things have changed.

`Rails.handleConfirm` can be overwritten, but that will not override the event listener that is already attached since `rails-ujs` was initialised. But no problem, let’s just write our own event handler and plug it into the new `rails-ujs` way of doing things. If you have a look at [the source code of the start part of `rails-ujs`](https://github.com/rails/rails/blob/master/actionview/app/assets/javascripts/rails-ujs/start.coffee) you see how event listeners are created. The code to add an event listener for our own method then looks like this:

```javascript
const handleConfirm = link => {
  // Do your thing
}
Rails.delegate(document, 'a[data-confirm-swal]', 'click', handleConfirm)
```

Alright, cool. It works for links with the attribute `data-confirm-swal="Are you sure?"` now 🎉 …__but wait__, if you have a delete link, the confirm dialog never shows up because the method never gets called. 🤔 Turns out the event listener for `method: :delete` is called earlier because it got initialised before our event listener for SweetAlert2. This is because the event listeners of `rails-ujs` are hooked up directly when evaluating the code.

If we want to add our event listener by requiring our Javascript code before `rails-ujs`, then we get into the Problem that `Rails.delegate` is not defined yet. When you require `rails-ujs` this is what’s happening:

* Define event handlers and `Rails.delegate`
* Fire `rails:attachBindings` event on document
* Attach event handlers (`Rails.start`)

So in order to get in between the definition of `Rails.delegate` and the execution of `Rails.start` we have to attach to the `rails:attachBindings` event. (For that to happen we need to require our script *before* `rails-ujs`!)

```javascript
document.addEventListener('rails:attachBindings', element => {
  Rails.delegate(document, 'a[data-confirm-swal]', 'click', handleConfirm)
})
```

🎉 Now everything works as expected 🎉

---

For the final solution [have a look at this demo project](https://github.com/store2be/rails-sweetalert) (with ES5 and ES6 version) or see the code below (only ES6).

```javascript
// This file has to be required before rails-ujs
// To use it change `data-confirm` of your links to `data-confirm-swal`
(function() {
  const handleConfirm = function(element) {
    if (!allowAction(this)) {
      Rails.stopEverything(element)
    }
  }

  const allowAction = element => {
    if (element.getAttribute('data-confirm-swal') === null) {
      return true
    }

    showConfirmationDialog(element)
    return false
  }

  // Display the confirmation dialog
  const showConfirmationDialog = element => {
    const message = element.getAttribute('data-confirm-swal')
    const text = element.getAttribute('data-text')

    swal({
      title: message || 'Are you sure?',
      text: text || '',
      type: 'warning',
      showCancelButton: true,
      confirmButtonText: 'Yes',
      cancelButtonText: 'Cancel',
    }).then(result => confirmed(element, result))
  }

  const confirmed = (element, result) => {
    if (result.value) {
      // User clicked confirm button
      element.removeAttribute('data-confirm-swal')
      element.click()
    }
  }

  // Hook the event before the other rails events so it works togeter
  // with `method: :delete`.
  // See https://github.com/rails/rails/blob/master/actionview/app/assets/javascripts/rails-ujs/start.coffee#L69
  document.addEventListener('rails:attachBindings', element => {
    Rails.delegate(document, 'a[data-confirm-swal]', 'click', handleConfirm)
  })

}).call(this)
```

---

To find this all out it took me some hours as I wasn’t familiar with the codebase of `rails-ujs`. But I learnt a lot along the way. Hopefully with this writeup I can help some other developers as well who want to use the latest version of Rails with SweetAlert2 and without jQuery.

---

If you use [Webpacker](https://github.com/rails/webpacker), there is an easy way to get in between the `rails-ujs` code and the start script (not tested yet):

```javascript
import Rails from 'rails-ujs';

const handleConfirm = () => {
  // Do your thing
}

// Add event listener before the other Rails event listeners like the one
// for `method: :delete`
Rails.delegate(document, 'a[data-confirm-swal]', 'click', handleConfirm)

Rails.start()
```
