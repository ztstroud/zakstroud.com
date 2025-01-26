<script lang="ts">
    import Aside from "$lib/Aside.svelte";
    import NextPrevNav from "$lib/NextPrevNav.svelte";

    import imageFlags from "./images/flags.png";
    import imageFlagRegister from "./images/flagRegister.png";
    import imageJumpDecider from "./images/jumpDecider.png";
    import imageRelativeJump from "./images/relativeJump.png";
    import imageJumpCalc from "./images/jumpCalc.png";
</script>

<svelte:head>
    <title>Jumping in Atom | Mini Computer</title>
</svelte:head>

<h1>Jumping in Atom</h1>
<p>
    Right now, Atom can execute a simple procedure like adding and subtracting a few numbers. However, it doesn't have
    the ability to make decisions of any kind. The solution to this problem is to add two things: a register comparison
    operation, and conditional jump operations. After comparing values, the jump operations can skip over or repeat
    code. These are the basis of things like <code>if</code> and <code>while</code> or <code>for</code> loops.
</p>
<p>
    Once I have implemented these instructions, I will be able to write a program that calculates the first Fibonacci
    number that is under a given value. This page covers changes between
    <a href="https://github.com/ztstroud/mini-computer/tree/register-atom">#register-atom</a> and
    <a href="https://github.com/ztstroud/mini-computer/tree/jump-atom">#jump-atom</a> in the git repository.
</p>

<h2>Types of Comparison</h2>
<p>
    There are six comparisons that I want to be able to execute: equal, not equal, less Than, less than or equal,
    greater than, and greater than or equal. Some of these comparisons will require that I make a distinction between
    signed and unsigned numbers, which has not been needed up to this point.
</p>
<p>
    Two of these operations, equal and not equal, don't depend on if the data is signed or unsigned. This is because,
    regardless of that fact, if you subtract to equal numbers you get zero, and if they are not equal you get something
    other than zero. This means that simply checking if the result of the operation is zero is sufficient to check for
    both equality and inequality.
</p>
<p>
    The remaining four operations differ depending on if you are operating on signed or unsigned data.
</p>

<h3>Ordering Comparisons</h3>
<p>
    I will refer to the remaining four comparisons as ordering comparisons as they establish and ordering of values. In
    order to evaluate these comparisons, I only need to define one more thing: how to determine if one value is less
    than another. Once that is defined, the other three can be defined using it and equality which I can already do. The
    equations for the remaining three are listed below, where <code>E</code> is equality and <code>L</code> is less
    than.
</p>
<table>
    <thead>
        <tr>
            <th>Comparison</th>
            <th>Equation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>less than or equal</td>
            <td><code>L | E</code></td>
        </tr>
        <tr>
            <td>greater than</td>
            <td><code>~L &amp; ~E</code></td>
        </tr>
        <tr>
            <td>greater than or equal</td>
            <td><code>~L</code></td>
        </tr>
    </tbody>
</table>
<p>
    This means that I only need to define the less than comparison for both signed and unsigned values.
</p>

<h4>Unsigned Less Than</h4>
<p>
    For unsigned numbers, determining if the target is less than the argument is simple. When subtraction occurs, if
    overflow happens then the argument must have been larger than the target, and so the target is less than the
    argument. That is it, just use the value of the carry from the subtraction.
</p>
<p>
    Note that it is not sufficient to check if the resulting signed value is negative. For instance, if you had a large
    number like <code>0xFFFF</code> and subtract zero, you will end up with a result that looks like a negative signed
    value, even though clearly the maximum unsigned value must be larger than zero.
</p>

<h4>Signed Less Than</h4>
<p>
    Signed numbers are more tricky than unsigned numbers. It is no longer sufficient to just check of overflow. For
    instance, if you take <code>0x0001</code> and subtract <code>0xFFFF</code> you get <code>0x0002</code>
    (<code>1 - -1 = 2</code>). This operation results in a carry, and this makes sense <em>if this were an unsigned
    operation</em>, as <code>0xFFFF</code> is much larger. But looking at this as a signed operation, it is clear that
    no overflow occurred, and that <code>0x0001</code> is not less than <code>0xFFFF</code>.
</p>
<p>
    What I want to define is a signed overflow, which will have the same simple property as carry does for unsigned
    numbers. It turns out that this has a fairly simple equation:
    <code>(Tn &amp; ~An &amp; ~Rn) | (~Tn &amp; An &amp; Rn)</code>, where <code>Tn</code>, <code>An</code>, and
    <code>Rn</code> are whether or not the target, argument, and result are negative respectively.
</p>
<Aside title="Deriving Signed Overflow">
    <p>
        I derived the above equation by looking at data that I wrote by hand for subtraction between two two bit
        integers and looking for patterns. First, the subtraction table itself, which is basically a 4 bit truth table:
    </p>
    <table>
        <thead>
            <tr>
                <td>T</td>
                <td>A</td>
                <td>R</td>
                <td>C</td>
                <td>O</td>
            </tr>
        </thead>
        <tbody>
            <!-- arg 0b00 -->
            <tr>
                <td><code>0b00</code></td>
                <td><code>0b00</code></td>
                <td><code>0b00</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b01</code></td>
                <td><code>0b00</code></td>
                <td><code>0b01</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b10</code></td>
                <td><code>0b00</code></td>
                <td><code>0b10</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b11</code></td>
                <td><code>0b00</code></td>
                <td><code>0b11</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <!-- arg 0b01 -->
            <tr>
                <td><code>0b00</code></td>
                <td><code>0b01</code></td>
                <td><code>0b11</code></td>
                <td><code>0b1</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b01</code></td>
                <td><code>0b01</code></td>
                <td><code>0b00</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b10</code></td>
                <td><code>0b01</code></td>
                <td><code>0b01</code></td>
                <td><code>0b0</code></td>
                <td><code>0b1</code></td>
            </tr>
            <tr>
                <td><code>0b11</code></td>
                <td><code>0b01</code></td>
                <td><code>0b10</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <!-- arg 0b10 -->
            <tr>
                <td><code>0b00</code></td>
                <td><code>0b10</code></td>
                <td><code>0b10</code></td>
                <td><code>0b1</code></td>
                <td><code>0b1</code></td>
            </tr>
            <tr>
                <td><code>0b01</code></td>
                <td><code>0b10</code></td>
                <td><code>0b11</code></td>
                <td><code>0b1</code></td>
                <td><code>0b1</code></td>
            </tr>
            <tr>
                <td><code>0b10</code></td>
                <td><code>0b10</code></td>
                <td><code>0b00</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b11</code></td>
                <td><code>0b10</code></td>
                <td><code>0b01</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
            <!-- arg 0b11 -->
            <tr>
                <td><code>0b00</code></td>
                <td><code>0b11</code></td>
                <td><code>0b01</code></td>
                <td><code>0b1</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b01</code></td>
                <td><code>0b11</code></td>
                <td><code>0b10</code></td>
                <td><code>0b1</code></td>
                <td><code>0b1</code></td>
            </tr>
            <tr>
                <td><code>0b10</code></td>
                <td><code>0b11</code></td>
                <td><code>0b11</code></td>
                <td><code>0b1</code></td>
                <td><code>0b0</code></td>
            </tr>
            <tr>
                <td><code>0b11</code></td>
                <td><code>0b11</code></td>
                <td><code>0b00</code></td>
                <td><code>0b0</code></td>
                <td><code>0b0</code></td>
            </tr>
        </tbody>
    </table>
    <p>
        This table includes the carry, which is calculated from the subtraction being performed. It also includes signed
        (O)verflow. This is the value that I want to compute, and here it is written by hand. It is true when the signed
        value is incorrect. By this, I mean if you take 1 and subtract -1, you should get 2. However, because I only
        have two bits you end up with -2.
    </p>
    <p>
        Intuitively, it seems like the sign of the target, argument, and result could be related to whether or not
        signed overflow occurred. If you have a negative number and subtract a positive number, you <em>should</em> end
        up with a negative number. If you end up with a positive number, overflow must have occurred even though no
        carry happened. I will simplify the table so that I only have the sign bit of the values. I will also remove
        the <code>0b</code> prefix to make the table easier to read.
    </p>
    <table>
        <thead>
            <tr>
                <td>Tn</td>
                <td>An</td>
                <td>Rn</td>
                <td>C</td>
                <td>O</td>
            </tr>
        </thead>
        <tbody>
            <!-- arg 0b00 -->
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <!-- arg 0b01 -->
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <!-- arg 0b10 -->
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <!-- arg 0b11 -->
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
        </tbody>
    </table>
    <p>
        Some of the rows of this table are duplicated, so I will remove those. Importantly, there are no rows where
        <code>Tn</code>, <code>An</code>, <code>Rn</code>, and <code>C</code> have the same value but <code>O</code> has
        a different value. If that were the case, then you would definitely need more information to determine if signed
        overflow had occurred.
    </p>
    <table>
        <thead>
            <tr>
                <td>Tn</td>
                <td>An</td>
                <td>Rn</td>
                <td>C</td>
                <td>O</td>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
        </tbody>
    </table>
    <p>
        At this point, I notice that for each value of <code>Tn</code>, <code>An</code>, <code>Rn</code>, and
        <code>C</code>, <code>O</code> is only true one time. This means that I can use any of them to bisect the table
        and simplify the problem. I don't see any reason to pick one over the other, so lets pick <code>Tn</code> and do
        the bisection. When <code>Tn = 0</code>:
    </p>
    <table>
        <thead>
            <tr>
                <td>An</td>
                <td>Rn</td>
                <td>C</td>
                <td>O</td>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
        </tbody>
    </table>
    <p>
        When <code>Tn = 1</code>:
    </p>
    <table>
        <thead>
            <tr>
                <td>An</td>
                <td>Rn</td>
                <td>C</td>
                <td>O</td>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>0</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>1</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
                <td><code>0</code></td>
            </tr>
            <tr>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>1</code></td>
                <td><code>0</code></td>
            </tr>
        </tbody>
    </table>
    <p>
        These tables are getting small enough that you can look at them and find a rule that produces
        <code>O</code>. For the <code>Tn = 0</code> table, I can see that <code>O = 1</code> only when both
        <code>An</code> and <code>Rn</code> are both 1, so you can have <code>An &amp; Rn = O</code>. Technically, I
        could include <code>C</code>, but it is redundant. It also cannot be used with only one of <code>An</code>
        or <code>Rn</code>, I need both of them.
    </p>
    <p>
        For the table when <code>Tn = 1</code>, I can similarly see that <code>O = 1</code> only when both
        <code>An</code> and <code>Rn</code> are both 0. That means that <code>~An &amp; ~Rn = O</code>. Again,
        <code>C</code> is redundant and insufficient without both <code>An</code> and <code>Rn</code>.
    </p>
    <p>
        Now I have two equations that I can use to calculate <code>O</code>, but I will need to combine them with
        <code>Tn</code>. What I want to say is that if <code>Tn = 0</code> then <code>O = An &amp; Rn</code> and if
        <code>Tn = 1</code> then <code>O = ~An &amp; ~Rn</code>. What you can do is gate each equation with
        <code>Tn</code> and then or them together so that you get the result of either. This gives us the original
        equation:
    </p>
    <div class="center">
        <code>(~Tn &amp; An &amp; Rn) | (Tn &amp; ~An &amp; ~Rn)</code>
    </div>
    <p>
        If <code>Tn = 0</code>, then the right hand side will always evaluate to 0. <code>~Tn</code> will always be 1,
        so you get the value of <code>An &amp; Rn</code>. Similarly, if <code>Tn = 1</code> then the left hand side will
        be 0. <code>Tn</code> will of course be 1, so you are left with the value of <code>~An &amp; ~Rn</code>.
    </p>
</Aside>

<h2>Flags</h2>
<p>
    The comparisons that I have discussed so far will be the core of the flags. As a summary, this includes a <b>zero
    flag</b>, which will let me test for equality, a <b>carry flag</b> which is defined well for both addition and
    subtraction, and a <b>signed overflow flag</b>, which so far is only well defined for subtraction. I also will
    include a <b>negative flag</b>, which just if the signed interpretation of the number is negative. In other words,
    is the most significant bit a one. All of this is encoded into the ALU, as shown below.
</p>
<img
    src={imageFlags}
    alt="Screen shot of flags logic"
    class="center"
/>
<p>
    The ALU has one four bit flag output, with each bit representing one of these flags. The next thing I need to to is
    save the flag output between instructions, which I will do with a register. This value needs to be saved because the
    comparison is part of a different instruction than the jump itself.
</p>
<img
    src={imageFlagRegister}
    alt="Screen shot of flag register"
    class="center"
/>
<p>
    I will save the value of the flags in this register for any instruction that writes the ALU output to the data
    buffer. This might change later, but for now this essentially correlates to executing the comparison. This also
    means that other register operations will update the flags, not just the comparison operation.
</p>

<h2>The Jump Instruction</h2>
<p>
    The jump instruction itself will move program execution to a new address. It will need two pieces of information:
    the type of jump, and where to jump to. The jump type can be on of the conditions described before, as well as jumps
    for other flag values. I also want a special value that will always jump.
</p>
<p>
    I need to allocate the bits in the jump instruction. First, I will take the next 4 bit op code <code>0b0100</code>.
    Next, I know of 13 types of jump that I want to be able to perform, so I will reserve the next 4 bits of the
    instruction for the jump type. This leaves me with three reserved typed, which may find use later. This leaves me
    with 8 bits that I can use to define a location.
</p>
<p>
    8 bits is not enough to describe a full 16 bit address. Instead, I will treat it as a signed 8 bit number and
    encodes an offset from the next instruction. The fact that it is relative to the next instruction is because the PC
    saves the address of the next instruction, rather than the current one. If a jump happens, then the offset will be
    added to the PC, rather than just going to the next instruction. This leaves me with the following format:
</p>
<div class="center">
    <code>0100JJJJOOOOOOOO</code>
</div>

<h3>Deciding to Jump</h3>
<p>
    Using the flags, I can determine if a jump should or should not be executed. You may have noticed the
    <code>jumpDecider</code> module in the previous screenshot. It takes the saved flags along with the jump type and
    determines if a jump should occur.
</p>
<img
    src={imageJumpDecider}
    alt="Screen shot of the internals of the jump decider"
    class="center"
/>
<p>
    The first four are fairly simple, and just pass a flag or its inverse. The next eight types are unsigned and signed
    comparisons. You might notice that there are two similar structures there. These encode the logic I talked about
    earlier, where you can define the four comparison operations using just equals and less than.
</p>
<p>
    As described, the difference is how less than is defined. As previously described, for an unsigned comparison I just
    use the carry value. This also means that I don't need a separate type for jumping when a carry happened, this one
    type can do both! Later when creating a language, I could include both of these so that the code is clear, but they
    both use one instruction.
</p>
<p>
    For signed comparisons, I use both the signed overflow and the negative flag. This is because if the argument was
    larger than the target you will get a negative value, unless you overflow in which case the result is inverted. Lets
    think about this with a few examples in four bits.
</p>
<table>
    <thead>
        <tr>
            <th>Equation</th>
            <th>Flags</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>3 - 5 = -2</code></td>
            <td><code>N</code></td>
            <td>The result is negative without overflow, so you get that three is less than five</td>
        </tr>
        <tr>
            <td><code>-4 - -2 = -2</code></td>
            <td><code>N</code></td>
            <td>The result is negative without overflow, so you get that negative four is less than negative two</td>
        </tr>
        <tr>
            <td><code>-1 - -3 = 2</code></td>
            <td>None</td>
            <td>The result is not negative without overflow, so you get that negative one is not less than negative three</td>
        </tr>
        <tr>
            <td><code>6 - 2 = 4</code></td>
            <td>None</td>
            <td>The result is not negative without overflow, so you get that six is not less than two</td>
        </tr>
        <tr>
            <td><code>5 - -5 = -6</code></td>
            <td><code>NO</code></td>
            <td>The result is negative but with overflow, so you get that five is not less than negative five</td>
        </tr>
        <tr>
            <td><code>-8 - 1 = 7</code></td>
            <td><code>O</code></td>
            <td>The result is not negative but with overflow, so you get that negative eight is less than one</td>
        </tr>
    </tbody>
</table>
<p>
    This XORed value is used as the less than value and is put through the same logic for less than or equal, greater
    than, and greater than or equal.
</p>
<p>
    The three reserved types follow, never allowing a jump, and the final type always jumps.
</p>

<h3>Executing the Jump</h3>
<p>
    Now I know if I want to jump or not, so the next step is to execute it. This involves two pieces. First, only jump
    when the control module requests it. Without this, the computer will jump randomly when the flags and type (which is
    really just some bits from the instruction) happen to look like a valid jump. This is achieved by ANDing the output
    of the <code>jumpDecider</code> with the <code>Jmp</code> control line.
</p>
<p>
    Second, add a module that adds the value of the PC with the jump offset. This is just a standard addition module,
    but I do have to extend the 8 bit offset to 16 bits, extending the sign bit. These two values are fed into the PC,
    with the combined jump signal enabling value storing, and sending the summed value as the value to store. 
</p>
<img
    src={imageRelativeJump}
    alt="Screen shot of main hardware circuit"
    class="center"
/>

<h2>Halt</h2>
<p>
    In addition to jump instructions, I added the HALT instruction. It is a generic instruction, and is defined as
    <code>0x0000FFFFFFFFFFFF</code>. It stops execution of the computer, and is useful for knowing when your program is
    done.
</p>

<h2>Calculating Fibonacci</h2>
<p>
    With this simple setup, I have the tools needed for control flow, which gives me the tools needed to write a simple
    program that computed Fibonacci numbers. Specifically, I would like to calculate the first Fibonacci number that is
    less than or equal to a given value <code>N</code>.
</p>
<pre><code>; setup values for Fibonacci
; assume that register 0 already has 0
Load Low 0x1 into register 1

; load N
Load Low 0xFF into register 2

LOOP:
Add register 1 to register 0

; If register 0 is now larger than register 2, register 1 is the answer
Compare register 0 to register 2
Jump to :R1 if greater than

Add register 0 to register 1

; If register 1 is now larger than register 2, register 0 is the answer
Compare register 1 to register 2
Jump to :R0 if greater than

Jump to :LOOP

R1:
Move register 1 to register 3
Jump to :HALT

R0:
Move register 0 to register 3

HALT:
Halt</code></pre>
<p>
    This program makes use of jump instructions ins various ways. One is a conditional jump, like an <code>if</code> in
    higher level languages. You can see this happening when a comparison is executed before a conditional jump. For this
    program, I use it to check if the computed value is greater than <code>N</code>.
</p>
<p>
    This program also needs a loop, continually executed until a value greater than <code>N</code> is found. This is
    achieved by always jumping back to the start of the loop. The only way out is through one of the <code>if</code>
    conditions.
</p>
<p>
    I also use a jump towards the end, skipping over the instructions after <code>:R0</code> if the program jumped to
    <code>:R1</code>. This doesn't have an exact parallel in higher level languages, at least that I can think of. It is
    a detail about how I get the right value into register three.
</p>
<p>
    One thing you might notice is that I used labels, but that the jump instruction takes an offset. In order to convert
    this code into instructions, I will have to calculate the offset from the usage of the label to its position. As a
    first step, I will convert other instructions, remove comments, and add line numbers.
</p>
<pre><code>00  0x2101
01  0x22FF
02  LOOP: 0x1001
03  0x1302
04  0x46?? ; to :R1
05  0x1010
06  0x1312
07  0x46?? ; to :R0
08  0x4F?? ; to :LOOP
09  R1: 0x1231
10  0x4F?? ; to :HALT
11  R0: 0x1230
12  HALT: 0x0FFF</code></pre>
<p>
    To fill in the offsets (<code>??</code> above), take the line number of the destination, and subtract the line
    number of the instruction after the source. This gives the following program, which is included as
    <code>prog.bin</code> in the git repository.
</p>
<pre><code>0x2101 0x22FF 0x1001 0x1302 0x4604 0x1010 0x1312 0x4603 0x4FF9 0x1231 0x4F01 0x1230 0x0FFF</code></pre>
<p>
    Running this will generate the largest Fibonacci number under <code>0x00FF</code>, which is <code>0x00E9</code>, and
    place it in register 3. This program won't work for any number, as the comparison take place after addition. This
    means the result could overflow, resulting in a smaller number. I do have an instruction to check for a carry, so I
    could take this into account, but I would rather move forwards with the hardware.
</p>

<h2>Register Jump Instruction</h2>
<p>
    While looking at jumping, I would like to add an instruction that uses a value in a register rather than the
    immediate value. This instruction will allow for both relative and absolute jumps, and this will solve for several
    gaps at once. This also provides a way to jump to <em>any</em> address by loading it into a register.
</p>
<p>
    The jump instruction uses every bit, and I don't want to take away a bit from the immediate value. Because of this,
    I will use the next op code, <code>0b0101</code>. This jump instruction still takes a jump type, so the next four
    bits will be the jump type. The register jump needs to know which register to look at, and for that purpose I will
    use the least significant four bits. I want to use these because they correspond to the argument in other
    operations, and it feels natural.
</p>
<div class="center">
    <code>0101JJJJXXXXAAAA</code>
</div>
<p>
    This leaves four bits unused. I will use one of them, bit 4, to switch between relative and absolute jumps. I don't
    have a use for the other three bits yet, but maybe I will find some later. This means the final instruction has the
    following form:
</p>
<div class="center">
    <code>0101JJJJXXXRAAAA</code>
</div>
<p>
    This instruction adds a few control lines that are fed into a new module, the <code>jumpCalc</code>. This replaces
    the addition circuit that simply added the immediate value to the PC.
</p>
<img
    src={imageJumpCalc}
    alt="Screen shot of flags logic"
    class="center"
/>
<p>
    This circuit take the PC, immediate, and the value of the current argument register. It also takes two control
    signals, <code>JReg</code> and <code>RRel</code>. When <code>JReg</code> is low, the immediate value plus PC is
    used. If it is high, then you have to look at the value of <code>RReg</code>. If it is high, then add the value of
    the register to the PC. If it is low, just jump to the value of the register. Taken together, this circuit allows
    for both relative jumps with the immediate value and both relative and absolute jumps with register values.
</p>
<p>
    I wrote a simple test program called `registerjump.bin`. It sets up two values in registers and then jumps back and
    forth between two register jump instructions.
</p>
<pre><code>0x2004 0x2102 0x5F10 0x0FFF 0x0FFF 0x0FFF 0x0FFF 0x5F01</code></pre>
<p>
    This instruction will become even more useful as I add the ability to read and write from memory.
</p>

<NextPrevNav
    prev={{ url: "/projects/mini-computer/register-atom", text: "Register Atom" }}
    next={{ url: "/projects/mini-computer/memory-atom", text: "Memory in Atom" }}
/>

