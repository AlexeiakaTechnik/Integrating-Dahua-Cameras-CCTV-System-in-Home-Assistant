
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
- **Stream all cameras reliably inside Home Assistant**,  
- **Use Dahua's advanced detection events (tripwires, IVS)** as triggers,  
- **Avoid cloud lock-in**,  
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

## 🛠️ 4. Cameras Installation Tips and Web-based Setup [↑](#-table-of-contents)

Setting up CCTV cameras isn't just about plugging them in. The physical mounting, angle, network planning, and web-based configuration all impact reliability, detection accuracy, and automation success.

### 📍 4.1 Physical Installation Tips [↑](#-table-of-contents)

- **Mounting height:** For human detection, 2.5–3 meters is ideal. Too high and you miss faces/details, too low and you decrese field of view and invite tampering.
- **Avoid glare:** Don’t point cameras directly at light sources (sun!, lamps, or car headlights). Use **turret-style cams** outdoors to reduce IR reflection.
- **Shelter from weather:** Even IP67-rated cams benefit from small roof covers. It protects lenses from raindrops and maintains clean image quality. While installing, make sure to use additional compatible camera mounts(IP67). They are sold separately. 
- **Cable management:** Always use weatherproof boxes or grommets for cable exits. Very important advice - do not neglect RJ45 waterproof connector caps, even if you are sure there is no water getting near the camera! One unfortunate rain/thunderstorm/flood and POE can short. For long-term durability, run PoE via proper CAT5e/6 cabling inside walls or conduit.
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
  Use Dahua’s [ConfigTool](https://www.dahuasecurity.com/support/downloadCenter) to scan and assign IP addresses, or access directly if your router lists new devices. I prefer doing it via WEB-UI, as long as cam gets IP from DHCP(enabled on default). Also, important thing - make sure to assign Static IPv4 to your cameras and NVR(s) or static DHCP binds in your router/dhcp server configs and in cam WEB-UI/Dahua tool - HA integration will use one IP to get data from Dahua device and we would not want it to change.

<details>
<summary>📸 IP Config (Click to Expand)</summary>

<img width="1643" height="679" alt="image" src="https://github.com/user-attachments/assets/75ef5e8d-88ac-44e6-bf71-8d486d1b3ddc" />

</details> 

> 🔐 _Security Tip:_ Disable UPnP on your router and change default Dahua passwords immediately.

- **Port Layout (defaults):**  
  - Web UI: `HTTP 80` / `HTTPS 443`
  - RTSP: `554` - change if you need
  - ONVIF - uses HTTP/HTTPS for communication: usually `8080` or `80`  or `443`
  - TCP/UDP: `37777`/`37778` - to communicate with NVR, Apps, etc., - network ports. ConfigTool often uses port `37020` for discovery.
  - Max Connection `10` (1~20) - can be used as an additional security layer if you are sure how many max connections(Live views, for example) can happen at a time

<details>
<summary>📸 Ports Config (Click to Expand)</summary>

<img width="1618" height="697" alt="image" src="https://github.com/user-attachments/assets/efbbe5a6-ef65-4cb8-a378-f2e6468ebf4d" />

</details>

- **Time Sync:**  
  Somehow, Dahua still has well-known issues with Time. Enabling NTP and syncing with your PC didn't work for me most times(my home lab + a few commercial installs). What I really recommend is choosing your Time Zone and setting up DST(Daylight Saving Time) if it happens in your region. Still, after that your Time Zone may be 1 hour ahead of real clock.. so try picking similar time zones and eventually you will set up normal time. Oh, and it's best to set up settings propagation from NVR to IP Cams for convenience and ease of set-up, just make sure you configure static IPs first.

Here are a few screenshots of System/Time WEB Config:
<details>
<summary>📸 System Config Propagation - on NVRs web-page(Click to Expand)</summary>

<img width="1310" height="777" alt="image" src="https://github.com/user-attachments/assets/ca31568e-4370-45e8-b076-246054c1e1dc" />

</details> 

<details>
<summary>📸 **Time Settings** (Click to Expand)</summary>

<img width="1220" height="877" alt="image" src="https://github.com/user-attachments/assets/9ae990e5-db76-4180-b0a6-f7804879052b" />

</details>

- **Stream Setup (RTSP):** [↑](#-table-of-contents)

IP Cameras usually have a few stream channels - Main Stream for high-qulity recordings, saved to HDD/NAS/Cloud and Sub Streams for Live Views(Mobile App, 3rd Party Services(Home Assistant), RTSP Web streams, etc.) Enable both **Main Stream** (for recordings) and **Sub Stream** (for dashboards or live view). Match resolutions and codecs to what go2rtc or HA supports best — typically H.264. 

<details>
<summary>📸 My Stream Config (Click to Expand)</summary>

<img width="1752" height="724" alt="image" src="https://github.com/user-attachments/assets/49e2f215-72a6-46bc-867b-2b4427dfe189" />

</details>

  Now, Dahua Cams/NVRs(and lots of other vendors) do not have a dedicated "RTSP ON/OFF" switch, and RTSP(_Real Time Streaming Protocol_) is enabled by default. 
What is important is to understand URL Link structure for Dahua Cams/NVRs. Those do differ vendor to vendor.

  Example:
      ```text
    rtsp://USERNAME:PASSWORD@CAMERA_IP:554/cam/realmonitor?channel=1&subtype=0
      ```
    Where:
    - `USERNAME` and `PASSWORD` are the credentials set up on cam web interface(or propagated from NVR), 
    - `CAMERA_IP` is cam's IPv4/IPv6 address
    - `554` Port is 554 by default
    - `channel=1` — first camera input (for NVRs or standalone cam)
    - `subtype=1` — sub stream (low-res, low bitrate), subtype=0 is usually Main Stream just FYI

 - **Regarding **ONVIF**(_Open Network Video Interface Forum_)** - it allows devices from different manufacturers to work together seamlessly, even if they don't have native compatibility. It also includes features like device discovery, configuration, user management, events, and PTZ (pan-tilt-zoom) control. And allows for it's own **network discovery** - this will become usefull later when we configure go2rtc Add-on for HA, so I recommend enabling it.

<details>
<summary>📸 ONVIF Config (Click to Expand)</summary>

<img width="1626" height="649" alt="image" src="https://github.com/user-attachments/assets/46d87547-33f8-4a41-9f14-12148b5c2965" />

</details>


- **User Management:** [↑](#-table-of-contents)
  Create a dedicated `homeassistant` user with only the needed permissions(live is enough). Avoid using admin credentials for automation integrations. Make sure to write down username and password - those are used to access RTSP link.
  Example:

<details>
<summary>📸 Add user (Click to Expand)</summary>

<img width="1605" height="712" alt="image" src="https://github.com/user-attachments/assets/b85ead90-bcc4-4571-979d-bb8de4436da5" />

</details>


This covers basic configs to manage Camera, get Live RTSP Stream from it, and discover them via ONVIF. But we need the events themselves to bind Home Assistant Automations to. In order to do this let's make sure Event(s) are configured correctly on the Camera(s)

- **Events:** [↑](#-table-of-contents)
  To cover all bases, I have **Video Detection**(Only Motion Detection), **Smart Motion Detection**(Recognizes Human/Vehicle silhouette), **Audio Detection**(Abnormality threshold based), **Smart Detection**(IVS plan), and **Abnormality Detection**(Network disconnect, Access violation, and Voltage shifts) enabled. Let's cover them one by one:

    - **Video Detection**(Motion Detection) - uses generic algorithm and sensetivity threshold to compare frames and trigger Event. See Screenshot for configuration. <details>
                                                                                                                                                                      <summary>📸 Video Detection Tab (Click to                                                                                                                                                                                     Expand)</summary>

        <img width="1665" height="826" alt="image" src="https://github.com/user-attachments/assets/53f87545-f60c-4383-9c2e-f70828b2844e" /> </details>
    - 
 
    - 
      

    - *





---

![image](PLACEHOLDER FOR PHOTOS OF INSTALLATION, RACK, POE BOX, SCREENSHOTS FROM DAHUA GUI)
---

## 🏠 5. Integrating with Home Assistant [↑](#-table-of-contents)

- Installing the [Dahua Integration](https://www.home-assistant.io/integrations/dahua/)
- HACS alternatives (if used)
- Adding cameras or NVR via IP + credentials
- Channel explanation (CAM1 = channel 1, etc.)
- Events supported: motion, IVS tripwire, tamper

![image](PLACEHOLDER FOR SCREENSHOTS FROM HA-DAHUA INTEGRATION)

---

## 📡 6. RTSP Streams and go2rtc Configuration [↑](#-table-of-contents)

- What is go2rtc and why use it?
- Installation via [Home Assistant Add-ons](https://github.com/AlexxIT/go2rtc)
- Obtaining RTSP URLs from Dahua
- go2rtc `config.yaml` sample
- Creating `Generic Camera` entities in HA
- Mentions of Frigate for advanced use cases

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
