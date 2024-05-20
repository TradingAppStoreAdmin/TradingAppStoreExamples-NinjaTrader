# TradingAppStoreExamples-NinjaTrader
## Description
TradingApp.Store offers a comprehensive software suite enabling vendors to verify user permissions for their products. The suite comprises digitally signed Dynamic Link Libraries (DLLs) containing API functions accessible via NinjaTrader scripts integrated into your software.

## Setup
Go to [vendors.tradingapp.store](https://vendors.tradingapp.store/app), create a vendor account, and click the 'Create Listing' button at the top right. Proceed to fill out the Product Listing form.  Instructions below.

### Product Name:
Fill out a Product Name at the top left and a SKU will be automatically produced at the bottom of the page (this will identify your product on our servers). 

### Product Description:
Create a formated product description in an editor like Word or Google Docs and then paste it into the Product Description text-box.  Read over it and do a final editing of the formatting to make sure it looks right.  The product description should fully explain your product, its features, benefits, and how it works.  Give as much detail as possible.  

### Images:
Add images and/or videos one at a time.  Most file common file types are accepted.  You can also use online videos from YouTube via the URL link.

### Listing Types:
Check all that apply to this product.  Your product may fit into more than one Listing Type.  

### Subscription Options:
Choose the type of billing scheme to use. Available options are: Lifetime, Annual, Monthly, Free Trial + Monthly, Free Trial + Annual, Free Trial + Lifetime.

### Webhook Link:
If you have a real-time listening application that works with Webhooks, paste the link to it here to be notified when a purchase for this product is made.

### Purchase Email:
If you would like email notifications upon purchases, place the receiving address here.

### Target Platform:
Choose NinjaTrader

### NinjaTrader Username
Enter your NinjaTrader Username. This value will be embedded into the license so that you can check it at runtime whenever you access our DLL.

### SKU:
Unique self-created product identifier that you will use in your script while accessing the DLL.

### Download MSI:
(NOTE: You must pick a 'Target Platform' and enter 'NinjaTrader Username' in order for the 'Download MSI' button to appear.)  Press this to download a copy of the 'TradingApp.Store License Manager' that is master-keyed to this specific product and specifically to your NinjaTrader username.  This provides you, the Vendor, the ability to test the integrations of your products with our DLLs.  After installing this MSI, launch the TradingApp.Store license manager application to see the generated license.  


![TradingApp.Store License Manager](licensemanager_screenshot.png)  

Your system is now ready for seamless integration with our platform.

## How it works
The DLL will automatically detect a license in the TradingAppStore/licenses folder and then will determine if the user has permission. If the license is expired, or a newer version of the license is required, then the DLL will automatically update the license to contain the new information. Consequently, users need only execute the installer once to access any trading apps or software included with their current or future purchases.
The TradingAppStore DLL also offers a hardware authorization option that only allows a certain number of devices to access one instance of your product. This adds an additional layer of security by preventing copies of a DLL / product from gaining permission.
You may download the installer for TradingAppStore from the vendor portal whenever you are in the process of creating a listing. All licenses created from the vendor portal are tagged with a “Debug” flag, so they will not have any functionality in release mode. Thus, BE SURE TO CHANGE THE DEBUG FLAG TO FALSE AFTER COMPLETION OF TESTING PHASES.

## DLL Inputs
The DLL must have 3 input values:
* string customerID :   username of the user
* string productID :    SKU of the product that was self generated above.
* bool debug :          set to True if you are testing to use Debug licenses distributed by the vendor portal. SET TO FALSE FOR RELEASE OR ELSE ANYONE WILL HAVE ACCESS TO YOUR PRODUCT

## Implementation
The following is an example implementation that halts the OnBarUpdate() event of an indicator in the case that the user does not have a valid subscription to the indicator.
```C#
// these using statements support our dll import and assembly import logic
using System.Reflection;
using System.Runtime.InteropServices;
// other using statements here...

namespace NinjaTrader.NinjaScript.Indicators
{
    public class MyCustomIndicator1 : Indicator
    {
        protected override void OnStateChange()
        {
            // onStateChange logic...
        }
	
        // Import method used to check if the current user has access to the program
        [DllImport("C:\\ProgramData\\TradingAppStore\\x64\\TASlicense.dll")]
        private static extern int UseMachineAuthorization(string productId, bool debug);

        // whether we ran hasPermission() yet or not
        private static bool ran = false;
		// caching the result of hasPermission() after first run
        private static bool verified = false;
        
        // helper method that returns whether or not user authentication was successful. Called from OnBarUpdate()
        private bool hasPermission()
        {
	    // if we already ran, return our cached result
            if (ran){
               return verified;
            }
            ran = true;

            //Before you use the dlls, you should first make sure that they have not been tampered with.
	    if (!VerifyDlls()){
		Print("DLLs denied");
                return false; // VERY IMPORTANT: Handle the case for if either verification fails. Do not use the library code! In this example, we simply return to terminate the program.
            }
            
            // set this to your product sku
            string productID = "Grant_CSecurityTest";
            bool debug = true; // VERY IMPORTANT: Only set this to true during testing. Actual implementation will have debug set to false.

            //Perform user authentication using TAS authorization
            int error_machine_auth = UseMachineAuthorization(productID, debug);

            if (error_machine_auth == 0)
            {
                verified = true;
                return true;
            }
            else
            {
                Print("Access denied. Error: " + error_platform_auth.ToString());
                return false;
            }
        }
	
        // helper method called by hasPermission() that returns whether or not the dlls are up to date and have not been tampered with.
        private bool VerifyDlls()
        {
            //This gets a one-time-use magic number from a utility dll
	    string magicNumber = (string)Assembly.LoadFrom(@"C:\ProgramData\TradingAppStore\x64\Utils_DotNet.dll").GetType("Utils").GetMethod("ReceiveMagicNumber", BindingFlags.Static | BindingFlags.Public, null, CallingConventions.Any, Type.EmptyTypes, 	null).Invoke(null, null).ToString();
            var jsonString = "{\"magic_number\" : \"" + magicNumber + "\"}";
            //Now, let's send that magic number to our server to be verified
            using (var client = new HttpClient())
            {
                var content = new StringContent(jsonString, Encoding.UTF8, "application/json");
                var response = client.PostAsync("https://tradingstoreapi.ngrok.app/verifyDLL", content).Result;

                if (response.StatusCode == System.Net.HttpStatusCode.OK)
                {
                    Print("DLL accepted"); //After verifying the DLLs, you can safely use them to authorize your customers.
                    return true;
                }
                else if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
                {
                    Print("DLL has been tampered with.");
                    return false;
                }
                else
                {
                    Print($"Error: {response.StatusCode}");
                    return false;
                }
            }
        }
	
	// indicator event. We cut off the logic in the case that the auth failed	
        protected override void OnBarUpdate()
        {
            // We call our hasPermission method here
            if (!hasPermission()){
		Print("User does not have permission!");
                return; // VERY IMPORTANT: Be sure to handle the case for when a user doesn't have access. In this example, we simply return to terminate the method before any further indicator logic ensues.
            }
            //Add your custom indicator logic here...
        }
    }
}

```

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
7 - internal error
8 - File Error
9 - other error
```

## Finishing Up
Go back to the Vendor Portal to complete your product setup.

### Upload Software Here:
Once your product is successfully integrated into our permissions system, take the product out of debug mode (see bool debug above), and export your project.  We recommend using NinjaTrader's Instructions here:  https://ninjatrader.com/support/helpGuides/nt8/NT%20HelpGuide%20English.html?distribution_procedure.htm
If you have accompanying files, workspaces, symbol lists, etc, it is required that you zip everything into one file, and then upload it here.  This is what will be distributed to end-users at the time of purchase or free trial.

### Sales Information - Set Price:
This is the price per period for the subscription term of the product.  Revenue splits are explained in the Vendor Policy (https://tradingapp.store/pages/vendor-policy).

### Send for approval:
Click here to send this listing for approval by TAS site moderators.  You will be notified by email upon acceptance or rejection.

## Other Notes
If you are planning on using other apps sold from TradingApp.Store as an end-user, you must first uninstall the vendor installation of the TAS License Manager using Windows 'Add or Remove Programs' in Settings and then install the MSI received at the time of purchase.  We are currently working on building a dual-mode License Manager, so for now, you have to either work with two computers, or uninstall/reinstall if you want to switch from vendor-mode to end-user mode.

## Further Help
If you need assistance in implementation, you may email support@tradingapp.store and we will respond as quickly as possible.
