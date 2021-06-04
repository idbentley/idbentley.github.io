---
layout: post
title: "Using python suds in Multi Threaded and Multi Process Environments"
date: 2014-09-16 13:32
categories: python suds multithread multiprocess
---

Working on a team that primarily builds integrations between systems, you will
inevitably end up writing integrations against SOAP based APIs.  If, like me,
your team uses python, you will likely end up using
[Suds](https://fedorahosted.org/suds/).  As you can see from this
[stackoverflow](http://stackoverflow.com/questions/206154/what-soap-client-libraries-exist-for-python-and-where-is-the-documentation-for)
post, there aren't a great deal of options for SOAP in python; and none are
phenomenal.

While Suds is not actively maintained, it has wide adoption, and is fairly
stable and  mature.  Suds is actually a pretty good abstraction around SOAP, and it
encourages idiomatic python.  I have my fair share of frustrations with Suds,
but for the most part, it "just works".  It does, however, have two major flaws
that make it unusable out of the box for my use-case.

## Suds is not Thread safe

### The Problem

For me, this manifested only in production (wsgi is designed such that most
development environments, including mine, are single threaded) with a 
SAXParseException:

{% highlight python %}
File "/venv_path/lib/python2.6/site-packages/suds/client.py", line 542, in __call__
  return client.invoke(args, kwargs)
File "/venv_path/lib/python2.6/site-packages/suds/client.py", line 602, in invoke
  result = self.send(soapenv)
File "/venv_path/lib/python2.6/site-packages/suds/client.py", line 649, in send
  result = self.failed(binding, e)
File "/venv_path/lib/python2.6/site-packages/suds/client.py", line 702, in failed
  r, p = binding.get_fault(reply)
File "/venv_path/lib/python2.6/site-packages/suds/bindings/binding.py", line 258, in get_fault
  faultroot = sax.parse(string=reply)
File "/venv_path/lib/python2.6/site-packages/suds/sax/parser.py", line 136, in parse
  sax.parse(source)
File "/usr/lib64/python2.6/xml/sax/expatreader.py", line 107, in parse
  xmlreader.IncrementalParser.parse(self, source)
File "/usr/lib64/python2.6/xml/sax/xmlreader.py", line 123, in parse
  self.feed(buffer)
File "/usr/lib64/python2.6/xml/sax/expatreader.py", line 211, in feed
  self._err_handler.fatalError(exc)
File "/usr/lib64/python2.6/xml/sax/handler.py", line 38, in fatalError
  raise exception
SAXParseException: <unknown>:15:2: mismatched tag
{% endhighlight %}

### The Solution

Ideally this could be fixed in suds properly, but since the library is
no-longer maintained, we opted for a simple workaround.  Basically we use a
[thread safe queue](https://docs.python.org/3/library/queue.html), to add a lock
around suds usage:

To Set Up:

{% highlight python %}
from Queue import Queue
from contextlib import contextmanager

MAX_THREADS = 5
suds_service_queue = Queue(MAX_THREADS)
for n in range(MAX_THREADS):
    suds_service_queue.put(construct_suds_client())

@contextmanager
def get_suds_client():
  # blocks until client available
  client = suds_service_queue.get()
  try:
    yield client
  finally:
    suds_service_queue.put(client)
{% endhighlight %}

To Use:

{% highlight python %}
from setup import get_suds_client
with get_suds_client() as client:
  client.do_stuff
{% endhighlight %}

This technique obviously requires additional resources compared to using a
single suds `Client` instance, but for us that trade-off is worth it.  If you
have flexible real-time requirements, or limited parallelism, you can make
`MAX_THREADS = 1` to achieve thread safety without any overhead.


## Suds is not Friendly in Multi-Process Environments

### The Problem

After we solved our thread safety problems, we ran into another exception coming
from scripts running on the same server as our wsgi application.  The `OSError`
exceptions were of the form:

{% highlight python %}
File "/venv_path/venv/lib/python2.6/site-packages/suds/client.py", line 109, in __init__
  options.cache = ObjectCache(days=1)
File "/venv_path/venv/lib/python2.6/site-packages/suds/cache.py", line 145, in __init__
  self.checkversion()
File "/venv_path/venv/lib/python2.6/site-packages/suds/cache.py", line 277, in checkversion
  self.clear()
File "/venv_path/venv/lib/python2.6/site-packages/suds/cache.py", line 251, in clear
  os.remove(os.path.join(self.location, fn))
OSError: [Errno 13] Permission denied: '/tmp/suds/suds-6882323804701353659-document.px'
{% endhighlight %}

There is an [open ticket](https://fedorahosted.org/suds/ticket/376) related to
this bug in suds, but we didn't want to fork suds to apply the patch provided.
Instead, we found an alternative solution that works well for us.

### The Solution

To understand our solution to this problem, we need to look more closely at how
to reproduce it.  First we must have a system in which there are at least two
users - in our case `apache` and `app-prod`.  The first of these users (say,
`apache`) to use a suds client will create a directory at `/tmp/suds` which it
will store the suds document cache.  Assuming the `umask` of this user is
configured in a standard way, and that `app-prod` is not in the `apache` group
the process run by `app-prod` will suffer the dreaded Permission denied `OSError`

Suds allows us to provide an `ObjectCache` object to the client constructor, and
in turn the `ObjectCache` constructor allows us to specify the path of the
document cache.

We initially considered adding the `pid` to the path to ensure permission safety, 
but were concerned with the side effect of creating many folders as the
lifetime of the machine goes on (every script that uses suds would create a new
folder, and our default system only cleans up `/tmp` on reboot.  Instead we
decided to use the `uid` of the process, which would protect us from permission
issues, while limiting the number of folders to the number of active users.

Check out the resulting class.

{% highlight python %}
import os
import suds
from suds.cache import ObjectCache

class SudsApi(object):
  _suds_client = None

  def __init(self, wsdl_url):
    self.wsdl_url = wsdl_url

  @property
  def client(self):
    if self._suds_client is None:
      cache_path = "/tmp/{0}-suds".format(os.getuid())
      cache = ObjectCache(cache_path, days=1)
      self._suds_client = suds.client.Client(self.wsdl_url,
        cache=cache)
    return self._suds_client
{% endhighlight %}

### Conclusion

Hopefully these tricks will help you sucessfully use `suds` in your production
environment.

