---
slug: trying-react-native-1
title: Trying React Native (Part 1)
author: David Wood
author_title: Ex-Infant, Planet Earth
author_image_url: https://www.dropbox.com/s/s7cjocf1dq712qm/Screenshot%202021-03-23%20at%2008.43.40.png?dl=1
---

## Prerequisites

We'll be running an iOS simulator, so you need a Mac - tough luck Edmund. 😈

You'll also need Homebrew.

## Goals

- Find the simplest working configuration for developing with React Native.
- Understand roughly how React Native works at runtime
- Understand how React Native code integrates with iOS and Android
- Understand the React Native build process
- (Optional) Integrate Typescript into the simplest working configuration
- (Optional) Find a way to build (for iOS and Android) using only shell commands

## Setup

To achieve a minimal working example, I've decided to start with the recommended default template and work backwards (removing everything non-essential).

:::note
I looked for the simplest template to start with, but couldn't find one simpler than the default. Most templates seem to think it's helpful to bundle in as much as possible...
:::

### Starting with a template

React Native has an [official CLI](https://github.com/react-native-community/cli) to administrate your project in various ways.
Our first use for it will be to get our starter template.

After the project is initialised, we will install the CLI to `node_modules`, so rather than install it globally (`npm i -g react-native`) for this one occasion, it makes sense to use `npx`.

    npx react-native init [project name] 

:::question What does the CLI do? Is it necessary?

The CLI handles [various tasks](https://github.com/react-native-community/cli/blob/master/docs/commands.md), some of which are relatively simple:
- Starting a development server
- Running device simulators

Others are more complicated:
- Bundling your code (TODO: Learn more about what this process involves)
- Installing and linking native dependencies (required by your project or its libraries) 
In theory, we could forgo the help of the CLI by inputting commands manually or by scripting a similiar tool.

We will use it to get started, but we should look out for any "magic" and try to understand what it is actually executing in those cases.
:::

:::todo 
For each React Native CLI command we use, note what is actually executed...
:::

Before we look at the code, let's make sure the template works by building the project.

### Building the template (before making changes)

For convenience, we'll ignore Android for now and build only for iOS.
Unfortunately, this requires us to install the entire Xcode IDE, 

Unfortunately, these tools seems to *only* be available as part of the full Xcode IDE installation.[^1]

We can at least use this [ neat CLI tool ](https://github.com/mas-cli/mas) to install it!
We'll install it with homebrew:

    brew install mas

Then we'll use it to install Xcode. 

    mas install 497799835

:::note
If you're wondering where the number "497799835" came from, it's [ Xcode's unique ID in the App Store ](https://apps.apple.com/us/app/xcode/id497799835). Yeah, I don't know why we needed that either.
:::

Once Xcode is installed, we have to use this magic command to ask our environment to use Xcode's set of dev tools (instead of the default `CommandLineTools`).

    xcode-select -s /Applications/Xcode.app/Contents/Developer

Finally, lets run the RN CLI's build command for iOS (via the `ios` command defined in our `package.json`):

    yarn ios

Wow! It not only built, but even ran our template successfully in an emulator!

![](https://previews.dropbox.com/p/thumb/ABGK0HNG96W-GgwlOpH75xVOvSCAhQMRmcF-yv0XT54vm87aURWQEUgEdoPx4VpUYY2hCwW5RoxneJgrSRmgQup1uBlrP_M_FnZvyPn4fFkODS56PmNCh1mo8GFVEqfWScA154srHlonx8mx8GY82F6Fn70Xvq5jfgjZftqrxvu3pcPDm2VGaM16pWyMpdFtZlx3CzWQ5s5Xm73qmk84MwK4QmwLjBcgbRbaqHPVeZJTTk609UjTURAJvPedsW2Bxw0_0zc_GescGIJSI4eWqd5x-CAAyI4aDxymVsLnveVYkiUPvtx5102HLzeZNVVhOgC0Ia9SPwuv0RqRtCRQUwQYlHmEJNVsKAS8fbxUa1SpKg/p.png?fv_content=true&size_mode=5)

:::question So what did that `run-ios` command do?

Helpfully, the React Native CLI printed the commands it ran:

    $ react-native run-ios
    info Found Xcode workspace "reactNativeDemo.xcworkspace"
    info Building (using "xcodebuild -workspace reactNativeDemo.xcworkspace -configuration Debug -scheme reactNativeDemo -destination id=ABC0887A-CADA-42B5-B5F1-2C423F29032F")

So the command to build (and presumably run) our project for iOS was: `xcodebuild -workspace reactNativeDemo.xcworkspace -configuration Debug -scheme reactNativeDemo -destination id=ABC0887A-CADA-42B5-B5F1-2C423F29032F`.

Let's break that down...

1. `-workspace reactNativeDemo.xcworkspace`

This looks fairly uninteresting - it's pointing to our project one way or another (though in this case by means of a particular metadata directory `*.xcworkspace`)

2. `-configuration Debug`

This refers to one of the 2 Xcode build configurations included with our template. 
It basically consists of a bunch of enveronment variables.
For example, here is `Pods-ReactNativeDemo.debug.xcconfig` (which defines our "Debug" configuration).

    ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES = YES
    CLANG_WARN_QUOTED_INCLUDE_IN_FRAMEWORK_HEADER = NO
    FRAMEWORK_SEARCH_PATHS = $(inherited) "${PODS_ROOT}/OpenSSL-Universal/Frameworks" "${PODS_XCFRAMEWORKS_BUILD_DIR}/OpenSSL"
    GCC_PREPROCESSOR_DEFINITIONS = $(inherited) COCOAPODS=1 FB_SONARKIT_ENABLED=1
    HEADER_SEARCH_PATHS = $(inherited) "${PODS_ROOT}/Headers/Public" "${PODS_ROOT}/Headers/Public/CocoaAsyncSocket" "${PODS_ROOT}/Headers/Public/DoubleConversion" "${PODS_ROOT}/Headers/Public/FBLazyVector" "${PODS_ROOT}/Headers/Public/FBReactNativeSpec" "${PODS_ROOT}/Headers/Public/Flipper" "${PODS_ROOT}/Headers/Public/Flipper-DoubleConversion" "${PODS_ROOT}/Headers/Public/Flipper-Folly" "${PODS_ROOT}/Headers/Public/Flipper-Glog" "${PODS_ROOT}/Headers/Public/Flipper-PeerTalk" "${PODS_ROOT}/Headers/Public/Flipper-RSocket" "${PODS_ROOT}/Headers/Public/FlipperKit" "${PODS_ROOT}/Headers/Public/RCT-Folly" "${PODS_ROOT}/Headers/Public/RCTRequired" "${PODS_ROOT}/Headers/Public/RCTTypeSafety" "${PODS_ROOT}/Headers/Public/React-Core" "${PODS_ROOT}/Headers/Public/React-RCTBlob" "${PODS_ROOT}/Headers/Public/React-RCTText" "${PODS_ROOT}/Headers/Public/React-callinvoker" "${PODS_ROOT}/Headers/Public/React-cxxreact" "${PODS_ROOT}/Headers/Public/React-jsi" "${PODS_ROOT}/Headers/Public/React-jsiexecutor" "${PODS_ROOT}/Headers/Public/React-jsinspector" "${PODS_ROOT}/Headers/Public/React-perflogger" "${PODS_ROOT}/Headers/Public/React-runtimeexecutor" "${PODS_ROOT}/Headers/Public/ReactCommon" "${PODS_ROOT}/Headers/Public/Yoga" "${PODS_ROOT}/Headers/Public/YogaKit" "${PODS_ROOT}/Headers/Public/glog" "${PODS_ROOT}/Headers/Public/libevent" "$(PODS_ROOT)/boost-for-react-native" "$(PODS_ROOT)/Headers/Private/React-Core" "$(PODS_TARGET_SRCROOT)/include/"
    LIBRARY_SEARCH_PATHS = $(inherited) "${PODS_CONFIGURATION_BUILD_DIR}/CocoaAsyncSocket" "${PODS_CONFIGURATION_BUILD_DIR}/DoubleConversion" "${PODS_CONFIGURATION_BUILD_DIR}/FBReactNativeSpec" "${PODS_CONFIGURATION_BUILD_DIR}/Flipper" "${PODS_CONFIGURATION_BUILD_DIR}/Flipper-DoubleConversion" "${PODS_CONFIGURATION_BUILD_DIR}/Flipper-Folly" "${PODS_CONFIGURATION_BUILD_DIR}/Flipper-Glog" "${PODS_CONFIGURATION_BUILD_DIR}/Flipper-PeerTalk" "${PODS_CONFIGURATION_BUILD_DIR}/Flipper-RSocket" "${PODS_CONFIGURATION_BUILD_DIR}/FlipperKit" "${PODS_CONFIGURATION_BUILD_DIR}/RCT-Folly" "${PODS_CONFIGURATION_BUILD_DIR}/RCTTypeSafety" "${PODS_CONFIGURATION_BUILD_DIR}/React-Core" "${PODS_CONFIGURATION_BUILD_DIR}/React-CoreModules" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTAnimation" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTBlob" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTImage" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTLinking" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTNetwork" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTSettings" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTText" "${PODS_CONFIGURATION_BUILD_DIR}/React-RCTVibration" "${PODS_CONFIGURATION_BUILD_DIR}/React-cxxreact" "${PODS_CONFIGURATION_BUILD_DIR}/React-jsi" "${PODS_CONFIGURATION_BUILD_DIR}/React-jsiexecutor" "${PODS_CONFIGURATION_BUILD_DIR}/React-jsinspector" "${PODS_CONFIGURATION_BUILD_DIR}/React-perflogger" "${PODS_CONFIGURATION_BUILD_DIR}/ReactCommon" "${PODS_CONFIGURATION_BUILD_DIR}/Yoga" "${PODS_CONFIGURATION_BUILD_DIR}/YogaKit" "${PODS_CONFIGURATION_BUILD_DIR}/glog" "${PODS_CONFIGURATION_BUILD_DIR}/libevent"
    OTHER_CFLAGS = $(inherited) -fmodule-map-file="${PODS_CONFIGURATION_BUILD_DIR}/YogaKit/YogaKit.modulemap" -fmodule-map-file="${PODS_ROOT}/Headers/Public/FlipperKit/FlipperKit.modulemap" -fmodule-map-file="${PODS_ROOT}/Headers/Public/React/React-Core.modulemap" -fmodule-map-file="${PODS_ROOT}/Headers/Public/yoga/Yoga.modulemap"
    OTHER_LDFLAGS = $(inherited) -ObjC -l"CocoaAsyncSocket" -l"DoubleConversion" -l"FBReactNativeSpec" -l"Flipper" -l"Flipper-DoubleConversion" -l"Flipper-Folly" -l"Flipper-Glog" -l"Flipper-PeerTalk" -l"Flipper-RSocket" -l"FlipperKit" -l"RCT-Folly" -l"RCTTypeSafety" -l"React-Core" -l"React-CoreModules" -l"React-RCTAnimation" -l"React-RCTBlob" -l"React-RCTImage" -l"React-RCTLinking" -l"React-RCTNetwork" -l"React-RCTSettings" -l"React-RCTText" -l"React-RCTVibration" -l"React-cxxreact" -l"React-jsi" -l"React-jsiexecutor" -l"React-jsinspector" -l"React-perflogger" -l"ReactCommon" -l"Yoga" -l"YogaKit" -l"glog" -l"libevent" -l"stdc++" -framework "AudioToolbox" -framework "CFNetwork" -framework "JavaScriptCore" -framework "MobileCoreServices" -framework "OpenSSL" -framework "Security" -framework "UIKit"
    OTHER_SWIFT_FLAGS = $(inherited) -D COCOAPODS -Xcc -fmodule-map-file="${PODS_CONFIGURATION_BUILD_DIR}/YogaKit/YogaKit.modulemap" -Xcc -fmodule-map-file="${PODS_ROOT}/Headers/Public/FlipperKit/FlipperKit.modulemap" -Xcc -fmodule-map-file="${PODS_ROOT}/Headers/Public/React/React-Core.modulemap" -Xcc -fmodule-map-file="${PODS_ROOT}/Headers/Public/yoga/Yoga.modulemap" -Xcc -DFB_SONARKIT_ENABLED=1
    PODS_BUILD_DIR = ${BUILD_DIR}
    PODS_CONFIGURATION_BUILD_DIR = ${PODS_BUILD_DIR}/$(CONFIGURATION)$(EFFECTIVE_PLATFORM_NAME)
    PODS_PODFILE_DIR_PATH = ${SRCROOT}/.
    PODS_ROOT = ${SRCROOT}/Pods
    PODS_XCFRAMEWORKS_BUILD_DIR = $(PODS_CONFIGURATION_BUILD_DIR)/XCFrameworkIntermediates
    SWIFT_INCLUDE_PATHS = $(inherited) "${PODS_CONFIGURATION_BUILD_DIR}/YogaKit"
    USE_RECURSIVE_SCRIPT_INPUTS_IN_SCRIPT_PHASES = YES

3. `-scheme reactNativeDemo`

For Xcode, a "Scheme" is simply a labelled collection of build settings (for each of the "actions" you can take - e.g. "Test"/"Build"/"Run", etc, see below).

![](https://previews.dropbox.com/p/thumb/ABG66bevfG8a9Day2hCcNXxbcQQaCL3vA91FWCJMe_fH6Wy1NJKRVMiAgVxUMZnRBG5aWsrH-RK9Tp8fg0Qq9M_XK6yeNLp6tb8QjgnxeJ0wduMU3teWqXmMXXiHFMu1TiMxae2txS1vz45joLrr0Epev_jo_suPrKHyzTaAocV_Df76Lj1RlQSukT1SVyNIDzQhmbPVZkyXhINESxQwb2That3gZC63vVDlbbg6ZN9l7mAmxz4Rk1tJD_H7rNiY0a-bV2I5FEsOxmIFXAlW56TwPph4LJrr9e-5GLU-9PLJvkT5PBBEpnysHyei1Cr74H1pETHAJ14jiPJWbAUC0Or_05xUhDEjzuSOCkG78DyVyw/p.png)

4. `-destination id=ABC0887A-CADA-42B5-B5F1-2C423F29032F`

This part is more interesting - it specifies the destination platform, in this case by ID. This ID corresponds to a device simulator which can be found among those listed by the developer tool `simctl`:

    xcrun simctl list devices | --context=10 grep ABC0887A-CADA-42B5-B5F1-2C423F29032F

This produces a list that includes a device with ID `ABC0887A-CADA-42B5-B5F1-2C423F29032F`

    === Devices ===
    -- iOS 14.4 --
    (...)
    iPhone 12 (ABC0887A-CADA-42B5-B5F1-2C423F29032F) (Shutdown)
    (...)

It therefore stands to reason that we can choose any other destination from that list to target a different simulator (e.g. an older model of iPhone) - this may be useful later!

:::
:::info
For more info on Xcode's CLI for building projects, check the manual entry: 

    man xcodebuild
:::

### 

[^1]: To elaborate on why this is necessary, the basic developer tools that are bundled with MacOS lack some tools. At least, I was able to find that `simctl` is not included - check for yourself in `/Library/Developer/CommandLineTools`. There might be other things missing, but if not, there may be hope that `simctl` can be obtained outside of the Xcode package and placed in `/Library/Developer/CommandLineTools/usr/bin`.

[^2]: 