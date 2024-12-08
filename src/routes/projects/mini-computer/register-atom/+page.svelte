<script lang="ts">
    import Aside from "$lib/Aside.svelte";
    import NextPrevNav from "$lib/NextPrevNav.svelte";

    import imageHardware from "./images/hardware.png"
    import imageControl from "./images/control.png"
</script>

<svelte:head>
    <title>Register Atom | Mini Computer</title>
</svelte:head>

<h1>Register Atom</h1>
<p>
    Register Atom is a very simple version that will allow me to write data into registers and perform simple operations
    on them.
</p>

<h2><code>noop</code> Instruction</h2>
<p>
    I would like to define a <code>noop</code> instruction that, does nothing. Specifically, because <code>0x0000</code>
    is the default state of RAM in Digital, I would like to have <code>0x0000</code> be my <code>noop</code> instruction
    so that by default nothing will happen. Technically, everything is a <code>noop</code> to start as nothing will
    happen until I implement some hardware, but defining this instruction will help me make some initial decisions about
    op codes.
</p>

<h2>Generic Instruction Op Code</h2>
<p>
    Specifically, <code>noop</code> will be part of the generic instruction set, and this means that I need to reserve
    the op code <code>0b0000</code> for them.
</p>

<h2>Register Operations Op Code</h2>
<p>
    I also want to have register operations, which I will assigned to op code <code>0b0001</code>. Register operations
    all have a similar shape. I want to be able to specify an operation to apply (separate from the op code) and specify
    two registers to act on, the target and the argument. Since I need 4 bits in order to address each of them, or 8
    overall, and 4 bits are already dedicated to the overall instruction op code I have 4 bits left over for the
    register op code.
</p>
<div class="center">
    <code>OOOORRRRTTTTAAAA</code>
</div>
<p>
    I will figure out what operations are supported over time, but for register Atom, I will start with a simple set of
    the following:
</p>
<table>
    <thead>
        <tr>
            <th>Code</th>
            <th>Operation</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th>
                <code>0000</code>
            </th>
            <td>Add</td>
            <td>Add the argument to the target</td>
        </tr>
        <tr>
            <th>
                <code>0001</code>
            </th>
            <td>Subtract</td>
            <td>Subtract the argument from the target</td>
        </tr>
        <tr>
            <th>
                <code>0010</code>
            </th>
            <td>Move</td>
            <td>Move the argument to the target</td>
        </tr>
    </tbody>
</table>
<p>
    Reflecting on these instructions, I just dedicated one sixteenth of my instructions to register operations. When I
    use bits to describe an address or data, you could think of each different address or piece of data being referenced
    as a distinct instruction. Here, since I am using two register addresses and four bits of data, I am actually
    defining <code>16 * 16 * 16 = 4096</code> instructions at once!
</p>

<h2>Setting Register Values</h2>
<p>
    Performing register operations isn't quite enough. All the registers start with the value <code>0x0000</code>, so
    you cannot usefully see the result of the operations I have defined. I want to be able to set data directly into the
    registers using an instruction.
</p>
<p>
    Since each register is 16 bits, but I only have twelve bit available in an instruction after using four for the op
    code, I will need at least two instructions to set a full 16 bits of data. If I used twelve bits for the data, I
    would have no bits left over to select a register. This could be fine, I could always write to a certain register
    and move the result to a different register if needed.
</p>
<p>
    However, since I need two instructions anyways, I could just write eight bits of data with each instruction. This
    leaves me with 4 bits, which I can use to specify a target register. One of the two instructions will write to the
    low bits of the specified register, and the other to the high bits. For the low bits, I will use op code
    <code>0b0010</code>, and for the high bits I will use <code>0b0011</code>.
</p>
<div class="center">
    <code>001HTTTTDDDDDDDD</code>
</div>
<p>
    Each of these op codes is fully used, so I have defined another two sixteenths of my instruction space with these
    simple instructions.
</p>

<h2>Implementation</h2>
<p>
    You can see more of my exact thoughts about the implementation by reading through the commits between the tags
    <a href="https://github.com/ztstroud/mini-computer/tree/initial">#initial</a> and
    <a href="https://github.com/ztstroud/mini-computer/tree/register-atom">#register-atom</a> in the git repository.
</p>
<img
    src={imageHardware}
    alt="Screenshot of hardware.dig"
    class="center"
/>

<h3>Pieces</h3>
<p>
    There are several core pieces of Atom that I had to create. The first is the <b>clock</b> and <b>phase</b>. The
    clock simply pulses on and off. Each of these signals causes the computer to move forward by one step. The phase
    module breaks these signals into two phases. During the first phase, the instruction to execute is loaded, and
    during the second it is executed.
</p>
<p>
    The next is the <b>PC</b>, which is a register that stores the address in RAM of the next instruction to execute.
    This is not one of the general purpose registers, and so the register operations I defined cannot access it. The PC
    is incremented during the loading phase.
</p>
<p>
    The address from the PC is fed into a <b>RAM</b> module. RAM currently just stores the instruction to execute, and
    the only address I need to use is the one from the PC. RAM does not depend on the clock signal yet, as only writing
    happens on the clock signal.
</p>
<p>
    The output of RAM is saved in the <b>instruction register</b>, which holds the value during execution. This seems a
    little pointless now, and it is because the value coming from RAM will not change during execution. However it is
    preparing for the future where while an instruction is executed, the value coming out may be changed. This would be
    true while loading a value from RAM into a register.
</p>
<p>
    The instruction from the instruction register is fed into the <b>control module</b>. The control module decodes an
    instruction into data and control signals. The data may be sent to other modules, and the control signals enable and
    control them. The control module also has an enable pin, and when it is low no data is written to any of the data
    outputs and no control signals are high. Ultimately, the control unit is the brain of the computer.
</p>
<p>
    Atom has a 16 bit <b>data bus</b> that is used to transfer data between different modules. For register Atom, there
    are three sources that can put data on the bus. These are the ALU, as well as two special values that represent
    writing the high or low bits to the current target register. The only module that reads from the bus is the register
    file, which does so that that it can write one of the aforementioned values to a register.
</p>
<Aside title="Why use a data bus?">
    <p>
        The reason I use a data bus is to reduce the number of direct data connections are needed on the computer.
        Technically, you could put a set of data signals between any two components, and this would allow you to very
        finely control when data is passed from one to another.
    </p>
    <p>
        However, the number of connections will grow with the square of the number of modules, because you need a gate
        between each of them. If you use a data bas, you only need to control when a module puts data on the bus and
        when it reads data from the bus. This grows linearly with the number of modules.
    </p>
</Aside>
<p>
    The <b>register file</b> contains the 16 registers available on Atom. The register file takes two address, one for
    the target and one for the argument. The contents of the target and argument are always fed into the ALU, and will
    store the value on the data bus when the control module says so.
</p>
<p>
    The <b>ALU</b> performs the register operations supported but Atom. It reads the target and argument values and
    writes it to the bus when the control modules says so.
</p>

<h3>The Control Module</h3>
<p>
    The control module itself is the most complicated module in the computer. It is in charge of decoding an instruction
    into data and control signals. Data is multi bit information, like an immediate value or a register address. This
    data will be sent to other modules. It is essentially a shortcut to describe many instructions at once. Control
    signals are used to enable other modules based upon an instruction, or control when or where data is sent. Each
    instruction will have a unique output of control signals.
</p>
<img
    src={imageControl}
    alt="Screenshot of control.dig"
    class="center"
/>
<p>
    I won't always describe the exact inner workings of the control module, but for this first implementation I want to
    describe my thoughts. First, I added an enable pin to the control module so that all signals can be controlled
    easily. When in the instruction loading phase, I don't want garbage data or the last instruction to be controlling
    anything. I could add logic throughout the computer so that the clock signal only pulses when in the execution
    phase. However, I think it is cleaner to pull all of that into the control module.
</p>
<p>
    The control module uses a decoder to decode the four op code bits. Each of the outputs represent one of the 16 op
    codes, and for each I can set relevant control signals and send sets of bits out as data.
</p>

<h2>A Small Program</h2>
<p>
    With these instructions defined, I could create a simple program can perform any register operation on any value
    that I define with these two loading operations. For one, this will let me figure out how to setup a program in RAM.
    It also will prove that the instructions work.
</p>
<p>
    In this program, I want to use every instruction I have defined. That means setting low and high bits as well as
    doing addition, subtraction, and a move. Here is a description of a simple program that does that.
</p>
<pre><code>Load 0x5 into the low bits of register 0
Load 0x3 into the low bits of register 1
Add the value of register 1 to register 0
Load 0x8 into the high bits of register 2
Subtract the value in register 0 from register 2
Move the value in register 2 to register 0
</code></pre>
<p>
    For each of these, I need to convert them into a hex instruction. Like the logic in the control module, I won't
    always go into detail here. However, for this first small program I will go through every step.
</p>
<Aside title="Hand converting instructions">
    <p>
        <code>Load 0x5 into the low bits of register 0</code>: Loading data into the low bits is done using op code
        <code>0x2</code>. Since I want to set the data in register 0, the next 4 bits will be <code>0x0</code>. Finally, I
        need to encode the data, <code>0x05</code>. This gives the final instruction <code>0x2005</code>.
    </p>
    <p>
        <code>Load 0x3 into the low bits of register 1</code>: Like the previous instruction, I will use op code
        <code>0x2</code>, but now with register <code>0x1</code> and the value <code>0x08</code>. This gives the final
        instruction <code>0x2108</code>.
    </p>
    <p>
        <code>Add the value of register 1 to register 0</code>: In order to add a value, I need to use op code
        <code>0x1</code>. For a register operation, I also need to specify the register operation. For addition, it is
        <code>0x0</code>. I also need to encode the target and the argument. The target is where the result will be stored,
        so for this instruction it is <code>0x0</code>, and the argument will be <code>0x1</code>. This give the final
        instruction <code>0x1001</code>.
    </p>
    <p>
        <code>Load 0x8 into the high bits of register 2</code>: Loading data into the high bits is done with op code
        <code>0x3</code>. The register I want to use is <code>0x2</code>. The data itself is <code>0x08</code>. Because I am
        setting the high bits and no other data has been written to register <code>0x2</code>, it will end up with the value
        <code>0x0800</code>. This gives the final instruction <code>0x3208</code>.
    </p>
    <p>
        <code>Subtract the value in register 0 from register 2</code>: Subtraction is also a register operation,
        <code>0x1</code>. Subtraction is register operation <code>0x1</code>. The target is register <code>0x2</code> and
        the argument is <code>0x0</code>. This gives the final instruction <code>0x1120</code>.
    </p>
    <p>
        <code>Move the value in register 2 to register 0</code>: Moving a value is also a register operation,
        <code>0x1</code>. Moving a value is register operation <code>0x2</code>. The target is register <code>0x0</code> and
        the argument is <code>0x2</code>. This gives the final instruction <code>0x1202</code>.
    </p>
</Aside>
<p>
    The final converted program is as follows:
</p>
<pre><code>0x2005 0x2108 0x1001 0x3208 0x1120 0x1202</code></pre>
<Aside title="Editing programs">
    <p>
        In order to create the <code>prog.bin</code> file, I have been using xxd. xxd can create hex dumps, and
        importantly can reverse the process and turn a hex dump back into a binary file.
    </p>
    <p>
        Originally, I looked at using Digital's hex import option. However, it seems to use an Intel hex format that
        requires some extra parameters like the length of the data. I quickly found xxd and like the
        simplicity, so I never got that far into looking into the format.
    </p>
    <p>
        I use xxd with NeoVim, and the editing process seems pretty smooth. I open NeoVim in binary mode, <code>nvim
        -b</code>. If you don't do this, extra bytes will be added to the binary file when saving because NeoVim adds a
        newline at the end of the file. When I first open the binary file, I execute xxd on the file:
        <code>.!xxd</code>. I can edit the hex dump, and when I am done I convert it back with <code>%!xxd -r</code> and
        then save the file.
    </p>
    <p>
        This doesn't violates my rule about not writing external code. I am just using this to ease the process of
        writing and sharing a list of instructions to execute. It doesn't write any instructions for me, I still have to
        manually enter them all in hex.
    </p>
</Aside>
<p>
    To execute the program, you need to write them into Atom's RAM module and turn on the computer. In Digital, you can
    set the <code>prog.bin</code> file as program memory, which will load it into RAM when stating the simulation. After
    running the program, you will see the following values in the register file:
</p>
<table>
    <thead>
        <tr>
            <th>Register</th>
            <th>Value</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th>
                <code>0x0</code>
            </th>
            <th>
                <code>0x07F8</code>
            </th>
        </tr>
        <tr>
            <th>
                <code>0x1</code>
            </th>
            <th>
                <code>0x0003</code>
            </th>
        </tr>
        <tr>
            <th>
                <code>0x2</code>
            </th>
            <th>
                <code>0x07F8</code>
            </th>
        </tr>
    </tbody>
</table>
<p>
    These values prove that Atom worked! I can load data into the registers and perform simple operations on them.
    However, Atom still lacks the ability to make any kind of decisions based on the values of the registers, which I
    will tackle next.
</p>

<NextPrevNav
    prev={{ url: "/projects/mini-computer/initial-hardware-planning", text: "Planning the Initial Hardware" }}
    next={{ url: "/projects/mini-computer/jump-atom", text: "Jumping in Atom" }}
/>

