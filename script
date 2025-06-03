// --- OCI AGGRESSIVE INSTANCE CREATION BOT ---
// --- Version: 3.0 (Public Release Refactor) ---

// --- I. CONFIGURATION ---
const CONFIG = {
    TRY_DIFFERENT_AD_ON_RETRY: true,
    INITIAL_INTERVAL_SECONDS: 7,
    MAX_INTERVAL_SECONDS: 15,
    INTERVAL_INCREMENT_SECONDS: 2,
    CONSECUTIVE_NO_CHANGE_ATTEMPTS_FOR_BACKOFF: 3,
    SESSION_KEEP_ALIVE_URL: "https://cloud.oracle.com/compute/instances",
    COMPUTE_IFRAME_SELECTORS: ["#compute-wrapper", "#sandbox-compute-container"],
    CREATE_BUTTON_SELECTORS: [
        ".oui-savant__Panel--Footer .oui-button-primary[data-test-id='primary-button']",
        ".oui-savant__Panel--Footer .oui-button-primary[type='button']",
        ".oui-savant__Panel--Footer .oui-button.oui-button-primary"
    ],
    CREATE_BUTTON_EXPECTED_TEXTS: ["Create", "Launch instance"],
    AD_RADIO_BUTTON_SELECTOR: ".oui-savant__card-radio-option input[type='radio']",
    PANEL_HEADER_SELECTOR: ".oui-savant__Panel--Header"
};

// --- II. SCRIPT SCOPE & STATE ---
(() => {
    'use strict';
    console.clear();
    const logPrefix = "%c◈ OCI AggroBot: ";
    const logStyle = (color = "#87CEFA", weight = "normal") => `background-color: #1C1C1C; color: ${color}; font-weight: ${weight}; border-left: 3px solid ${color}; padding: 2px 0;`;

    const scriptState = {
        computeWindow: null,
        createBtn: null,
        adElmts: [],
        currentAdIndex: 0,
        sessionWindow: null,
        statusElmt: null,
        currentIntervalSeconds: CONFIG.INITIAL_INTERVAL_SECONDS,
        countdown: CONFIG.INITIAL_INTERVAL_SECONDS,
        consecutiveNoChange: 0,
        clickAttempts: 0,
        lastClickLedToObservation: false,
        isRunning: true,
        stopSignal: false,
        effectiveAdCycling: CONFIG.TRY_DIFFERENT_AD_ON_RETRY
    };

    // --- III. UI & NOTIFICATION FUNCTIONS ---
    const showNotification = (message, color = "#00688c", duration = 3500) => {
        let notifEl = document.getElementById("oci-script-notification-v3");
        if (!notifEl) {
            notifEl = document.createElement("div");
            notifEl.id = "oci-script-notification-v3";
            document.body.appendChild(notifEl);
        }
        notifEl.setAttribute("style", `
            z-index: 99999999999999 !important; position: fixed !important; top: 20px !important; left: 50% !important;
            transform: translateX(-50%) !important; padding: 12px 25px !important; font-size: 1.1rem !important;
            font-family: "Oracle Sans", Arial, sans-serif !important; color: white !important;
            background-color: ${color} !important; border-radius: 999px !important; /* Pill shape */
            box-shadow: 0px 3px 15px -2px rgba(0,0,0,0.5) !important; text-align: center !important;
            opacity: 0; display: block !important;
            transition: opacity 0.4s ease-in-out, background-color 0.3s ease !important;`);
        // eslint-disable-next-line no-unused-expressions
        notifEl.offsetHeight; // Trigger reflow
        notifEl.textContent = message;
        notifEl.style.backgroundColor = color;
        notifEl.style.opacity = "0.95";
        if (duration > 0) {
            setTimeout(() => {
                if (notifEl) {
                     notifEl.style.opacity = "0";
                     setTimeout(() => { if(notifEl) notifEl.style.display = "none"; }, 500);
                }
            }, duration);
        }
    };

    const setupStatusDisplay = () => {
        if (!document.getElementById("oci-script-status-bar-v3")) {
            scriptState.statusElmt = document.createElement("div");
            scriptState.statusElmt.id = "oci-script-status-bar-v3";
            document.body.appendChild(scriptState.statusElmt);
        } else {
            scriptState.statusElmt = document.getElementById("oci-script-status-bar-v3");
        }
        scriptState.statusElmt.setAttribute("style", `
            z-index: 9999999999999 !important; position: fixed !important; bottom: 20px !important; left: 50% !important;
            transform: translateX(-50%) !important; padding: 10px 22px !important; font-size: 0.9rem !important;
            font-family: "Oracle Sans", Arial, sans-serif !important; color: white !important;
            background-color: #003f5c !important; border-radius: 999px !important; /* Pill shape */
            box-shadow: 0px -2px 10px -4px rgba(0,0,0,0.4) !important; white-space: nowrap !important;
            overflow: hidden !important; text-overflow: ellipsis !important; display: flex !important;
            align-items: center !important; justify-content: center !important;
            transition: background-color 0.5s ease-in-out, opacity 0.5s ease-in-out !important; opacity: 0.95 !important;`);
    };

    // --- IV. OCI INTERACTION & ELEMENT FINDERS ---
    const getComputeWindow = () => {
        for (const selector of CONFIG.COMPUTE_IFRAME_SELECTORS) {
            const iframe = document.querySelector(selector);
            if (iframe && iframe.contentWindow) return iframe.contentWindow;
        }
        for (const btnSelector of CONFIG.CREATE_BUTTON_SELECTORS) { // Fallback
            if (document.querySelector(btnSelector)) {
                console.warn(logPrefix + "No iframe matched. Assuming current window is compute window.", logStyle("#FFA500"));
                return window;
            }
        }
        return null;
    };

    const findCreateButtonInWindow = (targetWindow) => {
        if (!targetWindow) return null;
        for (const selector of CONFIG.CREATE_BUTTON_SELECTORS) {
            const btn = targetWindow.document.querySelector(selector);
            if (btn && CONFIG.CREATE_BUTTON_EXPECTED_TEXTS.some(text => btn.textContent?.trim().includes(text))) {
                return btn;
            }
        }
        return null;
    };
    
    const initializeOrReinitializeCoreElements = () => {
        scriptState.computeWindow = getComputeWindow();
        if (!scriptState.computeWindow) {
            console.error(logPrefix + "CRITICAL: OCI compute window/iframe NOT FOUND. Script halting.", logStyle("#FF0000", "bold"));
            showNotification("CRITICAL: OCI FRAME NOT FOUND. SCRIPT STOPPED.", "#D32F2F", 0);
            scriptState.isRunning = false;
            return false;
        }

        scriptState.createBtn = findCreateButtonInWindow(scriptState.computeWindow);
        scriptState.adElmts = Array.from(scriptState.computeWindow.document.querySelectorAll(CONFIG.AD_RADIO_BUTTON_SELECTOR));
        scriptState.effectiveAdCycling = CONFIG.TRY_DIFFERENT_AD_ON_RETRY && scriptState.adElmts.length > 1;

        if (CONFIG.TRY_DIFFERENT_AD_ON_RETRY && scriptState.adElmts.length <= 1) {
            console.warn(logPrefix + `AD cycling configured, but only ${scriptState.adElmts.length} AD(s) found. Effective AD cycling: OFF.`, logStyle("#FFA500"));
        }
        
        if (!scriptState.createBtn) {
            console.warn(logPrefix + "Could not find 'Create' button. Page might be loading or UI changed. Retrying scan.", logStyle("#FFA500"));
            return false;
        }
        return true;
    };

    // --- V. SESSION & AD MANAGEMENT ---
    const openOrRefreshSessionWindow = () => {
         if (scriptState.sessionWindow && !scriptState.sessionWindow.closed) {
            try {
                scriptState.sessionWindow.location.href = CONFIG.SESSION_KEEP_ALIVE_URL;
                scriptState.sessionWindow.blur(); window.focus();
            } catch (e) {
                console.warn(logPrefix + `Error refreshing session window: ${e.message}. Re-creating.`, logStyle("#FFA500"));
                scriptState.sessionWindow = window.open(CONFIG.SESSION_KEEP_ALIVE_URL, "_blank", "height=150,width=250,top=0,left=0,popup=true");
            }
        } else {
            scriptState.sessionWindow = window.open(CONFIG.SESSION_KEEP_ALIVE_URL, "_blank", "height=150,width=250,top=0,left=0,popup=true");
        }
        if (!scriptState.sessionWindow) {
            console.error(logPrefix + "SESSION KEEPALIVE POPUP BLOCKED. Allow popups for cloud.oracle.com.", logStyle("#FF0000", "bold"));
            showNotification("CRITICAL: Session Popup BLOCKED! Allow popups & restart.", "#D32F2F", 10000);
        }
    };
    
    const attemptNextAvailabilityDomain = async () => {
        if (scriptState.effectiveAdCycling) {
            scriptState.currentAdIndex = (scriptState.currentAdIndex + 1) % scriptState.adElmts.length;
            const adToClick = scriptState.adElmts[scriptState.currentAdIndex];
            if (adToClick && typeof adToClick.click === 'function') {
                adToClick.click();
                console.info(logPrefix + `Switched to AD: ${scriptState.currentAdIndex + 1}/${scriptState.adElmts.length}`, logStyle("#00BFFF"));
                showNotification(`Trying AD ${scriptState.currentAdIndex + 1}`, "#1E88E5", 2000);
                await new Promise(resolve => setTimeout(resolve, 350)); // Pause for AD switch
            } else {
                 console.warn(logPrefix + `Could not click AD radio button at index ${scriptState.currentAdIndex}.`, logStyle("#FFA500"));
            }
        }
    };

    // --- VI. CORE BOT LOGIC & LOOP ---
    const getCurrentFormattedTime = () => new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit', second: '2-digit' });

    const mainLoop = async () => {
        if (scriptState.stopSignal || !scriptState.isRunning) {
            console.info(logPrefix + "Stop signal or not running. Halting script.", logStyle("#FF6347", "bold"));
            if (scriptState.statusElmt) {
                scriptState.statusElmt.style.backgroundColor = "#C62828";
                scriptState.statusElmt.textContent = `SCRIPT STOPPED | ${getCurrentFormattedTime()}`;
            }
            if(scriptState.sessionWindow && !scriptState.sessionWindow.closed) scriptState.sessionWindow.close();
            return;
        }

        if (!initializeOrReinitializeCoreElements()) {
            if (scriptState.statusElmt) {
                scriptState.statusElmt.style.backgroundColor = "#FF6F00";
                scriptState.statusElmt.textContent = `Elements scan failed. Retrying in 5s | ${getCurrentFormattedTime()}`;
            }
            setTimeout(tick, 5000); // Use tick for consistency
            return;
        }
        
        if (scriptState.createBtn && scriptState.createBtn.disabled) {
             if (scriptState.statusElmt) {
                scriptState.statusElmt.style.backgroundColor = "#F9A825";
                scriptState.statusElmt.textContent = `'Create' Btn Disabled. Waiting... | ${getCurrentFormattedTime()}`;
             }
             scriptState.countdown = Math.max(3, Math.floor(CONFIG.INITIAL_INTERVAL_SECONDS / 2));
             setTimeout(tick, 1000);
             return;
        }

        if (scriptState.countdown > 0) {
            if (scriptState.statusElmt) {
                scriptState.statusElmt.style.backgroundColor = "#005A87";
                const adText = scriptState.effectiveAdCycling ? `${scriptState.currentAdIndex + 1}/${scriptState.adElmts.length}` : 'N/A';
                scriptState.statusElmt.textContent = `Attempt #${scriptState.clickAttempts + 1} in ${scriptState.countdown}s. Int: ${scriptState.currentIntervalSeconds}s. AD: ${adText} | ${getCurrentFormattedTime()}`;
            }
            scriptState.countdown--;
            setTimeout(tick, 1000);
            return;
        }

        scriptState.clickAttempts++;
        const adInfoForLog = scriptState.effectiveAdCycling ? `AD: ${scriptState.currentAdIndex + 1}/${scriptState.adElmts.length}` : 'AD Cycling Off';
        console.info(logPrefix + `Attempting click #${scriptState.clickAttempts}. Interval: ${scriptState.currentIntervalSeconds}s. ${adInfoForLog}`, logStyle("#00BFFF"));

        openOrRefreshSessionWindow();

        if (scriptState.clickAttempts > 1 && !scriptState.lastClickLedToObservation && scriptState.effectiveAdCycling) {
            await attemptNextAvailabilityDomain();
        }
        
        scriptState.createBtn = findCreateButtonInWindow(scriptState.computeWindow); // Re-find just before click
        if (scriptState.createBtn && !scriptState.createBtn.disabled) {
            scriptState.createBtn.click();
            const adStatusText = scriptState.effectiveAdCycling ? `${scriptState.currentAdIndex + 1}/${scriptState.adElmts.length}` : 'N/A';
             if (scriptState.statusElmt) {
                scriptState.statusElmt.style.backgroundColor = "#388E3C";
                scriptState.statusElmt.textContent = `CLICKED #${scriptState.clickAttempts}! AD: ${adStatusText} | Check OCI | ${getCurrentFormattedTime()}`;
             }
            console.log(logPrefix + `Clicked 'Create' (#${scriptState.clickAttempts}) at ${getCurrentFormattedTime()}.`, logStyle("#32CD32", "bold"));
            scriptState.lastClickLedToObservation = false; // Reset before check

            setTimeout(() => {
                const postClickCreateBtn = findCreateButtonInWindow(scriptState.computeWindow);
                const currentURL = scriptState.computeWindow ? scriptState.computeWindow.location.href : "";
                const likelyRedirected = currentURL.includes("/instances/") && !currentURL.includes("create");

                if (!postClickCreateBtn || postClickCreateBtn.disabled || likelyRedirected) {
                    console.info(logPrefix + "POST-CLICK: 'Create' btn change or redirect. LIKELY SUCCESS! Pausing 60s. VERIFY & STOP SCRIPT!", logStyle("#FFD700", "bold"));
                    if (scriptState.statusElmt) {
                        scriptState.statusElmt.style.backgroundColor = "#FBC02D";
                        scriptState.statusElmt.textContent = "SUCCESS? Paused 60s. VERIFY & STOP! | " + getCurrentFormattedTime();
                    }
                    showNotification("POSSIBLE SUCCESS! Verify & STOP SCRIPT if instance created!", "#FBC02D", 60000);
                    scriptState.lastClickLedToObservation = true;
                    setTimeout(tick, 60000); 
                    return;
                } else {
                    console.info(logPrefix + "POST-CLICK: 'Create' btn still present/enabled. Assuming 'Out of Capacity'.", logStyle("#FFA500"));
                     if (scriptState.statusElmt) {
                        scriptState.statusElmt.style.backgroundColor = "#EF6C00";
                        scriptState.statusElmt.textContent = `Click #${scriptState.clickAttempts} done. No change. Retrying. | ${getCurrentFormattedTime()}`;
                     }
                }
                scheduleNextAttempt();
            }, 2000); // Check after 2 seconds
            return; // Do not schedule next tick here; it's in the setTimeout above.

        } else {
            if (scriptState.statusElmt) {
                scriptState.statusElmt.style.backgroundColor = "#E53935";
                scriptState.statusElmt.textContent = `'Create' Btn issue before click #${scriptState.clickAttempts}. Rescanning. | ${getCurrentFormattedTime()}`;
            }
            console.warn(logPrefix + `'Create' button unavailable/disabled before click attempt #${scriptState.clickAttempts}. Rescanning.`, logStyle("#FF4500", "bold"));
        }
        scheduleNextAttempt();
    };
    
    const scheduleNextAttempt = () => {
        if (!scriptState.lastClickLedToObservation) {
            scriptState.consecutiveNoChange++;
        } else {
            scriptState.consecutiveNoChange = 0;
        }

        if (scriptState.consecutiveNoChange >= CONFIG.CONSECUTIVE_NO_CHANGE_ATTEMPTS_FOR_BACKOFF && !scriptState.lastClickLedToObservation) {
            scriptState.currentIntervalSeconds = Math.min(scriptState.currentIntervalSeconds + CONFIG.INTERVAL_INCREMENT_SECONDS, CONFIG.MAX_INTERVAL_SECONDS);
            console.info(logPrefix + `Interval increased to ${scriptState.currentIntervalSeconds}s.`, logStyle("#DAA520"));
            scriptState.consecutiveNoChange = 0;
        } else if (scriptState.lastClickLedToObservation) {
            scriptState.currentIntervalSeconds = CONFIG.INITIAL_INTERVAL_SECONDS;
        }

        scriptState.countdown = scriptState.currentIntervalSeconds;
        setTimeout(tick, 1000);
    };
    
    const tick = () => { // Renamed from mainLoopTick for clarity
        if (scriptState.stopSignal || !scriptState.isRunning) {
            mainLoop(); // Handles final logging and cleanup
            return;
        }
        mainLoop();
    };
    
    // --- VII. GLOBAL CONTROL & INITIALIZATION ---
    window.stopOCIAggroBot = () => { // Renamed for clarity
        scriptState.stopSignal = true;
        showNotification("STOP SIGNAL RECEIVED! Script will halt soon.", "#D32F2F", 5000);
        console.warn(logPrefix + "STOP SIGNAL RECEIVED via window.stopOCIAggroBot(). Script will halt.", logStyle("#DC143C", "bold"));
    };

    console.info(logPrefix + "Initializing...", logStyle("#FFA500", "bold"));
    setupStatusDisplay();
    
    if (!initializeOrReinitializeCoreElements()) {
        console.warn(logPrefix + "Initial element scan incomplete. Main loop will attempt to recover.", logStyle("#FFA500"));
    }
    openOrRefreshSessionWindow(); 

    console.info(logPrefix + "Starting Bot. Monitor console & OCI page. Type 'stopOCIAggroBot()' in console to stop.", logStyle("#20B2AA", "bold"));
    console.info(logPrefix + `Initial interval: ${CONFIG.INITIAL_INTERVAL_SECONDS}s, Max interval: ${CONFIG.MAX_INTERVAL_SECONDS}s.`, logStyle("#20B2AA"));

    if (CONFIG.TRY_DIFFERENT_AD_ON_RETRY) {
        if (scriptState.effectiveAdCycling) {
            console.info(logPrefix + `Cycling through ${scriptState.adElmts.length} Availability Domains.`, logStyle("#20B2AA"));
        } else {
             console.info(logPrefix + `AD cycling configured, but not enough ADs found (${scriptState.adElmts?.length || 0}). Effective AD cycling: OFF.`, logStyle("#FFA500"));
        }
    } else {
        console.info(logPrefix + `AD cycling is DISABLED by config.`, logStyle("#FFA500"));
    }

    showNotification("OCI AggroBot Initialized! Starting...", "#0277BD", 4000);
    tick(); // Start the main loop

})();
