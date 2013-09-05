---
title: GroupText
layout: post
---
[GroupText](https://github.com/mrw34/grouptext) is a web application used by a [community project in London](http://gcsesuccess.wordpress.com/) that provides free tutoring to local kids. It enables the project's tutors to contact individuals or groups of students by SMS without having direct access to their mobile phone numbers.

GroupText is written using Meteor and routes all messages and responses through the [Nexmo](https://www.nexmo.com/) platform, copying them via email to senior tutors to provide auditing and transparency. The code attempts to follow best-practice in terms of security, structure, and re-use of existing packages:

* Use of Meteor's new [check package](http://docs.meteor.com/#match) (with [audit-argument-checks](http://docs.meteor.com/#auditargumentchecks)) to validate user input
* Server-side routing (using [Meteor Router](https://github.com/tmeasday/meteor-router) via [Meteorite](https://github.com/oortcloud/meteorite)) to receive SMS callbacks from Nexmo
* Authentication using [accounts-ui](http://docs.meteor.com/#accountsui) with email verification
* Collection filtering for subscriptions from non-admin users, with appropriate allow/deny rules

Things I learnt:

* Be careful not to declare catch-all server-side routes - on reload they are applied before client-side routes, which confused me for a while
* Meteorite and [Bower](http://bower.io/) make life much easier. I recommend [hiding](https://github.com/mrw34/grouptext/blob/master/.bowerrc) your ```bower_components``` folder so that it's ignored by ```meteor bundle```
* Use of the [FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader) API combined with [Meteor methods](http://docs.meteor.com/#methods_header) is a neat solution for file uploading
* The simplest way to use Google Analytics is to just put your ```analytics.js``` file in the ```client``` folder

Meteor makes writing this type of real-time application refreshingly easy!

