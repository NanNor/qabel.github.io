---
title: "Components: Storage API"
---
# Storage component API


1. Upload method: 
    * generate new key or use existing
    * encrypt and authenticate collection of blobs with encryption method specified in CryptoObject
    * generate blob urls
    * upload collection of blobs
    * write key/blob url to blob objects (and crypto object if new key was generated)

          public void upload (collection of StorageBlob blobCollection,
              CryptoObject crypto)
    
5. Download method:
    * download collection of blobs (identified by blob urls)
    * authenticate and decrypt this collection
    * write blob data to blob objects
    
          public void download (collection of StorageBlob blobCollection,
              CryptoObject crypto)
    
7. Delete method:
    * delete a collection of blobs from specified urls
    
          public void delete (collection of StorageBlob blobCollection)  
    
    
## Used Objects

    class CryptoObject {

      private enum EncryptionMethod;
      private ... Keys; //TBD
      
      public CryptoObject (enum EncryptionMethod); //for new key generation
      public CryptoObject (enum EncryptionMethod, ... Keys); //for importing key
       
    } // not part of storage class but crypto class
    

    class StorageBlob {

      private byte[BLOB_SIZE] blobData;
      private string blobUrl;
      
      public byte[] getBlobData();
      public string getBlobUrl();
      
      
      public StorageBlob (byte[] blobData);
      public StorageBlob (string blobUrl);
    }

    

   