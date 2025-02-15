---
layout: default-layout
title: Dynamsoft Barcode Reader for C Language - User Guide
description: This is the user guide of Dynamsoft Barcode Reader for C Language.
keywords: user guide, c
needAutoGenerateSidebar: true
needGenerateH3Content: false
noTitleIndex: true
---


# User Guide for C Language

## System Requirements

- Operating systems:
   - Windows: 7, 8, 10, 2003, 2008, 2008 R2, 2012;
   - Linux x64: Ubuntu 14.04.4+ LTS, Debian 8+, etc;  
   - Linux arm 32bit;
   - Linux arm 64bit (contact us to get the SDK);
   - macOS 64bit: 10.12+ (contact us to get the SDK).


&nbsp; 

   
## Installation

You can download Dynamsoft Barcode Reader SDK from the [Dynamsoft website](https://www.dynamsoft.com/Downloads/Dynamic-Barcode-Reader-Download.aspx) and run the setup program. The trial installer includes a free trial license valid for 30 days.   
   
After installation, you can find samples for supported platforms in the **Samples** folder under the installation folder.  


&nbsp; 


## Getting Started: HelloWorld
1. Start Visual Studio and create a new Win32 Console Application in C. Let's name it `BarcodeReadDemo_C`.   
2. Add Dynamsoft Barcode Reader headers and libs in `BarcodeReadDemo_C.c`.   
   ```c
    #include <stdio.h>
    #include "<relative path>/DynamsoftCommon.h"
    #include "<relative path>/DynamsoftBarcodeReader.h"
    
    #ifdef _WIN64
    #pragma comment(lib, "<relative path>/DBRx64.lib")
    #else
    #pragma comment(lib, "<relative path>/DBRx86.lib")
    #endif
   ```
   
   Please replace `<relative path>` in the code with the relative path to the `BarcodeReadDemo_C.c` file. Typically, The `DynamsoftBarcodeReader.h` file can be found in `DBR-C_CPP-{version number}\DynamsoftBarcodeReader\Include\`, and the LIB files can be found in `DBR-C_CPP-{version number}\DynamsoftBarcodeReader\Lib\`.
 
3. Update the main function in `BarcodeReadDemo_C.c`.   
   ```c
   int main()
   {
       // Define variables
       void *hBarcode = NULL;
       int iRet = -1;
       int iIndex = 0;
       int iLicMsg = -1;
       TextResultArray* pResult = NULL;
       hBarcode = DBR_CreateInstance();

       // Initialize license prior to any decoding
       iLicMsg = DBR_InitLicense(hBarcode, "<your license key here>");

       //If error occurs to the license initialization
        if (iLicMsg != DBR_OK) 
       {
           printf("Failed to initialize the license successfully: %d\r\n%s\r\n", iLicMsg, DBR_GetErrorString(iLicMsg));
           return iLicMsg;
       }

       // Start decoding. Leave the template name empty ("") will use the settings from PublicRuntimeSettings
       DBR_DecodeFile(hBarcode, "<your image file full path>", "");

       // Get the text result
       iRet = DBR_GetAllTextResults(hBarcode, &pResult);

       // If error occurs
       if (iRet != DBR_OK) 
       {
           printf("Failed to read barcode: %d\r\n%s\r\n", iRet, DBR_GetErrorString(iRet));
           return iRet;
       }

       // If succeeds
       printf("%d total barcode(s) found. \n", pResult->resultsCount);
       for (iIndex = 0; iIndex < pResult->resultsCount; iIndex++)
       {
           printf("Result %d\n", iIndex + 1);
           printf("Barcode Format: %s\n", pResult->results[iIndex]->barcodeFormatString);
           printf("Text reads: %s \n", pResult->results[iIndex]->barcodeText);
       }

       // Finally release BarcodeResultArray and destroy instance
       DBR_FreeTextResults(&pResult);
       DBR_DestroyInstance(hBarcode);
       system("pause");
       return 0;
   }
   ```
   Please update `<your image file full path>` and `<your license key here>` in the code accordingly.   
   
4. Run the project.   

   Build the application and copy the related DLL files to the same folder as the EXE file. The DLLs can be found in `DBR-C_CPP-{version number}\DynamsoftBarcodeReader\Lib\{Platform}\`.

   To test, you can open the Command Prompt and execute the EXE file with a barcode image.
   
To deploy your application, make sure the DLLs are in the same folder as the EXE file. See the [Distribution](#distribution) section for more details.   


&nbsp; 


## Decoding Methods
The SDK provides multiple decoding methods that support reading barcodes from different sources, including static images,
video stream, files in memory, base64 string, bitmap, etc. Here is a list of all decoding methods:
- [DBR_DecodeFile]({{ site.c_methods }}decode.html#dbr_decodefile): Reads barcodes from a specified file (BMP, JPEG, PNG, GIF, TIFF or PDF).   
- [DBR_DecodeBase64String]({{ site.c_methods }}decode.html#br_decodebase64string): Reads barcodes from a base64 encoded string of a file.   
- [DBR_DecodeDIB]({{ site.c_methods }}decode.html#dbr_decodedib): Reads barcodes from a bitmap. When handling multi-page images, it will only decode the
current page.   
- [DBR_DecodeBuffer]({{ site.c_methods }}decode.html#dbr_decodebuffer): Reads barcodes from raw buffer.
- [DBR_DecodeFileInMemory]({{ site.c_methods }}decode.html#dbr_decodefileinmemory): Decodes barcodes from an image file in memory.   
   
You can find more samples in more programming languages at [Code Gallery](https://www.dynamsoft.com/Downloads/Dynamic-Barcode-Reader-Sample-Download.aspx).


&nbsp; 


## Barcode Reading Settings
Calling the [decoding methods](#decoding-methods) directly will use the default scanning modes and it will satisfy most of the needs. The SDK also allows you to adjust the scanning settings to optimize the scanning performance for different usage scenarios.   
   
There are two ways to change the barcode reading settings - using the PublicRuntimeSettings Struct or template. For new
developers, We recommend you to start with the PublicRuntimeSettings struct; For those who are experienced with the SDK,
you may use a template which is more flexible and easier to update.   

- [Use `PublicRuntimeSettings` Struct to Change Settings](#use-publicruntimesettings-struct-to-change-settings)   
- [Use A Template to Change Settings](#use-a-template-to-change-settings)   

### Use `PublicRuntimeSettings` Struct to Change Settings
Here are some common scanning settings you might find helpful:   
- [Specify Barcode Type to Read](#specify-barcode-type-to-read)   
- [Specify Maximum Barcode Count](#specify-maximum-barcode-count)   
- [Specify a Scan Region](#specify-a-scan-region)  

For more scanning settings guide, check out the [`PublicRuntimeSettings`](({{ site.structs }}PublicRuntimeSettings.html)) Struct.

#### Specify Barcode Type to Read
By default, the SDK will read all the supported barcode formats except Postal Codes and Dotcode from the image. If you know exactly the barcode format(s) you want to read, use `barcodeFormatIds` and `barcodeFormatIds_2` to specify the barcode format(s) to speed up the process and improve the accuracy. Check out [`Barcode Format Enumeration`]({{ site.enumerations }}format-enums.html) for full supported barcode list.   

For example, to read QR Code only, you can use the following code:   

```c
char sError[512];
PublicRuntimeSettings runtimeSettings;
//...Initialization code
DBR_GetRuntimeSettings(hBarcode, &runtimeSettings);
runtimeSettings.barcodeFormatIds = BF_QR_CODE; 
runtimeSettings.barcodeFormatIds_2 = BF2_NULL; 
DBR_UpdateRuntimeSettings(hBarcode, &runtimeSettings, sError, 512);
//...Decode and do something with the result
```

#### Specify maximum barcode count
By default, the SDK will read as many barcodes as it can. If you know exactly the barcode count or the maximum count you want to read, use `expectedBarcodesCount` to specify the count value to speed up the process.   

For example, to read two barcodes only, you can use the following code:   

```c
char sError[512];
PublicRuntimeSettings runtimeSettings;
//...Initialization code
DBR_GetRuntimeSettings(hBarcode, &runtimeSettings);
runtimeSettings.expectedBarcodesCount = 2;
DBR_UpdateRuntimeSettings(hBarcode, &runtimeSettings, sError, 512);
//...Decode and do something with the result
```

#### Specify a scan region
By default, the SDK will search the whole image for barcodes. This can lead to poor performance especially when
dealing with high-resolution images. If you know exactly where the barcode locates, use `region` to specify the barcode location.   

For example, to find the barcode located in the middle of the image, you can use the following code:   

```c
char sError[512];
PublicRuntimeSettings runtimeSettings;
//...Initialization code
DBR_GetRuntimeSettings(hBarcode, &runtimeSettings);
runtimeSettings.region.regionLeft = 25;
runtimeSettings.region.regionTop = 25;
runtimeSettings.region.regionRight = 75;
runtimeSettings.region.regionBottom = 75;
runtimeSettings.region.regionMeasuredByPercentage = 1; 
DBR_UpdateRuntimeSettings(hBarcode, &runtimeSettings, sError, 512);
//...Decode and do something with the result
```

### Use A Template to Change Settings
Besides the option of using the PublicRuntimeSettings struct, the SDK also provides [`DBR_InitRuntimeSettingsWithString`]({{ site.c_methods }}parameter-and-runtime-settings-advanced.html#dbr_initruntimesettingswithstring) and [`DBR_InitRuntimeSettingsWithFile`]({{ site.c_methods }}parameter-and-runtime-settings-advanced.html#dbr_initruntimesettingswithfile) APIs that enable you to use a template to control all the settings. With a template, instead of writing many codes to modify the settings, you can manage all the settings in a JSON file/string.    

```c
char sError[512];
//...Initialization code
DBR_InitRuntimeSettingsWithFile(hBarcode, "<Put your file path here>", CM_OVERWRITE, sError, 512);
//...Decode and do something with the result
```  

Below is a template for your reference. For more scanning settings guide, check out the [`Structure and Interfaces of Parameters`]({{ site.parameters }}structure-and-interfaces-of-parameters.html).

```json
{
   "ImageParameter" : {
      "BarcodeFormatIds" : [ "BF_ALL" ],
      "BinarizationModes" : [
         {
            "BlockSizeX" : 0,
            "BlockSizeY" : 0,
            "EnableFillBinaryVacancy" : 1,
            "ImagePreprocessingModesIndex" : -1,
            "Mode" : "BM_LOCAL_BLOCK",
            "ThreshValueCoefficient" : 10
         }
      ],
      "Description" : "",
      "ExpectedBarcodesCount" : 0,
      "GrayscaleTransformationModes" : [
         {
            "Mode" : "GTM_ORIGINAL"
         }
      ],
      "ImagePreprocessingModes" : [
         {
            "Mode" : "IPM_GENERAL"
         }
      ],
      "IntermediateResultSavingMode" : {
         "Mode" : "IRSM_MEMORY"
      },
      "IntermediateResultTypes" : [ "IRT_NO_RESULT" ],
      "MaxAlgorithmThreadCount" : 4,
      "Name" : "runtimesettings",
      "PDFRasterDPI" : 300,
      "Pages" : "",
      "RegionDefinitionNameArray" : null,
      "RegionPredetectionModes" : [
         {
            "Mode" : "RPM_GENERAL"
         }
      ],
      "ResultCoordinateType" : "RCT_PIXEL",
      "ScaleDownThreshold" : 2300,
      "TerminatePhase" : "TP_BARCODE_RECOGNIZED",
      "TextFilterModes" : [
         {
            "MinImageDimension" : 65536,
            "Mode" : "TFM_GENERAL_CONTOUR",
            "Sensitivity" : 0
         }
      ],
      "TextResultOrderModes" : [
         {
            "Mode" : "TROM_CONFIDENCE"
         },
         {
            "Mode" : "TROM_POSITION"
         },
         {
            "Mode" : "TROM_FORMAT"
         }
      ],
      "TextureDetectionModes" : [
         {
            "Mode" : "TDM_GENERAL_WIDTH_CONCENTRATION",
            "Sensitivity" : 5
         }
      ],
      "Timeout" : 10000
   },
   "Version" : "3.0"
}
```
## How to Distribute

Distribute the required library files with the applications using the Dynamsoft Barcode Reader SDK. The distribution files can be found under:

`DBR-C_CPP-{version number}\DynamsoftBarcodeReader\Lib\{Platform}\`

## How to Upgrade

### From version 8.0 to 8.x

Just replace the old assembly files with the ones in the latest version. Download the latest version [here](https://www.dynamsoft.com/Downloads/Dynamic-Barcode-Reader-Download.aspx). Your existing license for 8.0 is compatible with 8.x.

### From version 7.x

You need to replace the old assembly files with the ones in the latest version. Download the latest version [here](https://www.dynamsoft.com/Downloads/Dynamic-Barcode-Reader-Download.aspx).

Your previous SDK license for version 7.x is not compatible with the version 8.x. Please [contact us](https://www.dynamsoft.com/Company/Contact.aspx) to upgrade your license.

In v8.0, we introduced a new license tracking mechanism, <a href="https://www.dynamsoft.com/license-tracking/docs/about/index.html" target="_blank">License 2.0</a>. 

If you wish to use License 2.0, please refer to [this article](../../license-activation/set-full-license.md) to set the license.

After you upgraded your license to version 8.x:

- If you were using `DBR_InitLicense`, please replace the old license with the newly generated one.

- If you were using `DBR_InitLicenseFromServer` to connect to Dynamsoft server for license verification, then no need to change the license key. But please make sure the device has Internet connection.

- If you were using `DBR_InitLicenseFromServer` + `DBR_InitLicenseFromLicenseContent` to connect to Dynamsoft server once and use the SDK offline, please follow [these steps](../../license-activation/set-full-license-7.md#connect-once) to re-register the device.

- If you were using `DBR_InitLicenseFromLicenseContent` to use the SDK offline, please follow [these steps](../../license-activation/set-full-license-7.md#offline) to re-register the device.

### From version 6.x

We made some structural updates in the new version. To upgrade from 6.x to 8.x, we recommend you to review our sample code and re-write the barcode scanning module.

