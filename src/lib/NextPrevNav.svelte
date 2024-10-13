<script lang="ts">
    type LinkData = {
        url: string,
        text?: string,
    }

    export let next: LinkData | undefined = undefined;
    export let prev: LinkData | undefined = undefined;
</script>

<nav>
    {#if prev}
        <div class="prev">
            <a
                href={prev.url}
                rel="prev"
            >
                &lt; {prev.text ?? "Previous"}
            </a>
        </div>
    {/if}

    <!-- Without including this divider, the layout of the navigation looks wrong because it is not centered -->
    <div class="space"></div>

    {#if next}
        <div class="next">
            <a
                href={next.url}
                rel="next"
            >
                {next.text ?? "Next"} &gt;
            </a>
        </div>
    {/if}
</nav>

<style>
    nav {
        display: grid;

        grid-template-columns: 1fr 1px 1fr;
        grid-template-areas: "prev space next";

        gap: 1em;

        margin-top: 2em;
    }

    .prev {
        grid-area: prev;

        text-align: right;
    }

    .space {
        grid-area: space;

        background-color: currentcolor;
        height: 100%;
    }

    .next {
        grid-area: next;
    }
</style>

