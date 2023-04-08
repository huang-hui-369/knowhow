# Running an Application on a Device

You will need an Apple Developer Program account to run apps on a device and submit apps to the App Store. A free account will allow you to run on a device, but you will not be able to submit apps to the App Store without a paid account. 

## registered for a developer account

There are four important items in the provisioning process:

| Developer Certificate   | This certificate file is added to your Mac’s keychain, which you can view in Keychain Access. It is used to digitally sign your code. |
| ----------------------- | ------------------------------------------------------------ |
| App ID                  | The application identifier is a string that uniquely identifies your application on the App Store. Application identifiers typically look like this: com.bignerdranch.AwesomeApp, where the name of the application follows the name of your company.The App ID in your provisioning profile must match the bundle identifier of your application. A development profile can have a wildcard character (*) for its App ID and therefore will match any bundle identifier. To see the bundle identifier for the Quiz application, select the project from the top of the project navigator, then select the Quiz target. Finally, select the General pane from the menu across the top of the editor. |
| Unique Device ID (UDID) | This identifier is unique for each iOS device.               |
| Provisioning Profile    | This is a file that lives on your development device and on your computer. It references a Developer Certificate, a single App ID, and a list of the device IDs for the devices that the application can be installed on. This file is suffixed with .mobileprovision. |

When an application is deployed to a device, Xcode uses a provisioning profile on your computer to access the appropriate certificate. This certificate is used to sign the application binary. Then, the development device’s UDID is matched to one of the UDIDs contained within the provisioning profile, and the App ID is matched to the bundle identifier. The signed binary is then sent to your development device, where it is confirmed by the same provisioning profile on the device and, finally, launched.

## add the account to Xcode.

 Open Xcode and select Xcode → Preferences. Select the Accounts tab and click the + button in the bottom-left corner. Choose Apple ID and then sign in to your developer account.

![image-20221208180319064](D:\github\knowhow\ios-app\tutorial\firstApp\008-runInDevice.assets\image-20221208180319064.png)

##  Signing & Capabilities pane 

Select the Quiz project in the project navigator, select the Quiz target, and then open the Signing & Capabilities pane. Make sure the Automatically manage signing checkbox is checked. Then, from the Team drop-down menu, select your developer account.

![image-20221208180839951](D:\github\knowhow\ios-app\tutorial\firstApp\008-runInDevice.assets\image-20221208180839951.png)