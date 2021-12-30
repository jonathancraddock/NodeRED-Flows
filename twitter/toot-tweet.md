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

* Read RSS feed and get latest tweet
* Is it new?
* Is there an embedded image?
* Sanitise the body of the status
* Tweet it
* Store the latest GUID - see, 'is it new' above
* Repeat at fixed interval

![](https://github.com/jonathancraddock/Notes-on-NodeRED/blob/master/twitter/img/toot-tweet-01.png)
