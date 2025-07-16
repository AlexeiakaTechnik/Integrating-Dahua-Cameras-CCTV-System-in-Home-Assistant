
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

- **Home Assistant OS** (or Supervised / Docker setups)
- **Dahua NVR with Smart Motion Support**  
  _(Model used: INSERT MODEL HERE)_
- **Dahua IP Cameras**  
  _(Models used: INSERT MODELS HERE)_
- **POE Switch** for powering and connecting cams
- **LAN/WAN Configuration** – no need to expose ports externally
- **Sufficient storage** (based on bitrate, # of cams, retention period)

---

## 🧱 3. Dahua Cameras Overview & Integration Limitations [↑](#-table-of-contents)

_Explanation of the Dahua ecosystem: NVR vs. standalone cams, ONVIF/RTSP support, smart events, and limitations. Tips for buyers looking to install CCTV systems._  
Comparison to Reolink, Hikvision, etc.  
Discuss storage logic: NVR storage vs. motion-triggered FTP/NAS/cloud.

![image](PLACEHOLDER FOR IMAGES OF: CCTV SYSTEMS GUIDELINES, DAHUA CAMS)

---

## 🛠️ 4. Cameras Installation Tips and Web-based Setup [↑](#-table-of-contents)

### 📍 Physical Installation Tips
- Avoid glare, rain, and overexposure
- Secure cable management and weatherproofing
- Mounting height recommendations and blind spot avoidance

### 🌐 Web-Based Configuration
- Dahua’s IP config tool or direct access via IP
- Time sync issues & fixes
- RTSP, ONVIF setup
- User permissions
- Port layout explanation (HTTP, RTSP, etc.)

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
