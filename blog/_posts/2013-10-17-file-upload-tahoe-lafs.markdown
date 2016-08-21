---
layout: post
title: File Upload in Tahoe-LAFS
date: 2013-10-17 22:47:26.000000000 -07:00
---
### Introduction

In conjunction with the Google's Summer of Code (GSoC) program, I am improving Tahoe-LAFS on behalf of the Python Software Foundation. One of my responsibilities as an intern for the PSF is to blog about my project semiweekly. This post briefly explains Tahoe, and describes what I will be working on during the summer.

### About Tahoe-LAFS

Tahoe-LAFS is an open source cloud-based storage system which distributes user data over multiple servers, similar to Dropbox and other cloud-based storage services. However, unlike other services, Tahoe is built with provider-independent security. All user data is encrypted in such a way that the service provider cannot access it, guaranteeing that the user's privacy is respected. For more information on what Tahoe-LAFS is and how it works, check out the about page on the Tahoe-LAFS website [here](https://tahoe-lafs.org/trac/tahoe-lafs).

### Servers of Happiness

To accomplish data redundancy, Tahoe uses [erasure encoding](http://en.wikipedia.org/wiki/Erasure_code) to split the file into n pieces where only k pieces are required to reassemble the file. By default, Tahoe sets the encoding parameters to be n = 10 and k = 3 so that any file can be reassembled with three pieces even though ten pieces exist on the network.

While erasure encoding is great for redundancy purposes, it can be misleading. If all ten pieces of a given file are on the same server, data redundancy hasn't increased significantly. In the situation of server failure, the data is no better off in ten pieces compared to a single file on the server. Erasure encoding has only increased the size of the file without adding any system wide benefits.

To mitigate this situation, Tahoe uses a criteria called "Servers of Happiness", which introduces a new parameter h. Tahoe ensures that a subset of n shares are placed on h distinct servers, therefore guaranteeing increased redundancy. However, Servers of Happiness is only a test, and the current upload algorithm was not built to pass the new test. Therefore, problems arise in which the Servers of Happiness test can be met, but the uploader does not know how to do so, leading to upload failure. My project aims to fix this issue, and other issues which arise from the outdated uploader, by implemented an "Upload Strategy of Happiness" which knows how to meet the Servers of Happiness criteria. This will increase overall data redundancy, as well as remove inefficiencies in the uploader.

Today is the first day of programming in GSoC and I've just started to refactor the uploader to implement the Upload Strategy of Happiness. In my next post I hope to explain the new strategy I will be implementing, and discuss any progress I have made this week. To follow my progress subscribe to this blog, or follow me on Github and checkout the project branch [here](http://github.com/markberger/tahoe-lafs).
