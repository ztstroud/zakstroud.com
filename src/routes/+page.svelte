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
<div class="project clear-both left">
    <img src={miniComputerImage} alt="mini-computer screenshot">
    <h3>
        <a href="/projects/mini-computer/the-project">Mini Computer</a> |
        <a
            href="https://github.com/ztstroud/mini-computer"
            class="icon"
            target="_blank"
            title="View mini-computer on GitHub"
        ><GithubIcon /></a>
    </h3>
    <p>
        Programming languages built on top of virtual hardware, all built from scratch.
    </p>
</div>
<div class="project right">
    <h3>
        <a href="/">zakstroud.com</a> |
        <a
            href="https://github.com/ztstroud/zakstroud.com"
            class="icon"
            target="_blank"
            title="View zakstroud.com on GitHub"
        ><GithubIcon /></a>
    </h3>
    <p>
        This is where you are right now! My personal website, showing off my projects.
    </p>
</div>

<style>
    .project {
        margin: 1em 0;
    }

    .project img {
        max-height: 10em;
        max-width: 50%;
    }

    .project.left img {
        float: left;
        margin: 0 1em 0 0;
    }

    .icon :global(svg) {
        vertical-align: middle;
    }

    .project .icon {
        font-size: 1.3em;
    }
</style>

