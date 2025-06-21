<script>
    import GithubIcon from "$lib/icons/github.svelte";
    import LinkedinIcon from "$lib/icons/linkedin.svelte";

    import meImage from "./images/me.jpeg";
    import miniComputerImage from "./images/mini-computer.png";
    import plainZshImage from "./images/plain.zsh.png";
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

<div class="bio clear-both">
    <img
        src={meImage}
        alt="Me"
        class="right"
    >
    <p>
        I am a software developer at <b>Amazon</b> with over five years of experience building robust maintainable
        software at scale. Our UI framework, library, and build tools allow hundreds of teams to build consistent
        experiences across Amazon's federated architecture while allowing for site wide experimentation. I graduated
        from the University of Utah's computer science program, where I also worked as a teaching assitant. I love
        building developer tooling, creating concise ways for people to complete their work.
    </p>
</div>

<h2>Projects</h2>
<div class="project clear-both">
    <h3>
        <a href="/projects/mini-computer/the-project">Mini Computer</a> |
        <a
            href="https://github.com/ztstroud/mini-computer"
            class="icon"
            target="_blank"
            title="View mini-computer on GitHub"
        ><GithubIcon /></a>
    </h3>
    <img
        src={miniComputerImage}
        alt="mini-computer screenshot"
        class="left"
    >
    <p>
        Programming languages and compilers built on top of virtual hardware, all built from scratch with the aim of
        learning about foundational concepts of computing.
    </p>
</div>
<div class="project">
    <h3>
        plain.zsh |
        <a
            href="https://github.com/ztstroud/plain.zsh"
            class="icon"
            target="_blank"
            title="View plain.zsh on GitHub"
        ><GithubIcon /></a>
    </h3>
    <p>
        A simple zsh theme using simple ASCII characters that I use daily. It gives me just enough information about git
        repositories, running jobs, and command execution time.
    </p>
    <img
        src={plainZshImage}
        alt="Screenshot of plain.zsh in Alacritty"
        class="center"
    />
</div>
<div class="project">
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
        This is where you are right now! My personal website, showing off and documenting me and my projects.
    </p>
</div>

<style>
    .project {
        margin: 1em 0;
    }

    .bio img {
        max-width: 25%;
    }

    @media screen and (max-width: 400px) {
        .bio img {
            display: none;
        }
    }

    .project img {
        max-height: 10em;
    }

    .project img.left {
        max-width: 50%;
    }

    .icon :global(svg) {
        vertical-align: middle;
    }

    .project .icon {
        font-size: 1.3em;
    }
</style>

