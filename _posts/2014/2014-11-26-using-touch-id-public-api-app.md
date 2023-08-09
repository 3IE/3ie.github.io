---
title: "Using Touch ID public API into your own app"
date: "2014-11-26"
categories: 
  - "technical"
tags: 
  - "objective-c"
---

Released with iOS 7, Touch ID allows iPhone 5S and iPad Air users to authenticate with their fingerprint, until now only for AppStore purchases and device unlocking. With iOS 8, Apple released a public API that allows a developer to implement Touch ID authentication in a private app.

First include LocalAuthentication framework into your app via Target > Build Phases > Link Binary With Libraries.

[![LocalAuthentication framework](/assets/images/Screen-Shot-2014-06-03-at-11.43.46.png)](http://nscurious.com/wp-content/uploads/2014/06/Screen-Shot-2014-06-03-at-11.43.46.png)

Once it's done, this code snippet prompts the user to either authenticate via Touch ID, or via password, and the callback give you the process result.

```swift
Context *myContext = [[LAContext alloc] init];
NSError *authError = nil;
NSString *myLocalizedReasonString = @"Please authenticate to access your private photos"; //Provide the reason why you're requesting Touch ID authentication

if ([myContext canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&authError]) {
    [myContext evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics
              localizedReason:myLocalizedReasonString
                        reply:^(BOOL success, NSError *error) {
                            if (success) {
                                NSLog(@"Success");
                            } else {
                                NSLog(@"Failure: %@", error.localizedDescription);
                            }
                        }];
} else {
    NSLog(@"AuthError: %@", authError.localizedDescription);
}
```

However, Apple specifies you must provide the very good reason why you're requesting the user to authenticate in your app ; moreover, the UX impact is significant so that you may use this feature only when there is a clear necessity to.
<br>
<br>

---------------------------------------
<br>
Auteur: **herve.droit**
