---
title: شيفرة تحليلات جوجل على rails
date: 2016-07-14T15:51:15+03:00
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: Google Analytics
category: SEO
---

`app/assets/javascripts/google_analytics.js.coffee`

~~~coffee
class @GoogleAnalytics

  @load: ->
    # Google Analytics depends on a global _gaq array. window is the global scope.
    window._gaq = []
    window._gaq.push ["_setAccount", GoogleAnalytics.analyticsId()]

    # Create a script element and insert it in the DOM
    ga = document.createElement("script")
    ga.type = "text/javascript"
    ga.async = true
    ga.src = ((if "https:" is document.location.protocol then "https://ssl" else "http://www")) + ".google-analytics.com/ga.js"
    firstScript = document.getElementsByTagName("script")[0]
    firstScript.parentNode.insertBefore ga, firstScript

    # If Turbolinks is supported, set up a callback to track pageviews on page:change.
    # If it isn't supported, just track the pageview now.
    if typeof Turbolinks isnt 'undefined' and Turbolinks.supported
      document.addEventListener "page:change", (->
        GoogleAnalytics.trackPageview()
      ), true
    else
      GoogleAnalytics.trackPageview()

  @trackPageview: (url) ->
    unless GoogleAnalytics.isLocalRequest()
      if url
        window._gaq.push ["_trackPageview", url]
      else
        window._gaq.push ["_trackPageview"]
      window._gaq.push ["_trackPageLoadTime"]

  @isLocalRequest: ->
    GoogleAnalytics.documentDomainIncludes "local"

  @documentDomainIncludes: (str) ->
    document.domain.indexOf(str) isnt -1

  @analyticsId: ->
    # your google analytics ID(s) here...
    'UA-XXXXXXX-XX'

GoogleAnalytics.load()
~~~