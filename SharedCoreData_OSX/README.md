# SharedCoreData notes

Tom Harrington, @atomicbird

## CoreDataController

* Sets up persistent store(s) when `loadPersistentStores` is called. Various Core Data and ubiquity objects are available as read-only properties.

`loadPersistentStores`

* Dispatches to a background queue
* Calls `asyncLoadPersistentStores`

`asyncLoadPersistentStores`

* Calls `loadLocalPersistentStore` (which just contains static state info)
* If `[self iCloudAvailable]`,
    * calls `loadiCloudStore`.
    * If `SEED_ICLOUD_STORE` is `#define`d to `YES`, calls

            [self seedStore:_iCloudStore withPersistentStoreAtURL:[self seedStoreURL] error:&error]
    * Calls `deDupe`
    * If a file exists at `fallbackStoreURL`, dispatches to a background queue and calls

            [self seedStore:_iCloudStore withPersistentStoreAtURL:[self fallbackStoreURL] error:&blockError]

* If iCloud isn't available,
    * Calls `loadFallbackStore`
    * If `SEED_ICLOUD_STORE` is `#define`d, calls
    
            [self seedStore:_fallbackStore withPersistentStoreAtURL:[self seedStoreURL] error:&error]
    * Removes the fallback store (?)

* `loadiCloudStore`
    * Loads `_iCloudStore` from `[self iCloudStoreURL]` using standard iCloud setup
    * Adds self as a file presenter for the iCloud store.

* `loadFallbackStore`
    * Loads `_fallbackStore` from `[self fallbackStoreURL]` with no special setup.

* `seedStore:withPersistentStoreAtURL:error:`
    * Sets up a Core Data stack using the provided store URL.
    * Fetches data from this stack's MOC
    * For each instance found, calls `addPerson:toStore:withContext:` with the incoming store argument as the target
    * Overall effect: Copy each person from the store at the incoming URL to the provided target store

* `addPerson:toStore:withContext:`
    * Creates a duplicate Person to match the incoming object
    * Assigns it to the target store
    
* `iCloudAccountChanged:`
    * Callback for `NSUbiquityIdentityDidChangeNotification`
    * Calls `dropStores`
    * Updates the cached copy of the ubiquity token
    * Calls `loadPersistentStores` (see above).

* `dropStores`
    * Calls `removePersistentStore:error:` for any currently loaded store
