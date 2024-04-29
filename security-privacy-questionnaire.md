# Security and Privacy Questionnaire

This document answers the [W3C Security and Privacy Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/) for HTML Fullscreen Without A Gesture.

**2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?**

The User Agent's configuration for the origin can be inferred through attempted use (i.e. whether `requestFullscreen()` succeeds when `navigator.userActivation.isActive` is false). The same information may eventually be exposed more concretely through Permission API querying for feature detection and site UI or behavioral customization purposes.

**2.2 Do features in your specification expose the minimum amount of information necessary to enable their intended uses?**

Yes.

**2.3 How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?**

N/A; the feature does not deal with PII.

**2.4 How do the features in your specification deal with sensitive information?**

N/A; the feature does not deal with sensitive information.

**2.5 Do the features in your specification introduce new state for an origin that persists across browsing sessions?**

Yes; User Agent configurations can persist across browsing sessions, but are not under the site's control.

**2.6 Do the features in your specification expose information about the underlying platform to origins?**

No; excepting whether the feature is supported (e.g. available on desktop but not yet on mobile).

**2.7 Does this specification allow an origin to send data to the underlying platform?**

No.

**2.8 Do features in this specification enable access to device sensors?**

No.

**2.9 Do features in this specification enable new script execution/loading mechanisms?**

No.

**2.10 Do features in this specification allow an origin to access other devices?**

No.

**2.11 Do features in this specification allow an origin some measure of control over a User Agent’s native UI?**

Entering fullscreen hides User Agent UI; this feature enables fullscreen state control in additional circumstances.

**2.12 What temporary identifiers do the features in this specification create or expose to the web?**

None.

**2.13 How does this specification distinguish between behavior in first-party and third-party contexts?**

We've explicitly chosen not to introduce another policy-controlled feature name for automatic fullscreen. If the parent document delegates `fullscreen` to the child document, it accepts that its ability to automatically enter fullscreen is delegated as well.

So, third-party iframes may only enter fullscreen without a gesture if (a) the User Agent is configured to permit fullscreen without a gesture in the first-party context and (b) the third-party context is granted the preexisting `fullscreen` Permissions-Policy.

**2.14 How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?**

User Agents can choose whether or not to respect their configuration in Private Browsing or Incognito mode. Chromium has chosen to respect its settings in all modes.

**2.15 Does this specification have both "Security Considerations" and "Privacy Considerations" sections?**

Yes.

**2.16 Do features in your specification enable origins to downgrade default security protections?**

The Fullscreen API hides protective User Agent user interface elements, and the specification defines mitigations that are widely deployed in implementations. User Agents should explain new configuration options and adapt existing overlay mitigations to suit usage (e.g. advertise fullscreen state and exit instructions with user interactions after feature usage).

**2.17 How does your feature handle non-"fully active" documents?**

Handling is specified by the Fullscreen API itself; requests from non-"fully active" documents are rejected, and User Agents exit fullscreen during unloading document cleanup steps.

**2.18 What should this questionnaire have asked?**

The current set of questions is appropriate.
