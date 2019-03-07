# Screen capabilities redesign

* Proposal: [SDL-NNNN](NNNN-screen-capabilities-redesign.md)
* Author: [Ashwin Karemore](https://github.com/akaremor), [Kujtim Shala](https://github.com/kshala-ford)
* Status: **Awaiting review**
* Impacted Platforms: [Core / iOS / Android / Web / RPC]

## Introduction

This proposal is about changing display and screen capabilities to be provided over system capabilities.

## Motivation

The RPCs `RegisterAppInterfaceResponse` and `SetDisplayLayoutResponse` contain parameters about display and screen capabilities including text, image and button capabilities. In order to provide a more modern API desgin and ability to resize or reposition screens, this proposal should move metadata out of the repsonse RPCs into a system capability RPC.

## Proposed solution

A new system capability type is necessary in order to provide screen capabilities.

```xml
<enum name="SystemCapabilityType" since="4.5">
    <element name="SCREEN" since="5.x" />
</enum>
```

Apps requesting the screen capabilities can use `GetSystemCapability` and set the capability type to `SCREEN`. The response and `OnSystemCapability` need to be extended to hold screen capabilities.

```xml
<struct name="ScreenCapability" since="5.x">
  <param name="displayName" type="String" mandatory="false">
    <description>The name of the display the app is connected to.</description>
  </param>
  <param name="textFields" type="TextField" minsize="1" maxsize="100" array="true" mandatory="false">
    <description>A set of all fields that support text data. See TextField</description>
  </param>
  <param name="imageFields" type="ImageField" minsize="1" maxsize="100" array="true" mandatory="false">
    <description>A set of all fields that support images. See ImageField</description>
  </param>
  <param name="imageTypeSupported" type="ImageType" array="true" minsize="0" maxsize="1000" mandatory="false" />
  <param name="screenLayoutsAvailable" type="String" minsize="0" maxsize="100" maxlength="100" array="true" mandatory="false">
    <description>A set of all screen layouts available on headunit. To be referenced in SetDisplayLayout.</description>
  </param>
  <param name="numCustomPresetsAvailable" type="Integer" minvalue="1" maxvalue="100" mandatory="false">
    <description>The number of on-screen custom presets available (if any); otherwise omitted.</description>
  </param>
  <param name="buttonCapabilities" type="ButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" />
  <param name="softButtonCapabilities" type="SoftButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" />
</struct>
```

The above struct needs to be added as a parameter into system capability struct. The parameter needs to be of type array to be ready for multi-screen support.

```xml
<struct name="SystemCapability" since="4.5">
  <param name="screenCapabilities" type="ScreenCapability" array="true" minsize="1" maxsize="1000" mandatory="false" />
</struct>
```

As of now in a single-screen environment `screenCapabilities` is related to the app's screen on the main display, hence this parameter requires a minsize of 1, if it's requested by the app.

### Deprecate existing params

With above change, it will be possible to deprecate existing parameters in `RegisterAppInterfaceResponse` and `SetDisplayLayoutResponse`.

```xml
<function name="RegisterAppInterface" functionID="RegisterAppInterfaceID" messagetype="response" since="1.0">
  <param name="displayCapabilities" type="DisplayCapabilities" mandatory="false" deprecated="true" since="5.x">
    <description>See DisplayCapabilities. This parameter is deprecated and replaced by screen capabilities.</description>
    <history>
        <param name="displayCapabilities" type="DisplayCapabilities" mandatory="false" until="5.x"/>
    </history>
  </param>
  <param name="buttonCapabilities" type="ButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" deprecated="true" since="5.x">
    <description>See ButtonCapabilities. This parameter is deprecated and replaced by screen capabilities.</description >
    <history>
        <param name="buttonCapabilities" type="ButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" until="5.x">
    </history>
  </param>
  <param name="softButtonCapabilities" type="SoftButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" deprecated="true" since="5.x">
    <description>If returned, the platform supports on-screen SoftButtons; see SoftButtonCapabilities.  This parameter is deprecated and replaced by screen capabilities.</description >
    <history>
        <param name="softButtonCapabilities" type="SoftButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" since="2.0" until="5.x" />
    </history>
  </param>
  <param name="presetBankCapabilities" type="PresetBankCapabilities" mandatory="false" deprecated="true" since="5.x">
    <description>If returned, the platform supports custom on-screen Presets; see PresetBankCapabilities. This parameter is deprecated and replaced by screen capabilities.</description >
    <history>
        <param name="presetBankCapabilities" type="PresetBankCapabilities" mandatory="false" since="2.0" until="5.x" />
    </history>
  </param>
</function>

<function name="SetDisplayLayout" functionID="SetDisplayLayoutID" messagetype="response" since="3.0">
  <param name="displayCapabilities" type="DisplayCapabilities" mandatory="false" deprecated="true" since="5.x">
    <description>See DisplayCapabilities.  This parameter is deprecated and replaced by screen capabilities.</description>
    <history>
        <param name="displayCapabilities" type="DisplayCapabilities" mandatory="false" until="5.x"/>
    </history>
  </param>
  <param name="buttonCapabilities" type="ButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" deprecated="true" since="5.x">
    <description>See ButtonCapabilities.  This parameter is deprecated and replaced by screen capabilities.</description >
    <history>
        <param name="buttonCapabilities" type="ButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" until="5.x">
    </history>
  </param>
  <param name="softButtonCapabilities" type="SoftButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" deprecated="true" since="5.x">
    <description>If returned, the platform supports on-screen SoftButtons; see SoftButtonCapabilities. This parameter is deprecated and replaced by screen capabilities.</description >
    <history>
        <param name="softButtonCapabilities" type="SoftButtonCapabilities" minsize="1" maxsize="100" array="true" mandatory="false" until="5.x" />
    </history>
  </param>
  <param name="presetBankCapabilities" type="PresetBankCapabilities" mandatory="false" deprecated="true" since="5.x">
    <description>If returned, the platform supports custom on-screen Presets; see PresetBankCapabilities. This parameter is deprecated and replaced by screen capabilities.</description >
    <history>
        <param name="presetBankCapabilities" type="PresetBankCapabilities" mandatory="false" until="5.x" />
    </history>
  </param>
</function>
```

The information contained in the deprecated parameters will be made available with the newly proposed screen capability struct. In the next major release these parameters can be marked as removed.

### Automatic subscription to screen capabilities

As accepted in the app services proposal. The application can subscribe to system capabilities, including screen capabilities. In order to provide screen capabilities as soon as possible after app registration the application should be automatically be subscribed to screen capabilities. With this rule, Core should send a system capability notification with screen capabilities right after sending the response of the app registration. This approach results in a better performance compared to the need of the app to get/subscribe to screen capabilities. 

If an app sends `GetSystemCapability` with `SCREEN` type including `subscribe` flag, Core should ignore a subscription (`subscribe: true`) or unsubscription (`subscribe: false`) and return a message in `GetSystemCapabilityResponse` mentioning that `subscribe` parameter is ignored for `SCREEN` type. The response should contain the screen capability regardless.

Below scenario shows the expected RPCs being send at app registration:

1. App sends `RegisterAppInterface`
2. System responds with `RegisterAppInterfaceResponse`
3. System sends `OnSystemCapability` notification with screen capabilities
4. System sends `OnHMIStatus` notification 
5. System sends `OnPermissionsChange` notification

Another scenario is changing the layout using `SetDisplayLayout:

1. App sends `SetDisplayLayout`
2. System responds with `SetDisplayLayoutResponse`
3. System sends `OnSystemCapability` notification with screen capabilities

In both scenarios it should not be necessary for the app to subscribe to screen capabilities.

## Potential downsides

Moving screen metadata will cause more effort for OEMs and app consumers to implement this feature. The metadata needs to be send twice, in the responses but also in the system capability notification. However as the data is basically just a copy it is expected as an acceptable effort in favor of an improved API design.

## Impact on existing code

No impact to existing code yet. However the screen managers should be refactored to read screen capabilities notifications as well as the deprecated parameters.

## Alternatives considered

It is possible to have a manual subscription to screen capabilities. This is definitely a possible solution as the screen managers will perform the screen capabilities subscription. However, as this subscription will be made for 100% of the apps and as subscribing takes more time sending RPCs it was considered that autosubscription improves performance in this case.

Another alternative, similar to above, allows manual subscritpion but with automatic notifications after `RegisterAppInterfaceResponse` and `SetDisplayLayout`, regardless of the subscription. This would allow applications to subscribe, in order to get notified on HMI changes related to the screen caused by the system or the user. Still it provides automatic notifications if the screen related HMI change is caused by the app (e.g. by changing the layout). 
