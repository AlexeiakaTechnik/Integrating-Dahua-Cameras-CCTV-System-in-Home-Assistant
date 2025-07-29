
<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/a789fb5c-b4d1-4430-99a1-ca86ca08a189" />


# 🎥 Dahua Camera/CCTV System in Home Assistant
_Getting Dahua NVRs and cameras into Home Assistant with actionable smart events.  
Integration walkthrough with go2rtc, smart motion detection, light automations, RTSP streaming setup, and best-practice camera configs._

---

## 📚 Table of Contents

1. 🔍 [Introduction](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-1-introduction-) 
2. 🧰 [Requirements](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-2-requirements-)
3. 🧱 [Dahua Cameras Overview & Integration Limitations](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-3-dahua-cameras-overview--integration-limitations-)
4. 🛠️ [Cameras Installation Tips and Web-based Setup](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#%EF%B8%8F-4-cameras-installation-tips-and-web-based-setup-)
5. 🏠 [Integrating with Home Assistant](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-5-integrating-with-home-assistant-)
6. 📡 [RTSP Streams and go2rtc Configuration](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-6-rtsp-streams-and-go2rtc-configuration-)
7. ⚡ [Automations Based on Dahua Provided Events](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-7-automations-based-on-dahua-provided-events-)
8. 🖥️ [UI and Dashboards](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#%EF%B8%8F-8-ui-and-dashboards-)
9. 🔐 [Privacy and Security](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-9-privacy-and-security-)
10. 🎬 [Live Video Demonstration](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-10-live-video-demonstration-)  
11. 🧩 [Conclusions and Further Improvement](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-11-conclusions-and-further-improvement-) 
12. 🪪 [License](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-12-license-)
13. 👨‍💻 [Author and Inspiration](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-13-author-and-inspiration-)
14. 🔗 [Related Projects & Resources](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant#-14-related-projects--resources-)

---

## 🔍 1. Introduction [↑](#-table-of-contents)

Modern CCTV systems often come with their own closed ecosystems, mobile apps, and cloud dependencies. While this works for average users, advanced users (us with HA) and integrators increasingly seek **local control**, **event-based automations**, and **unified dashboards**.

This article documents my hands-on experience integrating a **Dahua NVR and multiple IP cameras** into **Home Assistant**, using tools like `go2rtc` for RTSP stream handling and the official Dahua integration for **smart event-based triggers**.

The goal was simple:  
- **Stream all cameras reliably inside Home Assistant**
- **Use Dahua's advanced detection events (tripwires, IVS, smart motion)** as triggers 
- **Avoid cloud lock-in**
- And **make security visuals and controls available on wall-mounted tablets and mobile phones.**

If you're looking for a **stable, private, and responsive** way to bring your CCTV system into your smart home, this guide covers the full pipeline: from camera installation and Dahua setup to smart lighting automations, dashboards, and UI best practices.

---

## 🧰 2. Requirements [↑](#-table-of-contents)

Before starting the integration, make sure you have the following or equivalent components in place.

### 🖥️ Core Software
- **Home Assistant OS**  
  _Also compatible with Supervised and Docker-based installs_
- **HACS (Home Assistant Community Store)** – we will install it later
- **go2rtc Add-on** – will install later

### 📸 CCTV Hardware
- **Dahua NVR** with Smart Motion/IVS Event Support  
  _Model used by me: `DHI-NVR2108HS-I`_
- **Dahua IP Cameras** with Tripwire / IVS Support / SmartMotion 
  _Models used by me: `DH-IPC-HFW3441E-SA`_

### 🌐 Network and Power
- **PoE Switch** (Power over Ethernet)  
  _Used to power and connect all cameras directly_
- **Structured LAN/WAN Setup**  
  _Router with internal-only access to NVR/cams; no port forwarding required, unless you want to use Dahua App in WAN_

### 💾 Storage & Performance
- **Sufficient NVR Storage Capacity**  
  _Depends on: camera resolution, bitrate, retention duration, 2-4Tb is usually enough_
- **Decent HA hardware**  
  _Raspberry Pi 4 is a minimal baseline. More cams = better CPU and storage. Also, dashboard tablets/phones should have decent performance to quickly load multiple streams_

---

## 🧱 3. Dahua Cameras Overview & Integration Limitations [↑](#-table-of-contents)

Dahua is one of the most widely used CCTV brands worldwide, known for offering a good balance between price, performance, and professional features. Their lineup includes:
- **NVRs** (Network Video Recorders) that centralize recording and event processing
- **IP cameras** that support ONVIF, RTSP, and in many cases, **smart detection events** like tripwire crossing, intrusion zones, and face detection

### 🧩 Ecosystem Breakdown
- Dahua NVRs are designed to work best with Dahua-brand cameras, but **ONVIF-compatible** models from other brands (e.g., Reolink, Hikvision) may work with limited features.
- **Smart Detection Events** (IVS) are often processed on-camera, with the NVR aggregating or forwarding them to third-party systems, like Home Assistant.
- Most Dahua cameras support **RTSP streaming**, which is very useful for integrating with HA dashboards and go2rtc to achieve the best performance results.

### ⚠️ Integration Limitations
While Dahua hardware is solid, integration into Home Assistant comes with some constraints:
- The **official Dahua HA integration** supports only **basic smart events** (motion, IVS tripwire, tamper). Advanced analytics (e.g., facial detection, people counting) are **not exposed** via the current API.
- **Two-way audio and PTZ control** is very limited or not available through Home Assistant, more on that later.
- Camera settings must still be configured through the **web interface or Dahua ConfigTool**.
- No official or clean MQTT support.

### 🛒 Advice for New Buyers
If you're building a new system:
- Choose cameras that support **IVS events (tripwire, intrusion zones)** — these work well in HA.
- Stick to **fixed lens models** unless you specifically need motorized zoom or PTZ.
- Consider using **turret-style cameras** outdoors — less glare and IR bleeding compared to domes.

### 📦 Local vs. Cloud Storage
- Dahua NVRs store footage locally on internal HDDs.
- Optionally, cameras can **send snapshots via FTP to NAS or external storage** on motion.
- Unlike cloud-based brands (e.g., Eufy, Arlo), Dahua systems allow **fully local operation**, which is ideal for privacy-conscious users.

In comparison with other known brands, like Hikvision, Dahua offers pretty much the same capabilities, depending on the exact model lines. Here is an example of comparison Hikvision vs Dahua vs Tiandy(another Chinese brand).

<details>
<summary>📸 Camera Comparison (Click to Expand)</summary>

<img width="1080" height="1135" alt="image" src="https://github.com/user-attachments/assets/ecf4e5db-5186-4be9-bda4-e833d15f5d54" />

</details>


---

## 🛠️ 4. Camera(s) Installation Tips and Web-based Setup [↑](#-table-of-contents)

Setting up CCTV cameras isn't just about plugging them in. The physical mounting, angle, network planning, and web-based configuration all impact reliability, detection accuracy, and automation success.

### 📍 4.1 Physical Installation Tips [↑](#-table-of-contents)

- **Mounting height:** For human detection, 2.5–3 meters is ideal. Too high and you miss faces/details, too low and you decrese field of view and invite tampering.
- **Avoid glare:** Don’t point cameras directly at light sources (sun!, lamps, or car headlights). Use **turret-style cams** outdoors to reduce IR reflection.
- **Shelter from weather:** Even IP67-rated cams benefit from small roof covers. It protects lenses from raindrops and maintains clean image quality. While installing, make sure to use additional compatible camera mounts(IP67). They are sold separately. 
- **Cable management:** Always use weatherproof boxes or grommets for cable exits. Very important advice - do not neglect RJ45 waterproof connector caps, even if you are sure there is no water getting near the camera! One unfortunate rain/thunderstorm/flood and POE can short contacts. For long-term durability, run PoE via proper CAT5e/6 cabling inside walls or conduit.
- **No blind spots:** Review camera fields of view in real time during installation. Avoid overly wide angles where humans become tiny blobs.
- Do not neglect **personal safety** on ladders/hights! Make sure someone is available to help you. 

> 🛠️ _Pro tip:_ Use your phone and a PoE tester or temporary injector to preview the stream live while adjusting position.

Here is a **short video of one of my cams**, just to demonstrate how it's mounted and provide some visual tips:

<details>
<summary>🎬 **Camera Mount** (Click to Expand)</summary>

[![HA - AJAX Demonstration](https://img.youtube.com/vi/RLNLpIZlIpg/0.jpg)](https://www.youtube.com/watch?v=RLNLpIZlIpg)

</details>

Another **video** **-** **overview of NVR, POE Switch, network and Security Monitoring Station** setup:

<details>
<summary>🎬 My Dahua Setup (Click to Expand)</summary>

[![HA - AJAX Demonstration](https://img.youtube.com/vi/ba6nMvc4q2w/0.jpg)](https://www.youtube.com/watch?v=ba6nMvc4q2w)

</details>

---

### 🌐 4.2 Web-Based Configuration [↑](#-table-of-contents)

Each Dahua camera has a built-in web UI, accessible via IP address. This is where most critical config is done before HA integration.

- **Initial IP Setup:**  
  Use Dahua’s [ConfigTool](https://www.dahuasecurity.com/support/downloadCenter) to scan and assign IP addresses, or access directly if your router lists new devices. I prefer doing it via WEB-UI, as long as cam gets IP from DHCP(enabled by default). Also, an important thing - make sure to assign Static IPv4 to your cameras and NVR(s) or static DHCP binds in your router/dhcp server configs, and inside cam/nvr WEB-UI/Dahua tool - HA integration will use one IP to get data from Dahua device, and we would not want it to change. <details>
                                                                                                <summary>📸 IP Config (Click to Expand)</summary>
                                                         <img width="1643" height="679" alt="image" src="https://github.com/user-attachments/assets/75ef5e8d-88ac-44e6-bf71-8d486d1b3ddc" /> </details> 

    > 🔐 _Security Tip:_ Disable UPnP on your router and change default Dahua passwords immediately.

- **Port Layout (defaults):**  
  - Web UI: `HTTP 80` / `HTTPS 443`
  - RTSP: `554` - change if you need
  - ONVIF - uses HTTP/HTTPS for communication: usually `8080` or `80`  or `443`
  - TCP/UDP: `37777`/`37778` - to communicate with NVR, Apps, etc., - network ports. ConfigTool often uses port `37020` for discovery.
  - Max Connection `10` (1~20) - can be used as an additional security layer if you are sure how many max connections(Live views, for example) can happen at a time
  <details>
                                                                  <summary>📸 Ports Config (Click to Expand)</summary>

  <img width="1618" height="697" alt="image" src="https://github.com/user-attachments/assets/efbbe5a6-ef65-4cb8-a378-f2e6468ebf4d" /> </details>

- **Time Sync:**  
  Somehow, Dahua still has well-known issues with Time. Enabling NTP and syncing with your PC didn't work for me most of the time(my home lab + a few commercial installs). What I recommend is choosing your Time Zone and enabling DST(Daylight Saving Time) if it happens in your region. Still, after that, your Time Zone may be 1 hour ahead of the real clock.. so try picking similar time zones, and eventually you will set achieve the correct time. Oh, and it's best to set up settings propagation from NVR to IP Cams for convenience and ease of set-up. Just make sure you configure static IPs first.

  Here are a few screenshots of System/Time WEB Config:
  <details>
                                                          <summary>📸 System Config Propagation - on NVRs web-page(Click to Expand)</summary>

    <img width="1310" height="777" alt="image" src="https://github.com/user-attachments/assets/ca31568e-4370-45e8-b076-246054c1e1dc" /> </0details> 

  <details>
    <summary>📸 **Time Settings** (Click to Expand)</summary>

    <img width="1220" height="877" alt="image" src="https://github.com/user-attachments/assets/9ae990e5-db76-4180-b0a6-f7804879052b" /> </details>

- **Stream Setup (RTSP):** [↑](#-table-of-contents)
  IP Cameras usually have a few stream channels - Main Stream for high-quality recordings, saved to HDD/NAS/Cloud, and Sub Streams for Live Views(Mobile App, 3rd Party Services(Home Assistant), RTSP Web streams, etc.) Enable both **Main Stream** (for recordings) and **Sub Stream** (for dashboards or live view). Match resolutions and codecs to what go2rtc or HA supports best — typically H.264.
  <details>
    <summary>📸 My Stream Config (Click to Expand)</summary>

  <img width="1752" height="724" alt="image" src="https://github.com/user-attachments/assets/49e2f215-72a6-46bc-867b-2b4427dfe189" /> </details>
  Now, Dahua Cams/NVRs(and lots of other vendors) do not have a dedicated "RTSP ON/OFF" switch, and RTSP(_Real Time Streaming Protocol_) is enabled by default. What is important is to understand URL Link structure for Dahua Cams/NVRs. Those do differ vendor to vendor.

  Example:

        `rtsp://USERNAME:PASSWORD@CAMERA_IP:554/cam/realmonitor?channel=1&subtype=0`

    Where:

     - `USERNAME` and `PASSWORD` are the credentials set up on cam web interface(or propagated from NVR)

     - `CAMERA_IP` is cam's IPv4/IPv6 address

     - `554` Port is 554 by default

     - `channel=1` — first camera input (for NVRs or standalone cam)

     - `subtype=1` — sub stream (low-res, low bitrate), subtype=0 is usually Main Stream, just FYI


 - **Regarding **ONVIF**(_Open Network Video Interface Forum_)** - it allows devices from different manufacturers to work together seamlessly, even if they don't have native compatibility. It also includes features like device discovery, configuration, user management, events, and PTZ (pan-tilt-zoom) control. And allows for it's own **network discovery** - this will become usefull later when we configure go2rtc Add-on for HA, so I recommend enabling it.
   <details>
      <summary>📸 ONVIF Config (Click to Expand)</summary>
    <img width="1626" height="649" alt="image" src="https://github.com/user-attachments/assets/46d87547-33f8-4a41-9f14-12148b5c2965" /> </details>


- **User Management:** [↑](#-table-of-contents)
  Create a dedicated `homeassistant` user with only the needed permissions(live is enough). Avoid using admin credentials for automation integrations. Make sure to write down the username and password - those are used to access the RTSP link.
  Example:

  <details>
    <summary>📸 Add user (Click to Expand)</summary>

    <img width="1605" height="712" alt="image" src="https://github.com/user-attachments/assets/b85ead90-bcc4-4571-979d-bb8de4436da5" /> </details>


**This covers basic configs to manage Camera, get Live RTSP Stream from it, and discover them via ONVIF. But we need the events themselves to bind Home Assistant Automations to. In order to do this, let's make sure Event(s) are configured correctly on the Camera(s)**

---

- **Events:** [↑](#-table-of-contents)
  
  To cover all bases, I have **Video Detection**(Only Motion Detection), **Smart Motion Detection**(Recognizes Human/Vehicle silhouette), **Audio Detection**(Abnormality threshold based), **Smart Detection**(IVS   plan), and **Abnormality Detection**(Network disconnect, Access violation, and Voltage shifts) enabled. Let's cover them one by one:

    - **Video Detection**(Motion Detection) - uses a generic algorithm and sensitivity threshold to compare frames and trigger an Event. See the Screenshot for the configuration. <details>
                                                                                                                                                                      <summary>📸 Video Detection Tab (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1665" height="826" alt="image" src="https://github.com/user-attachments/assets/53f87545-f60c-4383-9c2e-f70828b2844e" /> </details>
    - **Smart Motion Detection** - uses an advanced algorithm to recognize Human/Vehicle silhouettes to trigger Event. <details>
                                                                                                                                                                      <summary>📸 Smart Motion Detection Tab (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1560" height="520" alt="image" src="https://github.com/user-attachments/assets/70a83fbe-0377-4d69-a0d3-a6ea50f37dda" /> </details>
 
    - **Audio Detection** - uses built-in microphone and threshold set to fire an Event if the loudness of sound(db) exceeds said threshold. If you want to catch your neighbours' night party "on video" for evidence =) <details>
                                                                                                                                                                      <summary>📸 Audio Detection Tab (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1339" height="866" alt="image" src="https://github.com/user-attachments/assets/685ce275-d6da-4a00-9de8-c9e28fe58bff" /> </details>
      

    - **Smart Detection** - manually set borders and "tripwires" for the IVS (Intelligent Video Surveillance) video analytics algorithm to fire an Event if said borders were crossed. A more precise and advanced event scheme can be used alongside others to get more flags/stamps when searching events on the recorded timeline. <details>
                                                                                                                                                                      <summary>📸 Smart Plan & IVS Tabs (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1304" height="569" alt="image" src="https://github.com/user-attachments/assets/e88c948d-c23f-4dea-b4ef-50653793e4ec" />
        <img width="1781" height="880" alt="image" src="https://github.com/user-attachments/assets/474bcfed-3f6d-4326-9ba0-b665c1653b35" /> </details>
    - **Abnormality Detection** - uses system/device monitoring as Event triggers. Can be Brute force login attempts, Network or Power outages, IP conflicts, offline Storage, or even camera lid/casing opening events. I have Network disconnects, unsuccessful logins, and power events used as triggers. <details>
                                                                                                                                                                      <summary>📸 Smart Plan & IVS Tabs (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1357" height="406" alt="image" src="https://github.com/user-attachments/assets/d71e27da-34ab-4d61-9818-467c6b544347" /> </details>


One more **important thing regarding events** - you may have a question about "overlapping events", like - "what if we have all of those enabled simultaneously, and a screaming person runs into our camera's frame? Is there going to be a dozen similar recordings cluttering up our storage?" The short answer is - No. For a longer explanation, I have researched it for you! Additional read under spoiler:

<details>
<summary>📼 TL;DR – What happens if multiple events are triggered at once? (Click to Expand)</summary>

------------------------------------
**Usual Camera Behavior**
The camera itself sends individual event notifications (Motion, SMD, Audio, IVS, etc.) — each as a separate trigger. But it does not handle recording logic — it just says: "Hey! Something happened!"

**NVR Behavior – The Decision Maker**
The NVR controls recording, and here's what matters:

📅 Recording Schedule Type
In the NVR settings (under “Storage > Schedule”), you define which types of events should trigger recordings:

  - Regular – always recording
  - Motion – triggered by basic motion detection
  - Alarm – includes audio detection, IVS, SMD, etc.

MD+Alarm – combo events

If you have Motion + Audio + SMD + IVS all enabled, and they're set to trigger "Alarm Recording", the _NVR will start one continuous recording during that event window, even if multiple triggers fire_.
It will not create 3 separate video files — it records a unified stream, with a timestamped metadata log of each event.

**Important Notes:**
Dahua NVRs merge overlapping event triggers into one recording stream. The pre-record and post-record buffer applies once — not per trigger. For example, if SMD and audio fire within 2 seconds of each other, the NVR just keeps rolling. No duplicated storage or clutter, unless you explicitly configure snapshot uploads or FTP per event.

**Example Scenario:**
A person screams and runs across the camera's field of view, triggering:

  - Motion Detection (entire frame change)
  - Smart Motion (SMD) for Human
  - Audio Detection (scream)

**Result:**
A single recording clip is created (e.g., 18:03:22–18:03:58)
Each event is logged with its type and timestamp
You can search/filter later by event type

------------------------------------

</details>


---

## 🏠 5. Integrating with Home Assistant [↑](#-table-of-contents)

Now that your Dahua cameras and NVR are physically installed, configured, and confirmed to be working (ONVIF/RTSP tested, static IPs set, events firing), let’s integrate them into **Home Assistant** to begin automation and dashboard magic.

We’ll be using the **community-made Dahua integration by `@rroller`**, which supports motion detection, IVS events (tripwire, intrusion), tamper sensors, and more. It’s **not part of core HA**, so we install it via **HACS**.

---

### 📦 5.1 Installing Dahua Integration from HACS

You need to have HACS installed (see [HACS documentation](https://hacs.xyz/docs/setup/download)).

🔗 Integration [GitHub Repo](https://github.com/rroller/dahua) - for detailed description, installation guide.

#### 🪜 Step-by-step:
1. Open HA → **HACS → Integrations → + Explore & Download Repositories**  
2. Search for **“Dahua”**  
3. Select **“Dahua NVR/Camera Integration”** by `rroller`  
4. Click **Download**, then **Restart Home Assistant**

Once that’s done:

5. Go to **Settings → Devices & Services → + Add Integration**  
6. Search for **Dahua**  
7. Enter the IP address, ports, channel, and Events(Type) and login of your **NVR** or **camera**. I have added both, but it may be enough to add just IP Cams. 
   - You can use either HTTP (port 80) or HTTPS (port 443)
   - RTSP port is 554 by default. Change it if you use a different port
   - Make sure you select the right Channel(0 if you add single camera and 0,1,2,3,.. if you add cameras through NVR and Cams are assigned different channels in NVR config and NVR Config Channel 1 will be 0 when added, NVR Chanell 2 will be 1 and so on) 
   - Enter the credentials of the `homeassistant` user you have [created earlier](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant?tab=readme-ov-file#-42-web-based-configuration-) in Dahua's Web Interface.
   - Choose Events checkboxes depending on functions available for your camera, like Face Detection, parking Detection, Crowd or Rioting Detection, Object Placement/Removal, etc.
   - Once done, the Dahua integration entries will be populated with entities. When finished, I advise assigning proper names, rooms/areas, and labels to entities for easier navigation and management.

<details>
<summary>📸 Dahua Integration in HA (Click to Expand)</summary>

<img width="1444" height="864" alt="image" src="https://github.com/user-attachments/assets/40972427-6a45-4717-a5a2-9ae59c0f0b84" />

</details>

---

### 🎛️ 5.2 Understanding What Gets Added 

The integration will detect:
- All available **channels** on your NVR (CAM1, CAM2, ...)
- All supported **binary sensors** for events
- Other misc. and config/diagnostic entities
Such as:
  - ⚡🚨 `binary_sensor.cross_line_alarm, binary_sensor.cross_region_detection, binary_sensor.motion_alarm, etc.` for tripwire, motion detection events, etc.
  - 🛑 `camera.main, camera.sub, camera.sub_2, etc.` for stream channels
  - 🧠 `binary_sensor.smart_motion_human and binary_sensor.smart_motion_vehicle` for smart motion detection events

<details>
<summary>📸 Dahua Cam Entry (Click to Expand)</summary>

<img width="1442" height="884" alt="image" src="https://github.com/user-attachments/assets/8ec31438-8b5d-4319-b123-df317d53a598" />

</details>

You can now use these as triggers in automations!

**Quick Example:**

```yaml
alias: Tripwire trigger for gate alert
trigger:
  - platform: state
    entity_id: binary_sensor.binary_sensor.smart_motion_vehicle_garage_cam
    to: "on"
```

---

### 🚫 5.3 Integration Limitations

The current Dahua HACS integration **does not support:**
- Two-way audio or microphone streaming from HA to Camera(you can try going around it and add 1 or 2-way audio via go2rtc add-on, see [go2rtc Configuration section](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant?tab=readme-ov-file#-64-manual-configuration-go2rtcyaml) or check out my [Dahua VTO Doorbells Github Repo/Guide](https://github.com/AlexeiakaTechnik/Integration-of-Dahua-VTOs-Doorbells-into-Home-Assistant-and-creating-UI-for-it)).
- Full PTZ control or presets  
- Face detection / People counting / Smart tracking (can be circumvented by using Frigate or other AI object detection tool - see [Further Improvement section](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant?tab=readme-ov-file#-11-conclusions-and-further-improvement-) for more details)
- Event push via WebSocket — it's **poll-based**, meaning events may appear with a **1–5s delay**

Despite that, it’s **very stable and reliable**, and more than enough to trigger lights, alarms, notifications, or camera popups.

---

### 🛠️ 5.5 Troubleshooting Tips

- ✅ **Nothing is detected?** Double-check IP, ports, and that ONVIF + motion events are enabled in the camera/NVR.  
- 🔑 **Wrong credentials?** Try logging in via browser to test the same user/password.  
- 🔃 **Wrong channel detected?** Re-add the integration using direct camera IP instead of the NVR (or vice versa).  
- 🧼 **Reset + Retry:** Sometimes it helps to remove the integration, restart HA, and re-add cleanly.

---

This wraps up the **HA-level integration layer**.  
In the next chapter, we’ll cover how to **embed live RTSP feeds** into your dashboards and optimize them using the `go2rtc` add-on. 


---

## 📡 6. RTSP Streams and go2rtc Configuration [↑](#-table-of-contents)

While the Dahua integration brings smart event triggers into Home Assistant, it does not handle live video streams as fast as we need to populate an HA dashboard with multiple live Camera feeds.  
For that, we’ll use **go2rtc** — a powerful, low-latency multiprotocol stream proxy that works perfectly with Dahua's RTSP output.

This chapter covers setting up live video streams inside Home Assistant using Dahua’s RTSP and `go2rtc` add-on.

---

### ⚙️ 6.1 What is go2rtc and Why Use It?

[go2rtc](https://github.com/AlexxIT/go2rtc) is a lightweight stream aggregator that:
- Accepts RTSP, RTMP, WebRTC, HTTP, UDP, etc.
- Transcodes or relays streams on demand
- Supports **camera autodiscovery**, low-latency viewing, and **multi-client support**

In short, it’s the easiest and most efficient way to integrate Dahua live video into HA dashboards.

<img width="2560" height="1280" alt="image" src="https://github.com/user-attachments/assets/4b2599d8-4752-4b69-80cb-a3cb4d1f9979" />


### 🧠 6.1.1 Why go2rtc Feeds Perform Better Than `camera.main` / `camera.sub`

<details>
<summary>📼 Explained in detail (Click to Expand)</summary>

------------------------------------

While the Dahua integration automatically exposes RTSP-based entities like `camera.main`, `camera.sub`, and `camera.sub_2`, these often perform worse in dashboards compared to `Generic Camera` entities created manually via **go2rtc**.

Here’s why:

- 🔄 **Dahua camera entities** are routed through **Home Assistant’s internal camera proxy layer**, which:
  - Adds overhead by trying to "fetch" frames through HA core
  - May not maintain persistent streaming
  - Often results in **slow stream startup**, **higher latency**, and **greater CPU/RAM use**
  - Can stutter or buffer, especially when multiple streams are shown at once

- 🚀 **go2rtc**, by contrast:
  - Acts as a **dedicated RTSP relay/proxy**, optimized for real-time streaming
  - **Keeps camera streams active in memory** or reconnects intelligently on demand
  - **Avoids transcoding** or frame proxying unless explicitly configured
  - Supports **native MJPEG**, **RTSP**, **WebRTC**, and **HLS** delivery
  - Integrates directly with custom frontend cards (like [WebRTC Card](https://github.com/AlexxIT/webrtc))

> ✅ **Bottom Line:**  
> go2rtc reduces latency, avoids HA camera subsystem bottlenecks, and gives you **more control** over performance, compatibility, and stream delivery format.

This is especially noticeable on:
- Wall-mounted tablets (e.g., Fully Kiosk)
- Older Android dashboards
- Mobile views showing 2+ cameras simultaneously

**Best practice:**  
Avoid using `camera.main` / `camera.sub` from the Dahua integration for live dashboards. Instead, rely on go2rtc-powered `Generic Camera` or `WebRTC` entities for smooth and efficient video streaming.

------------------------------------

</details>

---

### 📥 6.2 Installing go2rtc via HA Add-on Store

1. Go to **Settings → Add-ons → Add-on Store**
2. Click **⋮ → Repositories**
3. Add:  
   https://github.com/AlexxIT/hassio-addons
4. Install **go2rtc** from the list
5. Start the add-on and optionally enable **"Start on boot"**, **Watchdog**, and **Show in Sidebar** for easier hopping.

---

### 🔎 6.3 Discovering Dahua RTSP Streams (Autodetect via ONVIF)

Once go2rtc is running:
- Go to **Settings → Addons → go2rtc** and open **Web UI** or just open go2rtc from sidebar
- Click **Add** → it will open a menu with available autodiscovery options, choose ONVIF, and it will scan your network for ONVIF-compatible devices
- Dahua cams or NVR channels should appear as options, like `name: IPs or Hostames` and `url: onvif://user:pass@1[IP or Hostname]`
- Select a stream URL and copy it into the `test` box, replace `user` and `pass` with your Cam's `homeassistant` user credentials, and press test
- After that, it will show available livestream URLs, depending on what was configured for your cam (e.g., `stream01` - most likely "main", `stream02`, or `snapshot`)
- Copy the URL you want to add from the right column, already with user and pass replaced, and go to the Config tab
- In the Config tab, add the  following lines:

```text
streams:
  [Cam name - like New_test_cam]:
    - [Your URL copied from previous step, like onvif://[user]:[pass]@[IP like 192.168.X.XXX]?subtype=MediaProfile00000]
  [Next Cam name]
    - [Next URL]
etc.
```
- Press **Save & Restart**
  
go2rtc will save these in its internal config

If autodetection doesn’t work, you can add streams manually.

<details>
<summary>📸 Screenshots from go2rtc (Click to Expand)</summary>

<img width="1915" height="720" alt="image" src="https://github.com/user-attachments/assets/8309297b-d8ec-459f-804b-198e2873503a" />
<img width="1105" height="781" alt="image" src="https://github.com/user-attachments/assets/568681a8-7464-4254-a45e-5832935c228e" />


</details>

---

### 📝 6.4 Manual Configuration (`go2rtc.yaml`)

Once go2rtc is running:
- Go to **Settings → Addons → go2rtc** and open **Web UI** or just open go2rtc from sidebar
- Go to Config tab
- In Config tab, add the following lines:

```text
streams:
  [Cam name - like New_test_cam]:
    - [Your Dahua Cam Main Stream RTSP URL, available already, ex. rtsp://[user]:@[pass]@[IP]/cam/realmonitor?channel=1&subtype=0&unicast=true&proto=Onvif]
    - [Your Dahua Cam Sub Stream RTSP URL, available already, ex. rtsp://[user]:@[pass]@[IP]/cam/realmonitor?channel=1&subtype=1]
  [Next Cam name]
    - [Next Main Stream URL]
    - [Next Sub Stream URL]
etc.
```
- Press Save & Restart

> 🔐 **Tip:** Don't forget to use a dedicated `homeassistant` user with minimum read permissions and password protect your admin user. If brute force protection or 2FA are availble - use them! CCTV Systems are a popular target for hacking, so avoid casual vulnerabilities and exploits.


<details>
<summary>📸 go2rtc config tab (Click to Expand)</summary>

<img width="1262" height="370" alt="image" src="https://github.com/user-attachments/assets/23b8b47c-3821-40e7-948f-be6f3f7230c6" />

</details>
---

### ⚠️ 6.5 Explaining go2rtc URLs 

You may have noticed that RTSP URLs are a bit different than what we have looked at in the [4.2 Web-Based Configuration](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant?tab=readme-ov-file#-42-web-based-configuration-) chapter under **Stream Setup (RTSP)** section. And we add two of them under one single camera. Let me explain why we do it this way.

Here’s a full breakdown of what each parameter means:

**🔢 Query Parameters**

| Parameter         | Purpose                                 | Recommendation                           |
|------------------|------------------------------------------|-------------------------------------------|
| `channel=1`       | Refers to CAM1 on NVR or standalone cam | Set based on which camera stream you want |
| `subtype=0`       | Main stream (high quality, high bitrate) | Use for popups or full-screen views       |
| `subtype=1`       | Sub stream (low-res, low bitrate)        | Use for dashboards or previews            |
| `unicast=true`    | Forces direct (unicast) stream delivery  | ✅ Always enable for go2rtc and HA         |
| `proto=Onvif`     | Requests ONVIF-compliant RTSP behavior   | Optional, but improves compatibility      |

`/cam/realmonitor` - This is the standard Dahua RTSP path for **live video streaming**


🎯 Why Use Both `subtype=0` and `subtype=1` in go2rtc?

By including **both streams** in your `go2rtc.yaml` entry:

```yaml
streams:
  front_yard_cam:
    - rtsp://...&subtype=0
    - rtsp://...&subtype=1
```

You enable **smart fallback and adaptive streaming**:
- go2rtc will automatically pick the appropriate stream based on the protocol requested:
  - **MJPEG or low-res view** → `subtype=1`
  - **WebRTC or full-screen view** → `subtype=0`
  But I prefer to set up my Dashboards manually - low-res subtype for views where multiple cards with live streams are loaded and hi-res subtypes for pop-up full view of a single camera, as will be explained in [UI Chapter](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant?tab=readme-ov-file#%EF%B8%8F-8-ui-and-dashboards-).

This gives you the **best of both worlds**:
- Efficient dashboards (fast, lightweight)
- Sharp popups and detailed views (when needed)

> 🧠 Pro Tip: You can test and compare streams manually in VLC or go2rtc debug UI to see the difference in resolution, latency, and CPU load.

---

### 🖼️ 6.5 Adding go2rtc Streams to Home Assistant

Now let's create Generic Camera entities via YAML or UI to use them in our UI cards. HA official cards and custom unofficial UI addons usually require `camera.cam_name` entity.

1. Go to **Settings → Devices & Services → Add Integration**
2. Type **Generic Camera** in search bar
3. For **stream source URL** use:
   rtsp://localhost:8554/[Cam name - as set up in go2rtc Config file]
4. You can ignore **still image URL** or set it up if you have set it up in go2rtc
5. Leave **RTSP transport protocol** as TCP if you have not changed it
6. And **Authentication** as Basic, since the link is taken from go2rtc addon, where we did not set up security
7. Enable **Verify SSL Certificate** so that dashbards accessed via remote connection(if your HA is accessible through your private Domain(ex. Cloudflare tunneling) or Nabu Casa Cloud) would show up without issues
8. Check new Generic Camera Feed

<details>
<summary>📸 Adding Generic Camera (Click to Expand)</summary>

<img width="1435" height="875" alt="image" src="https://github.com/user-attachments/assets/2eef4a64-7e92-4b93-a7b3-b58ab6065015" />

</details>

> 🧠 You can use go2rtc to consolidate multiple streams, rebroadcast to WebRTC, or create a mixed local/remote setup. Check out it's [Github](https://github.com/AlexxIT/go2rtc/) readme and see Streams tab in your go2rtc after you added your cams in go2rtc's Config file - there will be _links_ button in **Commands** column. Open it and there will be many generated or generatable links to different types of output streams you can further use in your HA dashboards.

<details>
<summary>📸 go2rtc links (Click to Expand)</summary>

<img width="1263" height="882" alt="image" src="https://github.com/user-attachments/assets/7e64b7a8-fc19-4bed-bc26-7c884645f32f" />

</details>

---

### 🚀 6.6 Performance Tips

- Use **substreams (`subtype=1`)** for dashboards to reduce bandwidth and CPU usage
- Main streams are better for popups, zoomed-in views, or snapshots
- go2rtc supports **motion JPEG**, **RTSP**, and **WebRTC** simultaneously — test what works best for your dashboard devices
- If using older tablets, MJPEG might be the most compatible option

---

### 🧩 6.7 What About Frigate or WebRTC?

- If you need **object detection**, check out [Frigate](https://frigate.video/)
- If you want **instant loading, no transcoding**, use go2rtc with WebRTC via [AlexxIT's WebRTC card](https://github.com/AlexxIT/webrtc)

We’ll revisit these advanced cases in the **"Further Improvements"** [chapter](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant?tab=readme-ov-file#-11-conclusions-and-further-improvement-).

---

Now that video feeds are live and integrated, we’re ready to start building **event-driven automations** based on smart triggers from Dahua.


---

## ⚡ 7. Automations Based on Dahua Provided Events [↑](#-table-of-contents)

Once your Dahua cameras are integrated into Home Assistant and their smart detection events (IVS, SMD, tamper, etc.) are exposed as `binary_sensor` entities, you can begin automating your home based on real-time surveillance input.

These event-based automations are the heart of a responsive, secure smart home setup — enabling actions like:
- Turning lights on when someone enters a zone
- Triggering alarms or snapshots on tripwire detection
- Muting music or pausing media when someone approaches
- Logging or notifying when tampering is detected

---

### 🚨 7.1 Types of Events You Can Use

Depending on your camera model and Dahua settings, the integration can expose:

| Event Type           | Binary Sensor Entity Example                         | Use Case Example                              |
|----------------------|------------------------------------------------------|-----------------------------------------------|
| Smart Motion (Human) | `binary_sensor.front_gate_camera_smart_motion_human` | Turn on lights, send snapshot                  |
| IVS Tripwire         | `binary_sensor.driveway_tripwire_crossed`            | Trigger TTS announcement, flash lights        |
| Tamper Detection     | `binary_sensor.garage_cam_tamper`                    | Send security alert or siren notification     |
| Audio Detection      | `binary_sensor.livingroom_camera_audio_detected`     | Flash warning light or log event              |

---

### 🧠 7.2 Best Practices for Event Automation

- Combine **multiple triggers** (motion, time, light state) for smarter logic  
- Use **sun elevation / offset** instead of static time for lighting  
- Avoid false positives: test IVS zones thoroughly  
- Always include a manual override (`input_boolean`) for disabling automations

---

### 📄 Let’s take a look at a real-world YAML automation I am using

<details>
<summary>📄 Example: Smart Motion-Based Courtyard Light Control(Click to Expand)</summary>

```yaml
alias: Outdoor Light Control via Dahua Smart Motion
description: "Automatically turns courtyard light ON/OFF based on Dahua camera motion events after sunset"
triggers:
  - platform: time_pattern
    minutes: "/5" ## Let's trigger automation every 5 minutes to check if the lights were left ON
    id: periodic_check
  - platform: state
    entity_id: binary_sensor.courtyard_camera_smart_motion_human ##lets use SmartMotion event as a trigger
    to: "on"
    id: motion_detected
  - platform: state
    entity_id: binary_sensor.courtyard_camera_smart_motion_human
    to: "off"
    id: motion_cleared
condition:
  - condition: state
    entity_id: input_boolean.outdoor_light_automation_enabled ##I use helpers to control automation's behavior from UI dashboard
    state: "on"
action:
  - choose:
      - alias: Turn ON Light After Sunset (on Motion)
        conditions:
          - condition: sun
            after: sunset
            after_offset: "01:00:00" ##I add offset because there is still enough light after sunset in my area
          - condition: trigger
            id: motion_detected
        sequence:
          - action: light.turn_on
            target:
              entity_id: light.courtyard_pool_stairs
      - alias: Turn OFF Light After Sunset (motion cleared)
        conditions:
          - condition: sun
            after: sunset
            after_offset: "01:00:00"
          - condition: trigger
            id: motion_cleared
        sequence:
          - action: light.turn_off
            target:
              entity_id: light.courtyard_pool_stairs
      - alias: Auto-Off After 10min Inactivity ##lets make sure lights turn off in no one is present
        conditions:
          - condition: sun
            after: sunset
            after_offset: "01:00:00"
          - condition: trigger
            id: periodic_check
          - condition: state
            entity_id: binary_sensor.courtyard_camera_smart_motion_human
            state: "off"
          - condition: device
            type: is_on
            entity_id: light.courtyard_pool_stairs
            for:
              minutes: 10
        sequence:
          - service: light.turn_off
            target:
              entity_id: light.courtyard_pool_stairs
mode: single
```

</details>

---

### 🧰 7.3 Optimization Tips

- ✅ **Use `choose:` structure** to branch logic based on trigger type  
- ☀️ Replace `after_offset` with an `input_datetime` for dynamic tuning  
- 🧠 Consider `for:` duration on motion trigger instead of relying only on periodic checks  
- 💡 Use scenes instead of direct `light.turn_on` for more ambiance  
- 🛑 Include `input_boolean` toggles for override control on the dashboard  

---

### 💡 7.3 Advanced Enhancements & Related Reading

This kind of smart light automation can be further improved by layering in **context-aware conditions** and **manual override mechanisms**.

Here are some powerful enhancements you may want to explore:

- 🔆 **Ambient Light (LUX) Sensors**  
  Instead of relying solely on sunset, use a calibrated LUX sensor to trigger lighting only when **natural light drops below a specific threshold**.  
  → Ideal for cloudy days, shaded areas, or indoor/outdoor transitions.

- 🛡️ **Integration with Security Systems**  
  Link this with your **alarm or security automation** logic:
  - Flash courtyard lights if the alarm is armed and motion is detected
  - Use visual deterrents as part of intrusion detection or perimeter breach logic

- 🗣️ **Temporary Pause via UI or Voice Assistant**  
  Add an `input_boolean` like `pause_light_automation` that you can toggle:
  - From a dashboard switch  
  - Via Alexa or Google Assistant voice command  
  - From a wall-mounted tablet  
  This is great if you want to **manually leave the light on** for a BBQ, chill-out night, or guest evening without motion-triggered cutoffs.

- 🧠 **Other Ideas to Explore**:
  - 🕒 Limit max ON time (e.g., light auto-turns off after 90 minutes regardless of motion)
  - 🌗 Use **sun elevation** instead of just `sunset + offset` for smoother seasonal behavior
  - 🔄 Switch from `time_pattern` to **timer helper + restartable delay** for better performance
  - 🕹️ Add a `scene` for night-mode lighting vs. full-bright floodlight based on context (e.g., after midnight, use dimmer light)

---

📚 **Want a deeper dive into smart lighting?**

👉 Check out my dedicated article:  
[💡 My Experience of Improving Light Automations in Home Assistant →](https://github.com/AlexeiakaTechnik/My-experience-of-improving-Light-Automations-in-Home-Assistant)

It includes real-world examples, iterative improvements, and performance insights gathered across multiple Home Assistant deployments.

In the next chapter, we’ll explore how to represent these smart events and video feeds visually using Home Assistant Dashboards, Popups, and Custom Cards.

---

## 🖥️ 8. UI and Dashboards [↑](#-table-of-contents)

<img width="1583" height="1058" alt="image" src="https://github.com/user-attachments/assets/eaf3990f-7ab7-444f-ab82-c882e565fdb6" />

Once your streams and event triggers are functional, the next step is building an intuitive and fast UI for daily interaction.

In this chapter, we’ll showcase a **multi-purpose Dahua camera control card** that:
- Shows a **live camera preview** with lower resolution optimized to include multiple cards like this in a single view
- Opens a **popup view with real-time stream** on tap
- Provides **quick-access buttons** to:
  - Open gates or doors
  - Toggle lights (including Dahua's IR lighting and built-in spotlight)
  - See changes **in real time**, obviously on cam live view
- Is styled to visually stand out on dashboards

This is a great example of how Dahua cameras + Home Assistant can form a **security and control dashboard** you’ll actually use.

---

### 🧩 Custom Elements Used

| Integration / Card       | Purpose                                   | Repo Link                                                                 |
|--------------------------|-------------------------------------------|---------------------------------------------------------------------------|
| `stack-in-card`          | Stack multiple cards with shared border   | [https://github.com/custom-cards/stack-in-card](https://github.com/custom-cards/stack-in-card) |
| `browser_mod`            | Used for popups on tap                    | [https://github.com/thomasloven/hass-browser_mod](https://github.com/thomasloven/hass-browser_mod) |
| `picture-glance`         | Base live camera view + entity overlay    | Built-in to Home Assistant                                               |
| `picture-elements`       | Custom tap-to-toggle icons over image     | Built-in to Home Assistant                                               |
| `lovelace-layout-card`   | Enables grid extended placement for cards | [https://github.com/thomasloven/lovelace-layout-card](https://github.com/thomasloven/lovelace-layout-card) |

---

### 🧪 Example Card: Live Camera + Controls 

Please carefully read ##comments!

<details>
<summary>📸 Card YAML Configuration with Entity Controls(Click to Expand)</summary>

```yaml
type: custom:stack-in-card
title: Entrance Level 1
card_mod: ##this is a styling section to beautify the card
  style: |
    ha-card {
      border-style: solid;
      border-width: 3px;
      border-radius: 5px;
      --ha-card-header-color: orange;
      --ha-card-header-font-size: 30px;
      min-height: 350px;
      height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
    }
mode: vertical
cards:
  - camera_view: live
    type: picture-glance
    entities:  ##here we add togglable entities to a quick glance view
      - entity: switch.gate_relay
      - entity: lock.door_front
      - entity: light.entryway_cam_spotlight
      - entity: light.entryway_cam_ir_light
      - entity: light.courtyard_lamp_posts
      - entity: light.courtyard_door_spotlight
    camera_image: camera.1_level_front_entrance_camera ##this is our Generic Camera entity we created in [chapter 6.5]
    entity: automation.main_hall_tv_turn_on_off_automation ##just a placeholder entity - choose any available that doesn't change states much
    hold_action: ##open larger view on hold action
      action: navigate
      navigation_path: >-
        https://YOUR_REMOTE_HA_DONAIN_ADDRESS/api/hassio_ingress/GO2RTC_ID/webrtc.html?src=1st_Level_Entrance_Cam&media=video+audio ##THIS IS IMPORTANT - for the stream to be available remotely, we use **link generated by go2rtc** addon, as mentioned in [🖼️ 6.5 Adding go2rtc Streams to Home Assistant chapter], particularly **webrtc.html local WebRTC viewer** link at the bottom of the page
    tap_action:
      action: fire-dom-event ##here we add browsermod functionality to open pop-ups on the device that currently displays the dashboard
      browser_mod:
        service: browser_mod.popup
        data:
          title: 1st Level Entrance
          timeout: 600000 ##set a timeout for how long the popup lasts
          content:
            type: picture-elements
            camera_image: camera.1_level_front_entrance_camera_hd ##this is our Generic Camera entity we created in [chapter 6.5], but with a Higher Resolution than the Main Dahua RTSP Stream
            camera_view: live
            elements: ##here we add toggleable entities to the popup view, note that we use style: to adjust their position and scale on live view
              - type: state-icon
                icon: mdi:spotlight
                style:
                  top: 85%
                  right: 5%
                  transform: scale(1.2, 1.2)
                tap_action:
                  action: toggle
                entity: light.entryway_cam_spotlight
              - type: state-icon
                icon: mdi:weather-night
                style:
                  top: 75%
                  right: 5%
                  transform: scale(1.2, 1.2)
                tap_action:
                  action: toggle
                entity: light.entryway_cam_ir_light
              - type: state-icon
                icon: mdi:gate-open
                style:
                  top: 65%
                  right: 5%
                  transform: scale(1.2, 1.2)
                tap_action:
                  action: toggle
                entity: switch.gate_relay
              - type: state-icon
                icon: mdi:door-open
                style:
                  top: 55%
                  right: 5%
                  transform: scale(1.2, 1.2)
                tap_action:
                  action: toggle
                entity: lock.door_front
              - type: state-icon
                icon: mdi:track-light
                style:
                  top: 45%
                  right: 5%
                  transform: scale(1.2, 1.2)
                tap_action:
                  action: toggle
                entity: light.courtyard_door_spotlight
              - type: state-icon
                icon: mdi:coach-lamp-variant
                style:
                  top: 35%
                  right: 5%
                  transform: scale(1.2, 1.2)
                tap_action:
                  action: toggle
                entity: light.courtyard_lamp_posts
columns: 1
view_layout: ##this is the element from lovelace_layout_card custom UI integration from HACS, I use them to position cards in grid with coordinates
  grid-area: c4

```

</details>

**This is what a pop-up may look like:**

<details>
<summary>📸 Cam Live Stream View in HA (Click to Expand)</summary>

<img width="1240" height="839" alt="image" src="https://github.com/user-attachments/assets/32f0ea16-c498-426f-82f8-8e5575cbcf9c" />


</details>
---

### 💡 Tips for Better Camera Dashboards

- 🖼️ Use `picture-elements` inside popups for **precise button placement**
- 🔁 Combine with `conditional` cards to **hide controls when not needed or when situational context requires them**
- 📱 Works great on **wall tablets** using Fully Kiosk Browser
- 🎨 Style with `card_mod` to differentiate security panels visually
- 🔊 Add `media_player` buttons to doorbell popups for announcements or TTS, especially cool if your doorbell can be used as a media_player entity in HA or you have a speaker/tablet/other player nearby

---

📚 **Want more on dashboard design for tablets and phones?**  
👉 See my article:  
[📱 Practical and Stylish Home Assistant Dashboards for Tablets & Phones →](https://github.com/AlexeiakaTechnik/Practial-and-stylish-Home-Assistant-Dashboards-for-Tablets-and-Mobile-Phones)

---

## 🔐 9. Privacy and Security [↑](#-table-of-contents)

In today's digital world, cybersecurity is **non-negotiable**, especially when dealing with surveillance systems. Whether you're a homeowner, business owner, or someone with high public visibility — your CCTV system can either protect you or become your biggest vulnerability. IP cameras and NVRs are among the most **frequent targets of automated hacking bots**, script kiddies, and advanced actors scanning the web 24/7 for exposed systems.

A compromised camera system can mean more than lost footage — it can result in stalkers, blackmail, physical break-ins, or public embarrassment.

This chapter outlines how to **avoid becoming a cautionary tale.**

---

### 🚫 Why You Should NOT Expose NVR or Camera Ports Publicly

Port forwarding your NVR or camera web interfaces (HTTP/HTTPS/RTSP) to the internet is **inherently risky**. If you must do it — take precautions:

#### 🧱 1. Use Firewalls Correctly
- Restrict incoming connections to only the exact ports you're forwarding
- Apply firewall rules **on both your router and the NVR/device itself**
- Only allow known public IPs (e.g., your phone's mobile network range)

#### 🔄 2. Keep Firmware Updated
- Update camera and NVR firmware regularly to patch critical vulnerabilities
- Subscribe to Dahua or vendor mailing lists for firmware notices

#### 🔐 3. Strong Authentication
- Never leave default passwords — **change them immediately**
- Use strong, unique passwords (12+ characters, random)
- Enable **2FA/MFA** where available

#### 🕵️ 4. Prefer VPNs Over Port Forwarding
- Set up OpenVPN, WireGuard, or Zerotier on your home network
- Or use a VPN-enabled router to restrict remote access only through encrypted tunnels
- **Never expose port 554 (RTSP) or 37777 (Dahua default) to the open internet**

#### 🧪 5. Monitor and Scan
- Use tools like `fail2ban`, `Uptime Kuma`, or HA logs to monitor for failed logins
- Regularly scan your network with tools like Nmap or Nessus

#### 📐 6. Use Network Segmentation
- Isolate your cameras on a separate VLAN or subnet
- Block internet access to local-only devices via router rules
- Only HA or NVR should have full access to them

#### 🔒 7. Close Unused Ports
- Disable RTSP if not using it
- Close ONVIF and HTTP ports if not required
- Change all default service ports to non-standard ones

#### 🛠️ 8. Additional Tips
- Consider **ngrok**, **Cloudflare Tunnel**, or **Nabu Casa** to access HA securely
- Use a **dedicated device** for externally accessible services
- Limit camera info exposure (e.g., no firmware banners or brand IDs)

---

### 🛡️ Secure Your Home Assistant

Home Assistant is often the gateway to your full smart home. A compromised HA instance can lead to:
- Device takeover
- Access to recorded clips and camera feeds
- Triggered automations or remote door/gate unlocks

**Best Practices:**
- Use **HTTPS** via Nabu Casa or Cloudflare Tunnels
- Enforce **strong passwords** and 2FA for **all admin users**
- Create dedicated users with **limited rights** for mobile dashboards or family
- Enable IP ban after failed login attempts (e.g., `ip_ban_enabled: true` in `configuration.yaml`)
- Review `home-assistant.log` regularly for suspicious activity

---

### 🧽 Privacy Controls for Guests and Indoor Spaces

Having guests over? Don’t want cameras in private zones?

Add dashboard toggles to:
- Temporarily disable bathroom/pool/bedroom cameras
- Mute indoor microphones or IR illuminators
- Visibly show guests when indoor recording is paused

This isn’t just about ethics or laws — it builds **trust and transparency**.

---

### 📡 Basic Router and Network Best Practices

| Practice              | Description                                                    |
|-----------------------|----------------------------------------------------------------|
| VLANs/Subnets         | Segment IoT devices from your main PC/mobile LAN               |
| Disable UPnP          | Prevent automatic opening of risky ports                       |
| Set DHCP Reservations | Lock camera IPs to fixed addresses to avoid IP rotation        |
| Use DNS Blocking      | Stop telemetry calls to unknown vendor cloud addresses         |
| Log Traffic           | Log incoming and outgoing traffic from critical devices        |

---

### 👁️ Why CCTV Systems Are a Hacker's Dream

Surveillance systems are:
- **Always on**
- Often **misconfigured**
- Commonly run **outdated firmware**
- Installed with **default passwords**
- Expose critical real-world info like **floorplans, routines, and people**

That makes them perfect targets for:
- Harassment and voyeurism
- Ransom and extortion
- Botnet inclusion (DDoS armies)
- Provocation (fake alerts, IR blinding, spotlight trolling)

> ⚠️ The more public your profile or business is, the more you should **treat your cameras like doors and vaults**. Because that’s what they are.

---

## 🎬 10. Live Video Demonstration [↑](#-table-of-contents)

_Youtube short showing live demonstration of courtyard lights turned on by Smart Motion Human event received from Dahua IP Camera:_

[![HA - Dahua - Automation Demo](https://img.youtube.com/vi/W4evMrAUYnw/0.jpg)](https://www.youtube.com/watch?v=W4evMrAUYnw)

---

## 🧩 11. Conclusions and Further Improvement [↑](#-table-of-contents)

Integrating CCTV into a smart home is one of those things that feels like **a hassle at first**… but once done properly, it becomes **indispensable**.
Gone are the days of a dozen buggy mobile apps with delayed notifications, cloud subscriptions, and forgotten cameras.  
With Home Assistant, you actually **use the security infrastructure you paid for** — in real-time, with automation, context awareness, and full local control.

---

### ✅ Integration Pros

- Full camera visibility inside your smart home dashboards
- Motion/IVS/tamper events as automation triggers
- Can react to activity in real-time (not just review it afterward), especially if notification automations are configured
- Dahua works well with HA once configured — solid mid-tier gear, but many other vendors do too! In any case **do research before buying!** Can't stress this enough!
- Combines well with lights, locks, doorbells, alarms, and security systems

### ⚠️ What to Watch Out For

- Initial setup is time-consuming
- Camera UI and config logic isn’t exactly user-friendly, but the more you do it - the less troubling it gets. And HA community is fantastic with sharing and caring, so you will probably find ready-made solutions!
- Not all camera models are equal, cheap vendors are cutting corners — again, **research before buying!**
- Some events/controls (e.g., 2-way audio) may not be accessible in HA easily

That said — the **benefit is huge**:  
With HA you’ll **notice if something breaks** (camera offline, motion stops), not just when disaster strikes, and, God forbid, you have to hand over footage to the Police and notice that some cameras have been offline for a whie. With Smart Home integration, you look at your cameras periodically!
More importantly — you gain a real **sense of control and awareness** over your environment, and can act **before** things escalate.

---

## 🚀 Further Improvements

Once you’ve built your basic integration — the next level is **real AI-powered video analytics**.  
Let’s talk about **Frigate NVR**.

---

### 🧠 What is Frigate?

<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/06831f41-d6fa-4cbc-a1e8-fe4a5731801a" />

[Frigate NVR](https://docs.frigate.video/) is an open-source NVR platform for real-time object detection, optimized for use with Home Assistant.  

It runs independently and can:
- Record 24/7 or by event
- Detect humans, pets, cars, bikes, specific objects, maybe even dust gathering on your sofa =)
- Generate **rich events** and **clips** with bounding boxes
- Support **license plate reading**, **mask detection**, and more
- Integrate with **deep learning models** and **semantic search**

> Unlike Dahua's limited smart detection — **Frigate lets you truly know what's going on**, with context and AI-powered insights.

---

### 🐕 What Can You Do With It?

- Turn on lights only when a human is detected (not just motion)
- Ignore trees and shadows blowing in the wind
- Notify if your **pet escapes the yard**
- Detect **robot vacuums** and **cleaning events** and make smart adjustments
- Fire alerts if a **door or window appears open** visually, without using physical sensors
- Build automations on **who** or **what** is seen — not just “motion” - Face recognition of impressive quality! 
- Tag events and search recordings using semantic filters and AI powered categorization

Imagine asking:  
> “Show me all clips from today where someone entered the garage after 6PM and carried a bag.”

And your Home Assistant with an integrated AI Agent and Frigate opens up a window with such events! This is the true magic-like future of Smart Homes in my opinion, and while commercial platforms are getting into it and cost... well A LOT, with HA, Frigate server and skills/time it's very real for anyone!

---

### 🧰 Hardware Considerations

Frigate is powerful… but it needs horsepower.

To run a serious Frigate setup, you’ll want:
- A dedicated mini-PC, server, virtual machine, or Docker host 
- Minimum **16GB RAM**, **SSD storage**, and **reliable LAN** and **mid-range CPU**
- **GPU (Coral, NVIDIA, Intel VAAPI, etc.)** for real-time detection
- Cameras with **consistent RTSP streams** (which you already have!)

Once deployed, Frigate can run standalone, or side-by-side with Home Assistant.  
Many users separate it into its own device or VM for better performance, scalability and reliability.

---

### 👷 My Personal Plans

At the time of writing, I haven’t deployed Frigate yet — but I’m **definitely planning to**.

> Right now my schedule is split between client work, article writing, HA tinkering, and generally getting consumed by the **DIY Smart Home OCD inducing rabbit hole(which I love <3)™**...  

The goal is to eventually add Frigate to my setup with:
- AI detection
- Smart recording + snapshots
- Fully searchable footage
- And **next-level automations** based on real-world activity

When I do — I will write up it's own GitHub article or die trying, so stay tuned!. 😉

---

## 🪪 12. License [↑](#-table-of-contents)

This project is licensed under the [MIT License](https://github.com/AlexeiakaTechnik/Integrating-Dahua-Cameras-CCTV-System-in-Home-Assistant/blob/main/LICENSE).

---

## 👨‍💻 13. Author and Inspiration [↑](#-table-of-contents)

_Alexei Halaim_  

Smart Home Professional, Integrator, Home Assistant enthusiast.
Practical experience integrating security systems and video surveillance into Home Assistant and building dashboards for real clients and personal projects.

Email: alexei.aka.technik@gmail.com
LinkedIn: [Link](https://www.linkedin.com/in/alexei-halaim-b62326172/)
Reddit: [Link](https://www.reddit.com/user/Unlikely-Tax-2700/)

---

## 🔗 14. Related Projects & Resources [↑](#-table-of-contents)

- [🔐 AJAX Security System Integration in Home Assistant](https://github.com/AlexeiakaTechnik/AJAX-Security-Integration-in-Home-Assistant)  
- [📱 Practical and Stylish HA Dashboards for Tablets & Phones](https://github.com/AlexeiakaTechnik/Practial-and-stylish-Home-Assistant-Dashboards-for-Tablets-and-Mobile-Phones)  
- [🚪 Integrating Dahua VTO Doorbells in Home Assistant](https://github.com/AlexeiakaTechnik/Integration-of-Dahua-VTOs-Doorbells-into-Home-Assistant-and-creating-UI-for-it)
