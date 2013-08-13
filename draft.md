## What's Quota API for?

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

## New Draft Proposal

### Overview

The main changes this draft is trying to make from
the [FPWD](http://www.w3.org/TR/quota-api/) are:

* **Promise**: The API should use Promise rather than callback syntax.
* **Getting and setting storage types**: The API should provide a way for webapps to get and set which storage type (i.e. *persistent* or *temporary*) each storage object belongs to.
* **Events**: The API should provide a way for the UA to notify webapps of significant events in local storage condition (e.g. when the remaining storage space is getting tight, when the UA is about to evict some of data stored by webapps etc).

On the other hand this draft is NOT (yet) addressing following requests:

* **More granularity in storage types or priorities**: The API should provide more granularity in storage types, rather than sticking to the rigid two types (*temporary* or *persistent*). For example it might be more convenient to specify storage types by numeric priorities, where lower priority indicates its data may be evicted earlier.
* **Size estimation of each storage object**: The API should provide a way for webapps to estimate how much space a particular storage object is consuming (i.e. how much space the app can free up by removing the storage object).
* **Trigger GC/compaction**: The API should provide a way for webapps to trigger storage GC/compaction.

While all of them sound reasonable or nice-to-have, I tentatively concluded
that they can be put off until the next revision iteration.
(I'm open to further discussions on them though)

### Storage types

This proposal uses the same storage types and terminologies
as defined in the [FPWD](http://www.w3.org/TR/quota-api/), i.e.
[temporary](http://www.w3.org/TR/quota-api/#temporary) and
[persistent](http://www.w3.org/TR/quota-api/#persistent).

    enum StorageType {
      "temporary",
      "persistent"
    };

### StorageQuota interface

Most of Quota API methods are provided via **StorageQuota** interface,
which can be accessible via *navigator.storageQuota* attribute.

    interface StorageInfo {
      unsigned long long usageInBytes;
      unsigned long long quotaInBytes;
    };

    interface StorageQuota {
      Promise<StorageInfo> queryStorageInfo(StorageType type);
      Promise<StorageInfo> requestQuota(StorageType type, unsigned long long newQuotaInBytes);

      Promise<StorageType> getStorageType((IDBObjectStore or Database or Entry) object);
      Promise<void> setStorageType((IDBObjectStore or Database or Entry) object, StorageType type);

      StorageWatcher createStorageWatcher(StorageType type);
    };

**supportedTypes** are list of all **StorageType**'s supported by the UA.

**queryStorageInfo** queries the current storage information for the given type.
This should work in the same way as how **queryUsageAndQuota** is supposed
to work in [FPWD](http://www.w3.org/TR/quota-api), except that it takes no
callbacks and returns **StorageInfo** by **Promise**.

**requestQuota** requests a new quota space for the given type.
This should work in the same way as how **requestQuota** is supposed
to work in [FPWD](http://www.w3.org/TR/quota-api), except that it also returns
**StorageInfo** by **Promise**, so that the webapps can get the udpated
storage information after the request is handled by UA.

**getStorageType** returns the current storage type for the given storage
object. For example it should return "temporary" for a FileEntry in temporary
FileSystem. This returns error (gets rejected) if the given object is not stored
in the storage backend that works with Quota API.

**setStorageType** sets the storage type of the given storage object.
For example, calling setStorageType("temporary", *object*) for a storage
object that belongs to *persistent* storage moves the object into
*temporary* storage. This returns error (gets rejected) if the target
storage does not have enough quota, or the storage backend does not support
changing storage types.

**createStorageWatcher** create a **StorageWatcher** object for
the given storage type.

### StorageWatcher interface

    interface StorageWatcher {
      attribute EventHandler onstoragelow;
      attribute EventHandler onstorageok;
      attribute EventHandler onstoragechange;
    };

**storagelow** event is fired when the remaining storage space becomes
lower than 10% of all available space for the type, or before the
quota backend triggers eviction (for *temporary* case), whichever
happens first. This event must be also fired when either one of these
conditions is already met when a StorageWatcher is created.

**storageok** event is fired when the remaining available storage goes out
of storagelow state.

**storagechange** event is fired about every 1 second or every time
the quota backend detects the amount of remaining storage is changed,
whichever is least frequent. This event must be fired before **storagelow**
or **storageok** when the remaining space crosses the threshold.
