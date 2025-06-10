# youtube-comment-delete-script


```
(async function autoDeleteYouTubeComments() {
    // Configuration
    const DELETE_ARIA_LABELS = ["Delete activity item", "Aktivitätselement löschen"]; // English and German
    const DELETE_DELAY_MIN = 50;
    const DELETE_DELAY_MAX = 250;
    const COMMENTS_BEFORE_PAUSE = 50;
    const PAUSE_TIME = 7500;
    const SCROLL_DELAY = 150;
    const MAX_SCROLL_ATTEMPTS = 7;
    const MAX_NEARBY_ATTEMPTS = 5;

    function sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    async function scrollToEnd() {
        let lastHeight = document.body.scrollHeight;
        let attempts = 0;

        console.log("Starting to scroll down to load all content...");

        while (attempts < MAX_SCROLL_ATTEMPTS) {
            window.scrollTo(0, document.body.scrollHeight);
            await sleep(SCROLL_DELAY);

            let newHeight = document.body.scrollHeight;

            if (newHeight === lastHeight) {
                attempts++;
                console.log(`No new content loaded. Attempt ${attempts}/${MAX_SCROLL_ATTEMPTS}.`);
            } else {
                attempts = 0;
                lastHeight = newHeight;
                console.log("New content loaded, continuing to scroll...");
            }
        }

        console.log("Finished scrolling. All content should now be loaded.");
    }

    async function scrollToTop() {
        console.log("Scrolling back to the top...");
        window.scrollTo(0, 0);
        await sleep(1000);
        console.log("Scrolled to the top.");
    }

    function findDeleteButtons() {
        const buttons = Array.from(document.querySelectorAll('button'));
        const deleteButtons = buttons.filter(button => {
            const ariaLabel = button.getAttribute('aria-label') || '';
            const innerText = button.innerText || '';
            const classList = button.classList || [];

            const matchesAnyLabel = DELETE_ARIA_LABELS.some(label => ariaLabel.startsWith(label));
            const isRevertButton = innerText.includes("Abbrechen") || innerText.includes("Cancel") || classList.contains("x95qze");

            return matchesAnyLabel && !isRevertButton;
        });

        return deleteButtons;
    }

    function getRandomDeleteDelay() {
        return Math.random() * (DELETE_DELAY_MAX - DELETE_DELAY_MIN) + DELETE_DELAY_MIN;
    }

    function pickNearbyDeleteButton(deleteButtons, currentIndex) {
        const nearbyRange = 2;
        const startIndex = Math.max(0, currentIndex - nearbyRange);
        const endIndex = Math.min(deleteButtons.length - 1, currentIndex + nearbyRange);
        const nearbyIndex = Math.floor(Math.random() * (endIndex - startIndex + 1)) + startIndex;
        return deleteButtons[nearbyIndex];
    }

    async function deleteComments() {
        let deletedCount = 0;
        let currentIndex = 0;
        let deleteButtons = findDeleteButtons();

        while (deleteButtons.length > 0) {
            const deleteButton = pickNearbyDeleteButton(deleteButtons, currentIndex);

            deleteButton.scrollIntoView({ behavior: 'smooth', block: 'center' });
            console.log("Scrolling to the delete button...");
            await sleep(1000);

            deleteButton.click();
            console.log("Delete button clicked.");

            deleteButtons = deleteButtons.filter(button => button !== deleteButton);
            deletedCount++;
            console.log(`Deleted ${deletedCount} comment(s). Remaining delete buttons: ${deleteButtons.length}`);

            currentIndex = deleteButtons.indexOf(deleteButton);
            const randomDelay = getRandomDeleteDelay();
            console.log(`Waiting for ${randomDelay.toFixed(0)}ms before checking for the next deletion.`);
            await sleep(randomDelay);

            if (deletedCount % COMMENTS_BEFORE_PAUSE === 0) {
                console.log(`Pausing for ${PAUSE_TIME / 1000} seconds after deleting ${deletedCount} comments...`);
                await sleep(PAUSE_TIME);
            }
        }

        console.log("All deletions completed.");
    }

    await scrollToEnd();
    await scrollToTop();
    await deleteComments();
})();
```
