
<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/a789fb5c-b4d1-4430-99a1-ca86ca08a189" />


# 🎥 Dahua Camera/CCTV System in Home Assistant (🚧UNDER CONSTRUCTION🚧)
_Getting Dahua NVRs and cameras into Home Assistant with actionable smart events.  
Integration walkthrough with go2rtc, smart motion detection, light automations, RTSP streaming setup, and best-practice camera configs._

---

## 📚 Table of Contents

1. [🔍 Introduction](#-1-introduction)  
2. [🧰 Requirements](#-2-requirements)  
3. [🧱 Dahua Cameras Overview & Integration Limitations](#-3-dahua-cameras-overview--integration-limitations)  
4. [🛠️ Cameras Installation Tips and Web-based Setup](#-4-cameras-installation-tips-and-web-based-setup)  
5. [🏠 Integrating with Home Assistant](#-5-integrating-with-home-assistant)  
6. [📡 RTSP Streams and go2rtc Configuration](#-6-rtsp-streams-and-go2rtc-configuration)  
7. [⚡ Automations Based on Dahua Provided Events](#-7-automations-based-on-dahua-provided-events)  
8. [🖥️ UI and Dashboards](#-8-ui-and-dashboards)  
9. [🔐 Privacy and Security](#-9-privacy-and-security)  
10. [🎬 Live Video Demonstration](#-10-live-video-demonstration)  
11. [🧩 Conclusions and Further Improvement](#-11-conclusions-and-further-improvement)  
12. [🪪 License](#-12-license)  
13. [👨‍💻 Author and Inspiration](#-13-author-and-inspiration)  
14. [🔗 Related Projects & Resources](#-14-related-projects--resources)  

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
  _Depends on: camera resolution, bitrate, retention duration, 2-4Tb is usually enogh_
- **Decent HA hardware**  
  _Raspberry Pi 4 is a minimal baseline. More cams = better CPU and storage. Also dashboard tablets/phones should have decent performance to quickly load multiple streams_

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
  Create a dedicated `homeassistant` user with only the needed permissions(live is enough). Avoid using admin credentials for automation integrations. Make sure to write down username and password - those are used to access RTSP link.
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
    - **Abnormality Detection** - uses system/device monitoring as Event triggers. Can be Brute force login attempts, Network or Power outages, IP conflicts, offline Storage, or even camera lid/casing opening events. I have Network disconnect, unsuccessful logins, and power events used as triggers. <details>
                                                                                                                                                                      <summary>📸 Smart Plan & IVS Tabs (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1357" height="406" alt="image" src="https://github.com/user-attachments/assets/d71e27da-34ab-4d61-9818-467c6b544347" /> </details>


One more **important thing regarding events** - you may have a question about "overlapping events", like - "what if we have all of those enabled simultaneously, and a screaming person runs in our camera's frame? Is there going to be a dozen similar recordings cluttering up our storage?" The short answer is - No. For a longer explanation, I have researched it for you! Additional read under spoiler:

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


![image](PLACEHOLDER FOR SCREENSHOTS FROM HA-DAHUA INTEGRATION)

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

In short: it’s the easiest and most efficient way to integrate Dahua live video into HA dashboards.

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
- Click **Add** → it will open menu with available autodiscovery options, choose ONVIF and it will scan your network for ONVIF-compatible devices
- Dahua cams or NVR channels should appear as options, like `name: IPs or Hostames` and `url: onvif://user:pass@1[IP or Hostname]`
- Select a stream url and copy it into the `test` box, replace `user` and `pass` with your Cam's `homeassistant` user credentials, press test
- After that it will show available livestream URLs, depending on what was configured for your cam (e.g., `stream01` - most likely "main", `stream02`, or `snapshot`)
- Copy URL you want to add from the right column, already with user and pass replaced, and go to Config tab
- In Config tab add following lines:

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


> 🧠 You can use go2rtc to consolidate multiple streams, rebroadcast to WebRTC, or create a mixed local/remote setup. Check out it's [Github](https://github.com/AlexxIT/go2rtc/) readme and see Streams tab in your go2rtc after you added your cams in go2rtc's Config file - there will be _links_ button in **Commands** column. Open it and there will be many generated(-able)

---

/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

### 🚀 6.6 Performance Tips

- Use **substreams (`subtype=1`)** for dashboards to reduce bandwidth and CPU usage
- Main streams are better for popups, zoomed-in views, or snapshots
- go2rtc supports **motion JPEG**, **RTSP**, and **WebRTC** simultaneously — test what works best for your dashboard devices
- If using older tablets, MJPEG might be the most compatible option

---

### 🧩 6.7 What About Frigate or WebRTC?

- If you need **object detection**, check out [Frigate](https://frigate.video/)
- If you want **instant loading, no transcoding**, use go2rtc with WebRTC via [AlexxIT's WebRTC card](https://github.com/AlexxIT/webrtc)

We’ll revisit these advanced cases in the **"Further Improvements"** chapter.

---

Now that video feeds are live and integrated, we’re ready to start building **event-driven automations** based on smart triggers from Dahua.

![image](PLACEHOLDER FOR SCREENSHOTS FROM GO2RTC AND HA CAMERA DASHBOARD)

---

## ⚡ 7. Automations Based on Dahua Provided Events [↑](#-table-of-contents)

- Supported Events: Motion, IVS line crossing, tamper, tripwire
- Automation ideas:  
  - Motion triggers lights at night  
  - Tripwire triggers alarm sound  
  - Face/tamper triggers snapshot archive
- Example YAML Automations

```yaml
alias: Porch Light On via Tripwire
trigger:
  - platform: state
    entity_id: binary_sensor.dahua_tripwire_front
    to: 'on'
condition:
  - condition: sun
    after: sunset
action:
  - service: light.turn_on
    target:
      entity_id: light.porch
```

![image](PLACEHOLDER FOR NODE-RED OR AUTOMATION FLOWCHARTS)

## 🖥️ 8. UI and Dashboards [↑](#-table-of-contents)

- Picture Glance cards with live stream  
- Popups with `browser_mod`  
- Custom controls (light, gate, alarm)  
- Best UI layout practices for tablets and phones

![image](PLACEHOLDER FOR DASHBOARD AND POPUP UI)

---

## 🔐 9. Privacy and Security [↑](#-table-of-contents)

- Why not expose NVR ports?  
- Use of VPN or local-only access  
- Secure HA: strong passwords, 2FA  
- UI buttons to disable indoor recording (e.g. bathrooms, pools)  
- Best practice router setup (VLANs, firewalls)

![image](PLACEHOLDER FOR UI SECURITY BUTTONS OR SETTINGS)

---

## 🎬 10. Live Video Demonstration [↑](#-table-of-contents)

_Embedded YouTube link showing:_
- Overview of physical setup  
- Live camera feeds on wall-mounted tablet  
- Example: night detection triggers lights  
- Example: tripwire triggers alert & popup

![image](PLACEHOLDER FOR YOUTUBE THUMBNAIL)

---

## 🧩 11. Conclusions and Further Improvement [↑](#-table-of-contents)

_Summary of integration pros/cons, reliability, and what worked best._

Possible upgrades:
- Adding Frigate for object detection  
- Face recognition with Dlib/Deepstack  
- Snapshot archival to NAS  
- Expand to multi-site camera control

---

## 🪪 12. License [↑](#-table-of-contents)

MIT / CC BY-NC-SA / or your preferred license.

---

## 👨‍💻 13. Author and Inspiration [↑](#-table-of-contents)

_Alexei aka Technik_  
Practical experience integrating security systems into Home Assistant and building dashboards for real clients and personal projects.

---

## 🔗 14. Related Projects & Resources [↑](#-table-of-contents)

- [🔐 AJAX Security System Integration in Home Assistant](https://github.com/AlexeiakaTechnik/AJAX-Security-Integration-in-Home-Assistant)  
- [📱 Practical and Stylish HA Dashboards for Tablets & Phones](https://github.com/AlexeiakaTechnik/Practial-and-stylish-Home-Assistant-Dashboards-for-Tablets-and-Mobile-Phones)  
- [🚪 Integrating Dahua VTO Doorbells in Home Assistant](https://github.com/AlexeiakaTechnik/Integration-of-Dahua-VTOs-Doorbells-into-Home-Assistant-and-creating-UI-for-it)
