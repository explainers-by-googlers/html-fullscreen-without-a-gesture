# How to experiment with HTML Fullscreen Without A Gesture in Chrome

A prototype of the proposal is available in Chrome M124.

**This is intended for experimentation purposes and is subject to change.**

## Enabling in Chrome
1. Download a version of Chrome M124+ with the feature implemented. (e.g. [Canary](https://www.google.com/chrome/canary/))
1. Enable the requisites features in Chrome:
    - [Launch with flags](https://www.chromium.org/developers/how-tos/run-chromium-with-flags) `--enable-features=AutomaticFullscreenContentSetting,IsolatedWebApps,IsolatedWebAppDevMode`
    - OR: Enable `chrome://flags/#automatic-fullscreen-content-setting`, `chrome://flags/#enable-isolated-web-apps`, `chrome://flags/#enable-isolated-web-app-dev-mode`
1. Allow a resource to exercise the feature:
    - Install an IWA, e.g. [iwa-windowing-example](https://github.com/michaelwasserman/iwa-windowing-example) and allow the "Automatic Fullscreen" setting via `chrome://settings/content/siteDetails?site=isolated-app://\<ID\>`
    - OR: Set [enterprise policy](https://www.chromium.org/administrators) to allow [an example](https://michaelwasserman.github.io/iwa-windowing-example/static/), e.g. in `/etc/opt/chrome/policies/managed/test_policy.json` on Linux:
  ```JSON
    "AutomaticFullscreenAllowedForUrls": [ "https://michaelwasserman.github.io" ]
  ```
1. Exercise the feature, e.g. in iwa-windowing-example:
    - Hover "request on mouseenter" when User Activation is not "Active"
    - Click "request after 6s", and refrain from further gestures
    - Click "open" for either Fullscreen Popup example implementation
1. Exercise multi-screen fullscreen features, e.g. in iwa-windowing-example:
    - use a device with multiple screens
    - click "request on click" and grant the [Window Management](https://developer.mozilla.org/en-US/docs/Web/API/Window_Management_API) prompt
    - allow the "Pop-ups and redirects" setting via `chrome://settings/content/siteDetails?site=isolated-app://\<ID\>`
    - click "one per screen" for either Fullscreen Popup example implementation
