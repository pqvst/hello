---
layout: post
title:  "Integrating iOS 12 Siri Shortcuts using Objective-C"
comments: true
---

A simple, straight-forward, step-by-step guide to implementing iOS 12 Siri Shortcuts using Objective-C. I noticed that the documentation for integrating the new iOS Shortcuts API is very sparse. After a lot of trial and error, combined with reading the official docs and various tutorials and forum threads, I managed to get everything to work.

![Shortcuts Demo](/assets/images/ShortcutsDemo.png)

## Introduction

Siri Shortcuts is a new feature in iOS 12 which enables 3rd party apps on iOS to be controlled using Siri. Essentially Siri "learns" as you use apps and then suggests actions. Furthermore, the new [Shortcuts app](https://itunes.apple.com/us/app/shortcuts/id915249334?mt=8){:target="_blank"} let's you build more complex sequences of actions, which is actually really cool.

For this tutorial I will be showing all of the steps I took to integrate shortcuts in [Unified Remote](https://itunes.apple.com/us/app/unified-remote/id825534179?mt=8){:target="_blank"}. Unified Remote is an app that turns your smartphone into a remote control for your computer. By integrating Shortcuts into Unified Remote you will be able to create shortcuts to a open remote on your device and send remote actions directly using Siri Shortcuts or the Shortcuts app.


## Implementing Shortcuts

Apps can [Donate Shortcuts](https://developer.apple.com/documentation/sirikit/donating_shortcuts?language=objc){:target="_blank"}, which basically just means making iOS aware of what shortcuts are available in your app. There are two ways to donate shortcuts:

1. Donating a *User Activity* - A screen of your app that can be resumed.
2. Donating an *Interaction* - An action that can be executed directly from a shortcut.

This tutorial will cover all required steps for donating and implementing both of the above. In my case, the goal was to donate a user activity whenever the user opens a remote in the app, and to donate an interaction when they trigger an action inside a remote.

![App Demo](/assets/images/AppDemo.png)

## Donating a User Activity

1) Choose a screen where you want to donate a user activity and create an `NSUserActivity`.

* Construct a `NSUserActivity` in `viewDidLoad` using an appropriate "activity type" string. 
* Choose a title.
* Make sure you enable `eligibleForPrediction`
* Add any `userInfo` that you need to be able resume the screen later.
* Set `requiredUserInfoKeys` (I'm not sure if this step is required).
* IMPORTANT: Assign your `NSUserActivity` to the view controller's built-in `userActivity` property. The view controller automatically invokes `becomeCurrent` and `resignCurrent` for us. At first I tried doing this manually but that does not seem to work. 

Be sure to wrap your code with `@available(iOS 12.0, *)` if your app supports older versions of iOS. Choose an appropriate "activity type". You can choose whatever you want, but it should match whatever your user activity is.

```objective_c
- (void)viewDidLoad
{
    // Create NSUserActivity for Siri Shortcuts
    if (@available(iOS 12.0, *)) {
        NSUserActivity* userActivity = [[NSUserActivity alloc] initWithActivityType:@"com.unified.UR.Remote"];
        userActivity.title = [NSString stringWithFormat:@"Open %@ Remote", self.remote.name];
        userActivity.eligibleForPrediction = YES;
        userActivity.userInfo = @{@"ID" : self.remote.ID};
        userActivity.requiredUserInfoKeys = [NSSet setWithArray:userActivity.userInfo.allKeys];
        self.userActivity = userActivity; // Calls becomeCurrent/resignCurrent for us...
    }
}
```

Also implement the `updateUserActivityState` function as well since (for whatever reason) the `userInfo` gets cleared by the system see [this Apple forums thread](https://forums.developer.apple.com/thread/9690){:target="_blank"} and [this medium article](https://medium.com/@edwardmp/common-misconception-when-creating-nsuseractivity-in-view-controller-3df8a825095b){:target="_blank"} for more details.

```objective_c
-(void)updateUserActivityState:(NSUserActivity *)userActivity 
{
    [userActivity addUserInfoEntriesFromDictionary:@{@"ID" : self.remote.ID}];
}
```

3) Edit your app's `Info.plist` file.

* Add a new row called `NSUserActivityTypes`.
* Right-click and select `Value Type > Array`.
* Add another row below.
* Right-click and select `Shift Row Right`.
* Then add your "activity type" that you chose above.

![NSUserActivityTypes](/assets/images/AddNSUserActivityTypesToPlist.png)

4) Test if your user activity is getting donated.

* Open the screen in your app.
* Go to iOS `Settings > Siri & Search` and check under `Suggested Shortcuts`. 

Note, you may have to trigger it a few times, or select `All Shortcuts` and search for your shortcut.

![NSUserActivityTypes](/assets/images/UserActivityDemo.png)


## Handling a User Activity

Simply add the `continueUserActivity` handler to your `AppDelegate`.

```objective_c

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> *restorableObjects))restorationHandler
{
    // Check to make sure it's the correct activity type
    if ([userActivity.activityType isEqualToString:@"com.unified.UR.Remote"]) 
    {
        // Extract the remote ID from the user info
        NSString* id = [userActivity.userInfo objectForKey:@"ID"];

        // Restore the user activity...
        ...

        return YES;
    }
    return NO;
}

```



## Donating an Interaction

1) Add an `Intent Definition File` to your main app target so that we can define a custom intent.

* Right-click in your project structure
* Select `New File...`
* Add a `SiriKit Intent Definition File`.

![IntentDefinitionFile](/assets/images/AddIntentDefinitionFile.png)

2) Create a new custom intent by clicking the `+` in the bottom left corner and give your intent a name.

![Intent](/assets/images/AddIntent.png)

3) Configure the intent settings and add any parameters that you need.

* Select a category (I'm not sure if this actually matters but I chose whatever seemed most fitting).
* Enter a default `Title` and `Description`.
* Add any parameters that you want to associate with your intent. Note that I add both the "internal" parameters needed for executing the intent later `remote` and `action`, plus parameters for constructing pretty UI messages `remoteName` and `actionName`.
* Add one or more shortcut types. A shortcut type is essentially just a combination of parameters. For example, if you have valid types that accept different combinations of parameters then you have to create a type for each combination.
* For each shortcut type, customize the title using the parameter values. 
* Make sure to enable `Supports background execution` if you want to.

![Configure Intent](/assets/images/ConfigureIntent.png)

4) Next you will want to actually implement where you donate the interaction shortcut. First of all, you need to import the `Intents/Intents.h` header to be able to use the `INInteraction` class. You also have to import the generated header for your custom intent. This was a bit tricky to figure out (and not documented at all). Basically it's just `[ClassPrefix][IntentName]Intent.h`.

```objective_c
// Import intent headers
#import <Intents/Intents.h>
#import "URActionIntent.h"
```

5) Find a suitable place in your code to "donate" the interaction shortcut. In my case, this is whenever a user triggers an action inside a remote control.

```objective_c
- (void)userTriggeredAnAction:(DataAction*)action
{
    // Constructor intent and set parameters
    // IMPORTANT: Parameters must match one of the shortcut types that you defined in the intents definition file. Otherwise the donation with fail.
    URActionIntent* intent = [[URActionIntent alloc] init];
    intent.remote = self.remote.ID;
    intent.remoteName = self.remote.name;
    intent.action = action.name;
    intent.actionName = [[action.name stringByReplacingOccurrencesOfString:@"_" withString:@" "] capitalizedString];

    // Set a suggested phrase (displayed when creating shortcuts)
    intent.suggestedInvocationPhrase = [NSString stringWithFormat:@"%@ %@", intent.remoteName, intent.actionName];

    // Donate the interaction shortcut
    INInteraction* interaction = [[INInteraction alloc] initWithIntent:intent response:nil];
    [interaction donateInteractionWithCompletion:^(NSError * _Nullable error) 
    {
        if (error) 
        {
            NSLog(@"Failed to donate interaction: %@ ", [error localizedDescription] );
        }
    }];
}
```


## Handling an Interaction

This was by far the most difficult step to figure out, especially since the documentation is very lacking.

1) Enable the `Siri Capability` for your main app target.

* Select your project in the project structure
* Select your main app target.
* Select the `Capabilities` tab.
* Enable `Siri`.

![Enable Siri Capability](/assets/images/EnableSiriCapability.png)

2) Add an `Intents Extension` to your project. (See [Apple Documentation](https://developer.apple.com/documentation/sirikit/creating_an_intents_app_extension?language=objc) for more details.)

* Click `File > New > Target...`
* Select `Intents Extension`
* Enter a product name (e.g. `Intents`)
* Set `Starting Point` to `None`
* Deselect `Include UI Extension`

![Add Intents Extension](/assets/images/AddIntentsExtension.png)

3) Add the `Intents Definition File` to your `Intents Extension` target. Note: I tried moving the intents definition file to a shared Cocoa Touch Framework that both the main app and the intents extension had access to, however that does not seem to work. So, I recommend just adding both targets to the target membership setting instead.

* Select the intents definition file that you created previously.
* Right-click `Show File Inspector`.
* Add your Intents Extension under `Target Membership`.

![Update Target Membership](/assets/images/UpdateIntentDefinitionFileTargetMemberships.png)

3) Enable your custom intent in the Intents Extension
* Open `Info.plist` for your Intents Extension
* Expand `NSExtension > NSExtensionAttributes > IntentsSupported`
* Add the *full* intent name to the list (i.e. `[ClassPrefix][IntentName]Intent`).

![Update Intents Extension Info Plist](/assets/images/UpdateIntentExtensionInfoPlist.png)

4) Open `IntentHandler.m` in your Intents Extension and implement your custom intent. Make sure to import your intent header and add your generated intent handling protocol as well (`[ClassPrefix][IntentName]IntentHandling`). 

```objective_c
#import "URActionIntent.h"

@interface IntentHandler () <URActionIntentHandling>

@end

@implementation IntentHandler

- (void)handleAction:(nonnull URActionIntent *)intent completion:(nonnull void (^)(URActionIntentResponse * _Nonnull))completion {
    // Do something...
}

@end
```

5) Keep in mind that the Intent Extension is a completely separate module, so if you need to access code from your main app you may need to refactor and share the code between the two targets. In my case, I decided to move a lot of my app backend logic to a new `Cocoa Touch Framework` target that both the main app target and the intents app target depend on. Note: Even "user defaults" are separate. However, you can share user defaults between both targets using [App Groups](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html).

![Shared Framework](/assets/images/SharedFramework.png)

6) To test and debug your `Intent Extension`, choose the intent target and click run. When asked to choose an app to run click `Siri`. Once Siri starts on your phone you can test your shortcut there, or switch to the Shortcuts app and test it there as well.

![Debug Intents Extension](/assets/images/DebugIntentsExtension.png)

---

Check out Unified Remote in the [App Store](https://itunes.apple.com/us/app/unified-remote/id825534179?mt=8) or on [Google Play](https://play.google.com/store/apps/details?id=com.Relmtech.Remote)!