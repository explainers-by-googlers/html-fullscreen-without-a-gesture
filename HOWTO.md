# How to experiment with HTML Fullscreen Without A Gesture in Chrome

A prototype of the proposal may be available in Chrome M123.

**This is intended for experimentation purposes and is subject to change.**

## Enabling in Chrome
1. Download a version of Chrome with the feature implemented. (e.g. [Canary](https://www.google.com/chrome/canary/))
1. Enable the requisites features in Chrome:
    - [Launch with flags](https://www.chromium.org/developers/how-tos/run-chromium-with-flags) `--enable-features=IsolatedWebApps,IsolatedWebAppDevMode,AutomaticFullscreenContentSetting`
    - OR: Enable chrome://flags/#enable-isolated-web-apps, chrome://flags/#enable-isolated-web-app-dev-mode, and chrome://flags/#automatic-fullscreen-content-setting
1. Navigate to a resource that exercises the feature
    - Install an IWA, e.g. https://github.com/michaelwasserman/iwa-windowing-example
    - OR: Set [enterprise policy](https://www.chromium.org/administrators) to allow [an example](https://michaelwasserman.github.io/iwa-windowing-example/static/), e.g. in `/etc/opt/chrome/policies/managed/test_policy.json` on Linux:
  ```JSON
    "AutomaticFullscreenAllowedForOrigins": [ "https://michaelwasserman.github.io" ]
  ```
1. Exercise the feature (e.g. hover "request on mouseenter" in iwa-windowing-example)
    - Grant [Window Management](https://developer.mozilla.org/en-US/docs/Web/API/Window_Management_API) permission and Unblock popups (chrome://settings/content/popups) to exercise multi-screen fullscreen features
      - (e.g. click "request on click" and grant the prompt, and then "open multiple" in iwa-windowing-example)
