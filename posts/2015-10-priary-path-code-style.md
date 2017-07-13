This is a quick breakdown&nbsp;of a simple technique I use when&nbsp;laying out code within methods.

Basically it refers back to a common rule used when drawing logic flow diagrams, that the primary path in a logic flow diagram will follow the central vertical line from top to bottom.

This indicates that any non-standard route will follow a logic path that deviates from the central vertical line, and perhaps rejoins it further on.

Working on this idea, code methods and logic flow diagrams should be written in a similar fashion - if there are fewer lines in a logic flow diagram, then it is a simpler diagram. A line in a flow diagram basically translates to a code block in code, so the fewer nested blocks there are, the fewer lines in the diagram there would be.

If two flows in a diagram share roughly equal likelihood, then this rule has less significance, as mutually exclusive nested blocks must exist within the code to represent this. Consider whether the method could be split out into separate methods if this is the case.

As personal preference, I try not to nest code more than 3 levels deeper than the initial depth of the method. (This will start at 2 levels deep for a class method, one deep for a function.)

Here's an example of some code:
<pre>
public function doSomething()
{
    $a = funcA();
    $b = funcB();
    if(null !== $a) {
        funcC();
        funcD();
        //more code goes here
    } else {
        throw new \Exception("a should not be null");
    }
}
</pre>

This code demonstrates a problem I see regularly, and it follows a pattern that breaks the primary path code style. In this example the primary path only executes `funcA()` and `funcB()` and then deviates into two parallel paths, instead of suggesting which path is most likely. Normally having this separation would be fine, because it is entirely possible that the two paths (the if/else blocks) are equally likely - but in this case the `else` case only exists to throw an `Exception` if the `if` statement is not met.

This breaks our rule, as an exception should <b>only</b> be thrown in cases of <b>exceptional</b> circumstances, hence the name. The only caveat to this is if the `doSomething()` method serves the primary purpose of throwing an exception in the first place, which in this case it does not.

This code can be simplified following the primary path code style as follows:

<pre>
public function doSomething()
{
    $a = funcA();
    $b = funcB();
    if(null === $a) {
        throw new \Exception("a should not be null");
    }
    funcC();
    funcD();
    //more code goes here
}
</pre>

Here we can see the primary path now says that `funcC()` and `funcD()` are part of the primary path, and has a greater likelihood if being executed than throwing the Exception does.
You should also note that this version of the code takes up less space on disk, as there is reduced indenting, and the `} else {` line is no longer required.

To a further extreme, I have seen code that looks like this, but with very many more lines:
<pre>
public function doSomething()
{
    $a = funcA();
    $b = funcB();
    if(null !== $a) {
        funcC();
        $d = funcD();
        if(null !== $d) {
            $e = funcE();
            $f = funcF();
            if(null !== $f) {
                //further horrible indented code
            } else {
                throw new FIsNullException("f is null, this should never happpen");
            }
        } else {
            throw new DIsNullException("d should not be null");
        }
        //more code goes here
    } else {
        throw new AIsNullException("a should not be null");
    }
}
</pre>
You can see how this pattern-ignoring coding has lead to much unnecessary indenting, and generally more difficult to read code. Here it is simplified:
<pre>
public function doSomething()
{
    $a = funcA();
    $b = funcB();
    if(null === $a) {
        throw new AIsNullException("a should not be null");
    }
    funcC();
    $d = funcD();
    if(null === $d) {
        throw new DIsNullException("d should not be null");
    }
    $e = funcE();
    $f = funcF();
    if(null === $f) {
        throw new FIsNullException("f is null, this should never happpen");
    }
    //further primary path code
    //more code goes here
}
</pre>
This code has **exactly** the same functionality, but now follows the pattern, and you can easily see the code that follows the primary path, and you've saved some disk space too.
<hr>
If you think about how `if` statements are actually processed and executed by the CPU, `else` statements don't really exist in assembly in the same way they do in abstracted languages like PHP - in assembly, an `if` statement is akin to a `JMP` to jump to a different instruction block depending on the outcome of a comparison statement, (==0 `JZ`, &gt;=n `JGE`, &lt;n `JL`, etc.)<a href="https://en.wikipedia.org/wiki/Branch_(computer_science)">[1]</a>. If the comparison doesn't match, the jump doesn't happen, and the execution point moves to the next instruction. It is therefore more processor efficient (albeit slightly) to have fewer 'if' statements that match the statement, as this means fewer jumps, and fewer instruction blocks.
Whether this performance gain translates upwards into highly abstract languages such as PHP is not likely to be noticed, or even detectable, but it's always good to try to consider the instructions you are sending to the CPU once in a while, and ask yourself, <i>is there a more logical way to walk through this code?</i>
