<script>
    import GithubIcon from "$lib/icons/github.svelte";
    import LinkedinIcon from "$lib/icons/linkedin.svelte";

    import miniComputerImage from "./images/mini-computer.png";
</script>

<svelte:head>
    <title>Home</title>
    <!-- TODO: add meta content -->
</svelte:head>

<header>
    <h1>
        Zak Stroud |
        <a
            href="https://github.com/ztstroud"
            class="icon"
            target="_blank"
            title="View my GitHub"
        ><GithubIcon /></a>
        <a
            href="https://linkedin.com/in/ztstroud"
            class="icon"
            target="_blank"
            title="View my Linkedin"
        ><LinkedinIcon /></a>
    </h1>
</header>

<h2>Projects</h2>

<div class="project">
    <img src={miniComputerImage} alt="mini-computer screenshot">
    <h2><a href="/projects/mini-computer/the-project">Mini Computer</a></h2>
    <p>
        Programming languages built on top of virtual hardware, all built from scratch.
    </p>
</div>

<style>
    .project {
        margin: 1em 0;
    }

    .project img {
        float: left;

        max-height: 10em;
        max-width: 50%;

        margin: 0 1em 0 0;
    }

    .project::after {
        content: "";
        display: block;
        clear: both;
    }

    .icon :global(svg) {
        vertical-align: middle;
    }
</style>

