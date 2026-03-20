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

# 📜 Scripts

## Script 1 — Remove `@` From Username


using System;

public class CPHInline
{
    public bool Execute()
    {
        string rawInput = args.ContainsKey("rawInput") ? args["rawInput"].ToString() : "";

        if (rawInput.StartsWith("@"))
        {
            rawInput = rawInput.Substring(1);
        }

        CPH.SetArgument("rawInput", rawInput);

        return true;
    }
}


## Script 2 — Fetch Random Twitch Clip

using System;
using System.Net.Http;
using Newtonsoft.Json.Linq;
using System.Collections.Generic;

public class CPHInline
{
    public bool Execute()
    {
        CPH.TryGetArg("clientId", out string clientId);
        CPH.TryGetArg("token", out string token);
        CPH.TryGetArg("targetUser", out string targetUser);
        CPH.TryGetArg("muteVOD", out bool muteVOD);

        CPH.SetArgument("noclipsforuser", true);

        try
        {
            using (var client = new HttpClient())
            {
                client.DefaultRequestHeaders.Add("Client-ID", clientId);
                client.DefaultRequestHeaders.Add("Authorization", $"Bearer {token}");

                var userResponse = client.GetStringAsync("https://api.twitch.tv/helix/users?login=" + targetUser).GetAwaiter().GetResult();
                var userId = JObject.Parse(userResponse)["data"]?[0]?["id"]?.ToString();

                if (!string.IsNullOrEmpty(userId))
                {
                    var clipsResponse = client.GetStringAsync("https://api.twitch.tv/helix/clips?broadcaster_id=" + userId + "&first=50").GetAwaiter().GetResult();
                    var clips = (JArray)JObject.Parse(clipsResponse)["data"];

                    if (clips != null && clips.Count > 0)
                    {
                        CPH.SetArgument("noclipsforuser", false);

                        var rand = new Random();
                        var selectedClip = (JObject)clips[rand.Next(clips.Count)];
                        
                        string clipId = selectedClip["id"].ToString();
                        double durationSeconds = (double)selectedClip["duration"];
                        
                        double totalMs = (durationSeconds * 1000) + 3000;

                        string url = "https://clips.twitch.tv/embed?clip=" + clipId +
                                     "&parent=localhost&autoplay=true&muted=" + muteVOD.ToString().ToLower();

                        CPH.SetArgument("clipUrl", url);
                        CPH.SetArgument("clipDuration", totalMs);

                        CPH.LogInfo("[Grabber] Clip: " + clipId + " | Duration: " + totalMs + "ms");

                        return true;
                    }
                    else
                    {
                        CPH.LogInfo("[Grabber] No clips found for user: " + targetUser);
                    }
                }
                else
                {
                    CPH.LogError("[Grabber] User not found: " + targetUser);
                }
            }
        }
        catch (Exception ex)
        {
            CPH.LogError("[Grabber] Exception: " + ex.Message);
        }

        return true;
    }
}
