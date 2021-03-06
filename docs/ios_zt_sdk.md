iOS + ZeroTier SDK
====

Welcome!

Imagine a flat, encrypted, no-configuration LAN for all of the instances of your iOS app. 

This short tutorial will show you how to enable ZeroTier functionality for your iOS app with little to no code modification. Check out our [ZeroTier SDK](https://www.zerotier.com/blog) page for more info on how the integration works.

***
**Step 1: Build iOS framework**

- `make ios_app_framework`
- This will output to `build/ios_app_framework/Release-iphoneos/ZeroTierSDK_iOS.framework`

**Step 2: Integrate SDK into project**

- Add the resultant framework package to your project
- Add `src` directory to *Build Settings -> Header Search Paths*
- Add `build/ios_app_framework/Release-iphoneos/` to *Build Settings -> Framework Search Paths*
- Add `ZeroTierSDK.frameworkiOS` to *General->Embedded Binaries*
- Add `src/SDK_XcodeWrapper.cpp` and `src/SDK_XcodeWrapper.hpp` to your project:
- Set `src/SDK_Apple-Bridging-Header.h` as your bridging-header in *Build Settings -> Objective-C Bridging-header*

**Step 3: Start the ZeroTier service**

Now find a place in your code to set up the ZeroTier service thread:

```
var service_thread : NSThread!
func zt_start_service() {
    let path = NSSearchPathForDirectoriesInDomains(NSSearchPathDirectory.DocumentDirectory, NSSearchPathDomainMask.UserDomainMask, true)
    start_service(path[0])
}
```

and then start it:

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), {
    self.service_thread = NSThread(target:self, selector:"zt_start_service", object:nil)
    self.service_thread.start()
});
```

**Step 4: Pick an API**

The following APIs are available for this integration:
- `Direct Call`: Consult [src/SDK_Apple-Bridging-Header.h](../../../../src/SDK_Apple-Bridging-Header.h).
- `Hook of BSD-like sockets`: Use BSD-like sockets as you normally would. This likely won't work for calls used by a third-party library.
- `Proxy of NSStream`: Create NSStream. Configure stream for SOCKS5 Proxy (127.0.0.1:PORT). Start Proxy. Use stream.

**Step 5: Join a network!**

Simply call `zt_join_network("nwid")`

***
**NSStream and SOCKS Proxy:**

As an example, here's how one would configure a NSStream object to redirect all network activity to the ZeroTier SOCKS proxy server:

```
// BEGIN proxy configuration
let myDict:NSDictionary = [NSStreamSOCKSProxyHostKey : "0.0.0.0",
                           NSStreamSOCKSProxyPortKey : 1337,
                           NSStreamSOCKSProxyVersionKey : NSStreamSOCKSProxyVersion5]

inputStream!.setProperty(myDict, forKey: NSStreamSOCKSProxyConfigurationKey)
outputStream!.setProperty(myDict, forKey: NSStreamSOCKSProxyConfigurationKey)
// END proxy configuration
```

