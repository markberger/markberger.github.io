---
layout: post
title: Upload Strategy of Happiness
date: 2013-10-17 23:00:03.000000000 -07:00
---
As I mentioned in a [previous post](/blog/file-upload-in-tahoe-lafs), my GSoC project is to implement a new upload algorithm in Tahoe, called the "Upload Strategy of Happiness". This algorithm was developed by [Kevan Carstensen](http://isnotajoke.com/), one of Tahoe's contributors, in order to meet the servers of happiness test by maximizing the distribution of file shares over a network. The relationship between shares and servers is modeled using a bipartite graph, where an edge exists from share n to server s if and only if s stores n.

The algorithm is as follows:

Let N be the set of all shares and S be the set of all servers.

1. Construct a bipartite graph A from the existing allocations on the network such that an edge exists from share n to server s if and only if s stores n.

2. Calculate the maximum matching graph of A using the Edmonds-Karp algorithm and let this graph be A-max.

3. Let N-existing be the set of all shares in A-max and let S-existing be the set of all servers in A-max. Construct the bipartite graph B such that the set of shares used by B is N - N-existing and the set of servers used by B is S - S-existing where an edge from each share in N - N-existing exists to each server in S - S-existing if server s can hold share
     n.

4. Calculate the maximum matching graph of B and let this graph be B-max.

5. Merge A-max and B-max. The product of merging A-max and B-max is the maximum distribution of shares over unique servers on the network.

This technique of calculating the maximum distribution is useful because it utilizes as many existing shares as possible, which reduces the amount of data transferred. Once the shares have been distributed over the maximum number of unique servers, the remaining shares are distributed as evenly as possible by looping over the set of servers until there are no more shares to distribute. Ultimately this will solve edge cases in Tahoe that prevented certain server configurations from accepting new files.

Note: If you are unfamiliar with the maximum matching of bipartite graphs but interested in learning more, I highly recommend chapter 26 of *Introduction to Algorithms*, titled *Maximum Flow*.

Last week I was able to implement the algorithm and start refactoring the current uploader. While the new uploader isn't finished yet, some of the test cases which used to fail now succeed, which is pretty exciting. Hopefully I will be able to clean up my code and work out some of the bugs this week. To view my progress checkout the Github branch [here](http://github.com/markberger/tahoe-lafs).
