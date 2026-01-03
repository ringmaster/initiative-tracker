<script lang="ts">
    import { getContext, onMount } from "svelte";
    import { tracker } from "../stores/tracker";
    import { SMALL_SCREEN_MODE } from "src/utils";
    import Updating from "./Updating.svelte";
    import type InitiativeTracker from "src/main";

    const plugin = getContext<InitiativeTracker>("plugin");
    const { updating, updateTarget } = tracker;

    const smallScreenMode = SMALL_SCREEN_MODE;
    $: isOpen = $updating.size > 0 && $updateTarget != null && smallScreenMode;
    
    let dialogElement: HTMLDialogElement;
    let focusableElements: HTMLElement[] = [];
    let currentFocusIndex = 0;
    
    // Handle ESC key to cancel the update
    function handleKeydown(e: KeyboardEvent) {
        if (e.key === "Escape" && isOpen) {
            e.preventDefault();
            tracker.clearUpdate();
            return;
        }
        
        // Tab trap for focus management
        if (e.key === "Tab" && isOpen) {
            if (!focusableElements.length) {
                setupFocusTrap();
            }
            
            if (focusableElements.length) {
                e.preventDefault();
                currentFocusIndex = e.shiftKey ? 
                    (currentFocusIndex - 1 + focusableElements.length) % focusableElements.length : 
                    (currentFocusIndex + 1) % focusableElements.length;
                focusableElements[currentFocusIndex].focus();
            }
        }
    }
    
    function setupFocusTrap() {
        if (!dialogElement) return;
        
        // Get all focusable elements in the dialog
        focusableElements = Array.from(
            dialogElement.querySelectorAll('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])')
        ) as HTMLElement[];
        
        // Find the index of the currently focused element
        const focusedElement = document.activeElement;
        currentFocusIndex = focusableElements.indexOf(focusedElement as HTMLElement);
        if (currentFocusIndex === -1) currentFocusIndex = 0;
    }
    
    $: if (isOpen && dialogElement) {
        // Focus the dialog when it opens for better accessibility
        setTimeout(() => {
            const inputElement = dialogElement.querySelector('input');
            if (inputElement) inputElement.focus();
            setupFocusTrap();
        }, 10);
    }
</script>

{#if isOpen}
<div class="modal-container">
    <dialog 
        class="updating-modal" 
        open={isOpen} 
        bind:this={dialogElement}
        on:keydown={handleKeydown}
        role="dialog"
        aria-modal="true"
    >
        <div class="updating-modal-header">
            <h3>{$updateTarget === "hp" ? "Hit Points" : "Armor Class"}</h3>
        </div>
        <div class="updating-modal-content">
            <Updating />
        </div>
    </dialog>
    <div class="modal-backdrop" on:click={() => tracker.clearUpdate()} role="presentation"></div>
</div>
{/if}

<style>
    .modal-container {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 1000;
        display: flex;
        justify-content: center;
        align-items: center;
        pointer-events: none;
    }

    .modal-backdrop {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0, 0, 0, 0.3);
        z-index: -1;
        pointer-events: auto;
        cursor: pointer;
    }

    .updating-modal {
        background-color: var(--background-primary);
        border-radius: 8px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        padding: 16px;
        max-width: 95%;
        width: min(500px, 95vw);
        max-height: 90vh;
        overflow-y: auto;
        z-index: 1001;
        pointer-events: auto;
        position: fixed;
        border: 1px solid var(--background-modifier-border);
    }
    
    .updating-modal-header {
        margin-bottom: 12px;
        border-bottom: 1px solid var(--background-modifier-border);
    }
    
    .updating-modal-header h3 {
        margin: 0 0 8px 0;
        font-size: 1.2em;
    }

    .updating-modal-content {
        width: 100%;
    }

    /* Dark mode adjustments */
    :global(.theme-dark) .updating-modal {
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.35);
    }
    
    /* Mobile adjustments */
    @media screen and (max-width: 768px) {
        .updating-modal {
            padding: 12px;
            border-radius: 6px;
            width: 95vw;
        }
        
        .updating-modal-header h3 {
            font-size: 1.1em;
        }
    }
</style>