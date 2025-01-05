<script lang="ts">
    import NextPrevNav from "$lib/NextPrevNav.svelte";
</script>

<svelte:head>
    <title>Planning the Initial Hardware | Mini Computer</title>
</svelte:head>

<h1>Planning the Initial Hardware</h1>
<p>
    I want the first hardware I create to be simple because I am more interested in the software side of the project. I
    want to get through my initial design to figure out where the rough spots are and what ideas work. The first step I
    am going to take is to think through some of the broad decisions I need to make.
</p>

<h2>Where do Programs Live?</h2>
<p>
    In previous experiments, I have stored programs in a ROM module. They could access separate RAM, and for simple
    programs this is fine. However, in this project I want to be able to compile programs, and I would like to be able
    to run them directly. If I use a ROM module, I have to manually write a compiled program into ROM to execute it.
</p>
<p>
    Because of this, I want to store all programs in RAM. This will introduce some (I think fairly minor) extra
    complexity, but it will be worth it in the long run. In the most basic implementation, I imagine that I will
    separate the loading of an instruction from the execution of the instruction.
</p>

<h2>How Many Bits?</h2>
<p>
    I need to figure out how many bits I will use for this computer. This will determine the size of instructions and
    values within the computer. In the past I have used 8 bits, but I ran into some problems. Values are limited, and I
    have to add additional complexity in order to handle larger numbers.
</p>
<p>
    In particular, with a pure 8 bit address space I only have 256 addressable locations in memory. This is simply not
    enough for the programs that I want to write and would have to be solved. I will want to be able to do math with
    addresses as well, which is part of the complexity I mentioned before.
</p>
<p>
    With all this in mind, I am choosing to start with a 16 bit computer. This will help me avoid some complexity and
    get to interesting software faster. I will want to solve some of the same types of problems later, but dealing with
    them should be pushed down the line until after I have started to create some interesting programs.
</p>
<p>
    This is also a freedom that I am given by using a simulator rather than a physical computer. In the physical world,
    as I increase the bit count I need more wires; the complexity is just pushed around. Using a simulator, doubling the
    bits takes almost no extra work.
</p>

<h2>How Many Registers?</h2>
<p>
    I also need to determine how many general purpose registers will be available. Intuitively, it feels like more
    registers is better so that I don't run out later and have to save values to the stack. However, the more registers
    you have, the more bits you need to be able to address them, which takes up space in the instruction.
</p>
<p>
    I don't have enough information to know where the balance is. If I have 8 registers, I only need three bits to
    specify a register. This would mean that I could specify three registers in 9 bits, or just about half the
    instruction. This would let me specify two sources and a separate destination and still have a lot of bits left
    over. I could also use 16 registers, but this likely will restrict me to using 2 registers in an instruction.
</p>
<p>
    For now, I am going to use 16 registers.
</p>

<h2>Instruction Set</h2>
<p>
    Since I have decided to use 16 bits, I need to decide how I am going to split them up for use in an instruction. At
    the most basic level, I am thinking that I will use the first 4 bits as an op code. This op code will let us pick
    between different types of instructions.
</p>
<div class="center">
    <code>OOOOXXXXXXXXXXXX</code>
</div>
<p>
    This gives us 16 op codes. Don't forget, this doesn't mean I have 16 operations I can perform, just 16 general
    categories, each of which can have 4096 sub-codes. I still have access to all 65,536 possible instructions. By
    breaking up the instructions into different operations, I can group instructions by their shape, like using the last
    8 bits as two register addresses or as an immediate value. Doing this makes it easier to keep track and implement
    instructions.
</p>
<p>
    I could spend the time now to try to figure out the specifics of all instructions, but I don't know enough about
    the details yet. Instead, I am going to list some of the general types of instructions I think I will need here. I
    am going to figure out the specifics of the instruction set as I build the hardware, and I will likely need to break
    them up further in order to implement them.
</p>

<h3>Register Operations</h3>
<p>
    Registers hold data that I can work on. Register operations will include things like mathematical and logical
    operations, comparisons, and writing literal values.
</p>

<h3>Jump Operations</h3>
<p>
    I am going to need the ability to control the flow of the program in order to make anything interesting.
</p>

<h3>Memory Operations</h3>
<p>
    16 registers is clearly not enough to store the data for all programs that I are going to want to write. I will want
    to store values on the stack and on the heap. To do this, I will need to be able to save and read values between
    memory and registers.
</p>

<h3>I/O Operations</h3>
<p>
    For this project, I am going to want at least two things: the ability to input text into the simulated computer, and
    the ability to save data between executions. I am thinking that I will model these as I/O operations that interact
    with generic peripheral devices. Doing this, I keep the door open to other potential devices.
</p>

<h3>Generic Operations</h3>
<p>
    I know there will also be other operations that I want to be able to execute that don't fit neatly into one of these
    categories. I am thinking about having an address register, and I will need a way to move data to that. I also want
    some kind of halt instruction that at least pauses execution of the program.
</p>

<h2>A Name</h2>
<p>
    I don't want to keep referring to this as the hardware, so I want to give this first piece of hardware a name.
    Because it is my first building block and I want to keep it simple, I am going to call this first hardware
    implementation Atom.
</p>

<NextPrevNav
    prev={{ url: "/projects/mini-computer/the-project", text: "The Project" }}
    next={{ url: "/projects/mini-computer/register-atom", text: "Register Atom" }}
/>

