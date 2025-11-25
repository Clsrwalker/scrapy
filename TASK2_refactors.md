Task 2. Refactoring summary

1. Project information

Upstream repository: https://github.com/scrapy/scrapy  
My fork: https://github.com/Clsrwalker/scrapy  
Branch used for the changes: assignment/refactor

For Task 2 I did all my changes directly in Scrapy. In the end almost everything I touched lives in these two modules:

- scrapy/utils/request.py
- scrapy/utils/asyncio.py

From these files I picked three refactorings for Set I, implementation level, and three for Set II, design level.

2. Set I – implementation level refactorings

2.1 Extract method for fingerprint headers and payload

Refactoring name: Extract Method  
Location: scrapy/utils/request.py, class FingerprintBuilder, methods _build_headers, build_payload, build_hash

Old file link, before refactoring:  
https://github.com/scrapy/scrapy/blob/master/scrapy/utils/request.py  

New file link, after refactoring:  
https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/request.py

Before this change, the code that built the fingerprint was more or less one big piece. It decided which headers to include, it built the payload, and it did the hashing, all together.  

Now there is a small helper class called FingerprintBuilder and I split the work into a few clear methods:

- _build_headers builds the headers dictionary used in the fingerprint  
- build_payload returns the dict with method, url, body and headers  
- build_hash takes that payload, turns it into JSON and computes the SHA1 hash  

So instead of one long function with many steps mixed together, there are three short methods. That makes the flow easier to understand and it is simpler to change only one part later if needed.

2.2 Rename parameter in _parallel_asyncio to worker_func

Refactoring name: Rename parameter  
Location: scrapy/utils/asyncio.py, function _parallel_asyncio

Old file link:  
https://github.com/scrapy/scrapy/blob/master/scrapy/utils/asyncio.py  

New file link:  
https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/asyncio.py

In _parallel_asyncio there is a coroutine that processes every item taken from a queue. That function was passed in as a parameter. I renamed that parameter to worker_func.

Inside the inner worker coroutine the code now does something like:

    await worker_func(item, *args, **kwargs)

This reads quite naturally. You can guess what worker_func is just from the name. The behaviour is exactly the same as before. The only goal here is to make the code easier to read when someone looks at the function signature and the loop.

2.3 Introduce explaining variable use_asyncio in _select_call_later_scheduler

Refactoring name: Introduce Explaining Variable  
Location: scrapy/utils/asyncio.py, function _select_call_later_scheduler

Old file link:  
https://github.com/scrapy/scrapy/blob/master/scrapy/utils/asyncio.py  

New file link:  
https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/asyncio.py

The helper _select_call_later_scheduler decides whether to use the asyncio based scheduler or the Twisted based one. The decision is based on is_asyncio_available.

I added a small local variable called use_asyncio:

    use_asyncio = is_asyncio_available()

Then the function returns the asyncio scheduler or the Twisted scheduler depending on this variable. The logic did not change, but now the condition reads more like a normal sentence, which makes it a bit more friendly to read and also easier to log or debug later.

3. Set II design level refactorings

3.1 Extract class FingerprintBuilder

Refactoring name: Extract Class  
Location: scrapy/utils/request.py, class FingerprintBuilder

Old file link:  
https://github.com/scrapy/scrapy/blob/master/scrapy/utils/request.py  

New file link:  
https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/request.py

Originally the logic for building a fingerprint was spread across the top level fingerprint function and the RequestFingerprinter class. It did URL canonicalisation, header handling, body handling and hashing, plus some cache work.

I introduced a separate helper class called FingerprintBuilder. This class has one main job, it knows how to take a Request and the processed headers and turn that into a stable hash.  

After this change the roles are clearer:

- FingerprintBuilder handles the detailed steps of building the fingerprint payload and computing the hash  
- RequestFingerprinter takes care of the public API, header processing and the cache, and then calls FingerprintBuilder when it needs the actual hash  

This separation makes the code a bit easier to test and reason about.

3.2 Move field:fingerprint cache into RequestFingerprinter

Refactoring name: Move Field  
Location: scrapy/utils/request.py, class RequestFingerprinter attribute _cache and module level _fingerprint_cache

Old file link:  
https://github.com/scrapy/scrapy/blob/master/scrapy/utils/request.py  

New file link:  
https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/request.py

Before this refactor the cache used for fingerprints was stored in a module level variable called _fingerprint_cache. This is a global piece of mutable state, which is not ideal.

In the new code RequestFingerprinter has an instance attribute called _cache. It is a WeakKeyDictionary keyed by Request. The fingerprint method uses this instance cache. At the module level there is still a name _fingerprint_cache, but now it just points at the default RequestFingerprinter instance cache. This is to keep backwards compatibility with existing code and tests that import it.

So the cache field moved from the module to the class. It makes RequestFingerprinter more self contained and it opens the possibility to create different fingerprinters with their own caches if that is ever needed.

3.3 Replace conditional with polymorphism for call_later scheduler

Refactoring name: Replace Conditional with Polymorphism  
Location: scrapy/utils/asyncio.py, classes _BaseCallLaterScheduler, _AsyncioCallLaterScheduler, _TwistedCallLaterScheduler, plus helper _select_call_later_scheduler and function call_later

Old file link:  
https://github.com/scrapy/scrapy/blob/master/scrapy/utils/asyncio.py  

New file link:  
https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/asyncio.py

The function call_later has to schedule a function that will be called after some delay. There are two ways to do this in Scrapy, one uses the asyncio event loop and one uses the Twisted reactor. Which one to use depends on the environment.

Instead of putting an if else inside call_later each time, I added a small strategy style structure:

- _BaseCallLaterScheduler defines a schedule method  
- _AsyncioCallLaterScheduler implements schedule using asyncio.get_event_loop and call_later  
- _TwistedCallLaterScheduler implements schedule using reactor.callLater  
- _select_call_later_scheduler chooses one of these based on is_asyncio_available  

Now call_later just asks _select_call_later_scheduler for the right scheduler and calls its schedule method. The logic that decides which backend to use is in one place and it is hidden behind the object interface.

If Scrapy wants to add another backend in the future, a new scheduler class can be added without changing the main call_later function.

4. Build and test instructions

To check that these refactorings do not break the project, I used my fork and ran the following commands:

-pip install -e .  
- pip install pytest pytest-twisted  
- pytest tests/test_utils_request.py tests/test_utils_asyncio.py  

The tests above all passed after the changes.

5. pull request information

Pull request URL: not opened yet, plan is to open a PR from branch assignment/refactor in my fork to the main scrapy/scrapy master branch  
Status at submission time: not opened  
Merged commit link: not applicable yet
