<script lang="ts">
    import NextPrevNav from "$lib/NextPrevNav.svelte";

    import callretImage from "./images/callret.png";
    import improvedGeneralImage from "./images/improvedGeneral.png";
    import pushpopImage from "./images/pushpop.png";
    import moreGenericImage from "./images/moreGeneric.png";
</script>

<svelte:head>
    <title>Procedures in Atom | Mini Computer</title>
</svelte:head>

<h1>Procedures in Atom</h1>
<p>
    Up next, I want to add the ability to call procedures or functions. This will give me the ability to reuse pieces
    of code. This will require me to start to divide up memory into various regions, namely somewhere to store the
    program and somewhere to store the stack. This page describes change between
    <a href="https://github.com/ztstroud/mini-computer/tree/atom-refactor">#atom-refactor</a> and
    <a href="https://github.com/ztstroud/mini-computer/tree/atom-procedure">#atom-procedure</a> in the git repository.
</p>

<h2>What is Needed?</h2>
<p>
    The first question I need to answer is, what do I even need? When calling a procedure I need to perform a jump,
    changing the PC to the instructions of the procedure. What makes this different from a jump is that it needs to be
    reversible. When the procedure is over, execution should resume where the call was made. This means that I need to
    be able to save the value of the PC somewhere that can be restored.
</p>
<p>
    I could write the PC to a special register that can be recalled later. This would work for a single procedure call,
    and for certain types of work it would be sufficient. However, it would not allow nested calls. Instead, I am going
    to introduce a stack where values of the PC can be written. When a procedure is called, the PC is pushed onto the
    stack, and when it is finished you restore the PC and pop the value from the stack.
</p>
<p>
    This stack is just a part of memory. I will keep track of a pointer that walks up and down as values are pushed and
    popped from the stack. This memory will need to be offset from program memory, which starts from from
    <code>0x0000</code>. For now, I have made the decision to start the stack from <code>0x8000</code>, which is halfway
    through the computers memory. This is enforced by the hardware.
</p>

<h3>Instructions</h3>
<p>
    As described above, I need a pair of instructions. One to jump to a procedure, and one to return from it. When
    jumping, I need to save the value of the PC to the stack so that the return instruction can jump back.
</p>
<p>
    For the jump instruction, I want to be able to jump both by a relative offset, and to a value in a register. The
    immediate value will take 8 bits, and the register identifier will take 4 bits. However, I only need one of them at
    a time. After taking up four bits for the op code <code>0b1000</code>, I have 4 bits left over. I can use one of
    those to switch between an immediate or register jump. For this purpose I will pick bit 11. This leaves me with two
    variants of the instruction. When jumping with an immediate, it looks like <code>0b10000XXXIIIIIIII</code>. When
    jumping to a register, it looks like <code>0b10001XXXXXXXAAAA</code>.
</p>
<p>
    Both of these have unused bits, and I would like to find something to do with them. For the register jump, I could
    enable relative jumps, and have enough room to specify two registers or an extra immediate value. The immediate
    variation has less wiggle room. I could use those three bits to expand the range of the immediate, or could add more
    flags if I find a use for them.
</p>
<p>
    The return instruction doesn't require any arguments. Because of this, I am going to use a generic instruction for
    it. As a reminder, all generic instructions start with <code>0b0000</code>. Right now, there are only two
    instructions reserved, <code>0b0000000000000000</code> and <code>0b0000FFFFFFFFFFFF</code>. I am going to simply
    make the return instruction <code>0b0000000000000001</code>. This is just the next free instruction, and also makes
    it really easy to see where procedure returns are memory.
</p>

<h3>Implementation</h3>
<p>
    First, I need a way to store a stack pointer (SP). In theory, I could reserve one of the 16 general purpose
    registers as the SP. That would give me a lot of freedom in manipulating it, which could be good for
    experimentation. However, I need to be able to manipulate the SP register when pushing and popping values, which is
    not something that I can do right now. Since that would be involved anyways, I am going to add a special SP
    register. To start, all I need to be able to do is increment and decrement the pointer, which will be used as a
    value is pushed or popped respectively.
</p>
<p>
    For the actual instructions, I need to figure out what actions need to occur. First up, the push instruction, which
    needs to: write the value of PC to RAM at address in the SP, increment the SP, and jump to the call address.
    Primarily, this instruction is very similar to a jump, just with information saved to the stack. The return
    instruction is also very similar to a jump, except that we need to use a value from memory. More specifically, it
    needs to: write the value in RAM at the SP to the PC and decrement the stack pointer.
</p>
<img
    src={callretImage}
    alt="Atom with call and return instructions implemented"
    class="center"
/>
<p>
    While implementing this, I started to layout how I will identify more general instructions. Right now, I have a
    simple solution for HALT that just checks if the lower 12 bits are high and the opcode is zero. It only needs to
    send one signal out, so this is acceptable. However, as I add more instructions which will also be more complex, I
    need a better solution. This is true of the return instruction, which needs to send a variety of signals and values.
</p>
<p>
    I like what I have for op codes, and I want this new system to tie into that. Each instruction is parallel signal
    that can branch of into the signals it needs:
</p>
<img
    src={improvedGeneralImage}
    alt="Control unit with extra handling for generic instructions from 0x0000 to 0x000F"
    class="center"
/>
<p>
    For the new return instruction, I want to identify <code>0b0000000000000001</code>. The circuit on the left checks
    that bits [4-15] are all zero. If so, the decoder on the right is able to represent instructions <code>0x0000</code>
    though <code>0x000F</code>. This only works because zero is noop and is guaranteed to do nothing. If this was not
    the case, I would need to be able to disable the decoder.
</p>

<h3>Testing</h3>
<p>
    I need to test both of these instructions. I want to be sure that the call instruction jumps to a procedure, and
    that the return instruction returns to the call location. I could just set a value in and after the procedure, but I
    thought it would be neat to propagate a value through registers. By that, I mean that I will set <code>r0</code> to
    some value. Within the procedure, I copy from <code>r0</code> to <code>r1</code>. After the procedure call, I move
    <code>r1</code> to <code>r2</code>. If the call instruction fails, only <code>r0</code> will contain the value. If
    the return instruction fails, <code>r0</code> and <code>r1</code> will contain the value, but <code>r2</code> will
    not.
</p>
<pre><code>SETLO r0 0x01

CALL 0x02
MOV r1 r2
HALT

; procedure propagating value
MOV r0 r1
RET

; halt if RET failed
HALT
</code></pre>
<p>
    Another feature of the stack is that a function can call itself, which I also want to test specifically.
</p>
<pre><code>SETLO r0 0x05
SETLO r1 0x01

CALL 0x01;&gt;proc
HALT

;=proc
SUB r0 r1
JEQ 0x01
CALL 0xFD;&gt;proc
RET
</code></pre>
<p>
    This test uses a procedure to recursively decrement <code>r0</code> to zero. Each call decrements it by one. If the
    value was one, the recursive call is skipped.
</p>

<h3>Limitations</h3>
<p>
    Being able to nest procedure calls is good, but there is still a fundamental limitation. Since there are finite
    registers, nested calls will have to share them. That means the procedure making the call (the caller) cannot know
    that the registers will have the same value after the call is complete. The function been called (the callee)
    itself does not have the capacity to restore the registers to their original values.
</p>
<p>
    This severely limits the usability of procedures. You could manage some procedures, but it would require the caller
    to know everything about the callee, <em>including the entire tree of calls it makes</em>. That system is not very
    flexible. Modifying a function like this would be very difficult as they get large.
</p>

<h2>Pushing and Popping Data</h2>
<p>
    The solution is to allow arbitrary data to be placed on the stack. This allows for the caller to save data that it
    can recover after a procedure has completed. It also allows the callee to restore registers as it completes, by
    saving the values to the stack at the start and restoring them at the end.
</p>

<h3>Instructions</h3>
<p>
    Like call and ret, I need two instructions: one to push data to the stack and one to pop data from the stack. Each
    of the instructions needs to identify one register. Combined with an op code, there are 8 bits left over. Because
    the instructions themselves don't need that many bits, I am going to put them under the generic instructions op
    code. I am going to define a new category within generic instructions for single register operations. Those
    instructions will have <code>0b0001</code> in bits [8-11]. Bits [4-7] will identify the target register, and bits
    [0-3] will select an operation.
</p>
<div class="center">
    <code>00000001TTTTOOOO</code>
</div>
<p>
    Push will be operation <code>0b0000</code> and pop will be operation <code>0b0001</code>.
</p>

<h3>Implementation</h3>
<p>
    These instructions will very similar to the call and return instructions. Instead of writing the PC to or from the
    stack, we use a register. This means there is no jumping, just data. The image below shows the main circuit after
    adding push and pop, but it looks effectively identical to when call and ret were added. The entirety of the
    difference between the two sets of instructions is contained within the control unit.
</p>
<img
    src={pushpopImage}
    alt="Atom with push and pop instructions implemented, which looks the same as after call and ret"
    class="center"
/>
<p>
    The interesting change in the control unit is the addition of circuitry that can identify single register
    operations. Single register operations specifically check that the bits [8-15] are <code>0b00000001</code>. I also
    cleaned up the circuitry for checking the lower 12 bits, grouping them into fours which let me reuse some of them
    for the single register operations.
</p>
<img
    src={moreGenericImage}
    alt="Control unit with logic extended for single register operations"
    class="center"
/>

<h3>Testing</h3>
<p>
    To test push, I just push a series of values onto the stack. The expectation is that these values will be in memory
    where the stack stars.
</p>
<pre><code>SETHI r0 0xDE
SETLO r0 0xAD
PUSH r0
SETHI r0 0xBE
SETLO r0 0xEF
PUSH r0
SETHI r0 0x12
SETLO r0 0x34
PUSH r0
SETHI r0 0x56
SETLO r0 0x78
PUSH r0
HALT
</code></pre>
<p>
    Popping relies on pushing, so I left it as a separate test. The test for pop itself involves pushing two values onto
    the stack. Both values were written to <code>r0</code>, and after the values were pushed I zero out <code>r0</code>.
    After that, the values must be recovered from the stack. The expectation is that I will have the correct values in
    registers <code>r0</code> and <code>r1</code>.
</p>
<pre><code>SETHI r0 0xDE
SETLO r0 0xAD
PUSH r0
SETHI r0 0xBE
SETLO r0 0xEF
PUSH r0
SUB r0 r0
POP r1
POP r0
HALT
</code></pre>
<p>
    Next up, I am going to be using all these procedure tools and start to see what I can create on Atom.
</p>

<NextPrevNav
    prev={{ url: "/projects/mini-computer/atom/testing-and-reorganizing/", text: "Testing and Reorganizing Atom" }}
    next={{ url: "/projects/mini-computer/atom/fibonacci/", text: "Fibonacci in Atom" }}
/>

