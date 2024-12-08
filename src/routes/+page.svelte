<script>
    import miniComputerImage from "./images/mini-computer.png";
</script>

<svelte:head>
    <title>Home</title>
    <!-- TODO: add meta content -->
</svelte:head>

<header>
    <h1>Zak Stroud</h1>
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
</style>

