<nav>
    <a href="/">Home</a> |
    <a href="/projects">Projects</a>
</nav>
<main>
    <slot />
</main>

<style>
    @import "./theme.css";
    @import "./styles.css";
    @import "./tables.css";

    nav,
    main {
        width: 100%;
        max-width: 48rem;

        margin: 0 auto;
    }
</style>

