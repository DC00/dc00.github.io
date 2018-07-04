---
layout: post
title: "Artist Engagement on Genius.com"
categories: music web scraping
---





Design

Paginate through the verified artist pages, extracting the url and iq for each artist.
Go to the artist profile page to scrape the follower and annotation counts. The annotation count is tricky because there is no element with a hard number. Had to use a continuous scroll technique to count the annotations.

Ran into a lot of errors from page load, element visibility, and processing issues. Fixed these by adding time delays, smart select statements, and most importantly batching the asynchronous calls.

S/o to this guy for this good starter on batching with puppeteer: [CITE]

Had to batch the annotation fetching because otherwise it would crash trying to do 30 at a time.
Broke data element into batches of 2 and waited for those to resolve before moving on.

Originally used cheerio to find all of the annotation elements on each scrollToBottom call. This was super slow because cheerio has to load the entire html tree which got worse the more annotation an artist had. Improved the performance of this method by utilizing the format of the element selector: .... nth-child(8). Tested if n annotations existed by substituting a number for a hypothetical selector. If that returned an element then there were at least n annotations and I could continue scrolling.


Other Notes:

Running puppeteer in headless mode gets rid of some of the inconsistent element loads
