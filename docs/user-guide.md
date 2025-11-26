# 📖 YouLy+ User Guide

## ⚙️ Configuration
Access the settings dashboard by clicking the **YouLy+ icon** in your browser's toolbar.

---

### 🏠 General
**Core Functionality**
*   **Enable YouLy+:** Master switch to toggle the extension on/off.
*   **Word-by-Word Highlighting:** Enables "Karaoke" style animations where specific words light up in sync with the audio.
*   **Use SponsorBlock API:** Uses crowdsourced data to skip non-music segments (intros, outros), improving lyric timing accuracy.

**Lyrics Sources**
Select where YouLy+ retrieves lyrics from.

| Option | Description |
|:-------|:------------|
| **Lyrics+ (KPoe)** | **(Recommended)** Aggregates multiple sources (Apple Music, Musixmatch, Spotify). |
| **LRCLIB** | An open-source, community-driven lyrics database. |
| **Custom KPoe** | Connect to a self-hosted lyrics server. |

---

### 🎨 Appearance
Customize how the lyrics look and feel.

**Text Display**
*   **Larger Text Mode:** When using Romanization, choose which text takes priority: the **Original Lyrics** or the **Pronunciation**.

**Visual Effects**
*   **Blur Inactive Lyrics:** Focuses on the current line by blurring previous and upcoming lines.
*   **Dynamic Background:** Enables animated album artwork backgrounds in the player view.
*   **Song Palette:** Automatically colors the lyrics to match the album art.

**Performance**
*   **Lightweight Mode:** Disables heavy visual effects to save battery.
*   **Hide Off-screen Lyrics:** Removes invisible lyric lines to improve performance.

---

### 🈯 Translation & Romanization
YouLy+ supports real-time translation and pronunciation guides (Romanization) for non-Latin scripts (e.g., Japanese, Korean).

**Romanization Behavior**
*   **Primary Source:** The extension automatically attempts to fetch high-quality official Romanization from **Lyrics+ (KPoe)** first. This is available for the vast majority of songs.
*   **Fallback:** If official Romanization is missing, it generates it using the provider selected below.

**Fallback Providers (Translation & Romanization)**
| Provider | Setup |
|:---------|:------|
| **Google Translate** | Works out of the box. Good for general use. |
| **Gemini AI** | **(Advanced)** Requires a free API Key. Offers context-aware, higher-quality translations. |

---

### 🎧 Player Interface Controls
When lyrics are active, you will see a control dock in the **bottom-right corner** of the lyrics pane containing two buttons:

**1. <span class="material-symbols-outlined">translate</span> Display Menu**
Click this button to open the toggle menu:
*   **Show Translation:** Toggles the translated lyrics line below the original.
*   **Show Pronunciation:** Toggles the romanized text above the original characters.

**2. <span class="material-symbols-outlined">refresh</span> Reload Button**
*   Forces the extension to re-fetch lyrics. Use this if the timing is off or if the wrong song data was loaded.

> **Note:** The interface supports **i18n**, meaning button labels and menus will automatically appear in your browser's local language (e.g., "Tampilkan Terjemahan" for Indonesian users).

---

### 📂 Local Lyrics
If a song has no lyrics online, you can upload your own.

*   **Supported Formats:** `.lrc` (Simple), `.elrc` (Enhanced), `.ttml` (Apple Music), and `.json`.
*   **Management:** Upload/Delete lyrics via the "Local Lyrics" tab in settings.

---

### 💾 Cache
To speed up loading and save data, YouLy+ saves lyrics locally.

*   **Aggressive:** Caches for 2 hours.
*   **Moderate:** Caches for 1 hour.
*   **None:** Always fetches fresh data.