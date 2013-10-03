## New Draft Proposal

### Overview

This draft is for the next version of Quota API WD, updating
[FPWD](http://www.w3.org/TR/quota-api/):

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

    partial interface Navigator {
      readonly attribute StorageQuota storageQuota;
    };

    [NoInterfaceObject] interface StorageInfo {
      unsigned long long usage;
      unsigned long long quota;
    };

    [NoInterfaceObject] interface StorageQuota {
      // List of all storage types supported by the UA.
      readonly attribute StorageType[] supportedTypes;

      // Queries the current storage info.
      Promise<StorageInfo> queryInfo(StorageType type);

      // Requests a new quota for persistent storage.
      Promise<StorageInfo> requestPersistentQuota(unsigned long long newQuota);
    };

### StorageWatcher interface

    // Watches storage changes. storagechange event is fired about every
    // 'rate' seconds or every time the quota backend detects the amount
    // of usage or quota is changed, whichever is least frequent.
    [Constructor(StorageType type, unsigned long rate)]
    interface StorageWatcher : EventTarget {
      readonly attribute StorageType type;
      readonly attribute unsigned long rate;

      attribute EventHandler onstoragechange;
    };

    interface StorageEvent : Event {
      readonly attribute unsigned long long usage;
      readonly attribute unsigned long long quota;
    };

