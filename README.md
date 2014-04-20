# Ask Cerebro

> DEPRECATED: askcerebro will shut down on 04/26/14.

## What ?

Cerebro is an API-like endpoint that returns information about a package.

Releases type, versions and urls are returned as a JSON so it can be used for rolling upgrades of common software packages.

It's aimed for people who like to compile from sources as it always returns the source.tar.gz of packages.

You feed it `disks` and it tells you everything about them.



## Why ?

I got tired having to check for the latest version of Nginx, Redis, etc and switching variables in Chef, Fabric, Capistrano and wherever you can think. 

For example, looking at the [Logstash cookbook](https://github.com/lusis/chef-logstash) from Lusis and contributors:

	node['logstash']['agent']['version'] - The version of Logstash to install. Only applies to jar install method.
	
Logstash agent `['version']` can now be easily updated when a new release is available.


I needed something easy, fast, easy to maintain and was using it personally with restricted IPs. A friend asked me if he could used it as well, so I just removed all IPs restrictions and put it on Github.



## Where ?

Just make a call to `askcerebro.com/package` where `package` is the package you want to get information about.


## How ?

#### JSON API

Here is an example:

	$> curl -i www.askcerebro.com/redis
	
Answer should be 200 OK if found with a **JSON** answer:


	HTTP/1.1 200 OK
	Date: Tue, 12 Mar 2013 18:50:59 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

	{
	  "redis": {
	    "releases": [
	      "stable",
	      "legacy"
	    ],
	    "stable": {
	      "md5": "c4be422013905c64af18b1ef140de21f",
	      "sha256": "3b9439636c58ca06bee538a0f7298e02a33fcf98b8fa845c0b0cf8567751e948",
	      "url": "http://redis.googlecode.com/files/redis-2.6.13.tar.gz",
	      "version": "2.6.13"
	    },
	    "legacy": {
	      "md5": "13b9955a924be15b5fe67f970a5386b5",
	      "sha256": "d71b6372f42fcbdc77a9601f1dd6a029ed57f7f77ac3b18bfed8670fb8c74697",
	      "url": "http://redis.googlecode.com/files/redis-2.4.18.tar.gz",
	      "version": "2.4.18"
	    }
	  }
	}
	
If a package is not available in Cerebro (for example `perl`), you'll get a 404 HTTP NOT FOUND with an "NA" answer:

	$> curl -i www.askcerebro.com/perl
		
	HTTP/1.1 404 Not Found
	Date: Tue, 12 Mar 2013 18:51:35 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

	NA
	
If you're trying to hit an endpoint unknown to Cerebro, you'll get a **404 HTTP NOT FOUND** answer:

	$> curl -i www.askcerebro.com/shady/endpoint 
	
	HTTP/1.1 404 Not Found
	Date: Tue, 12 Mar 2013 18:51:35 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive
	
If Cerebro is **down**, you'll get a **503 Service Temporarily Unavailable**:

	$> curl -i www.askcerebro.com/down
	
	HTTP/1.1 503 Service Temporarily Unavailable
	Date: Wed, 04 Dec 2013 23:41:28 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

#### HUMAN API

You can directly get the **plain text** answer about a package, depending of the release type.

Currently only `version`, `url`, `md5` and `sha256` are supported.

Here is an example:


	$> curl -i www.askcerebro.com/mongodb/nightly/url
	
	HTTP/1.1 200 OK
	Date: Tue, 12 Mar 2013 19:02:25 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

	http://github.com/mongodb/mongo/tarball/v2.2-latest
	

And another:

	curl -i www.askcerebro.com/redis/legacy/version
	
	HTTP/1.1 200 OK
	Date: Tue, 12 Mar 2013 19:03:35 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

	2.4.18
	
	

## How to add a package to Cerebro ?


Just add another `disk` with the following YAML structure:


	new_package:
  		releases: ['stable', 'development', 'legacy']
  		stable:
    		version: 1.2.7
   			url: http://example.com/download/stable-1.2.7.tar.gz
   			md5: abcdefghijkl124345
   			sha256: d71b6372f42fcbdc77a9601f1dd6a029ed57f7f77ac3b18bfed8670fb8c74697
  		development:
    		version: 1.3.14
    		url: http://github.example.com/download/latest.tar.gz
    		md5: abcdefghijkl124345
    		sha256: d71b6372f42fcbdc77a9601f1dd6a029ed57f7f77ac3b18bfed8670fb8c74697
  		legacy:
    		version: 1.0.15
    		url: http://example.com/download/legacy-1.0.15.tar.gz
    		md5: abcdefghijkl124345
    		sha256: d71b6372f42fcbdc77a9601f1dd6a029ed57f7f77ac3b18bfed8670fb8c74697
    		
Once I merge your disk, you should be able to ask Cerebro about it within couple mins.


## How to use it with Chef ?

You could technically use it for rolling upgrades (be careful), with something like this (YMMV):

```ruby
require "net/http"
require "uri"
require "json"
require 'chef/log'

begin
  redis_info = URI.parse("http://www.askcerebro.com/redis")
  info = JSON.parse(Net::HTTP.get_response(redis_info).body)
  redis_version = info['redis']['stable']['version']
  node.default[:redis][:server][:version] = redis_version.chomp
rescue => e
  Chef::Log.info("Cerebro is down, error is #{e}")
  Chef::Log.info("Falling back to default attribute version.")
  node.default[:redis][:server][:version] = "2.6.10"
end

node.default[:redis][:server][:addr]    = "127.0.0.1"
```	

Please also look at the [ruby example](https://gist.github.com/scalp42/5164178).
	

## Need Help or Want to Contribute ?

All contributions are welcome: ideas, patches, documentation, bug reports, complaints, and even something you drew up on a napkin.

It is more important to me that you are able to contribute and get help if you need it.

That said, some basic guidelines, which you are free to ignore :)

- Have a problem you want **askcerebro** to solve for you ? You can email me personally (scalisia@gmail.com)
- Have an idea or a feature request? File a ticket on Github, or email me personally (scalisia@gmail.com) if that is more comfortable.
- If you think you found a bug, it probably is a bug. Please file a ticket on Github.
- If you want to add new `disks`, best way is to fork this repo and send me a pull request. If you don't know git, I also accept diff(1) formatted patches - whatever is most comfortable for you.

**Programming is not a required skill. Whatever you've seen about open source with maintainers or community members saying "send patches or die" -  you will not see that here.**

## Contributors

[Thank you for contributing !](https://github.com/scalp42/askcerebro/graphs/contributors)
