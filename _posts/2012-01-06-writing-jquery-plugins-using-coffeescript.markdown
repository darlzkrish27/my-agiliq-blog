---
layout: post
title:  "Writing jQuery plugins using Coffeescript"
date:   2012-01-06 11:54:11+05:30
tags: coffeescript
author: shabda
---
So you want to write a Jquery plugin. If you know jQuery and Coffeescript, this would be amazingly easy.

I will walk you through writing a jQuery plugin which will allow us to add alternating colors to alternating rows.

Here is the plugin in its entirety.

    $ = jQuery
    $.fn.zebraTable = (options) ->
        defaults = 
            evenColor: '#ccc'
            oddColor : '#eee'

        options = $.extend(defaults, options)
        @each ->
            $("tr:even", this).css('background-color', options.evenColor)
            $("tr:odd" , this).css('background-color', options.oddColor)


Lets look at what we did.

1. We bound $ to jQuery object.
2. We created an anonymous functions and added this to jQuery, by assigning it to ` $.fn.zebraTable`
3. After this we will be able to do `$("selector").zebraTable()` or `$(selector).zerbraTable(optionsDict)
4. This function will set background-color on alternating rows.

Lets look at the compiled Javascript output

    (function() {
      var $;

      $ = jQuery;

      $.fn.zebraTable = function(options) {
        var defaults;
        defaults = {
          evenColor: '#ccc',
          oddColor: '#eee'
        };
        options = $.extend(defaults, options);
        return this.each(function() {
          $("tr:even", this).css('background-color', options.evenColor);
          return $("tr:odd", this).css('background-color', options.oddColor);
        });
      };

    }).call(this);
    
And you can call it like this.

    $(function() {
      $("table").zebraTable(  {
          evenColor: 'red',
          oddColor: 'yellow'
        });
    });

Salient features.

1. Since Coffeescript will enclose the code in a function `$ = jQuery` doesn't overwrite everything, and there is no need to enclose the plugin in its own closure.
2. Inside the plugin `this` refers to the selected jQuery object, so there is no need to do `$(this)`.

----
You can see the whole code on [github here](https://github.com/shabda/coffeescript-jquery-plugin-example)

