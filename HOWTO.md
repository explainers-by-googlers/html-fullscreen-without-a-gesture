# How to experiment with HTML Fullscreen Without A Gesture in Chrome

A prototype of the proposal may be available in Chrome M123.

**This is intended for experimentation purposes and is subject to change.**

## Enabling in Chrome
1. Download a version of Chrome with the feature implemented. (e.g. [Canary](https://www.google.com/chrome/canary/))
1. Enable the requisites features in Chrome:
    - [Launch with flags](https://www.chromium.org/developers/how-tos/run-chromium-with-flags) `--enable-features=IsolatedWebApps,IsolatedWebAppDevMode,AutomaticFullscreenContentSetting`
    - OR: Enable `chrome://flags/#enable-isolated-web-apps`, `chrome://flags/#enable-isolated-web-app-dev-mode`, and `chrome://flags/#automatic-fullscreen-content-setting`
1. Allow a resource to exercise the feature:
    - Install an IWA, e.g. [iwa-windowing-example](https://github.com/michaelwasserman/iwa-windowing-example) and allow the "Automatic Fullscreen" setting via `chrome://settings/content/siteDetails?site=isolated-app://\<ID\>`
    - OR: Set [enterprise policy](https://www.chromium.org/administrators) to allow [an example](https://michaelwasserman.github.io/iwa-windowing-example/static/), e.g. in `/etc/opt/chrome/policies/managed/test_policy.json` on Linux:
  ```JSON
    "AutomaticFullscreenAllowedForOrigins": [ "https://michaelwasserman.github.io" ]
  ```
1. Exercise the feature:
    - e.g. hover "request on mouseenter" in iwa-windowing-example
1. Exercise multi-screen fullscreen features, e.g. in iwa-windowing-example:
    - use a device with multiple screens
    - click "request on click" and grant the [Window Management](https://developer.mozilla.org/en-US/docs/Web/API/Window_Management_API) prompt
    - allow the "Pop-ups and redirects" setting via `chrome://settings/content/siteDetails?site=isolated-app://\<ID\>`
    - click "open multiple"
