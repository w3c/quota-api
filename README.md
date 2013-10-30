Quota Management API
--------------------

Quota Management API is for helping webapps to manage their storage data
stored on the user's local device more wisely and efficiently.
Namely, the API is aiming to enable:

* Webapps to **query the current data usage**, i.e. how much data is stored by the app on the user's local device,
* Webapps to **request a new storage quota** to store bigger data in a way that the UA and the user can agree with, and
* Webapps to **efficiently manage their data** when the remaining available space on the user's local device is getting tight.

The API is intended to work in a unified way across multiple storage APIs.
Namely, the API is expected to work with following APIs:

* [Indexed Database API](http://www.w3.org/TR/IndexedDB/)
* [Application Cache](http://www.whatwg.org/specs/web-apps/current-work/multipage/offline.html#offline)
* [Web SQL Database](http://www.w3.org/TR/webdatabase/)
* File System API ([File API: Directories and System](http://www.w3.org/TR/file-system-api/), [new proposal by Mozilla](http://lists.w3.org/Archives/Public/public-webapps/2013AprJun/0382.html))
* [Navigation Controller](https://github.com/slightlyoff/NavigationController)

Note that some of the storage APIs listed above are no longer in active
maintenance, but the UA should make the APIs work with the Quota API if
it supports them.

Currently it's in FPWD status.

* [FPWD](http://www.w3.org/TR/quota-api/)
* [New API draft](http://htmlpreview.github.io/?http://github.com/kinu/quota-api/blob/master/draft.html)
