<div class="cols">
    <div>
        <div class="title">
            <b>Pros</b>
        </div>
        <slot name="pros" />
    </div>
    <div class="divider"></div>
    <div>
        <div class="title">
            <b>Cons</b>
        </div>
        <slot name="cons" />
    </div>
</div>

<style>
    .cols {
        display: grid;

        grid-template-columns: 1fr 1px 1fr;
        gap: 1em;
    }

    .title {
        text-align: center;
    }

    .divider {
        background-color: currentcolor;
    }
</style>

