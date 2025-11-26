# DIY Smart Home Family Calendar (Skylight Clone)

![Main Dashboard View](assets/main_view.jpeg)

## 📖 Introduction
My wife has been recently bombarded in social media with ads for smart home calendars (Skylight, Cozyla, Hearth) and was ready to spend over $300 on one. Before giving her the green light, I asked for a chance to research them.

I realized most offered similar functionality but differed significantly in price. Most importantly, I didn't see any outstanding feature that I couldn't implement in **Home Assistant**.

**The Goal:** A WAF-approved (Wife Acceptance Factor), countertop-friendly touchscreen calendar that integrates deep into our smart home without monthly fees.

## 💡 Why DIY?
Choosing the DIY route with Home Assistant provided several benefits over buying a Skylight/Hearth display:
* **No Monthly Fees:** Avoids subscriptions for "premium" features.
* **Seamless Integration:** It talks to our lights, chores (Grocy), and presence sensors.
* **Old Hardware:** Repurposed a Mini PC and a standard monitor.
* **Privacy:** No vendor lock-in or risk of the company shutting down.

## 🛠 Hardware Selection
The only essential purchase was the screen. It needed to be touch-enabled and aesthetically pleasing for a countertop.

* **Monitor:** [HP Engage 15-inch Touchscreen](https://computers.woot.com/offers/hp-engage-16t-fhd-monitor). I chose this over generic portable monitors because it includes a built-in **Speaker, Webcam, and Microphone**, allowing for future voice control or video calls.
* **Compute:** An old Mini PC (NUC/Tiny PC) running Windows/Linux in Kiosk mode, or a Raspberry Pi 4.

## ✨ Features
* **Family-wide & Individual Views:** Toggle specific family members' calendars on/off.
* **Two-way Sync:** Edit events on the screen or on our phones (Google Calendar).
* **"Add Event" Popup:** A custom UI to add events to specific calendars directly from the screen.
* **Weather & Date:** Beautiful, glanceable header.
* **Responsive:** Automatically adjusts day-count based on screen width (Mobile vs Desktop).

---

## ⚙️ Installation Guide
*Note: This setup uses a **YAML Package** to automatically create all the necessary helpers, scripts, and variables for you. You do not need to create them manually.*

### 1. Prerequisites (HACS)
You must have [HACS](https://hacs.xyz/) installed. Please install the following **Frontend** integrations:
* `week-planner-card`
* `bubble-card`
* `config-template-card`
* `card-mod`
* `better-moment-card`
* `browser_mod` (Required for the popups to work)
* `layout-card` (Required for the Sections view)

### 2. The Backend (The Brains)
1.  Open your `configuration.yaml` file in Home Assistant.
2.  Ensure you have this line added under `homeassistant:` to enable packages:
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```
3.  Create a folder named `packages` in your HA config directory (if you don't have one).
4.  Download [packages/family_calendar.yaml](packages/family_calendar.yaml) from this repo.
5.  Place the file inside your `packages/` folder.
6.  **Restart Home Assistant**.

### 3. The Calendars
You can use **Google Calendars** or **Local Calendars**.

**Option A: Local Calendar (Easiest)**
1.  Go to **Settings > Devices & Services**.
2.  Add the **Local Calendar** integration.
3.  Create calendars named exactly: `Alice`, `Bob`, `Charlie`, `Daisy`, `Family`.
    * *If you use these names, the code works out of the box!*

**Option B: Google Calendar**
1.  Open `packages/family_calendar.yaml`.
2.  Scroll to the `add_google_calendar_event` script.
3.  Update the `calendar_map` to point to your real Google entities:
    ```yaml
    calendar_map:
      "Alice": "calendar.alice_gmail_com"
      "Bob": "calendar.bob_work_account"
    ```

**Setting up Holidays:**
Since Home Assistant updates, Holidays are now added via UI:
1.  Go to **Settings > Devices & Services > Add Integration > Holiday**.
2.  Select your country.
3.  Check the entity ID (e.g., `calendar.holidays`). If it differs from the default, update it in the dashboard YAML.

### 4. The Dashboard (The Look)
1.  Create a new Dashboard View (Set View Type to **Sections**).
2.  Copy the code from [dashboard.yaml](dashboard.yaml).
3.  **Customize:**
    * **Search & Replace:** Replace `person.alice` with your actual family member entities.
    * **Weather:** Replace `weather.home` with your weather provider.
    * **Background:** Update the image URL at the bottom of the yaml.

---

## 📐 How It Works (Under the Hood)

### Filter Logic
The `week-planner-card` does not natively support hiding specific calendars on the fly. To solve this, I used **Input Texts** acting as Regex filters.
* When you click a person's button, it toggles their filter between `.*` (Show everything) and `^$` (Show nothing).
* `config-template-card` injects these variables into the calendar card dynamically.

### Event Creation Script
The "Add Event" popup uses a single script that handles logic for multiple people and event types (All Day vs Timed).

```yaml
# Simplified Logic Example
target_calendar: "{{ calendar_map.get(states('input_select.calendar_select')) }}"

choose:
  - conditions: "All Day Event is ON"
    action: calendar.create_event (start_date, end_date)
  - conditions: "All Day Event is OFF"
    action: calendar.create_event (start_date_time, end_date_time)
```    

