Task 1 – Smell detection on Scrapy

Project and tool

For this assignment I chose the Scrapy project (https://github.com/scrapy/scrapy).  
It is a Python web crawling framework with much more than 10k lines of code, lots of GitHub stars, and recent commits, so it matches the project selection rules.

To detect smells I used DPy 1.7.4 (trial version).  
Because the trial only works on less than 10k LOC at a time, I ran it on folders instead of the whole repo:

- scrapy/core
- scrapy/utils

DPy produced CSV files with implementation smells and design smells. I read those CSVs, then opened the real code and checked each smell by hand. From that process I picked 6 examples:

- 3 implementation smells (true positive, false positive, false negative)
- 3 design smells (true positive, false positive, false negative)

Below I explain each one in my own words.


Implementation smells (Set I)

1) True positive – complex method in ExecutionEngine.close_spider_async

Smell type: complex  long method 
Category: true positive  
File: scrapy/core/engine.py  
Location: class ExecutionEngine, method close_spider_async ，around lines 560–633

DPy marks close_spider_async as a complex method. After reading it, I agree.

This async method is responsible for closing almost everything related to a spider:

- it closes the slot,
- closes the downloader,
- closes the scraper,
- maybe closes the scheduler,
- sends the spider_closed signal,
- closes stats,
- and finally calls the spider_closed callback.

Every step is wrapped in a try/except with logging. So inside one method we have:

- resource clean up,
- error handling,
- logging,
- signals,
- statistics.

The control flow is long, there are a lot of branches, and many different things are happening at once. It is not easy to understand at a glance, and it is also harder to test single parts of it.

So in my opinion this really is a complex/long method smell, not just a cosmetic warning. That is why I treat this as a true positive.


2) False positive – “long statement” in is_asyncio_available

Smell type: long statement (implementation smell)  
Category: false positive  
File: scrapy/utils/asyncio.py  
Location: function is_asyncio_available (around lines 30–70)

DPy reports a “long statement” here. When I open the file, the only long thing I see is the docstring. The actual code is about:

- checking if a reactor is installed,
- raising RuntimeError if not,
- otherwise returning is_asyncio_reactor_installed().

All real code lines are already wrapped nicely. There is no very long boolean expression and no crazy one-line chain of calls. The long text is only the comment (the docstring) that explains future behaviour of this helper.

So the function itself is short and clear. I do not think this is an implementation smell in practice, because the logic is simple and easy to read. For this reason I consider this smell from DPy a false positive.


3) False negative – old fingerprint function in scrapy/utils/request.py

Smell type: long / complex method (implementation smell)  
Category: false negative  
File: scrapy/utils/request.py  
Location: old version of the top-level function fingerprint(request, include_headers=None, keep_fragments=False)

Here I talk about the version before my refactoring. In that older version, the top-level fingerprint function did a lot of different things:

- normalised and canonicalised the URL, with an option to keep or drop fragments,
- handled the include_headers parameter and normalised header names,
- looped over request.headers to build a header structure,
- built a JSON-friendly payload with method, url, body and headers,
- managed a global cache (built the cache key and read/wrote the cache),
- computed the SHA1 hash and returned it.

All this logic was inside a single function. It mixed URL logic, header logic, hashing and caching. For me this clearly looks like a “too long and too busy” function.

However, DPy did not flag this function as a long or complex method in my run. From my point of view it still counts as a smell (the function does too many things), so I use it as an example of a false negative.

Later I refactored this part by introducing a small helper class FingerprintBuilder and some smaller methods, and by moving the cache into RequestFingerprinter. That also confirms that the original version was not very clean.


Design smells (Set II)

1) True positive – feature envy in Stream.close

Smell type: feature envy (design smell)  
Category: true positive  
File: scrapy/core/http2/stream.py  
Location: class Stream, method close (starting around line 380)

DPy reports a feature envy smell on Stream.close. After reading this method, I think this is fair.

The close method takes a StreamCloseReason and then has a long if/elif chain for different cases:

- MAXSIZE_EXCEEDED,
- ENDED,
- CANCELLED,
- RESET,
- CONNECTION_LOST,
- INACTIVE,
- INVALID_HOSTNAME.

In some of these branches it uses a lot of data from self._protocol.metadata, for example:

- metadata["ip_address"],
- metadata["uri"].host,
- metadata["uri"].port,
- and other values from the protocol.

Stream is supposed to represent a single HTTP/2 stream, but here it knows quite a lot about how the underlying H2ClientProtocol stores its metadata. It even builds error messages using those details.

That is why this feels like classic “feature envy”: the Stream object is reaching into another object (the protocol) and using its internal fields heavily, instead of working mainly with its own state. So I agree with the tool and treat this as a true positive design smell.


2) False positive – “deficient encapsulation” in CaselessDict

Smell type: deficient encapsulation ，design smell
Category: false positive  
File: scrapy/utils/datatypes.py  
Location: class CaselessDict (near the top of the file, around lines 30–90)

DPy reports a “deficient encapsulation” smell for CaselessDict. This class is a small subclass of dict. It overrides methods like __getitem__, __setitem__, __delitem__ and __contains__. Inside these methods it calls dict.__getitem__ and friends directly, but always after running a helper like normkey or normvalue.

The pattern is something like:

 __getitem__ → dict.__getitem__(self, self.normkey(key))
 __setitem__ → dict.__setitem__(self, self.normkey(key), self.normvalue(value))

This is a standard way to extend a built-in container in Python. The abstraction here is still “a dict-like mapping”, and CaselessDict just adds case-insensitive keys. It does not leak any extra internal state beyond what dict already exposes.

So I do not really see an encapsulation problem here. My guess is that DPy flags this because of the direct calls to dict.__getitem__, but in this case that is just how you reuse the base implementation. So I treat this as a false positive.


3) False negative – god class ExecutionEngine

Smell type: god class/ multifaceted abstraction ，design smell 
Category: false negative  
File: scrapy/core/engine.py  
Location: class ExecutionEngine (roughly lines 100–633)

ExecutionEngine is the main engine class in Scrapy. It does a lot of different things at once:

- starts and stops the engine,
- opens and closes spiders,
- talks to the scheduler, downloader and scraper,
- drives start requests and normal requests,
- sends and receives many framework signals,
- logs events and updates stats,
- deals with Twisted and asyncio integration.

The class is quite large, and many methods share state through attributes like self.spider, self._slot, self.running, self.paused, and so on. It is basically the “brain” of the whole crawling process.

From a design point of view, this looks like a typical “god class”: one single class that knows too much and controls too many parts of the system. It has several responsibilities that could be split into smaller components.

DPy did not detect a design smell for this class in my run, but I still think it is a design smell ，multi‑responsibility, very central, very big. So I use ExecutionEngine as my false negative example on the design side.
