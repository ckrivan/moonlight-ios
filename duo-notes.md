# iPhone Duo redesign notes
Journey: Select Host (MainFrameViewController) → app grid (same controller) → stream (StreamFrameViewController); Settings side panel (SettingsViewController in SWRevealViewController)
Stack: UIKit, Objective-C, storyboards (iPhone.storyboard / iPad.storyboard), iOS target 15.0, no UIScene manifest. Fork of moonlight-stream/moonlight-ios at 02dc978. Built with Xcode 27.2 / iOS 27.2 SDK and Xcode 27.3 (Studio).
Stage: 5  Next: on the Duo, pair "Moonlight Duo" with the PC, start a stream open, close and reopen the phone, and report what the picture does

## Findings
- App Store build ran in the scaled compatibility window (user screenshot 16 Nov 2026): older SDK. Change — rebuild with the 27.x SDK [T461]
- StreamFrameViewController: stream view and scroll view sized once from `self.view.frame`, labels centered by frame math — Change [T461, HIG]
- VideoDecoderRenderer: display layer fitted only at init and decoder reset — Change [HIG]
- Settings "Safe Area" / "Full": sizes read once from the first window when Settings opens — Change [T461, HIG]
- `[UIScreen mainScreen]` for maximumFramesPerSecond — Change [T461]; UIAppView rasterizationScale still uses it (cosmetic, left)
- Settings / MainFrame pad 20 pt when any left or right safe-area inset exists, same both sides — Decision, not changed yet [T461]
- OnScreenControls picks its layout by `userInterfaceIdiom` — Decision, not changed [HIG, layout]
- Info.plist UIRequiresFullScreen = true, landscape only — Decision (HIG: games may lock; T461: inner display doesn't honor orientations)
- Bars: UINavigationController with a text "Settings" item; SWRevealViewController is a custom side panel — Decision, not changed [T462]
- Ready: 0 before the rebuild; N/A: camera, multiwindow, tab bars, scene accessories, table pose

## Decisions
- Display change during a stream: recommended keep the stream running and refit the picture (may letterbox); chosen same (agent, 2026-11-16) — a reconnect at the new size drops the game for seconds; HIG prefers a new aspect ratio over letterboxing, so offer "restart at the new size" later
- Stream size: recommended "Safe Area"/"Full" follow the display in use when the stream starts; chosen same (agent, 2026-11-16)
- Orientation: recommended keep landscape lock; chosen same (agent, 2026-11-16)
- App identity: recommended separate app "Moonlight Duo", com.ckrivan.moonlightduo, beside the App Store app; chosen same (agent, 2026-11-16) — needs its own pairing
- Settings panel as a managed split view / vertical bars: pending (user)

## Done / verified
- Stream view, scroll view, labels follow the view's bounds (StreamFrameViewController.m) — compiled; not yet seen on a fold
- StreamView.layoutSubviews refits the video layer and on-screen controls (StreamView.m, OnScreenControls.m/.h) — compiled; not yet seen on a fold
- "Safe Area"/"Full" recomputed at stream start, NSUserDefaults `duoFollowDisplay` (SettingsViewController.m, MainFrameViewController.m) — compiled; not run
- iOS and tvOS simulator builds: BUILD SUCCEEDED (16 Nov 2026)
- Scene life cycle: SceneDelegate in AppDelegate.h/.m, UIApplicationSceneManifest in Limelight-Info.plist, UIMainStoryboardFile keys removed — first install crashed at launch in _UIApplicationEvaluateRuntimeIssueForNoSceneLifecycleAdoption (crash report 16 Nov); after the change the device console showed the app running and finding the host PC
- Release build signed with the fork owner's development certificate, installed on a physical iPhone Duo with devicectl 16 Nov 2026 — launch on device: run, stays up
- Default resolution is now "Full" (follow the display) until one is chosen — the first stream had asked for 1280x720 (Sunshine log)
- Device Hub iPhone Duo simulator (Xcode 27.3), run 16 Nov: app fills the display closed (678x466 pt) and open (951x669 pt), survives every hinge change, never restarted. Defects seen: host list scroll view keeps its 678 pt width after opening (icons hug the left); open + rotated to the tall frame gives a letterboxed 669x471 window with clipped content; Settings panel (570 pt) pushes the main view off the right edge, worst closed; no fold-specific layout at 104 degrees. Stream screen not exercised (simulator not paired)
- Next journey: host list width on open, tall-frame handling, Settings panel
- Nothing committed
