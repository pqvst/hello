---
layout: post
title: Notarize any macOS application
tags: [Tech, Tutorial, macOS]
---

Since macOS Catalina [notarization](https://developer.apple.com/documentation/xcode/notarizing_macos_software_before_distribution) is required to avoid installed apps from being blocked because "Apple cannot check it for malicious software". For anyone who already has an Apple developer account (or is willing to create one) you can easily notarize any macOS application using the Xcode command line tools.

![](/assets/img/notarize/notarization.png)

*Note that if you are shipping a DMG file, you only need to notarize the DMG itself, you do not have to notarize the contents of the DMG file.*

## Generate app-specific password
Generate an app-specific password for your apple developer account:
[https://support.apple.com/en-us/HT204397](https://support.apple.com/en-us/HT204397)

## Start notarization
Open a terminal and define some variables:

```
FILE=/path/to/my/file.dmg
BUNDLE_ID=com.my.app.bundle
USERNAME=me@my-apple-account.com
PASSWORD=my-app-specific-password
```

Start notarization:

```
xcrun altool --notarize-app -f ${FILE} --primary-bundle-id ${BUNDLE_ID} -u ${USERNAME} -p ${PASSWORD}
```

After some time, it should complete and return a UUID:

```
No errors uploading '...'.
RequestUUID = 00001111-2222-3333-4444-555566667777
```

Copy the UUID and define it as a variable:

```
UUID=...
```

## Check progress
Use the following command to check the notarization progress:

```
xcrun altool --notarization-info ${UUID} -u ${USERNAME} -p ${PASSWORD}
```

Once it completes, it should say something like this:

```
No errors getting notarization info.

          Date: ...
          Hash: ...
    LogFileURL: ...
   RequestUUID: ...
        Status: success
   Status Code: 0
Status Message: Package Approved
```

## Staple and verify
Now, staple the file:

```
xcrun stapler staple ${FILE}
```

And validate it:

```
xcrun stapler validate ${FILE}
spctl --assess -vvv --type install ${FILE}
```
