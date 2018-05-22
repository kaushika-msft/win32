---
title: How To Create a Reference Device
description: This topic shows how to create a reference device that implements a highly accurate, software implementation of the runtime.
ms.assetid: '00d3f5f2-02c6-4ff4-82a9-0726ad4a8cb3'
---

# How To: Create a Reference Device

This topic shows how to create a reference device that implements a highly accurate, software implementation of the runtime. To create a reference device, simply specify that the device you are creating will use a reference driver. This example creates a device and a swap chain at the same time.

**To create a reference device**

1.  Define initial parameters for a swap chain.
    ```
        DXGI_SWAP_CHAIN_DESC sd;
        ZeroMemory( &amp;sd, sizeof( sd ) );
        sd.BufferCount = 1;
        sd.BufferDesc.Width = 640;
        sd.BufferDesc.Height = 480;
        sd.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
        sd.BufferDesc.RefreshRate.Numerator = 60;
        sd.BufferDesc.RefreshRate.Denominator = 1;
        sd.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
        sd.OutputWindow = g_hWnd;
        sd.SampleDesc.Count = 1;
        sd.SampleDesc.Quality = 0;
        sd.Windowed = TRUE;
    ```

    

2.  Request a feature level that implements the features your application will need. A reference device can be successfully created for the Direct3D 11 runtime.

    ```
        D3D_FEATURE_LEVEL FeatureLevels = D3D_FEATURE_LEVEL_11_0;
    ```

    

    See more about feature levels in the [**D3D\_FEATURE\_LEVEL**](d3d-feature-level.md) enumeration.

3.  Create the device by calling [**D3D11CreateDeviceAndSwapChain**](d3d11createdeviceandswapchain.md).


```
    HRESULT hr = S_OK;
    D3D_FEATURE_LEVEL FeatureLevel;

    if( FAILED (hr = D3D11CreateDeviceAndSwapChain( NULL, 
                    D3D_DRIVER_TYPE_REFERENCE,
                    NULL, 
                    0,
                    &amp;FeatureLevels, 
                    1, 
                    D3D11_SDK_VERSION, 
                    &amp;sd, 
                    &amp;g_pSwapChain, 
                    &amp;g_pd3dDevice, 
                    &amp;FeatureLevel,
                    &amp;g_pImmediateContext )))
    {
        return hr;
    }
```



You will need to supply the API call with the reference driver type from the [**D3D\_DRIVER\_TYPE**](d3d-driver-type.md) enumeration. After the method succeeds, it will return a swap chain interface, a device interface, a pointer to the feature level that was granted by the driver, and an immediate context interface.

For information about limitations creating a reference device on certain feature levels, see [Limitations Creating WARP and Reference Devices](overviews-direct3d-11-devices-limitations.md).[How to Use Direct3D 11](how-to-use-direct3d-11.md)

## Related topics

<dl> <dt>

[Devices](overviews-direct3d-11-devices.md)
</dt> <dt>

[How to Use Direct3D 11](how-to-use-direct3d-11.md)
</dt> </dl>

 

 



