# Toot-Tweet

Testing a NodeRED flow to read a Mastodon RSS feed and 'tweet' the latest 'toot'.

### Twitter API

* [Info](https://developer.twitter.com/en/support/twitter-api)  

An API key is required.

### Mastodon

* [Mastodon](https://joinmastodon.org/)  
* [https://fosstodon.org/@mootParadox](https://fosstodon.org/@mootParadox)  
* [https://fosstodon.org/@mootParadox.rss](https://fosstodon.org/@mootParadox.rss)  

Mastodon is an ethical, federated, community-based social media platform. For this purpose, the RSS feed provides sufficiant info, and it's freely accessible.

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/lastitem.png)

* GUID (permalink to Mastodon toot)
* Payload (body of Twitter status)
* Media (image attachment)

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/lastitem-fields.png)

-----

### NodeRED Flow

Initial test:

* Read RSS feed and get latest toot
* Is it new?
* Is there an embedded image?
* Sanitise the body of the status
* Tweet it
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
^- *only picking up first image, if there are several...*

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
// mdg.guid = permalink appended to status
// msg.media = attached image

return msg;
```
^- *replacing some common HTML entities with 'escaped' versions...*
