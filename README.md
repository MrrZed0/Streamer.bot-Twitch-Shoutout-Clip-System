# 🎬 Streamer.bot Twitch Shoutout Clip System

A Streamer.bot C# script setup that:
- Removes `@` from usernames in shoutout commands
- Fetches a random Twitch clip from the target user
- Sends clip data to OBS for playback
- Falls back gracefully if no clips exist

---

## ✨ Features

- Automatically removes `@username`
- Fetches random Twitch clips via API
- Sends clip to OBS Browser Source
- Supports mute/unmute playback
- Auto-calculates clip duration
- Detects when user has no clips

---

## ⚙️ Required Arguments Setup

Create these arguments in Streamer.bot before running the scripts.

### Clip Control Args

- noclipsforuser (bool)  
  true = no clips found  
  false = clips found  

- clipUrl (string)  
  Twitch embed clip URL  

- clipDuration (number)  
  Duration in milliseconds  

---

### Audio Control

- muteClip (bool)  
  true = muted  
  false = sound enabled  

---

### Twitch API Credentials

Get credentials here:  
https://twitchtokengenerator.com/

- twitch_clientId  
- twitch_token  

---

## 🖥️ OBS Setup

After running the script:

1. Set Browser Source URL  
   → Use: `clipUrl`

2. Enable Browser Source  

3. Add Delay  
   → Use: `clipDuration`  
   OR set custom (example: 10000 ms)

4. Disable Browser Source  

---

## 🔁 Optional Logic (Recommended)

Add an IF condition after the script:

If `noclipsforuser == true`  
→ Send chat shoutout  

If `noclipsforuser == false`  
→ Play OBS clip overlay  

---

## 💡 Example Command Flow
!so @username
↓
Script 1 (remove @)
↓
Script 2 (fetch clip)
↓
IF noclipsforuser
→ Chat shoutout
ELSE
→ Play clip in OBS



---

## 🚀 Result

You now have a fully automated shoutout system that:
- Plays random Twitch clips
- Handles users with no clips
- Integrates with OBS

---

## ⚠️ Notes

- `parent=localhost` must match your OBS setup
- Twitch API rate limits apply
- Make sure OBS Browser Source is enabled

---

Streamer.bot Twitch Shoutout Clip System #steamer.bot #twitchalertoverlay #twitchshoutout #twitchshoutoutoverlay Twitch shoutout overlay streamer.bot twitch shoutout overlay


