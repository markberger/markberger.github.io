---
layout: post
title: Introducing Tally
date: 2013-10-17 21:50:14.000000000 -07:00
---
   Over the past few days I have been working on [tally](http://www.github.com/markberger/tally), an IRC
    bot that helps keep track of your open source project. It provides
    a handful of useful features that make collaborating over IRC easier, and
    writing plugins is painless.

   When discussing tickets on IRC, type "#<ticketnum\>" and tally will bring up
    the name of the ticket and a link to the ticket for others to view.

    markberger: Can someone help me out with #1382?
    tally: #1382 (immutable peer selection refactoring and enhancements)
    tally: https://tahoe-lafs.org/trac/tahoe-lafs/ticket/1382

   It will also post information about activity on the ticket tracker:

    tally: Ticket #1057 (Alter mutable files to use servers of happiness) updated by markberger
    tally: https://tahoe-lafs.org/trac/tahoe-lafs/ticket/1057#comment:10

   And if you are running multiple bots, you can set tally to ignore messages that
    contain the name of another user or phrase.

   Currently tally only supports Trac, but I want to eventually add Github support. It is
    written in Go and the source code can be found [on Github](https://github.com/markberger/tally).
    Right now I am running an instance of tally on #tahoe-lafs at freenode and
    everything is working smoothly. You can view the source for tahoe-bot [here](https://github.com/markberger/tahoe-bot).
    If you are using tally and have any issues, please let me know and I will try to address them. Or if you've
    solved an issue yourself, open up a pull request and I will merge the patch.
