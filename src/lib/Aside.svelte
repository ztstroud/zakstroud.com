<script lang="ts">
    export let title: string;

    let open = true;

    function handleEvent(event: Event | KeyboardEvent) {
        if("key" in event) {
            if(event.key == " " || event.key == "Enter") {
                open = !open;
            }
        } else {
            open = !open;
        }
    }
</script>

<aside
    data-open="{ open }"
>
    <div
        class="trigger"
        role="button"
        tabindex="0"
        on:click={handleEvent}
        on:keyup={handleEvent}
    >
        <p class="title">
            <b>{title}</b>
        </p>
    </div>
    <div class="content">
        <slot/>
    </div>
</aside>

<style>
    aside {
        padding: 0 1rem;

        background-color: var(--aside-background-color);
        border: 2px solid var(--aside-border-color);

        overflow: auto;
    }

    .trigger {
        cursor: pointer;
    }

    .title {
        font-size: 1.1em;
    }

    .title::before {
        content: "";
        display: inline-block;
        position: relative;
        box-sizing: border-box;

        margin-left: 0.2em;
        margin-right: 0.5em;

        width: 0.6em;
        height: 0.6em;

        border-top: 2px solid var(--text-primary-color);
        border-right: 2px solid var(--text-primary-color);

        vertical-align: middle;
        bottom: 0.1em;

        transform: rotate(45deg);

        transition: transform 0.1s, bottom 0.1s;
    }

    aside[data-open="true"] .title::before {
        bottom: 0.2em;
        transform: rotate(135deg);
    }

    aside[data-open="false"] .content {
        display: none;
    }

    .content {
        border-top: 1px dashed var(--aside-border-color);
    }
</style>

