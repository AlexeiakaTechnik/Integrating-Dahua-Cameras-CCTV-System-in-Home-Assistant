
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

Before starting the integration, make sure you have the following or equal components in place.

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

_Explanation of the Dahua ecosystem: NVR vs. standalone cams, ONVIF/RTSP support, smart events, and limitations. Tips for buyers looking to install CCTV systems._  
Comparison to Reolink, Hikvision, etc.  
Discuss storage logic: NVR storage vs. motion-triggered FTP/NAS/cloud.

![image](PLACEHOLDER FOR IMAGES OF: CCTV SYSTEMS GUIDELINES, DAHUA CAMS)

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

### 📍 Physical Installation Tips

- **Mounting height:** For human detection, 2.5–3 meters is ideal. Too high and you miss faces/details, too low and you decrese field of view and invite tampering.
- **Avoid glare:** Don’t point cameras directly at light sources (sun!, lamps, or car headlights). Use **turret-style cams** outdoors to reduce IR reflection.
- **Shelter from weather:** Even IP67-rated cams benefit from small roof covers. It protects lenses from raindrops and maintains clean image quality. While installing, make sure to use additional compatible camera mounts(IP67). They are sold separately. 
- **Cable management:** Always use weatherproof boxes or grommets for cable exits. Very important advice - do not neglect RJ45 waterproof connector caps, even if you are sure there is no water getting near the camera! One unfortunate rain/thunderstorm/flood and POE can short. For long-term durability, run PoE via proper CAT5e/6 cabling inside walls or conduit.
- **No blind spots:** Review camera fields of view in real time during installation. Avoid overly wide angles where humans become tiny blobs.
- Do not neglect **personal safety** on ladders/hights! Make sure someone is available to help you. 

Here is a short video of one of my cams, just to demonstrate how it's mounted:

!VIDEO CAMERA MOUNT

> 🛠️ _Pro tip:_ Use your phone and a PoE tester or temporary injector to preview the stream live while adjusting position.

---

### 🌐 Web-Based Configuration

Each Dahua camera has a built-in web UI, accessible via IP address. This is where most critical config is done before HA integration.

- **Initial IP Setup:**  
  Use Dahua’s [ConfigTool](https://www.dahuasecurity.com/support/downloadCenter) to scan and assign IP addresses, or access directly if your router lists new devices. I prefer doing it via WEB-UI, as long as cam gets IP from DHCP(enabled on default). Also, important thing - make sure to assign Static IPv4 to your cameras and NVR(s) or static DHCP binds in your router/dhcp server configs and in cam WEB-UI/Dahua tool - HA integration will use one IP to get data from Dahua device and we would not want it to change.
  
- **Time Sync:**  
  Enable NTP and sync with your Home Assistant device or local NTP server. Time mismatch causes major issues in event automation and recordings.

- **Stream Setup (RTSP):**  
  Enable both **Main Stream** (for recordings) and **Sub Stream** (for dashboards or live view). Match resolutions and codecs to what go2rtc or HA supports best — typically H.264.

- **User Management:**  
  Create a dedicated `homeassistant` user with only the needed permissions. Avoid using admin credentials for automation integrations.

- **Port Layout (defaults):**  
  - Web UI: `HTTP 80` / `HTTPS 443`  
  - RTSP: `554`  
  - ONVIF: `8999` or `80`  
  - ConfigTool often uses port `37020` for discovery
  
> 🔐 _Security Tip:_ Disable UPnP on your router and change default Dahua passwords immediately.

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
