# Toot-Tweet v01

Trying out a NodeRED flow to read a Mastodon RSS feed and 'tweet' the latest 'toot'.

## Story

I've almost given up on Social Media and, so far as Facebook is concerned, I'm 18 months clean! But, admittedly, I do still make some use of Twitter to follow select individuals, companies and podcasts, and I'd prefer to keep up appearances with some authentic looking activity on my timeline. Cue the idea to cross-post activity from Mastodon (which I do like) onto Twitter (which has long since gone to hell in a handbasket).

There are several tools that will automate this, a couple of examples being:

* [IFTTT](https://ifttt.com/)  
* [Zapier](https://zapier.com/)

IFTTT is nice and easy, but I found it tempramental and subject to slow rate limits. It will cross-post your content, but it's limited to posting text and a link back to the source. Zapier is a useful tool, far superior to IFTTT, but £23/month is not a viable "personal" level subscription, and their free tier is too limited.

What I wanted was this:

* Cross-post a 'toot' onto Twitter, more or less in realtime  
* Embed any attached image in the tweet (don't just link to it)  
* But, do append a link back to the source  

All fairly simple, but it does require basic logic to deal with posts that do (or don't) include an image and, because there will always be an appended permalink, longer toots will need to be truncated to allow for that.

Sounds like a job for NodeRED... my low-code tool of choice! :-)

-----

### Twitter API

* [Info](https://developer.twitter.com/en/support/twitter-api)  

An API key is required.

### Mastodon

* [Mastodon](https://joinmastodon.org/)  
* [https://fosstodon.org/@mootParadox](https://fosstodon.org/@mootParadox)  
* [https://fosstodon.org/@mootParadox.rss](https://fosstodon.org/@mootParadox.rss)  

Mastodon is an ethical, federated, community-based social media platform. There's an API, but for this purpose the RSS feed provides sufficiant info, and it's freely accessible.

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/lastitem.png)

I'll be using:

* GUID (provides a permalink to source toot)  
* Payload/Description (will become the body of new Twitter status)  
* Media/Enclosure (image attachment)  

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/lastitem-fields.png)

-----

### NodeRED Flow

Initial test v01:

* Read RSS feed and get the latest toot
* Is it new?
* Does it contain an embedded image?
* Sanitise and format the body of the status
* Tweet it (with image, if required) and append a permalink
* Store the latest GUID - see, 'is it new?' above
* Repeat at fixed interval

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/toot-tweet-01.png)

#### Get Latest Toot

```javascript
//set array position, typically '0' for 'latest' toot
let arrPos = 0;

//get latest item from rss feed: msg.payload.rss.channel[0].item[0]
msg.lastitem = msg.payload.rss.channel[0].item[arrPos];

//get permalink to original toot
msg.guid = msg.lastitem.guid[0]._;

//if there's an attached image, get the link: msg.lastitem.enclosure[0].$.url
if ( typeof msg.lastitem.enclosure !== 'undefined') {
  msg.url = msg.lastitem.enclosure[0].$.url;
} else {
  msg.url = false;
}

//get source 'description' of toot: msg.lastitem.description[0]
msg.rawtext = msg.lastitem.description[0];

return msg;
```
^- *only picking up **first** image...*

#### Build Tweet Status Update

```javascript
//if there's an image, write it to 'msg.media'
//otherwise, leave it undefined so that it will be ignored
if ( msg.img == 'yes' ) { msg.media = msg.payload; }

//sanitise HTML source text ready for tweet
let text = msg.rawtext;
let santext = text.replace('<br />','\n').
            replace(/<(.|\n)*?>/g, '').
            replace(/&apos;/g,'\'').
            replace(/&quot;/g,'\"').
            replace(/&amp;/g,'\&').
            replace(/&lt;/g,'\<').
            replace(/&gt;/g,'\>');

//truncate to first 240 chars (if required) leaving space for permalink
//trim any trailing spaces (particularly before appending ellipsis, if required)
let text240 = santext.length > 240 ? santext.substring(0, 240).trim() + '...' : santext.trim();

//append Mastodon permalink to sanitised text
msg.payload = text240+'\n\nSource -> '+msg.guid;

//sending to Twitter node:
// msg.payload = status
// msg.guid = permalink appended to status
// msg.media = attached image

return msg;
```
^- *replacing some common HTML entities with 'escaped' versions...*

#### Example v01

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/toot-flow-tweet-v01.png)
