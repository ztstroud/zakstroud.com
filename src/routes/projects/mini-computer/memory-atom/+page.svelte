<script lang="ts">
    import Aside from "$lib/Aside.svelte";
    import NextPrevNav from "$lib/NextPrevNav.svelte";

    import circuitImage from "./images/circuit.png";
</script>

<svelte:head>
    <title>Memory in Atom | Mini Computer</title>
</svelte:head>

<h1>Memory in Atom</h1>
<p>
    The next gap I am going to fill is access to the memory of Atom. The concept of reading and writing to memory will
    be important for various problems that I will eventually want to solve. This includes things like the following:
</p>
<ul>
    <li>Setting up data before program execution, like a jump table</li>
    <li>Using more than 16 registers of data, at which point some data needs to be pushed off into memory</li>
    <li>Adding procedure calls, in which some registers need to be preserved and I need to save context about where to return</li>
</ul>
<p>
    For now, I am just going to deal with reading and writing data from memory into or out of a register. This
    page describes change between
    <a href="https://github.com/ztstroud/mini-computer/tree/jump-atom">#jump-atom</a> and
    <a href="https://github.com/ztstroud/mini-computer/tree/memory-atom">#memory-atom</a> in the git repository.
</p>

<h2>Where and What to Read or Write</h2>
<p>
    First up, I need to figure out what instructions are involved in loading memory to and from registers. I need to
    represent a 16 bit address, and I need to be able to select what register to read to or write from. I can use one
    register to store the address, which mean in the instruction I only need 4 bits to specify which register to use
    and 4 bits to specify the source/destination. Because I need to use 4 bits for the op code, I have 4 bits left
    over. There are some interesting choices I could make around these last four bits.
</p>
<p>
    I could sum two registers to create and address. This would be useful for things like indexing into an array,
    preventing me from having to perform addition before a memory operation. However, it isn't free from problems. Since
    no register is reserved to be a zero value, indexing base on a single register would be more difficult and would
    require zeroing out a register if this were the only option.
</p>
<p>
    I could use the remaining 4 bits as an immediate value added to the register. This would prevent the problem in the
    first strategy because you can always use 0. But, it doesn't let you dynamically add values based on registers, you
    must make the decision while writing the program. What it does enable is referencing into a small amount of data
    that is organized consistently. It would be useful for grabbing data out of something like a struct, which always
    same shape in memory. Because there are only 4 bits, could only reference 16 individual values, but in many cases
    that may be sufficient.
</p>
<p>
    These four bits could also be used to store some extra information. For instance, instead of using two top level op
    codes for read and write, one of these bits could toggle between read and write. They could also be reserved for
    selecting devices for I/O and saving even more top level op codes. I wouldn't get any of the other features, and I
    do think that this would sacrifice some readability of the hex values, but that eventually won't matter as much once
    I start to create languages.
</p>
<p>
    All of these ideas have merits. At some point, I will almost certainly come back and implement multiple options for
    different circumstances. For now, I am going to implement the small immediate value because it feels like that will
    be a pain, but I would not be surprised if I rapidly decide that I need other forms.
</p>
<Aside title="Further options">
    <p>
        There are even more interesting ideas that could be explored in this vein. For instance, I could save 4 bits in
        the instruction itself by having a special register that is read to or written from. This could be one of the
        standard 16 registers, or it could be an independent register. If using an independent register, I would likely
        have a set of instructions to move data to/from a standard register, but the memory instruction itself would
        have more bits. With this special register, there are various things that I could do.
    </p>
    <ul>
        <li>The address could be a register value plus a 16 bit immediate</li>
        <li>The address could be the sum of two register values plus a 8 bit immediate</li>
        <li>The address could be the sum of three register values</li>
        <li>The address could be the sum of a register and a register shifted by an immediate value</li>
        <li>I could have a 24 bit immediate value</li>
    </ul>
    <p>
        I could also write the address to a special register instead. If you needed to do a lot of operations on top of
        one base address, this could be great. These options convey different benefits, but also different downsides. I
        will revisit some of these as I start actually writing programs, at which point I expect to have a better idea of
        which kind of operations I want.
    </p>
</Aside>

<h2>Instructions</h2>
<p>
    The next available op codes are <code>0b0110</code> and <code>0b0111</code>, which I will use for reading and
    writing respectively. Both instructions will need to specify an immediate, a data register, and an address register.
    I want to keep the hardware simple, so I will use pre-existing instructions to inform how I arrange these three
    fields so that I can re-use circuitry.
</p>
<p>
    Register operations already use 2 registers and an immediate (specify the type of register operation there). The
    target is both read from and written to, so that can be the data register. The argument register is only read from,
    so that can be the address register. The immediate will be the immediate. This give the following forms:
</p>
<div class="center">
    <code>1010IIIIDDDDAAAA</code>
</div>
<div class="center">
    <code>1011IIIIDDDDAAAA</code>
</div>

<h2>Implementation</h2>
<p>
    In order to read data, I need to be able to load data from RAM onto the data bus, and I need to be able to specify
    an address other than the PC. Sending data to the bus is straight forward, controlled by a new control line. For the
    address itself, I need to switch between the PC and the newly calculated address based upon the execution phase.
</p>
<p>
    Because I have already handled getting the address, writing is simply a matter of adding a new control line that
    enables storing and connecting the data input to the data bus.
</p>
<img
    src={circuitImage}
    alt="Screenshot of the full circuit of Atom with memory instructions"
    class="center"
/>
<p>
    This is getting a bit messy, and I would like to take some time to fix it up, but I will cover that in another
    installation. I will need to create some tests to make sure I don't break anything. I have started this process and
    I am keeping some of my programs around. I wrote a simple test for the memory instructions, which reads a value from
    memory, adds it to itself, and writes it back at that location:
</p>
<pre><code>Read data at address register 0 + 4 into register 1
Add the value of register 1 to register 1
Write the data in register 1 to address register 1 + 0
HALT
0x0008 ; this is the data at 0x0004
</code></pre>
<p>
    This is translated to the following:
</p>
<div class="center">
    <code>0x6410 0x1011 0x7011 0x0FFF 0x0008</code>
</div>
<p>
    After execution, the value <code>0x0010</code> should appear in address <code>0x0010</code>.
</p>

<NextPrevNav
    prev={{ url: '/projects/mini-computer/jump-atom', text: 'Jumping in Atom' }}
/>

