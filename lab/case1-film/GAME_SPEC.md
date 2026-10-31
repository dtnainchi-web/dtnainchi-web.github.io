# قضية بالحروف — Game Spec (v1, for Claude Code)

Owner: Youness Nadi. Platform: **Android first (Google Play)**, Flutter. Language: **Arabic only, full RTL**.
Case 1 data (dialogue, puzzles, all media file names): `case1.json` next to this file.
Playable story preview of case 1: https://dtnainchi-web.github.io/lab/case1-film/ — the game must feel like this preview, plus real puzzles.

## 1. The idea
A detective game in a fictional Arab harbour city, «المَرسى». The player is a new inspector. Inspector Karim is the mentor.
Each case = story cutscenes (lip-sync videos) + **Arabic crossword puzzles whose words ARE the clues of the case** + interrogation + accusation.
Rule: every solved word reveals evidence tied to the case or a suspect. Words, dialogue and story are always connected, never separate.

## 2. Art & media (already produced, do NOT generate new art)
- Stylized 3D animated-film look, warm amber + teal, dark navy UI with gold accents (#e9b96a / #ffd98f), font Cairo (UI) + Amiri (titles).
- Cutscenes = vertical 9:16 MP4 lip-sync videos (720p). Show with cinematic letterbox bars (9% top/bottom) and subtitles at the bottom: speaker name in gold + line text.
- Voice-over lines = evidence image full screen + Karim's voice (MP3) + subtitle.
- Download every file from `cdn_prefix + filename` in case1.json into the project as bundled assets. Compress videos for the app (H.264, ~540x960 or 720x1280, CRF ~30, AAC 96k, target ≤ 2 MB each). Convert PNG to WebP. Case 1 total must stay small (target ≤ 35 MB).

## 3. Core loop of a case
`intro cutscene → puzzle → evidence voice-over → interrogation cutscenes → puzzle → objection → puzzle → accusation → reveal → case closed (stars)`
Exact order for case 1: see `flow` in case1.json.

### 3.1 Puzzle (crossword)
- Classic Arabic **letter wheel + crossword grid** (like popular Arabic word games): the player swipes letters on a wheel to form words that fill a small crossword grid.
- Generate a valid crossword layout from the puzzle's word list (words must cross on shared letters; Arabic letters in isolated form on tiles; grid RTL).
- Each word shows its **clue** (from case1.json) in a list; when solved, the clue card turns into the evidence card (image if `evidence` exists) and is added to the notebook.
- If a word has `then_line`, play that voice-over line right after solving it.
- Hint button: reveals one letter. 3 free hints per case; later more hints via a rewarded ad (opt-in only).

### 3.2 Notebook (دفتر المفتش)
- Always reachable: list of suspects (portrait/still + what they said) and evidence collected so far.

### 3.3 Objection (اعتراض)
- After puzzle p2: the player opens Omar's statement (line 7) and must pick the evidence that contradicts it (the lantern). Correct → play lines 9, 10. Wrong → short Karim hint text, try again (no penalty except stars).

### 3.4 Accusation
- Choose between suspects in `accusation.suspects`. Wrong → play `wrong_line` (16) and return to the choice (lose 1 star). Right → reveal lines 12–15.

### 3.5 Stars
- 3 stars: no wrong accusation and ≤ 1 hint. 2 stars: one mistake or 2–3 hints. 1 star: otherwise. Save progress locally.

## 4. Screens
1. Splash (no ads) → 2. Welcome for new players (Karim line 0 is the welcome) → 3. Case map/list (case 1 open, case 2 "قريباً" only if case 2 content exists — never show empty placeholders in a release) → 4. Case player (cutscenes + puzzles) → 5. Notebook → 6. Case closed (stars) → 7. Settings: sound on/off, music volume, privacy options (UMP), credits/licences, privacy policy link.

## 5. Content & storage
- Cases 1 and 2 are **bundled in the app**. Later cases will be downloaded from a server (Cloudflare R2) — design a `CaseRepository` interface now (bundled implementation only for v1).
- Downloads (later): check free space before downloading; on low space show a friendly Arabic message; delete partial files on failure; never crash. Delete videos of finished downloaded cases, keep progress.
- The game must be fully playable offline.

## 6. Ads (add only in the release milestone, after everything works)
- AdMob + Unity Ads fallback, same strict consent rules as the owner's other apps (UMP before any ad SDK; personalized only with real consent).
- Interstitial only at natural breaks (after "case closed", never during cutscenes or puzzles), frequency cap. Rewarded = optional extra hints. No banner over the puzzle wheel.

## 7. Store rules (mandatory)
Follow `C:\dev\STORE_RULES.md` (or the copy in C:\dev\quiz_cqc). Target API 36, R8 keep rules, no forbidden permissions (only INTERNET, ACCESS_NETWORK_STATE, AD_ID when ads exist), target audience 13+, Arabic content only, no gore.

## 8. Milestones
- **M1 (now):** project setup + download/compress case 1 assets + case player with cutscenes, voice-overs, subtitles, all 3 crossword puzzles, notebook, objection, accusation, stars, local save. Playable end-to-end on an Android emulator/BlueStacks in debug. No ads.
- M2: welcome flow polish, settings, music/SFX, case 2 content (provided later).
- M3: ads + consent, release build, store checklist from STORE_RULES §10.
