# FirebaseUI for iOS — Auth

FirebaseUI is an open-source library for iOS that provides simple, customizable UI
bindings on top of [Firebase](https://firebase.google.com) SDKs to eliminate
boilerplate code and promote best practices.

FirebaseUI provides a drop-in auth solution that handles the UI flows for
signing in users with email addresses and passwords, Google Sign-In, and
Facebook Login. It is built on top of [Firebase Auth](https://firebase.google.com/docs/auth).

The FirebaseUI Auth component implement best practices for authentication on
mobile devices and websites, which can maximize sign-in and sign-up conversion
for your app. It also handles edge cases like account recovery and account
linking that can be security sensitive and error-prone to handle correctly.

FirebaseUI can be easily customized to fit in with the rest of your app's visual
 style, and it is open source, so you aren't constrained in realizing the user
 experience you want.

Compatible FirebaseUI clients are also available for [Android](https://github.com/firebase/firebaseui-android/tree/master/auth)
and [Web](https://github.com/firebase/firebaseui-web/).

## Table of Contents

1. [Installation](#installation)
2. [Usage instructions](#using-firebaseui-for-authentication)
3. [Customization](#customizing-firebaseui-for-authentication)

## Installation
### Importing FirebaseUI components for auth
Add the following line to your `Podfile`:
```ruby
pod 'FirebaseUI/Auth'
```

### Configuring sign-in providers
To use FirebaseUI to authenticate users you first need to configure each provider you want to use in
their own developer app settings. Please read the *Before you begin* section of the Firebase
Auth guides at the following links:

- [Email and password](https://firebase.google.com/docs/auth/ios/password-auth#before_you_begin)
- [Google](https://firebase.google.com/docs/auth/ios/google-signin#before_you_begin)
- [Facebook](https://firebase.google.com/docs/auth/ios/facebook-login#before_you_begin)


## Using FirebaseUI for Authentication

### Configuration

All operations, callbacks, UI customizations are done through an `FIRAuthUI`
instance. The `FIRAuthUI` instance associated with the default `FIRAuth`
instance can be accessed as follows:

```swift
// swift
import Firebase
import FirebaseAuthUI

/* ... */

FIRApp.configure()
let authUI = FIRAuthUI.default()
// You need to adopt a FIRAuthUIDelegate protocol to receive callback
authUI?.delegate = self
```

```objective-c
// objc
@import Firebase
@import FirebaseAuthUI
...
[FIRApp configure];
FIRAuthUI *authUI = [FIRAuthUI defaultAuthUI];
// You need to adopt a FIRAuthUIDelegate protocol to receive callback
authUI.delegate = self;
```

This instance can then be configured with the providers you wish to support:

```swift
// swift
import FirebaseGoogleAuthUI
import FirebaseFacebookAuthUI
import FirebaseTwitterAuthUI

let providers: [FIRAuthProviderUI] = [
  FIRGoogleAuthUI(),
  FIRFacebookAuthUI(),
  FIRTwitterAuthUI(),
]
self.authUI?.providers = providers
```

```objective-c
// objc
@import FirebaseGoogleAuthUI
@import FirebaseFacebookAuthUI
@import FIRTwitterAuthUI
...
NSArray<id<FIRAuthProviderUI>> *providers = @[
                                             [[FIRGoogleAuthUI alloc] init],
                                             [[FIRFacebookAuthUI alloc] init],
                                             [[FIRTwitterAuthUI alloc] init],
                                             ];
_authUI.providers = providers;
```

For Google sign in support, add custom URL schemes to your Xcode project
(step 1 of the [implement Google Sign-In documentation](https://developers.google.com/firebase/docs/auth/ios/google-signin#2_implement_google_sign-in)).

For Facebook sign in support, follow step 3 and 4 of
[Facebook login documentation](https://developers.google.com/firebase/docs/auth/ios/facebook-login#before_you_begin)
, and add custom URL schemes following step 5 of [Facebook SDK for iOS-Getting started documentation](https://developers.facebook.com/docs/ios/getting-started).

Finally add a call to handle the URL that your application receives at the end of the
Google/Facebook authentication process.

```swift
// swift
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey : Any]) -> Bool {
  let sourceApplication = options[UIApplicationOpenURLOptionsKey.sourceApplication] as! String?
  if FIRAuthUI.default()?.handleOpen(url, sourceApplication: sourceApplication) ?? false {
    return true
  }
  // other URL handling goes here.
  return false
}
```

```objective-c
// objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options {
  NSString *sourceApplication = options[UIApplicationOpenURLOptionsSourceApplicationKey];
  return [[FIRAuthUI defaultAuthUI] handleOpenURL:url sourceApplication:sourceApplication];
}
```

### Sign In

To start the authentication flow, obtain an `authViewController` instance from
`FIRAuthUI`.  In order to leverage FirebaseUI for iOS you must display the
`authViewController`; you can present it as the first view controller of your
app or present it from another view controller within your app.  In order to
present the `authViewController` obtain as instance as follows:

```swift
// swift

// Present the auth view controller and then implement the sign in callback.
let authViewController = authUI!.authViewController()

func authUI(_ authUI: FIRAuthUI, didSignInWithUser user: FIRUser?, error: Error?) {
  // handle user and error as necessary
}
```

```objective-c
// objc
UIViewController *authViewController = [authUI authViewController];
// Use authViewController as your root view controller,
// or present it on top of an existing view controller.

- (void)authUI:(FIRAuthUI *)authUI didSignInWithUser:(nullable FIRUser *)user error:(nullable NSError *)error {
  // Implement this method to handle signed in user or error if any.
}
```

## Customizing FirebaseUI for authentication
### Terms of Service (ToS) URL customization:

The Terms of Service URL for your application, which is displayed on the
email/password account creation screen, can be specified as follows:

```swift
// swift
let kFirebaseTermsOfService = URL(string: "https://firebase.google.com/terms/")!
authUI?.tosurl = kFirebaseTermsOfService
```

```objective-c
// objc
authUI.TOSURL = [NSURL URLWithString:@"https://example.com/tos"];
```

### Custom strings

You can override the default messages and prompts shown to your users. This can
be useful for things such as adding support for other languages besides English.

In order to do so:

```swift
// swift
authUI?.customStringsBundle = NSBundle.mainBundle() // Or any custom bundle.
```

```objective-c
// objc
authUI.customStringsBundle = [NSBundle mainBundle]; // Or any custom bundle.
```

The bundle should include [.strings](https://github.com/firebase/FirebaseUI-iOS/blob/master/FirebaseAuthUI/Strings/en.lproj/FirebaseAuthUI.strings)
files that have the same names as the default files, namely `FirebaseAuthUI`,
`FirebaseGoogleAuthUI`, and `FirebaseFacebookAuthUI`. Each string in these files
should have the same key as its counterpart in the default `.strings` files.

### Custom sign-in screen

You can customize everything about the authentication method picker screen,
except for the actual sign-in buttons.

In order to do so, create a subclass of `FIRAuthPickerViewController`  and
customize it to your needs. Provide `FIRAuthUI` with an instance of your
subclass by implementing the delegate method
`authPickerViewControllerForAuthUI:` as follows:

```swift
// swift
func authPickerViewController(for authUI: FIRAuthUI) -> FIRAuthPickerViewController {
  return CustomAuthPickerViewController(authUI: authUI)
}
```

```objective-c
// objc
- (FIRAuthPickerViewController *)authPickerViewControllerForAuthUI:(FIRAuthUI *)authUI {
  return [[CustomAuthPickerViewController alloc] initWithAuthUI:authUI];
}
```
