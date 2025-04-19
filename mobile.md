Developing a mobile app for Open edX involves choosing between:
	1.	📱 Building your own mobile app using the official Open edX mobile source code (React Native or native Android/iOS)
	2.	⚙️ Using a white-labeled mobile platform like Edly Go
	3.	🛠️ Fully custom mobile app consuming Open edX APIs (LMS + MFEs + your services)

Let’s walk through all options, including whether Edly Go is the right choice for you.

⸻

✅ Option 1: Use the Official Open edX Mobile Apps (Open Source)

📦 Repos:
	•	openedx/edx-app-android
	•	openedx/edx-app-ios

📲 Features:
	•	Enroll/view courses, watch videos
	•	Download content offline
	•	Track progress and scores
	•	Login via OAuth2/SAML
	•	Supports Open edX themes

🛠️ How to Use:
	•	Fork the app repo
	•	Customize branding (logo, theme, icons)
	•	Configure API endpoints (config.json)
	•	Rebuild and publish to app stores

// Example LMS config
{
  "HOST_URL": "https://your-openedx.example.com",
  "API_URL_HOST": "/api",
  "OAUTH_CLIENT_ID": "your-oauth-client-id"
}

⚙️ Requirements:
	•	Knowledge of Java/Kotlin (Android), Swift/Obj-C (iOS)
	•	Build system (Gradle/Xcode)
	•	Open edX OAuth setup (via Tutor or native LMS)

✅ Best for:
	•	Teams with in-house mobile skills
	•	Need for full control and offline support

⸻

🔁 Option 2: Use a White-Label App like Edly Go

📱 What is Edly Go?
	•	A commercial white-label mobile solution built on top of Open edX
	•	Offered by Edly, an official Open edX partner
	•	Offers a branded mobile experience quickly, with LMS integration

💡 Features:
	•	Prebuilt mobile app (Android/iOS)
	•	Push notifications
	•	Supports custom themes
	•	Integration with Tutor-based Open edX
	•	Hosted option (or self-hosted with guidance)

💰 Licensing:
	•	Commercial, with subscription or licensing fees
	•	Can save months of development time

✅ Best for:
	•	Orgs that want a mobile app fast
	•	Lack in-house mobile dev team
	•	Want professional support & maintenance

⸻

🧱 Option 3: Build a Custom Mobile App using Open edX APIs

🔗 Use APIs like:
	•	/api/mobile/v1/ – course discovery, enrollments, progress
	•	/api/user/v1/account/login_session/ – authentication
	•	/api/discussion/, /api/certificates/, etc.

🧠 Build with:
	•	React Native / Flutter / Kotlin / Swift
	•	Your own UX/UI logic and layout
	•	Custom AI tutor integrations, dashboards, chatbots, etc.

✅ Best for:
	•	Total customization
	•	Integrating GenAI tutors, learning analytics, etc.
	•	Advanced workflows (e.g., microlearning, gamification)

⸻

🔍 Summary: Which Path Should You Take?

Approach	DIY Open edX App	Edly Go	Fully Custom App
Mobile Skills Needed	✅ High	❌ None	✅ High
Brand Customization	✅ Yes	✅ Yes	✅ Full Control
Offline Support	✅ Yes	✅ Yes	Optional
AI/Custom Features	🟡 Limited	🟡 Some Support	✅ Fully Custom
Time to Market	❌ Slow (weeks)	✅ Fast (days)	❌ Slow (custom)
Best For	Full control devs	Rapid deployment	Custom AI apps



⸻

🚀 Recommendation for You

Since you’re:
	•	Already customizing LMS and MFEs
	•	Deploying AI services
	•	Comfortable with Tutor and APIs

✅ Use Edly Go if you want a quick and supported mobile rollout.

OR

🛠️ Use the official Open edX mobile apps if you want to integrate your AI tutor directly and have in-house mobile devs.

⸻

Would you like:
	•	Help customizing the official edX Android app?
	•	A starting React Native app that uses Open edX APIs and talks to your AI tutor backend?
	•	A deep dive into LMS OAuth2 setup for mobile apps?

Happy to assist you on either path!

_____

Great! Here’s your comprehensive guide to customizing the official Open edX Android app step-by-step — with branding, server configuration, and integrations like OAuth, push notifications, and AI tutor endpoints.

⸻

✅ Overview: What You Can Customize

Feature	Customizable?	How
App name & icon	✅ Yes	Android manifest + drawable
LMS/Studio URL	✅ Yes	config.json + build.gradle
OAuth credentials	✅ Yes	config.json
Colors & themes	✅ Yes	colors.xml, styles, branding
Push notifications	✅ Yes	Firebase setup
AI integration	✅ Yes	Custom fragment / API call



⸻

🛠 Step-by-Step: Customize and Build

1. 📥 Clone the Official Repo

git clone https://github.com/openedx/edx-app-android.git
cd edx-app-android

(Optional: checkout a specific release, e.g., Olive):

git checkout open-release/olive.master



⸻

2. ⚙️ Set Up Android Studio
	•	Open the project in Android Studio
	•	Let Gradle sync
	•	Install SDKs if prompted (API 33+ recommended)

⸻

3. 🧾 Configure LMS Server and OAuth

🔧 Edit config/config.json

{
  "HOST_URL": "https://your-openedx.domain",
  "API_HOST_URL": "https://your-openedx.domain",
  "OAUTH_CLIENT_ID": "your-client-id-from-openedx",
  "ENVIRONMENT_DISPLAY_NAME": "Your LMS",
  "FEEDBACK_EMAIL_ADDRESS": "support@yourdomain.com"
}

🗝️ Get OAuth Client ID
	•	Create one in LMS Django Admin (/admin/oauth2/client/)
	•	Callback URL: org.edx.mobile://<any> (e.g. org.edx.mobile://oauth2redirect)

⸻

4. 🎨 Customize Branding

📱 App Name & Icon
	•	Edit app/src/main/res/values/strings.xml:

<string name="app_name">Your LMS</string>

	•	Replace ic_launcher.png in res/mipmap-*/

🎨 Colors & Styles
	•	Modify res/values/colors.xml, styles.xml

<color name="primaryColor">#004c97</color>
<color name="accentColor">#00b0f0</color>



⸻

5. 📲 Add Push Notifications (Optional)

If using Firebase:
	•	Set up Firebase in the console
	•	Download google-services.json
	•	Place it in app/ directory
	•	Enable Firebase in build.gradle and Manifest

⸻

6. 🤖 Add Custom Features (e.g., AI Tutor)

You can:
	•	Add a new Activity or Fragment (e.g., AITutorFragment)
	•	Create a layout with a question input and a “Send” button
	•	Use Retrofit to call your FastAPI AI backend

📁 Example: AITutorFragment.kt

class AITutorFragment : Fragment() {
    private lateinit var apiService: AITutorService

    override fun onCreateView(...) {
        val view = inflater.inflate(R.layout.fragment_ai_tutor, container, false)
        view.findViewById<Button>(R.id.ask_button).setOnClickListener {
            val question = view.findViewById<EditText>(R.id.question_input).text.toString()
            askAI(question)
        }
        return view
    }

    fun askAI(question: String) {
        apiService.askAI("Bearer " + AuthTokenProvider.getAccessToken(), question)
            .enqueue(object : Callback<AIResponse> {
                override fun onResponse(...) {
                    // update UI with reply
                }
            })
    }
}



⸻

7. ▶️ Build & Run
	•	Run on emulator or device from Android Studio
	•	Sign APK if building for release

⸻

🔐 LMS Configuration Notes

Make sure Tutor LMS has:
	•	OAuth client (/admin/oauth2/client/)
	•	CORS allowed origins (CORS_ORIGIN_WHITELIST)
	•	Mobile APIs enabled (/api/mobile/v1/)
	•	Push notifications (edx_notifications optional plugin)

⸻

✅ Summary

Task	File / Tool
LMS/OAuth Config	config/config.json
Branding	res/values/, mipmap/
Custom UI or AI features	MainActivity.kt, Fragment.kt
API Communication	Retrofit, AuthTokenProvider
Build APK	Android Studio or ./gradlew assemble



⸻

Would you like a working AI tutor fragment + layout ready to copy into the app? Or help wiring up the LMS token for secure API access to your FastAPI backend?

____

## using whitelabel
Yes, you can use Edly Go with your own custom Tutor-based Open edX deployment, including your custom LMS theme — but there are important architectural and practical considerations. Let’s break this down:

⸻

✅ What is Edly Go?

Edly Go is a white-label mobile app solution provided by Edly, built on top of the official Open edX mobile apps (Android + iOS). It is designed to work with:
	•	Open edX backends (including Tutor)
	•	Your own LMS instance (not just hosted by Edly)
	•	Your custom branding and theming

⸻

📦 What Edly Go Offers You

Feature	Supported	Notes
Connects to your Tutor LMS	✅	Your own domain, API keys
Custom logo & theme	✅	Provided during onboarding
Push notifications	✅	Firebase integration included
OAuth2 login	✅	Configurable for your LMS
Offline video access	✅	Like official apps
Multi-org / multi-tenant	✅	Optional per plan
Custom features (e.g. AI)	🟡 Maybe	Ask Edly for integration feasibility



⸻

🧠 How It Works with Your Tutor Deployment

🔐 1. OAuth Setup

You need to provide:
	•	HOST_URL (e.g. https://your-lms.com)
	•	OAUTH_CLIENT_ID from your LMS (configured in /admin/oauth2/client)
	•	Callback URL for mobile auth: org.edx.mobile://<redirect>

Tutor supports this via the oauth plugin (tutor plugins enable oauth).

⸻

🎨 2. Custom Theme

Edly Go lets you provide:
	•	Logo, splash screen
	•	Primary/secondary colors
	•	Fonts (optional)
	•	App name, organization ID

This won’t use your LMS’s HTML/CSS theme, but it visually matches it on the mobile app.

If your LMS theme has specific branding or UI language, Edly can match this look in the app’s UI config during white-label setup.

⸻

🔗 3. API Compatibility

Your LMS needs to expose:
	•	/api/mobile/v1/
	•	/api/user/v1/
	•	OAuth endpoints (/oauth2/authorize, /oauth2/access_token)
	•	JWT auth, if used for protected APIs

Tutor already includes this via default + openedx plugin.

⸻

🧩 4. AI/Custom Features

If you’re building:
	•	AI tutor APIs (FastAPI backend)
	•	Custom course dashboards
	•	Embedded chatbot or hints

Edly Go does not automatically support custom APIs, but:
	•	You can request integration (if you’re on a paid plan)
	•	You may build a wrapper app using Edly Go as a base (less common)

⸻

✅ Summary: Can You Use Edly Go?

Use Case	Possible with Edly Go?	Notes
Use own Tutor LMS backend	✅ Yes	Fully supported
Apply LMS branding/theme	✅ Yes	Provided during app setup
OAuth2 login via your LMS	✅ Yes	LMS must support /oauth2/
LMS hosted on your own infra	✅ Yes	Not hosted by Edly
Add AI tutor or custom APIs	🟡 Possible	May need extra work or plan upgrade
Offline video, course browsing, etc.	✅ Yes	Standard features



⸻

🚀 Next Steps

If you’re ready:
	1.	📩 Contact Edly: Tell them you want to use Edly Go with your own Tutor-based Open edX.
	2.	🧾 Prepare:
	•	Your LMS URL + API endpoints
	•	OAuth client ID + callback
	•	Logo, color scheme, app name
	3.	🧪 Test: Edly can give you an APK build or access to test version.

⸻

Would you like a checklist of what to prepare for Edly Go onboarding, or help configuring your Tutor LMS to work seamlessly with mobile OAuth and API endpoints?


Yes — Edly Go supports both Android and iOS 🎉

In fact, iOS support is a core part of the Edly Go offering, just like the official Open edX mobile apps.

⸻

📱 What Edly Go Offers for iOS

Feature	iOS Support	Notes
Branded mobile app	✅ Yes	Your logo, name, icons
Push notifications	✅ Yes	Via Apple Push Notification Service (APNs)
OAuth2 login	✅ Yes	Via your LMS (Tutor)
Offline video	✅ Yes	Works like Android version
Custom theme/colors	✅ Yes	Matches LMS branding
App Store deployment	✅ Optional	Edly can publish or give you .ipa
LMS-hosted backend	✅ Yes	Supports custom domains
Device-specific customization	🟡 Limited	Optional per your plan or request



⸻

🚀 Options for iOS Deployment

Option A: Edly Publishes for You
	•	You provide branding assets, bundle ID, and App Store developer account.
	•	Edly builds and publishes the app to the App Store under your name.

Option B: You Build & Publish
	•	Edly gives you the .ipa file or Xcode project.
	•	You sign and publish using your Apple Developer account.

You must enroll in the Apple Developer Program ($99/year) to publish.

⸻

🔑 iOS OAuth Setup for Your LMS

Just like Android, your LMS must support OAuth2:
	•	LMS URL: https://your-lms.com
	•	OAuth Client ID: From /admin/oauth2/client
	•	Redirect URI: Something like
org.edx.mobile://<your_redirect_path> (used by the iOS app)

✅ Tutor + oauth plugin handles this perfectly.

⸻

✅ Summary

Feature	Supported on iOS?
Tutor-based LMS backend	✅ Yes
Branding + theming	✅ Yes
Push + offline content	✅ Yes
LMS OAuth2 integration	✅ Yes
App Store deployment	✅ Yes (manual or Edly-managed)



⸻

Would you like:
	•	A checklist of iOS provisioning assets you need to prepare?
	•	Guidance for configuring OAuth for mobile login in your LMS?
	•	Or a full guide to publishing under your own Apple Developer account?

____

No — Edly Go is not open source.

It is a commercial, proprietary white-label mobile app solution built by Edly, a service provider and official Open edX partner. While it is based on the open-source Open edX mobile apps (Android/iOS), Edly Go itself includes proprietary customizations and is licensed as a paid product.

⸻

🔍 Breakdown: Edly Go vs. Open Source

Feature	Edly Go	Open edX Mobile Apps
Open source	❌ No	✅ Yes (edx-app-android)
Based on Open edX app	✅ Yes (heavily customized)	✅ Native edX mobile apps
Self-hosting allowed	❌ Not directly	✅ Yes (you build it yourself)
iOS/Android support	✅ Yes	✅ Yes
Custom theme/logo	✅ Included in setup	✅ You do it yourself
AI, analytics, extensions	🟡 Some built-in / requestable	✅ You can develop freely
Support & SLA	✅ Commercial support	❌ Community only
Cost	💰 Commercial licensing / subscription	✅ Free (DIY only)



⸻

✅ What You Can Do Open Source

If you want to retain full control, you can:
	•	Clone and customize the official mobile apps:
	•	edx-app-android
	•	edx-app-ios
	•	Configure it to talk to your Tutor LMS
	•	Add features like:
	•	GenAI assistant
	•	Custom dashboards
	•	LMS-integrated chat or feedback

You’ll be responsible for:
	•	Maintaining the code
	•	Managing app store publishing
	•	Ensuring security and API compatibility

⸻

🤔 So When Should You Choose Edly Go?

Choose Edly Go if…
You want to launch quickly with minimal dev work
You don’t have a mobile dev team
You need iOS + Android support with SLA
You want visual consistency with your LMS theme
You’re okay with a subscription-based solution



⸻

✅ Summary
	•	❌ Edly Go is not open source
	•	✅ It’s built on top of Open edX mobile, but sold as a product
	•	✅ You can always use the official open-source apps if you want full control

⸻

Would you like:
	•	A starter fork of the open-source Android app preconfigured for Tutor?
	•	Help comparing Edly Go pricing/features vs. DIY path?
	•	A hybrid strategy (e.g., use open-source + outsource app store management)?

Let me know how deep you want to go — I can tailor a solution.

____

Great question — Edly Go is a professionally packaged and branded mobile solution based on the official Open edX mobile apps (which are open source), but it includes a wide range of customizations, enhancements, and polish that go beyond what’s available out of the box.

Here’s a breakdown of the customizations and added features typically found in Edly Go:

⸻

🎨 1. White-Label Branding & Theming

Feature	Description
App name	Customized to your organization
Logo and splash screen	Your branding during app launch
Custom icons	Branded app icons in all resolutions
Theme and colors	Primary, secondary colors, font tweaks
Language and locale	Customized labels, possibly multi-lingual

🟢 Fully customizable during onboarding

⸻

🔐 2. Custom LMS Integration

Feature	Description
Custom LMS domain	Connects to your Tutor-based or native LMS
OAuth2 support	Configured to authenticate with your LMS
Organization awareness	Supports multi-org use cases
Token-based SSO	Seamless login experience across web/mobile

🟢 Works with self-hosted Open edX too

⸻

🧠 3. UX/UI Enhancements

Feature	Description
Improved navigation	Better UI structure for discoverability
Enhanced course browsing	Refined filters, views, and category layout
Progress tracking widgets	Modern visuals for completion, performance
Light/dark mode	Optional for visual preference

🟡 Based on Open edX UX but improved

⸻

📲 4. Push Notifications

Feature	Description
Firebase-based notifications	Via Firebase Cloud Messaging (FCM) for Android / APNs for iOS
Course updates & reminders	Notify users of deadlines, announcements
Admin-controlled campaigns	You can configure notifications manually or via LMS dashboard (with plugin)

🟢 Built-in and integrated

⸻

🎥 5. Offline Video & Caching Enhancements

Feature	Description
Video download manager	For course content; better UX than default
Resume playback	Pick up where the user left off
Bandwidth controls	Quality/resolution adjustments (optional)

🟢 Based on open source, enhanced by Edly

⸻

🧩 6. Custom Integrations (Optional)

Edly Go supports advanced customizations for enterprise clients:

Feature	Description
Custom dashboard cards	E.g., AI tutor prompt, leaderboards
Analytics & LRS	Integrate with analytics dashboards
AI Tutor	Optional endpoint/widget integration
Third-party API plugins	Via API gateway (e.g., attendance, HR data)

🟡 Available on request — depends on your plan & scope

⸻

🧾 7. App Store Readiness

Feature	Description
Signed, ready-to-publish APK & IPA	Provided as part of onboarding
Apple Developer help	Assist with provisioning if you self-publish
App Store deployment	Can publish under your account or Edly’s

🟢 Saves huge time vs. DIY app store setup

⸻

🛠️ 8. Optional Plugins & Services
	•	LTI provider/viewer for mobile-optimized embedded tools
	•	Custom course discovery engine
	•	Multi-tenant UX (org-specific branding per user login)
	•	SCORM / xAPI support (mobile-aware)

⸻

✅ Summary: What’s Customized in Edly Go?

Category	Customization Examples
Branding	Logo, colors, app name, splash screen
LMS Integration	OAuth, domain, SSO, token exchange
UI Enhancements	Navigation, progress visuals, theming
Offline & Media UX	Download manager, resume playback
Notifications	Firebase/APNs, push from LMS or campaign UI
Extensibility	AI Tutor APIs, analytics, 3rd party support
App Delivery	App Store builds, provisioning support



⸻

🧠 Final Note

Edly Go is not just a reskin — it’s a production-grade mobile app product that abstracts away the heavy lifting while staying compatible with Open edX.

⸻

Would you like:
	•	A feature comparison sheet of Edly Go vs. official Open edX mobile?
	•	Advice on integrating your AI tutor into Edly Go or official apps?
	•	Or a checklist to prepare if you’re onboarding with Edly?

Just say the word!