# TradingAppStoreExamples-NinjaTrader
## Description
TradingApp.Store offers a comprehensive software suite enabling vendors to verify user permissions for their products. The suite comprises digitally signed Dynamic Link Libraries (DLLs) containing API functions accessible via NinjaTrader scripts integrated into your software.

## Setup
Go to vendors.tradingapp.store, create a vendor account, and click the "Create Listing" button at the top right. Fill out a product SKU (this will identify your testing product on our servers), target platform (NinjaTrader), and an optional username that will be embedded into the license. This is useful if you'd like to offload machine authorization to a third party, like if you have your own username/password login system that you'd like to use. If no username, put "none". After filling out the details, click "Generate MSI" and proceed to install the downloaded MSI file. You can then launch the TradingApp.Store license manager application to see the generated license.  

![TradingApp.Store License Manager](licensemanager_screenshot.png)  

Your system is now ready for seamless integration with our platform.

## How it works
The DLL will automatically detect a license in the TradingAppStore/licenses folder and then will determine if the user has permission. If the license is expired, or a newer version of the license is required, then the DLL will automatically update the license to contain the new information. Consequently, users need only execute the installer once to access any trading apps or software included with their current or future purchases.
The TradingAppStore DLL also offers a hardware authorization option that only allows a certain number of devices to access one instance of your product. This adds an additional layer of security by preventing copies of a DLL / product from gaining permission.
You may download the installer for TradingAppStore from the vendor portal whenever you are in the process of creating a listing. All licenses created from the vendor portal are tagged with a “Debug” flag, so they will not have any functionality in release mode. Thus, BE SURE TO CHANGE THE DEBUG FLAG TO FALSE AFTER COMPLETION OF TESTING PHASES.

## Implementation
After downloading and executing the MSI installer, navigate to C:\ProgramData\TradingAppStore\x64 . Copy the TASDotNet.dll file and paste it into your Documents\NinjaTrader\bin\custom folder. Then, in your NinjaScript file, add that newly added file in the bin\custom folder as a reference to your project by right clicking and selecting “references”.
To access the DLL function, the following lines can be inserted into your software source files:
```C#
using static UserPermission
//…
Print("Starting...");
UserPermission p = new UserPermission();
bool debug =  true, tasAuth = false;
int error = p.GetPermission("NinjaTrader-" + "ACCOUNT-NAME" , "MY-PRODUCT-SKU", debug, tasAuth);
Print(error);
```

Please make sure that the end user knows to copy the TAS_DotNet DLL into the Documents\NinjaTrader\bin\custom folder as well or else your application will throw an error.


## DLL Inputs
The DLL must have 4 input values:
* string customerID :   username of the user
* string productID :    SKU of the product to be checked.
* bool debug :          set to True if you are testing to use Debug licenses distributed by the vendor portal. SET TO FALSE FOR RELEASE OR ELSE ANYONE WILL HAVE ACCESS TO YOUR PRODUCT
* bool TASauth :        Enable this to use our system for user authorization via hardware identifiers. Otherwise, you can use another system like Username / Password

## DLL Return Values
The DLL will return various error values based on numerous factors. It is up to your application how to handle them.
```
0 - no error
1 - expired
2 - wrong customerId
3 - cannot use Debug license in Release Mode
4 - invalid productId
5 - Too many user instances. Only for TAS Authorization. Contact support@tradingapp.store
6 - billing attempt not found... likely expired
8 - File Error
9 - other error
```
## Other Notes
If you are planning on using other apps sold from TradingApp.Store, you must first uninstall the vendor installation and delete the TradingAppStore folder located at C:/ProgramData/ . This will insure that there will be no conflict between the license generated whenever you buy a real product and the debug license used for testing.

## Further Help
If you need assistance in implementation, you may email support@tradingapp.store and we will respond as quickly as possible.
