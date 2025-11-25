Task 2 – Refactoring summary

1. Project information

Upstream repository: https://github.com/scrapy/scrapy  
My fork: https://github.com/Clsrwalker/scrapy  
Branch used for the changes: assignment/refactor

All refactorings are inside Scrapy, mainly in:
- scrapy/utils/request.py
- scrapy/utils/asyncio.py

2. Set I – implementation-level refactorings

2.1 Extract method for fingerprint headers and payload  
Refactoring name: Extract Method  
Location: scrapy/utils/request.py, class FingerprintBuilder, methods _build_headers, build_payload, build_hash

Old file link (before refactoring): https://github.com/scrapy/scrapy/blob/master/scrapy/utils/request.py  
New file link (after refactoring): https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/request.py

2.2 Rename parameter in _parallel_asyncio to worker_func  
Refactoring name: Rename parameter  
Location: scrapy/utils/asyncio.py, function _parallel_asyncio

Old file link: https://github.com/scrapy/scrapy/blob/master/scrapy/utils/asyncio.py  
New file link: https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/asyncio.py

2.3 Introduce explaining variable use_asyncio in _select_call_later_scheduler  
Refactoring name: Introduce Explaining Variable  
Location: scrapy/utils/asyncio.py, function _select_call_later_scheduler

Old file link: https://github.com/scrapy/scrapy/blob/master/scrapy/utils/asyncio.py  
New file link: https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/asyncio.py

3. Set II – design-level refactorings

3.1 Extract class FingerprintBuilder  
Refactoring name: Extract Class  
Location: scrapy/utils/request.py, class FingerprintBuilder

Old file link: https://github.com/scrapy/scrapy/blob/master/scrapy/utils/request.py  
New file link: https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/request.py

3.2 Move field: fingerprint cache into RequestFingerprinter  
Refactoring name: Move Field  
Location: scrapy/utils/request.py, class RequestFingerprinter (_cache) and module-level _fingerprint_cache

Old file link: https://github.com/scrapy/scrapy/blob/master/scrapy/utils/request.py  
New file link: https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/request.py

3.3 Replace conditional with polymorphism for call_later scheduler  
Refactoring name: Replace Conditional with Polymorphism  
Location: scrapy/utils/asyncio.py, classes _BaseCallLaterScheduler, _AsyncioCallLaterScheduler, _TwistedCallLaterScheduler; helper _select_call_later_scheduler; function call_later

Old file link: https://github.com/scrapy/scrapy/blob/master/scrapy/utils/asyncio.py  
New file link: https://github.com/Clsrwalker/scrapy/blob/assignment/refactor/scrapy/utils/asyncio.py

4. Build and test instructions

- pip install -e .
- pip install pytest pytest-twisted
- pytest tests/test_utils_request.py tests/test_utils_asyncio.py

5. Pull request information

Pull request URL: (not opened yet; will open from origin/assignment/refactor to scrapy/scrapy master)  
Status at submission time: not opened  
Merged commit link (if any): n/a
