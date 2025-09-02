// Configuration object for script settings
let config = {
    nextQuestionDelay: 1500,
    alternativeDelay: 500,
    videoDelay: 1000,
    retryDelay: 2000,
    speedrunMode: true,
    debugMode: false,
    autoStart: true
};

// Script state tracking object
let scriptState = {
    isRunning: false,
    questionsResolved: 0,
    videosWatched: 0,
    startTime: null
};

// Chrome runtime message listener
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
    if (message.type === "configUpdate") {
        config = {
            ...config,
            ...message.config
        };
        console.log("🔄 Configuration updated:", config);
    } else if (message.type === "getStatus") {
        sendResponse(scriptState);
    } else if (message.type === "toggleScript") {
        scriptState.isRunning = message.status;
        if (scriptState.isRunning && !scriptState.startTime) {
            scriptState.startTime = Date.now();
        }
        console.log((scriptState.isRunning ? "▶️" : "⏸️") + " Script " + (scriptState.isRunning ? "started" : "paused"));
    }
    return true;
});

function initializeScript() {
    // ASCII art and version log
    console.log(`
    █████╗ ██╗     ██╗   ██╗██████╗  █████╗     ██╗  ██╗██████╗ ██╗      ██████╗ ██╗████████╗
    ██╔══██╗██║     ██║   ██║██╔══██╗██╔══██╗    ╚██╗██╔╝██╔══██╗██║     ██╔═══██╗██║╚══██╔══╝
    ███████║██║     ██║   ██║██████╔╝███████║     ╚███╔╝ ██████╔╝██║     ██║   ██║██║   ██║   
    ██╔══██║██║     ██║   ██║██╔══██╗██╔══██║     ██╔██╗ ██╔═══╝ ██║     ██║   ██║██║   ██║   
    ██║  ██║███████╗╚██████╔╝██║  ██║██║  ██║    ██╔╝ ██╗██║     ███████╗╚██████╔╝██║   ██║   
    ╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═╝    ╚═╝  ╚═╝╚═╝     ╚══════╝ ╚═════╝ ╚═╝   ╚═╝   
    
    🔰 v1.3.3.7 | ${config.speedrunMode ? "ULTRA INSTINCT MODE ACTIVATED 🔥" : "NORMAL MODE"} | DEBUG: ${config.debugMode ? "ON 🐛" : "OFF"}
    `);

    // Debug logging function
    function debugLog(...args) {
        if (config.debugMode) {
            console.log(...args);
        }
    }

    // Adjust delay based on speedrun mode
    function adjustDelay(delay) {
        return config.speedrunMode ? Math.floor(delay / 2) : delay;
    }

    // Navigate to next question/task
    function navigateToNextTask() {
        if (!scriptState.isRunning) return;

        debugLog("⏳ BYPASSING...");
        const nextButtonSelectors = [
            ".bootcamp-next-button", 
            ".task-actions-button-next", 
            "a[href*=\"/next\"]", 
            ".task-body-actions-button", 
            "a:contains(\"Próxima\")"
        ];

        for (const selector of nextButtonSelectors) {
            const nextButton = document.querySelector(selector);
            if (nextButton) {
                debugLog("🎯 Button found: " + selector);
                setTimeout(() => {
                    if (!scriptState.isRunning) return;
                    try {
                        nextButton.click();
                        scriptState.questionsResolved++;
                        debugLog("✅ Moved to next question");
                    } catch (error) {
                        console.error("❌ Click error:", error);
                        const href = nextButton.getAttribute("href");
                        if (href) {
                            window.location.href = href;
                        }
                    }
                }, adjustDelay(config.nextQuestionDelay));
                return true;
            }
        }
        return false;
    }

    // Auto-select correct multiple choice answers
    function handleMultipleChoiceQuestions() {
        if (!scriptState.isRunning) return;

        const checkboxInput = document.querySelector("input[type=\"checkbox\"]");
        if (checkboxInput) {
            const correctAlternatives = Array.from(document.querySelectorAll(".alternativeList-item[data-correct=\"true\"] input"));
            if (correctAlternatives.length > 0) {
                correctAlternatives.forEach((alternative, index) => {
                    setTimeout(() => {
                        if (!scriptState.isRunning) return;
                        if (!alternative.checked) {
                            alternative.click();
                            debugLog("✅ H4CK3D: Alternative " + (index + 1) + "/" + correctAlternatives.length);
                        }
                        if (index === correctAlternatives.length - 1) {
                            setTimeout(navigateToNextTask, adjustDelay(config.nextQuestionDelay));
                        }
                    }, adjustDelay(config.alternativeDelay * index));
                });
                return true;
            }
        } else {
            const singleCorrectAlternative = document.querySelector(".alternativeList-item[data-correct=\"true\"] input");
            if (singleCorrectAlternative) {
                singleCorrectAlternative.click();
                debugLog("✅ H4CK3D: Single question resolved");
                setTimeout(navigateToNextTask, adjustDelay(config.nextQuestionDelay));
                return true;
            }
        }
        return false;
    }

    // Decode base64 encoded strings for sorting blocks
    function decodeBase64String(encodedString) {
        try {
            let decodedString = atob(encodedString);
            try {
                decodedString = atob(decodedString);
            } catch {}
            return new TextDecoder("utf-8").decode(new Uint8Array([...decodedString].map(char => char.charCodeAt(0))));
        } catch (error) {
            console.error("Decoding error:", error);
            return encodedString;
        }
    }

    // Solve sorting tasks
    function handleSortingTask() {
        if (!scriptState.isRunning) return;

        try {
            const sortDestination = document.getElementById("sortBlocksDestination");
            const blocksContainer = document.querySelector(".blocks");
            if (!sortDestination || !blocksContainer) return false;

            const correctOrder = blocksContainer.getAttribute("data-correct-order");
            const decodedOrder = decodeBase64String(correctOrder).split(",");
            debugLog("📋 Decoded order:", decodedOrder);

            sortDestination.innerHTML = "";
            const availableBlocks = Array.from(document.querySelectorAll(".block"));
            debugLog("🧩 Available blocks:", availableBlocks.map(block => block.getAttribute("data-text")));

            // Normalize text for comparison
            const normalizeText = text => text.normalize("NFD").replace(/[\u0300-\u036f]/g, "").toLowerCase().trim();

            let blocksMoved = 0;
            decodedOrder.forEach(orderItem => {
                if (!scriptState.isRunning) return;

                const normalizedOrderItem = normalizeText(orderItem);
                const matchingBlock = availableBlocks.find(block => {
                    const normalizedBlockText = normalizeText(block.getAttribute("data-text"));
                    return normalizedBlockText === normalizedOrderItem || 
                           normalizedBlockText.includes(normalizedOrderItem) || 
                           normalizedOrderItem.includes(normalizedBlockText);
                });

                if (matchingBlock && matchingBlock.parentElement) {
                    sortDestination.appendChild(matchingBlock.parentElement);
                    blocksMoved++;
                    debugLog("✅ Block moved: " + matchingBlock.getAttribute("data-text"));
                }
            });

            // Submit sorted blocks
            if (blocksMoved === decodedOrder.length) {
                const submitButton = document.getElementById("submitBlocks");
                if (submitButton) {
                    submitButton.removeAttribute("disabled");
                    submitButton.click();
                    debugLog("📤 Answer submitted!");

                    setTimeout(() => {
                        if (!scriptState.isRunning) return;

                        if (!document.querySelector(".taskErrorOpinion:not(.hidden)")) {
                            debugLog("✅ Correct answer!");
                            setTimeout(navigateToNextTask, adjustDelay(1500));
                        } else {
                            debugLog("❌ Incorrect answer, retrying...");
                            const retryButton = document.getElementById("tryAgain");
                            if (retryButton) {
                                retryButton.click();
                                setTimeout(handleSortingTask, adjustDelay(1000));
                            }
                        }
                    }, adjustDelay(1000));
                }
            }
        } catch (error) {
            console.error("❌ Error:", error);
        }
    }

    // Mark video as watched
    function handleVideoTask() {
        if (!scriptState.isRunning) return;

        const playButton = document.querySelector(".vjs-big-play-button");
        if (playButton) {
            debugLog("▶️ Starting video...");
            playButton.click();
        }

        const currentUrl = window.location.href;
        const urlParts = currentUrl.split("/");
        const courseId = urlParts[4];
        const taskId = urlParts[6];

        fetch(`https://cursos.alura.com.br/course/${courseId}/task/${taskId}/mark-video`, {
            method: "POST",
            credentials: "include",
            headers: {
                "Content-Type": "application/json",
                Cookie: document.cookie
            }
        }).then(() => {
            debugLog("✅ Video completed");
            scriptState.videosWatched++;
            if (scriptState.isRunning) {
                setTimeout(navigateToNextTask, adjustDelay(config.videoDelay));
            }
        }).catch(error => {
            console.error("❌ Video error:", error);
            if (scriptState.isRunning) {
                setTimeout(navigateToNextTask, adjustDelay(config.videoDelay * 2));
            }
        });
    }

    // Finish unit button handler
    function finishUnit() {
        const finishButton = document.querySelector(".finishSection-content-button");
        if (finishButton) {
            finishButton.click();
            console.log("\"Finish Unit\" button clicked successfully!");
        } else {
            console.log("\"Finish Unit\" button not found.");
        }
    }

    // Handle course evaluation form
    function handleCourseEvaluation() {
        const randomScore = Math.floor(Math.random() * 11);
        const scoreInput = document.querySelector(`input[name="netPromoteScore"][value="${randomScore}"]`);
        
        if (scoreInput) {
            scoreInput.checked = true;
            console.log(`Rating option ${randomScore} selected.`);
            
            setTimeout(() => {
                const continueButton = document.getElementById("buttonFinishAndContinueToDegree");
                if (continueButton) {
                    continueButton.click();
                    console.log("\"Finish course and continue Formation\" button clicked successfully!");
                } else {
                    console.log("\"Finish course and continue Formation\" button not found.");
                }
            }, 1000);
        } else {
            console.log("Rating option not found.");
        }
    }

    // Main task handler
    function handleCurrentTask() {
        if (!scriptState.isRunning) return;

        debugLog("🔍 Analyzing activity...");

        if (document.querySelector(".alternativeList")) {
            handleMultipleChoiceQuestions();
        } else if (document.querySelector(".sortBlocks")) {
            handleSortingTask();
        } else if (document.querySelector(".video-player-wrapper") || document.querySelector(".vjs-big-play-button")) {
            handleVideoTask();
        } else if (document.querySelector(".finishSection-content-button")) {
            finishUnit();
        } else if (document.querySelector(".evaluation-form-group")) {
            handleCourseEvaluation();
        } else {
            debugLog("⚠️ Unrecognized activity type");
            navigateToNextTask();
        }
    }

    // Mutation observer to detect page changes
    const observer = new MutationObserver(mutations => {
        mutations.forEach(mutation => {
            if (mutation.addedNodes.length && scriptState.isRunning) {
                setTimeout(handleCurrentTask, adjustDelay(1500));
            }
        });
    });

    observer.observe(document.body, {
        childList: true,
        subtree: true
    });

    // Retry handling interval
    setInterval(() => {
        if (!scriptState.isRunning) return;

        const retryButton = document.getElementById("tryAgain");
        if (retryButton && !retryButton.classList.contains("hidden")) {
            debugLog("🔄 Retrying...");
            retryButton.click();
            setTimeout(handleCurrentTask, adjustDelay(config.retryDelay));
        }
    }, adjustDelay(config.retryDelay));

    // Check for next button interval
    setInterval(() => {
        if (!scriptState.isRunning) return;

        const nextButton = document.querySelector(".bootcamp-next-button");
        if (nextButton && !document.querySelector(".taskErrorOpinion:not(.hidden)")) {
            navigateToNextTask();
        }
    }, 3000);

    debugLog("🚀 Script started!");

    // Auto-start if configured
    if (config.autoStart) {
        scriptState.isRunning = true;
        scriptState.startTime = Date.now();
        setTimeout(handleCurrentTask, 1000);
    }
}

// Ensure script runs after DOM is fully loaded
if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", initializeScript);
} else {
    initializeScript();
}
