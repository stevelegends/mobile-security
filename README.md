## Storing Sensitive Info
Never store sensitive API keys in your app code.
Anything included in your code could be accessed in plain text by anyone inspecting the app bundle.
Tools like [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) and [react-native-config](https://github.com/luggit/react-native-config/) are great for adding environment-specific variables like API endpoints,
but they should not be confused with server-side environment variables,
which can often contain secrets and API keys.

If you must have an API key or a secret to access some resource from your app, the most secure way to handle this would be to build an orchestration layer between your app and the resource. This could be a serverless function (e.g. using AWS Lambda or Google Cloud Functions) which can forward the request with the required API key or secret. Secrets in server side code cannot be accessed by the API consumers the same way secrets in your app code can.

## Async Storage
[Async Storage](https://github.com/react-native-async-storage/async-storage) is a community-maintained module for React Native that provides an asynchronous, unencrypted, key-value store. Async Storage is not shared between apps: every app has its own sandbox environment and has no access to data from other apps.


|  Do use async storage when...	 | Don't use async storage for...|
|---|---|
| Persisting non-sensitive data across app runs	  |  Token storage|
| Persisting Redux state	  | Secrets  |
| Persisting GraphQL state	  |   |
| Storing global app-wide variables	  |   |

## Secure Storage
[iOS - Keychain Services
](https://developer.apple.com/documentation/security/keychain_services) 
allows you to securely store small chunks of sensitive info for the user. This is an ideal place to store certificates, tokens, passwords, and any other sensitive information that doesn’t belong in Async Storage.
[Android - Keystore](https://developer.android.com/training/articles/keystore)
system lets you store cryptographic keys in a container to make it more difficult to extract from the device.
* [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
* [react-native-keychain](https://github.com/oblador/react-native-keychain)


## Authentication and Deep Linking
Deep links are not secure and you should never send any sensitive information in them.

The reason deep links are not secure is because there is no centralized method of registering URL schemes. As an application developer, you can use almost any url scheme you choose by configuring it in Xcode for iOS or adding an intent on Android.

There is nothing stopping a malicious application from hijacking your deep link by also registering to the same scheme and then obtaining access to the data your link contains. Sending something like app://products/1 is not harmful, but sending tokens is a security concern.

When the operating system has two or more applications to choose from when opening a link, Android will show the user a [Disambiguation dialog](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) and ask them to choose which application to use to open the link. On iOS however, the operating system will make the choice for you, so the user will be blissfully unaware. Apple has made steps to address this issue in later iOS versions (iOS 11) where they instituted a first-come-first-served principle, although this vulnerability could still be exploited in different ways which you can [read more about here](https://thehackernews.com/2019/07/ios-custom-url-scheme.html). Using [universal links](https://developer.apple.com/ios/universal-links/) will allow linking to content within your app securely in iOS.

## OAuth2 and Redirects

The OAuth2 authentication protocol is incredibly popular nowadays, prided as the most complete and secure protocol around. The OpenID Connect protocol is also based on this. In OAuth2, the user is asked to authenticate via a third party. On successful completion, this third party redirects back to the requesting application with a verification code which can be exchanged for a JWT — a JSON Web Token. JWT is an open standard for securely transmitting information between parties on the web.

On the web, this redirect step is secure, because URLs on the web are guaranteed to be unique. This is not true for apps because, as mentioned earlier, there is no centralized method of registering URL schemes! In order to address this security concern, an additional check must be added in the form of [PKCE](https://oauth.net/2/pkce/).

[React-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth) can support PKCE only if your Identity Provider supports it.
It wraps the native [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) and [AppAuth-Android](https://github.com/openid/AppAuth-Android) libraries and can support PKCE.

![image](https://github.com/user-attachments/assets/28927195-0ec5-4beb-8c13-f4c8001a861f)

## Network Security
**SSL Pinning** 

Using https endpoints could still leave your data vulnerable to interception. With https, the client will only trust the server if it can provide a valid certificate that is signed by a trusted Certificate Authority that is pre-installed on the client. An attacker could take advantage of this by installing a malicious root CA certificate to the user’s device, so the client would trust all certificates that are signed by the attacker. Thus, relying on certificates alone could still leave you vulnerable to a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).


