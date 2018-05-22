---
Description: 'DDE shares are a machine resource.'
ms.assetid: '98d24300-52cc-4f0d-b74f-c58b823ac5f3'
title: DDE Shares
---

# DDE Shares

\[Network DDE is no longer supported. Nddeapi.dll is present on Windows Vista, but all function calls return NDDE\_NOT\_IMPLEMENTED.\]

DDE shares are a machine resource. They are similar to file shares because they are used to control access to a resource. With file shares, the resource is a file. With DDE shares, the resource is dynamically exchanged data. The type of data exchanged is determined by the server application that supplies the data and the client application that requests the data.

The server calls the [**NDdeShareAdd**](nddeshareadd.md) function to create the DDE share, which is stored in the DDE share database manager (DSDM).

The client starts the DDE conversation by connecting to the DDE share. The client must call the [**DdeInitialize**](_win32_ddeinitialize_cpp) function to initialize DDEML and call the [**DdeConnect**](_win32_ddeconnect_cpp) function to connect to the DDE share. In the **DdeConnect** call, the client specifies the service name as follows:

\\\\*ComputerName*\\NDDE$

where *ComputerName* is the name of the computer running the server application. The NDDE$ indicates that the topic provided to [**DdeConnect**](_win32_ddeconnect_cpp) is the DDE share name on the remote computer named *ComputerName*.

There are three types of DDE shares: old style, new style, and static. It is typical to support only the static type. The names of static shares use the following convention: *ShareName*$.

 

 


