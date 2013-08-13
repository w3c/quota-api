## New Draft Proposal

### Overview

This draft is trying to address following requests and feedbacks given to
the [FPWD](http://www.w3.org/TR/quota-api/):

* **Promise**: The API should use Promise rather than callback syntax.
* **Getting and setting storage types**: The API should provide a way for webapps to get and set which storage type (i.e. *persistent* or *temporary*) each storage object belongs to.
* **Events**: The API should provide a way for the UA to notify webapps of significant events in local storage condition (e.g. when the remaining storage space is getting tight, when the UA is about to evict some of data stored by webapps etc).

This draft is NOT (yet) addressing following requests:

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

Most of Quota API methods are provided via StorageQuota interface,
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

**supportedTypes** are list of all StorageType's supported by the UA.

**queryStorageInfo** queries the current storage information for the given type.
This should work in the same way as
how [queryUsageAndQuota](http://www.w3.org/TR/quota-api/#widl-StorageQuota-queryUsageAndQuota-void-StorageUsageCallback-successCallback-StorageErrorCallback-errorCallback) is supposed
to work in [FPWD](http://www.w3.org/TR/quota-api), except that it takes no
callbacks and returns StorageInfo object via Promise.

**requestQuota** requests a new quota space for the given type.
This should work in the same way as how [requestQuota](http://www.w3.org/TR/quota-api/#widl-StorageQuota-requestQuota-void-unsigned-long-long-newQuotaInBytes-StorageQuotaCallback-successCallback-StorageErrorCallback-errorCallback)
is supposed to work in [FPWD](http://www.w3.org/TR/quota-api),
except that it also returns
StorageInfo object via Promise, so that the webapps can get the udpated
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

**createStorageWatcher** create a [StorageWatcher](https://github.com/kinu/quota-api/blob/master/draft.md#storagewatcher-interface) object for
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
whichever is least frequent. This event must be fired before *storagelow*
or *storageok* when the remaining space crosses the threshold.
