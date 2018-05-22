---
Description: 'You can import and export symmetric keys and asymmetric keys with CNG. And you can use key export and import functionality to move keys between machines.'
ms.assetid: '37bda1e0-5dd2-455c-9627-4e7e1b0e04d3'
title: Key Import and Export
---

# Key Import and Export

You can import and export [*symmetric keys*](https://msdn.microsoft.com/library/windows/desktop/ms721625#-security-symmetric-key-gly) and asymmetric keys with CNG. And you can use key export and import functionality to move keys between machines.

## Symmetric keys

To import or export symmetric (or session) keys in which the same key is used to encrypt and decrypt some data, you can use the [**BCryptImportKey**](bcryptimportkey-func.md) and [**BCryptExportKey**](bcryptexportkey-func.md) functions. Typically, you first export a key by using the **BCryptExportKey** function before importing by using the **BCryptImportKey** function. The functions are designed to enable encryption of exported and imported keys by using the *hExportKey* and *hImportKey* parameters; however, the Microsoft implementation of these functions does not support encryption of exported and imported keys.

## Asymmetric keys

To import asymmetric (or [*public/private*](https://msdn.microsoft.com/library/windows/desktop/ms721603#-security-public-private-key-pair-gly)) key pairs in which one key is used to encrypt and the other is used to decrypt some data, you can use either of the [**BCryptImportKeyPair**](bcryptimportkeypair.md) or [**NCryptImportKey**](ncryptimportkey-func.md) functions. A CNG provider must encode the key pair by using a supported [*key BLOB*](https://msdn.microsoft.com/library/windows/desktop/ms721590#-security-key-blob-gly) type. [**BCryptExportKey**](bcryptexportkey-func.md) can be used to create the encoded key BLOB. [CNG Structures](cng-structures.md) describes the key BLOB types and structures that Microsoft Key Storage Provider supports.

For [**BCryptExportKey**](bcryptexportkey-func.md) to create a persisted key pair, the input key BLOB must contain a [*private key*](https://msdn.microsoft.com/library/windows/desktop/ms721603#-security-private-key-gly). [*Public keys*](https://msdn.microsoft.com/library/windows/desktop/ms721603#-security-public-key-gly) are not persisted.

The key name and export policy are not part of the [*BLOB*](https://msdn.microsoft.com/library/windows/desktop/ms721569#-security-blob-gly) structure defined in [CNG Structures](cng-structures.md). However, if a BLOB is of an opaque BLOB type (such as the memory image of an internal key state), the BLOB might contain the key name and export-policy properties.

The following procedure describes how to import a persisted private key with its properties.

**To import a persisted key**

1.  Create a persisted key by using the [**NCryptCreatePersistedKey**](ncryptcreatepersistedkey-func.md) function.
2.  Set any desired properties on the key object by using the [**NCryptSetProperty**](ncryptsetproperty-func.md) function.
3.  Set the import key BLOB as a property on the key, with the BLOB type as the property name.
4.  Finalize the persisted key import by using the [**NCryptFinalizeKey**](ncryptfinalizekey-func.md) function.

 

 


