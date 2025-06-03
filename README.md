# OCI Aggressive Instance Creation Bot

A browser console script designed to aggressively attempt to create compute instances on Oracle Cloud Infrastructure (OCI), particularly useful when dealing with "Out of Capacity" issues for certain shapes or Availability Domains (ADs).

**Disclaimer:** This script interacts directly with the OCI web console. Oracle may change its UI at any time, which could break this script. Use this script at your own risk. Understand what it does before using it. Continuously monitor its activity. This tool is for educational and experimental purposes, primarily to overcome temporary capacity limitations for personal or development use. Do not use this in a way that violates OCI's terms of service.

## Features

*   **Aggressive Retries:** Continuously tries to click the "Create" button on the OCI instance creation page.
*   **Dynamic Interval Backoff:** Starts with an initial aggressive interval and increases it if capacity issues persist, up to a maximum interval.
*   **Availability Domain (AD) Cycling:** Optionally cycles through available ADs to find one with capacity.
*   **Session Keep-Alive:** Opens a small popup window to navigate to an OCI page, attempting to keep your OCI session active.
*   **Visual Feedback:**
    *   **Status Bar:** A floating pill-shaped status bar at the bottom of the screen shows current status, countdown, and attempts.
    *   **Notifications:** Pill-shaped notifications for important events (start, stop, potential success, errors).
*   **Console Logging:** Detailed logging in the browser's developer console.
*   **Configurable:** Key parameters like retry intervals and CSS selectors can be adjusted in the script's `CONFIG` section.
*   **Stop Function:** Can be stopped by typing `stopOCIAggroBot()` in the console.

## Prerequisites

*   A modern web browser (Chrome, Firefox, Edge, etc.) with developer tools access.
*   An active OCI account and session.
*   Popups must be allowed for `cloud.oracle.com` for the session keep-alive feature to work reliably.

## How to Use

1.  **Navigate:** Open your web browser and navigate to the OCI console, specifically to the "Create Compute Instance" page. Configure your desired instance shape, image, network, SSH keys, etc., *before* running the script.
2.  **Open Developer Console:**
    *   Most browsers: Press `F12`, or right-click on the page and select "Inspect" or "Inspect Element", then go to the "Console" tab.
3.  **Copy Script:** Copy the entire JavaScript code (`oci-aggro-bot.js` or the code block above).
4.  **Paste and Run:** Paste the copied script into the developer console and press `Enter`.
5.  **Monitor:** The script will start running. Observe the on-screen status bar, notifications, and console logs.
    *   The script will automatically attempt to find the "Create" button (often within an iframe) and start the click cycle.
    *   If it detects a potential success (e.g., the create button disappears or the page redirects), it will pause for a longer duration (e.g., 60 seconds) and notify you to manually verify and stop the script.
6.  **Stop the Script:** To stop the script at any time, type `stopOCIAggroBot()` into the console and press `Enter`.

## Configuration (Top of the Script)

The script includes a `CONFIG` object at the top. You can modify these values before pasting the script into the console:

*   `TRY_DIFFERENT_AD_ON_RETRY` (boolean): `true` to cycle through ADs, `false` to stick to the initially selected one.
*   `INITIAL_INTERVAL_SECONDS` (number): Starting interval in seconds between click attempts.
*   `MAX_INTERVAL_SECONDS` (number): Maximum interval the script will back off to.
*   `INTERVAL_INCREMENT_SECONDS` (number): How many seconds to add to the interval during backoff.
*   `CONSECUTIVE_NO_CHANGE_ATTEMPTS_FOR_BACKOFF` (number): How many clicks with no apparent change (still "Out of Capacity") before increasing the interval.
*   `SESSION_KEEP_ALIVE_URL` (string): URL used by the session keep-alive popup.
*   `COMPUTE_IFRAME_SELECTORS` (array of strings): CSS selectors to find the OCI compute iframe. The script tries these in order. **May need adjustment if OCI UI changes.**
*   `CREATE_BUTTON_SELECTORS` (array of strings): CSS selectors to find the "Create" or "Launch instance" button. **May need adjustment if OCI UI changes.**
*   `CREATE_BUTTON_EXPECTED_TEXTS` (array of strings): Possible text content of the create button (to help identify it).
*   `AD_RADIO_BUTTON_SELECTOR` (string): CSS selector for Availability Domain radio buttons. **May need adjustment if OCI UI changes.**

**Important:** The CSS selectors (`COMPUTE_IFRAME_SELECTORS`, `CREATE_BUTTON_SELECTORS`, `AD_RADIO_BUTTON_SELECTOR`) are crucial and most likely to break if Oracle updates its console UI. If the script fails to find elements, these selectors are the first thing to check and update.

## Troubleshooting

*   **Script doesn't find "Create" button / "OCI FRAME NOT FOUND":**
    *   Ensure you are on the correct "Create Compute Instance" page in OCI.
    *   OCI might have updated its UI. You may need to update the `COMPUTE_IFRAME_SELECTORS` or `CREATE_BUTTON_SELECTORS` in the script's `CONFIG`. Use your browser's developer tools (Inspector) to find the new selectors.
*   **Session Popup Blocked:** Ensure your browser allows popups from `https://cloud.oracle.com`. The script will show a notification if it detects this.
*   **Script stops unexpectedly:** Check the developer console for error messages.

## License

This project is released under the MIT License. See the `LICENSE` file for details (you'll need to create this file if you don't have one, a standard MIT license text will do).

---
