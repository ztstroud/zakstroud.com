<script lang="ts">
    import NextPrevNav from "$lib/NextPrevNav.svelte";
</script>

<svelte:head>
    <title>The Project | Mini Computer</title>
</svelte:head>

<h1>The Project</h1>
<p>
    In this project, I am going to create various virtual computers from scratch and build programming languages on top
    of them.
</p>

<h2>The Goal</h2>
<p>
    I want to learn about how computers and programming languages work. I think a interesting way to do this will be to
    build it all from scratch. At the end of the project, I will have a variety of functional languages.
</p>

<h2>Limitations</h2>
<p>
    I am imposing some limitations that I think will make this experience more informative, unique, and interesting.
    These are:
</p>
<ul>
    <li>
        I am not allowed to write <em>any</em> code outside of the virtual computers. I want to simulate the experience
        of creating a computer completely on my own. This fundamentally changes the kinds of problem I will face and
        the solutions I will need to create because modern hardware and languages will give me access to tools not
        available in the virtual computer.
    </li>
    <li>
        I am not going to look up information about how anything is implemented in real hardware or software until I
        have attempted to solve it myself. I am limited to what I have learned over my studies and work. I don't just
        want to implement an existing architecture, I want to think through and solve problems on my own. Some of my
        ideas will be wrong, and I fully expect to walk into traps that have already been solved, but that is how we
        learn! Later, I will take the time to compare my mistakes with history, and look at solutions that have been
        developed in reality.
    </li>
</ul>

<h2>The Tools</h2>
<p>
    I am going to start out using a tool called <a href="https://github.com/hneemann/Digital">Digital</a> to build and
    simulate the circuitry. While I think a physical circuit would be neat, I want to be able to preserve a history of
    the circuit in a useable and shareable form.
</p>
<img
    src="https://raw.githubusercontent.com/hneemann/Digital/refs/heads/master/distribution/screenshot.png"
    alt="Screenshot of Digital"
    class="center"
>
<p>
    Outside of Digital, I am allowed to use a text editor to write notes so that they can be shared and kept in version
    control along side the computer.
</p>

<h2>Past Experience</h2>
<p>
    I have some idea of how a computer physically functions. In college, I was exposed to some basic hardware
    architecture, worked with x86 assembly, and wrote code in C, including things like a memory allocator. I have a
    working understanding of the stack, and the basics of how it works in hardware. I also know that there are decades
    of development in this space, and that there are many complexities that I don' understand at all, and many I don't
    even know about.
</p>
<p>
    In college, I created a very basic simulated computer that could execute a small instruction set. I had friends that
    were in a class and I was interested from the moment I heard about it. That project had similarities to this in that
    I didn't have very much information and just followed my intuition, and that made it exciting.
</p>
<p>
    Since then, I have tinkered with little virtual computers, but nothing with the ambitions of this project, and
    without the focus on software. I will take some learnings from those experiments into this project, and try to avoid
    some the problems I have faced in the past.
</p>

<h2>Points of Interest</h2>
<p>
    There are various topics that I know about that I look forward to learning more about. I have included some of them
    below, and a bit of what I know (or think I know) about them.
</p>

<h3>ABIs</h3>
<p>
    Application Binary Interfaces, the lower level sibling of APIs. I have heard of them, and my general understanding
    is that they are an agreement about how hardware like registers and the stack will be used, in particular with
    regards to function calling. I don't even know all of the concepts that are covered by an ABI, and this is certainly
    one of the concepts that I want to do actual research on later, comparing my solution that what exists today and
    through history.
</p>

<h3>Bootstrapping</h3>
<p>
    My understanding of bootstrapping is that you can use a tool, in this case I am thinking about a compiler a
    compiler, to create itself. Each version that you create will be slightly more powerful than the last, thanks to the
    fact that more power allows you to build a more complicated tool.
</p>
<p>
    In the case of this project, I am expecting to start by hand-converting assembly like instructions into my machine
    instructions. Using that, I will build a program that can convert assembly instructions into machine instructions.
    At each level, I will be able to improve the language by building its compiler in itself until I can build an even
    more complicated language, at which point the process repeats.
</p>

<h3>Data Structures and Algorithms</h3>
<p>
    I took a data structures and algorithms course in college. It was fun, and I truly enjoyed learning about and
    creating them. This project is going to allow me to go through that process again, because I will have to create all
    of them that I use from scratch. This is one of those things that could be fundamentally different if I used modern
    tools. While I likely would want to implement them at some point, I would have them from the get go in a modern
    language, which would allow me to solve problems significantly easier than if I am limited to my simulation.
</p>

<h3>Multiple Architectures</h3>
<p>
    Over the course of the project, I would like to experiment with different ways to build the hardware. Right now,
    this mainly revolves around instructions, like how complicated an individual instruction is, how instructions are
    represented, or how immediate data is handled.
</p>
<p>
    Having multiple architectures will also let me experiment with making higher level languages that compile to various
    instruction sets. Maybe I will even be able to look at creating some kind of JVM like tool that allows one program
    to execute on multiple computers without being compiled for each, or creating an intermediate language that allows
    more complex languages to be compiled to various architectures easily.
</p>

<NextPrevNav
    next={{ url: "/projects/mini-computer/atom/initial-hardware-planning/", text: "Planning the Initial Hardware" }}
/>

