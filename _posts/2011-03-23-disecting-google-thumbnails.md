---
layout: post
title: Dissecting Google thumbnails
---

Today I had an itch to look into how Google generates the preview thumbnails when you do a search.

<img src="http://i.imgur.com/NhS6D.png"/><!-- more -->

## Google results page

The first thing I discovered is that the thumbnails are only loaded the first time you click a magnifying glass <img src="http://i.imgur.com/vkShs.png"/>. In the case where you access a previous query, the thumbnails are loaded from the cache once the results are rendered to the screen.

## JSONP /webpagethumbnail request

After your first click the magnifying glass, 10 JSONP calls (1 per search result) are made to `http://clients1.google.com/webpagethumbnail`.

An example request for my search query `site:reddit.com programming`:

    c=11
    r=2
    f=2
    s=300:585
    hl=en
    gl=ca

    query=programming
    d=http://www.reddit.com/programming
    b=1
    j=google.vs.r
    a=IFs

A few values are hardcoded in the page's HTML (before the search results are even loaded), namely the thumbnail size `s` and locale values `hl` and `gl`:

`"kfeUrlPrefix":"/webpagethumbnail?c=11&r=2&f=2&s=300:585&query=&hl=en&gl=ca"`

The next values are what interest me though:

- `query` only contains the keyword I searched and not my entire query `site:reddit.com programming`. I find this particularly interesting as this "slicing" logic seems to be done client-side.
- `d` contains the full URL of the given result item.
- `j` contains the JSONP callback function
- `a` contains a 3 character checksum to prevent 3rd party requests (from what I concluded) that is obtained with the results HTML (in this case `IFs`):

<img src="http://i.imgur.com/UXiE2.png"/ >

## JSONP /webpagethumbnail response

The thumbnails are sent with a expiry time of 1 day from a server running `snapshot_btfe` (likely the codename of their thumbnail server). No surprise there.

The structure of the returned JSON goes as follows:

    {
      "s": "b",
      "b": 1,
      "dim": [302, 585],
      "ssegs": [ "data:image/jpeg...", "data:image/jpeg..." ],
      "ssegs-heights: [405, 180],
      "tbts": [ ... ],
      "url": "http://www.reddit.com/r/programming"
    }

- `dim` contains the total width and height of the thumbnails
- `ssegs` contains an array of strings each composed of a data uri with a segment of the thumbnail
- `ssegs-heights` contains the height of each segment
- `tbts` contains an array of text that will be overlayed on top of the thumbnails
- `url` contains the url of the requested page

At this time I am unsure what `s` and `b` are used for.

## Building the thumbnail

The thumbnail appears to be split into segments when the height is greater than 405 pixels. I'm guessing this is either for performance reasons or compatibility ([IE8 supports max 32KB data URIs](http://en.wikipedia.org/wiki/Data_URI_scheme#Disadvantages))?

<img src="http://i.imgur.com/7AalF.jpg" /> <img src="http://i.imgur.com/kyCnx.jpg" />

Both segments are simply appended one after the other in the preview bubble.

## Building the overlay text

As I previously explained the overlay text data is contained within the JSON in the `tbts` array.

Each text overlay has an entry in the array with the following values:

- `box` contains the dimension and position (top, left) of the thumbnail highlight
- `txt` contains the HTML text that is displayed in the overlay
- `txtBox` contains the dimensions and position of the text box

For example:

    {
      "box": {
        "h": 10,
        "l": 211,
        "t": 71,
        "w": 74
      },
      "txt": "A reddit for discussion and news about computer <em>programming</em> <b>...</b>",
      "txtBox": {
        "h": 42,
        "l": 0,
        "t": 25,
        "w": 300
      }
    }

A `div` is then appended for each box and textbox into the preview bubble which gives the end result:

<img src="http://i.imgur.com/f4bvN.png" />

## Unanswered questions

Unfortunately there are many unanswered questions. I would really be greatful if someone at Google made an official post about how the thumbnails work.

Specifically:

- How are the thumbnail images generated? I'm guessing they are using a headless version of Chrome?
- How are the position of the boxes calculated?
- What kind of infrastructure is behind the thumbnail service?
- What is the ratio of pages that are currently thumbnailed?
- Will there ever be an open API?

-Christian Joudrey
