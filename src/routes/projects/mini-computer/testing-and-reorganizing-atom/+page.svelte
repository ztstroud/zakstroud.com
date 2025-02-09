<script lang="ts">
    import Aside from "$lib/Aside.svelte";
    import NextPrevNav from "$lib/NextPrevNav.svelte";

    import initialImage from "./images/initial.png";
    import busImage from "./images/bus.png";
    import consolidateImage from "./images/consolidate.png";
    import consolidateHighlightedImage from "./images/consolidateHighlighted.png";
    import reorganizedHighlightedImage from "./images/reorganizedHighlighted.png";
    import reorganizedImage from "./images/reorganized.png";
</script>

<svelte:head>
    <title>Testing and Reorganizing Atom | Mini Computer</title>
</svelte:head>

<h1>Testing and Reorganizing Atom</h1>
<p>
    At the end of adding memory instructions, Atom was looking a bit messy. I want to clean up the layout of Atom, and
    consolidate some of the circuitry to clean everything up. However, I am worried that I will break something as I do
    this, and right now I don't have a great way to tell if I do. This page describes change between
    <a href="https://github.com/ztstroud/mini-computer/tree/memory-atom">#memory-atom</a> and
    <a href="https://github.com/ztstroud/mini-computer/tree/atom-refactor">#atom-refactor</a> in the git repository.
</p>

<h2>How to Know if Something is Broken?</h2>
<p>
    The solution to this problem is to write tests! With these, I will be able to validate that Atom still works like it
    did before. Digital has some testing tools, but they are for testing direct inputs and outputs of a circuit, and not
    the contents of RAM. They could be useful to add, especially for things like the ALU, jump decider, or control unit.
    I may add these later, but I want to start somewhere else. I am going to write a small series of programs that can
    be loaded into RAM, and each of which verify that some set of instructions work as expected.
</p>

<h2>Testing</h2>
<p>
    First thing first, I need to know when a test is complete. I already added the HALT instruction, and the computer
    halting is a clear choice. <em>A test is done when the computer halts</em>. The first test that I will write is
    simply a single HALT instruction. If the computer halts, the test has passed.
</p>
<p>
    After writing this test, I realized that I needed to record the expected value of my tests, so I added a new
    <code>TEST.md</code> file that documents the purpose and expected value of each test. In addition, it is in
    execution order. The tests are going to build on each other, like using HALT to end the test, so I want to know that
    certain instructions work so that I can use them in other tests.
</p>

<h3>Testing Set</h3>
<p>
    Set is another very basic instruction that I will need to use in other tests. I want to check a couple things: can I
    set low bits, can I set high bits, can I set both? The test is simple, just execute set instructions on a few
    registers.
</p>
<div class="center">
    <code>20ab 31cd 2257 3223 0fff</code>
</div>
<p>
    This test found a bug introduced when adding the memory read/write instructions. They include a constant in the
    upper four bits of the immediate that were always forced to zero. When trying to load a value with a one in any of
    those bits, the compute crashes.
</p>

<h3>Testing Arithmetic</h3>
<p>
    Testing arithmetic requires some values, which I can now set using either of the set instruction.
</p>
<div class="center">
    <code>2005 2102 1000 1220 1121 0fff</code>
</div>

<h3>Testing Comparisons and Jumps</h3>
<p>
    Testing comparisons and jumps is a bit more tricky. There are many type of jump for different conditions, and I
    would like to avoid having to load up many different programs. Instead, I want to have one program that tells me
    which type of jump failed. I will use register zero to store the exit code. If it is zero when the computer halts,
    all test passed, otherwise its value indicates which test failed.
</p>
<p>
    For each type of jump, I will jump over two instructions. One sets the value of register zero, and the other is a
    halt. If the jump doesn't happen, a value will be set and the computer will immediately halt. A single test from the
    program is shown below:
</p>
<div class="center">
    <code>1283 1081 4402 2004 0fff</code>
</div>
<p>
    This moves a predefined value into a register, performs addition, and executes a jump. If the jump fails,
    <code>0x04</code> is written to register zero and the program halts.
</p>
<Aside title="Assembling with Regex in Neovim">
    <p>
        This program is going to be a little bit long, and it is going to be annoying to write all the instructions and
        convert them by hand. I have decided that I can use Neovim to help me convert individual instructions. I have
        decided that this doesn't violate my "no tools" rules because it doesn't do anything other than shuffle
        characters on a line around. It doesn't compute offsets for jumps or help me reference values in memory. It can
        just turn a string like <code>SETLO r1 0xAB</code> into <code>21AB</code> and then apply some formatting so that
        everything is compatible with <code>xxd</code>.
    </p>
    <p>
        The way this works is by applying a series of commands in <code>atom/tools/regex-assemble</code> using
        <a href="https://neovim.io/doc/user/repeat.html#_using-vim-scripts"><code>:source</code></a>. The commands them
        selves are primarily a series of regular expressions that can extract characters and put them into a four digit
        hex value.
    </p>
    <p>
        To do this, I needed to standardize how I write my fake assembly. I decided that the names of instruction would
        be in upper case, so I instructions are called things like <code>ADD</code> or <code>SETLO</code>. I didn't set
        any limit on the length, which is one of those things that I think might be harder in my first real assembler.
        Registers are all a hex identifier prefixed with <code>r</code>, like <code>r5</code> or <code>rC</code>.
        Literal values are two hex digits prefixed with <code>0x</code>.
    </p>
    <p>
        After converting all instructions from assembly to four digits of hex, I need to format the hex values so that
        <code>xxd</code> can convert it to binary. <code>xxd</code> needs each line to start with an address, so I group
        the hex values into groups of 8. I then prefix each line with <code>0x00000000: </code>, select all lines except
        the first and then increment them all using <code>16g&lt;C-a&gt;</code>. This actually increments each by 16, as
        each line has 16 bytes. I then delete the <code>0x</code> from the start of each line.
    </p>
    <p>
        This tool lets me convert larger programs like the following, which is the full test for comparisons and jumps.
    </p>
    <pre><code>; setup values
SETLO r1 0x03

SETLO r2 0x08

SETHI r3 0xFF ; max unsigned / -1 signed
SETLO r3 0xFF

SETHI r4 0x80 ; min signed

SETHI r5 0x7F ; max signed
SETLO r5 0xFF

; tests
JMP 0x02
SETLO r0 0x01 ; always jump failed
HALT

CMP r1 r1
JEQ 0x02
SETLO r0 0x02 ; jump equal failed
HALT

CMP r1 r2
JNE 0x02
SETLO r0 0x03 ; jump not equal failed
HALT

MOV r3 r8
ADD r8 r1
JCR 0x02
SETLO r0 0x04 ; addition carry failed
HALT

SUB r8 r8 ; set r8 to zero
SUB r8 r1
JCR 0x02
SETLO r0 0x05 ; subtraction carry failed
HALT

MOV r5 r8
SUB r5 r3
JOV 0x02
SETLO r0 0x06 ; signed positive subtract negative overflow failed
HALT

MOV r4 r8
SUB r8 r1
JOV 0x02
SETLO r0 0x07 ; signed negative subtract positive overflow failed
HALT

MOV r1 r8
SUB r8 r2
JNG 0x02
SETLO r0 0x08 ; negative failed
HALT

CMP r1 r2
JL 0x02
SETLO r0 0x09 ; unsigned less than failed
HALT

JLE 0x02
SETLO r0 0x0A ; unsigned less than or equal failed (less than)
HALT

CMP r1 r1
JLE 0x02
SETLO r0 0x0B ; unsigned less than or equal failed (equal)
HALT

CMP r2 r1
JG 0x02
SETLO r0 0x0C ; unsigned greater than failed
HALT

JGE 0x02
SETLO r0 0x0D ; unsigned greater than or equal failed (greater than)
HALT

CMP r1 r1
JGE 0x02
SETLO r0 0x0E ; unsigned greater than or equal failed (equal)
HALT

CMP r3 r1
JLU 0x02
SETLO r0 0x0F ; signed less than failed
HALT

JLEU 0x02
SETLO r0 0x10 ; signed less than or equal failed (less than)
HALT

CMP r3 r3
JLEU 0x02
SETLO r0 0x11 ; signed less than or equal failed (equal)
HALT

CMP r1 r3
JGU 0x02
SETLO r0 0x12 ; signed less than failed
HALT

JGEU 0x02
SETLO r0 0x13 ; signed less than or equal failed (greater than)
HALT

CMP r3 r3
JGEU 0x02
SETLO r0 0x14 ; signed less than or equal failed (equal)
HALT

HALT ; finished with no errors</code></pre>
    <p>
        I think it strikes a good balance. It doesn't provide so much functionality that writing my own assembler will
        be pointless, but it makes life easier. A full assembler will help me with things like labels and preparing data
        in memory.
    </p>
</Aside>

<h3>Testing Register Jump</h3>
<p>
    I also need to test register jumps. I have decided not to test every jump type because the circuitry is the same
    between all jump types. I just want to focus on relative and absolute jumps. To that end, the program is very
    simple:
</p>
<pre><code>SETLO r1 0x02
RRJMP r1
SETLO r0 0x01
HALT

SETLO r1 0x07
RJMP r1
SETLO r0 0x02
HALT</code></pre>

<h3>Testing Memory Read and Write</h3>
<p>
    Testing memory introduces an interesting new concept, and that is that <em>data can be encoded directly into memory
    when the program is loaded</em>. For this test, I include <code>0x0008</code> in memory at <code>0x0004</code>. I
    read that, add it to itself, and write it back into memory <em>at the result</em> (which isn't necessary, but is
    neat). Because the value is <code>0x0008</code>, it is written back to <code>0x0010</code>.
</p>
<pre><code>READ r0 0x4 r1
ADD r1 r1
WRITE r1 0x r1
HALT
0x0008</code></pre>

<h2>Refactoring</h2>
<p>
    This small suite of tests covers all instructions. There are definitely still some holes in it, but I think this is
    sufficient for now. After solving the bug revealed by the set test, all test pass and the computer is working. As a
    reminder, it looks like this.
</p>
<img
    src={initialImage}
    alt="Atoms initial state"
    class="center"
/>
<p>
    First, I want to centralize around a core bus, including more than just the data line. This way I can route various
    lines to many components without there being random wires all over the place.
</p>
<img
    src={busImage}
    alt="Atom centralized around a bus"
    class="center"
/>
<p>
    I also moved the control signal to tunnels, which removes a lot of wiring. I would like to being them back over
    time, and let them inform how components are laid out. I am also going to merge some components together and create
    new sub circuits for things like the address calculator.
</p>
<img
    src={consolidateImage}
    alt="Atom with circuits consolidated"
    class="center"
/>
<p>
    There are some components that produce an output that is only fed into one other circuit. To make that a bit easier
    to see, I will highlight different lines in different colors.
</p>
<img
    src={consolidateHighlightedImage}
    alt="Atom with circuits consolidated and lines highlighted"
    class="center"
/>
<p>
    In particular, I notice that the jumpCalc and the jumpDecider only feed into the PC itself. I will reorganize them
    so that they are closer to the PC.
</p>
<img
    src={reorganizedHighlightedImage}
    alt="Atom with circuits reorganized and lines highlighted"
    class="center"
/>
<p>
    This does mean that the Flags are now on the bus, but I may want to use them for other things later so I think that
    is fine. You can see that the new PC (dark green) and the final decision to jump (cyan) are much shorter because
    they are near to the PC. Without highlight, you can see the current state of Atom below.
</p>
<img
    src={reorganizedImage}
    alt="Atom with circuits reorganized"
    class="center"
/>
<p>
    I think that this is better, but how much or even if it is better depends on the viewer. I have re-run all the test
    that I wrote at the begging on this circuit, and everything still works, so I feel safe with these updates.
</p>

<NextPrevNav
    prev={{ url: "/projects/mini-computer/memory-atom", text: "Memory in Atom" }}
    next={{ url: "/projects/mini-computer/procedures-in-atom", text: "Procedures in Atom" }}
/>

