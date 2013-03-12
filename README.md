# Ask Cerebro


## What ?

Cerebro is an API-like endpoint that returns information about a package.

Releases type, versions and urls are returned as a JSON so it can be used for rolling upgrades of common software packages.

It's aimed for people who like to compile from sources as it always return source.tar.gz of packages.

You feed him `disks` and it tells you everything about them.



## Why ?

I got tired having to check for the latest version of Nginx, Redis, etc and switching variables in Chef, Fabric, Capistrano and wherever you can think. 

For example, looking at the [Logstash cookbook](https://github.com/lusis/chef-logstash) from Lusis and contributors:

	node['logstash']['agent']['version'] - The version of Logstash to install. Only applies to jar install method.
	
Logstash agent `['version']` can now be easily updated when a new release is available.


I needed something easy, fast, easy to maintain and was using it personally with restricted IPs. A friend asked me if he could used it as well, so I just removed all IPs restrictions and put it on Github.



## Where ?

Just make a call to `www.askcerebro.com/package` where `package` is the package you want to get information about.


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

	{"redis":
		{"releases":["stable","legacy"],
		 "stable":{"version":"2.6.10","url":"http://redis.googlecode.com/files/redis-2.6.10.tar.gz"},
		 "legacy":{"version":"2.4.18","url":"http://redis.googlecode.com/files/redis-2.4.18.tar.gz"}
		 }
	}
	
If a package is not available in Cerebro (for example `perl`), you'll get a 404 HTTP NOT FOUND with an "not available." answer:

	$> curl -i www.askcerebro.com/perl
		
	HTTP/1.1 404 Not Found
	Date: Tue, 12 Mar 2013 18:51:35 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

	not available.
	
If you're trying to hit an endpoint unknown to Cerebro, you'll get a 404 HTTP NOT FOUND answer:

	$> curl -i www.askcerebro.com/shady/endpoint 
	
	HTTP/1.1 404 Not Found
	Date: Tue, 12 Mar 2013 18:51:35 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive
	

#### HUMAN API (because I could not come up with a better name)

You can directly get the **plain text** answer about a package, depending of the type release.

Currently only `version` and `url` are supported.

Here is an example:


	$> curl -i www.askcerebro.com/mongodb/nightly/url
	
	HTTP/1.1 200 OK
	Date: Tue, 12 Mar 2013 19:02:25 GMT
	Content-Type: text/plain; charset=utf-8
	Transfer-Encoding: chunked
	Connection: keep-alive

	http://github.com/mongodb/mongo/tarball/v2.2-latest
	

And another:

	curl -i http://localhost:8080/redis/legacy/version
	
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
  		development:
    		version: 1.3.14
    		url: http://github.example.com/download/latest.tar.gz
  		legacy:
    		version: 1.0.15
    		url: http://example.com/download/legacy-1.0.15.tar.gz
    		
Once I merge your disk, you should be able to ask Cerebro about it within couple mins.
	
	

## Need Help or Want to Contribute ?

All contributions are welcome: ideas, patches, documentation, bug reports, complaints, and even something you drew up on a napkin.

It is more important to me that you are able to contribute and get help if you need it.

That said, some basic guidelines, which you are free to ignore :)

- Have a problem you want **askcerebro** to solve for you ? You can email me personally (anthony@taskrabbit.com)
- Have an idea or a feature request? File a ticket on Github, or email me personally (anthony@taskrabbit.com) if that is more comfortable.
- If you think you found a bug, it probably is a bug. Please file a ticket on Github.
- If you want to add new `disks`, best way is to fork this repo and send me a pull request. If you don't know git, I also accept diff(1) formatted patches - whatever is most comfortable for you.

**Programming is not a required skill. Whatever you've seen about open source with maintainers or community members saying "send patches or die" -  you will not see that here.**

This part was borrowed from [John Vincent](https://github.com/lusis) and [Jordan Sissel](https://github.com/jordansissel). I strongly stand behind this state of mind.