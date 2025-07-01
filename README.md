# Explainer for HTML Fullscreen Without A Gesture

This proposal reflects Chrome's implementation of the Automatic Full Screen content setting:
  - https://chromestatus.com/feature/6218822004768768
  - https://chromeenterprise.google/policies/#AutomaticFullscreenAllowedForUrls
  - chrome://settings/content/automaticFullScreen

This explainer was drafted as an early design sketch by Chrome's Web Apps & Capabilities (Fugu) team to describe the problem below and solicit feedback on the proposed solution.
See the relevant [issue](https://github.com/whatwg/fullscreen/issues/234) and [pull request](https://github.com/whatwg/fullscreen/pull/235) for [Element.requestFullscreen()](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen).

## Proponents

- Chrome's Web Apps & Capabilities (Fugu) team

## Participate
- https://github.com/explainers-by-googlers/html-fullscreen-without-a-gesture/issues
- https://github.com/whatwg/fullscreen/issues/234
- https://github.com/whatwg/fullscreen/pull/235

## Table of Contents

<!-- Update this table of contents by running `npx doctoc README.md` -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Introduction](#introduction)
- [Goals](#goals)
- [Use cases](#use-cases)
- [Enable User Agent configurations that permit fullscreen requests without transient activation](#enable-user-agent-configurations-that-permit-fullscreen-requests-without-transient-activation)
  - [Spec changes](#spec-changes)
  - [Example](#example)
  - [How this solution would solve the use cases](#how-this-solution-would-solve-the-use-cases)
- [Detailed design discussion](#detailed-design-discussion)
  - [Permission API integration](#permission-api-integration)
  - [UI changes](#ui-changes)
  - [Feature detection](#feature-detection)
- [Security Considerations](#security-considerations)
- [Privacy Considerations](#privacy-considerations)
- [Considered alternatives](#considered-alternatives)
  - [Multiple fullscreens from one gesture](#multiple-fullscreens-from-one-gesture)
  - [Fullscreen popups](#fullscreen-popups)
- [Stakeholder Feedback / Opposition](#stakeholder-feedback--opposition)
- [References & acknowledgements](#references--acknowledgements)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

Sites entering [HTML Fullscreen](https://fullscreen.spec.whatwg.org/) require [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) from a user gesture. While that is generally a useful safeguard, it also prohibits some advanced windowing capabilities.

This explainer proposes an algorithmic change to [`Element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen), enabling User Agent configurations that permit fullscreen requests without transient activation.

## Goals

Unlock a broad set of advanced fullscreen capabilities to enable smooth end-user experiences.

## Use cases

Virtual Desktop Infrastructure (VDI) clients connect users with remote desktop or app content from a local device, providing immersive computing experiences on the web. Those apps present compelling use cases for showing content fullscreen without stringent transient activation requirements, particularly on multi-screen devices.

* Show fullscreen remote desktop content on multiple displays with one user gesture
* Automatically extend a fullscreen desktop session onto a newly connected display
* Swap fullscreen windows between displays with one user gesture
* Request fullscreen after transient activation expiry, e.g. slow remote host response
* Open a popup and make it fullscreen with one user gesture
* Apply remote app fullscreen state locally, e.g. on app launch or system events

This functionality would also prove useful for finance, medical, signage, gaming, creativity, and presentation web app use cases.

## Enable User Agent configurations that permit fullscreen requests without transient activation

### Spec changes

This potential solution extends step #5 of the [`Element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen) algorithm to accommodate user agent configurations that permit fullscreen requests without transient activation. See an illustrative PR: [WIP: Enable configuration for fullscreen without transient activation #235](https://github.com/whatwg/fullscreen/pull/235)

Before (*[sic]* [orientation dfn link broken](https://github.com/whatwg/fullscreen/issues/215)):
* [This](https://webidl.spec.whatwg.org/#this)’s [relevant global object](https://html.spec.whatwg.org/multipage/webappapis.html#concept-relevant-global) has [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) or the algorithm is [triggered by a user generated orientation change](https://w3c.github.io/screen-orientation/#dfn-triggered-by-a-user-generated-orientation-change).

After:
* [This](https://webidl.spec.whatwg.org/#this)’s [relevant global object](https://html.spec.whatwg.org/multipage/webappapis.html#concept-relevant-global) has [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation), the algorithm is [triggered by a user generated orientation change](https://w3c.github.io/screen-orientation/#dfn-triggered-by-a-user-generated-orientation-change), or the user agent has been configured to permit fullscreen without [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation).

### Example 

```js
initiateMultiScreenFullscreenButton.addEventListener('click', async () => {
  try {
    const permissionStatus = await navigator.permissions.query({name: 'fullscreen', allowWithoutGesture: true});
    if (permissionStatus.state != 'granted') {
      // Permission is not granted; each window will need a separate gesture to enter fullscreen.
    }
  } catch (error) {
    // Permission is not supported; each window will need a separate gesture to enter fullscreen.
  }

  try {
    const permissionStatus = await navigator.permissions.query({name: 'window-management'});
    if (permissionStatus.state != 'granted') {
      // Permission is not yet granted; the user will be prompted, or each window will need to be placed on other screens manually.
    }
  } catch (error) {
    // Permission is not supported; each window will need to be placed on other screens manually.
  }


  let screenDetails = await window.getScreenDetails();

  // Make the current window fullscreen on its current screen.
  document.documentElement.requestFullscreen({screen : screenDetails.currentScreen});

  // Open a fullscreen popup on each other screen.
  for (let s of screenDetails.screens.filter(s => s !== screenDetails.currentScreen)) {
    let popup = window.open(getUrlForScreen(s), '_blank', `popup,left=${s.availLeft},top=${s.availTop},width=${s.availWidth},height=${s.availHeight}`);
    if (!popup) {
      // Popups are being blocked; this window will need a separate gesture to open each popup window.
    } else {
      popup.addEventListener('load', () => { popup.document.documentElement.requestFullscreen({screen : s}); });
    }
  }
});
```

### How this solution would solve the use cases

This uses existing [Window Management API](https://w3c.github.io/window-management/) surfaces to [request detailed screen information](https://w3c.github.io/window-management/#usage-overview-screen-details) then place [windows](https://w3c.github.io/window-management/#usage-overview-place-windows-on-a-specific-screen) and [fullscreen content](https://w3c.github.io/window-management/#usage-overview-place-fullscreen-content-on-a-specific-screen) on specific screens. Users must grant Window Management permission, and configure their user agent to unblock popups and (NEW) permit fullscreen without transient activation in the relevant context.

Otherwise, corresponding functionality requires cumbersome user interactions to open and fullscreen each window:
* The user must click once to open each popup, or interact with blocked popup UIs, if popups are blocked.
* The user must manually place each popup on its intended screen, if Window Management permission is not granted.
* The user must click each popup to enter fullscreen, if fullscreen without transient activation is blocked.

## Detailed design discussion

The [Window Management API](https://w3c.github.io/window-management/) provides relevant multi-screen content placement features ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window_Management_API)).

### Popup blocking precedent

User Agents widely support configurations that block or allow the creation of popup windows without transient activation. That is reflected in [The rules for choosing a navigable](https://html.spec.whatwg.org/multipage/document-sequences.html#the-rules-for-choosing-a-navigable) when a new [top-level traversable](https://html.spec.whatwg.org/multipage/document-sequences.html#top-level-traversable) is being requested, as invoked by [`Window.open()`](https://html.spec.whatwg.org/multipage/nav-history-apis.html#dom-open-dev):

* If currentNavigable's [active window](https://html.spec.whatwg.org/multipage/document-sequences.html#nav-window) does not have [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) and the user agent has been configured to not show popups (i.e., the user agent has a "popup blocker" enabled)
  * The user agent may inform the user that a popup has been blocked.

This proposal aims to offer similar flexibility for fullscreen window management, so VDI clients and other advanced web apps can offer high quality fullscreen experiences that match user expecations.

### Permissions API integration

This proposal suggests adding a new [powerful feature](https://w3c.github.io/permissions/#dfn-powerful-feature) named `fullscreen` to the [permissions-registry](https://w3c.github.io/permissions-registry). The [Permissions API](https://www.w3.org/TR/permissions/) would support [query](https://www.w3.org/TR/permissions/#query-method) method calls with a custom [PermissionDescriptor](https://www.w3.org/TR/permissions/#dom-permissiondescriptor) that includes a boolean to signify whether the query pertains to fullscreen requests made with or without transient activation from a user gesture.

```IDL
dictionary FullscreenPermissionDescriptor : PermissionDescriptor {
    boolean allowWithoutGesture = false;
};
```

This enables sites to [query](https://developer.mozilla.org/en-US/docs/Web/API/Permissions/query) the user agent's pertinent configuration state in the relevant context.

```JS
navigator.permissions.query({name: 'fullscreen', allowWithoutGesture: true});
```

When permission is `granted`, sites can generally enter [HTML Fullscreen](https://fullscreen.spec.whatwg.org/) without [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) from a user gesture.

This proposal suggests the `{name: 'fullscreen', <options>}` descriptor shape in order to support future prospective queries pertaining to the Fullscreen API. The Permissions API may yield a `TypeError` exception for queries when `allowWithoutGesture` is false or unspecified, as it does for unknown descriptor names, representing that no corresponding permission state exists.

### Permissions Policy integration

This proposal suggests reusing the existing [policy-controlled feature](https://github.com/w3c/webappsec-permissions-policy/blob/main/features.md) named `fullscreen`, defined in the [Fullscreen API Standard](https://fullscreen.spec.whatwg.org/#permissions-policy-integration), to convey configuration state to specific frames.

For example, allowing the `fullscreen` policy-controlled feature on an embedded iframe also conveys any `automatic-fullscreen` permission grant to the embedded context (i.e. `<iframe allow="fullscreen *">`). Similarly, denying the `fullscreen` policy-controlled feature denies any `automatic-fullscreen` permission grant.

So if an embedded iframe is allowed the `fullscreen` policy-controlled feature, and the top-level frame has been granted the `automatic-fullscreen` permission, then the embedded frame can also generally enter [HTML Fullscreen](https://fullscreen.spec.whatwg.org/) without [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation).

### Feature detection

Feature detection is valuable for sites that wish to use this feature. The proposed Permissions API integration enables sites to determine whether the user agent has been configured to allow [HTML Fullscreen](https://fullscreen.spec.whatwg.org/) without [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) in the given context. Sites can also determine whether the user agent potentially supports such configuration in the given context, by checking if their Permissions API query yields a result or an exception.

### UI changes

User agents share some common patterns around UI treatments for setting configurations, [Fullscreen UI](https://fullscreen.spec.whatwg.org/#ui), and blocked popup notifications. This proposal doesn’t make specific UI recommendations.

## Security Considerations

Fullscreen web content poses spoofing risks and other usable security concerns. Sites entering fullscreen without transient activation will exacerbate those concerns.

User agents may provide controls (e.g. application settings) rather than permission prompts. That allows savvy users or enterprise administrators to grant this infrequently needed powerful web capability with adequate disclaimers in trusted contexts, while preventing sites from prompting users in drive-by web experiences. User agents could reasonably restrict this configuration to security-sensitive apps, like Chrome’s [Isolated Web Apps](https://chromestatus.com/feature/5146307550248960).

User Agents could improve their own fullscreen user interfaces to better convey window states and state transitions, especially in sensitive situations. User agents could present blocking or persistent user interface surfaces with prominent security context when sites enter fullscreen without transient activation.

User Agents can also implement mitigations against suspected abuse of this new feature, like requiring transient activation to re-enter fullscreen for a short time after the user exits fullscreen.

## Privacy Considerations

This feature does not directly expose any information to sites, and there are no significant privacy considerations to note.

## Considered alternatives

### Multiple fullscreens from one gesture

Another [explainer](https://github.com/w3c/window-management/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) considered initiating multi-screen experience from a single user gesture. That was far more complex and restrictive, but most importantly, VDI web development partners ultimately require some functionality without any user gesture.

### Fullscreen popups 

Another [explainer](https://github.com/w3c/window-management/blob/main/EXPLAINER_fullscreen_popups.md) and ongoing [chrome experiment](https://chromestatus.com/feature/6002307972464640) ([blog post](https://developer.chrome.com/blog/fullscreen-popups-origin-trial/)) considered creating fullscreen popup windows, but that did not meet VDI web development partner requirements, particularly for making pre-existing windows fullscreen, and waiving user gesture requirements.

### Permissions alternatives

An alternative Permissions API query descriptor shape might be useful if using `fullscreen` were not a suitable powerful feature name: `navigator.permissions.query({name: 'automatic-fullscreen'});`.

A `Document.fullscreenRequiresTransientActivation` boolean could parallel the existing [`Document.fullscreenEnabled`](https://fullscreen.spec.whatwg.org/#ref-for-dom-document-fullscreenenabled), but seems inherently inferior to Permissions API integration.

A new permissions policy-controlled feature would only be useful if sites needed to convey `fullscreen`, but *not* the `automatic-fullscreen` permission.

A hacky alternative could require sites to call requestFullscreen on a [detached DOM element](https://github.com/explainers-by-googlers/locked-mode/issues/6) without a gesture, then assess the error type or message. This would require changing a lack of transient activation to take precedence over the detached node error, potentially changing Error types thrown, and sites would need fragile per-browser logic:

```JS
if (!navigator.userActivation.isActive) {
  document.createElement('div').requestFullscreen().catch(e => {
    if (e instanceof TypeError) {
      if (e.message === "Permissions check failed")
        console.log("Fullscreen requires a gesture");
      else if (e.message === "Element is not connected")
        console.log("Fullscreen does not require a gesture");
  });
}
```

## Stakeholder Feedback / Opposition

Web Developers have requested relevant functionality in the [Window Management API Issue tracker](https://github.com/w3c/window-management/issues):
 - [window.open should support the 'fullscreen' option #7](https://github.com/w3c/window-management/issues/7)
 - [Feature Request: Initiate a multi-screen experience from a single user activation #98](https://github.com/w3c/window-management/issues/98)
 - [Feature request: Fullscreen support on multiple screens #92](https://github.com/w3c/window-management/issues/92)

Publicly stated positions from implementors have been requested:
- https://github.com/mozilla/standards-positions/issues/1020
- https://github.com/WebKit/standards-positions/issues/345

Additional discussion can be found on [Fullscreen API Issues](https://github.com/whatwg/fullscreen/issues):
- [Entering fullscreen without transient activation #234](https://github.com/whatwg/fullscreen/issues/234)
  - PR [WIP: Enable configuration for fullscreen without transient activation #235](https://github.com/whatwg/fullscreen/pull/235)
- This compliments [Proposal: Supporting fullscreen requests in multi-screen environments. #161](https://github.com/whatwg/fullscreen/issues/161)

## References & acknowledgements

Many thanks for valuable feedback and advice from many folks, especially:

- ayuishii
- bradtriebwasser
- takumif
