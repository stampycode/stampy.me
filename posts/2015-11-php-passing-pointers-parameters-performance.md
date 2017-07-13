A little adds up to a lot, and in the world of code, a tiny change in code performance can have a big impact on application performance overall.

So, pointers aren't a "thing" in PHP. This article is about the use of PHP References, and I used the word 'pointer' in the title because it is alliterative. So there :)

<hr>

<h3>how to test performance of small blocks</h3>
So there are plenty of theories on how to test code performance, but this simple script below can trivially compare 2 blocks of code. Both blocks must be executed an equal number of times, with as little risk of other system interference as possible. This means you don't run one block of code 1000 times and then run another block 1000 times to compare them - this risks external influences affecting one of the execution cycles much more heavily than the other, tainting your results. So you run one, then the other, then repeat, capturing times for each execution as it happens.

<pre>
function funcA() {
    //do some code
};
function funcB() {
    //do some other code
};
//setup (only run once):
function changeDataA() {}
function changeDataB() {}

$loops = 5000;
$timeA = 0.0;
$timeB = 0.0;

ob_start();
for($i=0; $i<$loops; ++$i) {
    $start = microtime(1);
    funcA();
    $timeA += microtime(1) - $start;

    $start = microtime(1);
    funcB();
    $timeB += microtime(1) - $start;
}
ob_end_clean();

$timeA = round(1000000 * ($timeA / $loops), 3);
$timeB = round(1000000 * ($timeB / $loops), 3);

echo "
TimeA averaged $timeA microseconds
TimeB averaged $timeB microseconds
";
</pre>
So I'll be using this code to show how performance differs when writing your code slightly different ways. Sometimes the difference is very small, and you are welcome to reproduce these tests and come to your own conclusions, this blog contains only my professional opinions.

Sometimes the code set up cost should also be taken into account - if the code in question is called a small number of times in a single execution of your code then the set up cost of a block of code will have a bigger impact on performance. I will include the code I used to perform these tests below so you can see where I have included set up cost in the calculation.

<hr>
<h2>passing variables by reference</h2>
Passing variables by reference is when you use the `&amp;` prefixed to an argument in a function, so any changes made to that variable in the function will also affect the variable that was passed into the function.
These parameters are called references, passing 'By Reference' being the opposite of passing 'By Value' which is when you pass a variable without the '&amp;' prefix.

Example:
<pre>
function funcA() {
    $str = str_shuffle("0123456789");
    $str = changeData1($str);
    strlen($str);
};
function funcB() {
    $str = str_shuffle("0123456789");
    changeData2($str);
    strlen($str);
};
//setup (only run once):
function changeData1($data) {
    return $data . " World";
}
function changeData2(&$data) {
    $data .= " World";
}
</pre>
results in the following execution times:
<pre>
TimeA averaged 2.774 microseconds
TimeB averaged 2.748 microseconds
</pre>
So not a lot of difference there, but you can already see the `funcB()` function is very slightly faster.

<b>But...</b> Consider what happens you are working with larger data blobs. All of a sudden you are calling a function `changeData1()` with large amounts of data that PHP has to allocate memory for, copy it, alter it, then remove the original (and later garbage-collect it). This can happen when modifying the contents of a file, for example:
<pre>
function funcA() {
    $str = str_shuffle("0123456789");
    $str = str_repeat($str, 100000);
    $str = changeData1($str);
    strlen($str);
};
function funcB() {
    $str = str_shuffle("0123456789");
    $str = str_repeat($str, 100000);
    changeData2($str);
    strlen($str);
};
//setup (only run once):
function changeData1($data) {
    return $data . " World";
}
function changeData2(&$data) {
    $data .= " World";
}
</pre>
Outputs:
<pre>
TimeA averaged 542.497 microseconds
TimeB averaged 294.44 microseconds
</pre>
It was ~45% faster to use a reference here.
So when working with large strings, it is a lot more performant to alter the existing string, than to copy the passed string and return it.

<hr>

<h3>On a large array:</h3>
<pre>
function funcA() {
    $str = str_shuffle("0123456789");
    $str = str_repeat($str, 10000);
    $str = explode('0', $str);
    $str = changeData1($str);
    count($str);
};
function funcB() {
    $str = str_shuffle("0123456789");
    $str = str_repeat($str, 10000);
    $str = explode('0', $str);
    changeData2($str);
    count($str);
};
//setup (only run once):
function changeData1($data) {
    $data[] = " World";
    return $data;
}
function changeData2(&$data) {
    $data[] = " World";
}
</pre>
outputs:
<pre>
TimeA averaged 2980.23 microseconds
TimeB averaged 2028.161 microseconds
</pre>
So we took ~32% off the processing time here by passing our variable by reference.

<hr>

<h3>we can also use the reference operator `&amp;` in a regular `foreach` loop.</h3>
Here's an example:
<pre>
function funcA() {
    $str = str_shuffle("0123456789");
    $str = str_repeat($str, 1000);
    $str = explode('0', $str);
    foreach($str as $k => $v) {
        $str[$k] .= 'a'; //<-- we have to look up the key every iteration!
    }
    count($str);
};
function funcB() {
    $str = str_shuffle("0123456789");
    $str = str_repeat($str, 1000);
    $str = explode('0', $str);
    foreach($str as &$v) { //<-- & used here, and no key required any more
        $v .= 'a'; //<-- modify the value in the array using the reference 
    }
    count($str);
};
</pre>
And our survey says:
<pre>
TimeA averaged 296.358 microseconds
TimeB averaged 160.289 microseconds
</pre>
Sooooo.... We nearly improve the performance of this code by <b>~54%</b>, just by using a reference instead of modifying the array values by key.

<hr>

<h3>using a reference to reduce calls to nested variables</h3>

Why write this code:
<pre>
$myBigArray['firstVar']['secondVar']['thirdVar'][] = "FOO";
$myBigArray['firstVar']['secondVar']['thirdVar'][] = "BAR";
$myBigArray['firstVar']['secondVar']['thirdVar'][] = "Hello";
$myBigArray['firstVar']['secondVar']['thirdVar'][] = "World";
</pre>
when you can do this instead:
<pre>
$thirdVar =& $myBigArray['firstVar']['secondVar']['thirdVar'];
$thirdVar[] = "FOO";
$thirdVar[] = "BAR";
$thirdVar[] = "Hello";
$thirdVar[] = "World";
</pre>
Much better :)

<hr>

<h3>use `unset` when you want to redefine the reference </h3>

When working with references, it can be pretty easy to accidentally modify data you didn't intend to, consider this:
<pre>
$bar = "bar";
$foo =& $bar;
$foo = null;
</pre>
This code sets `$bar` to null. The reference `$foo` is still pointed at `$bar`.
If you want to stop `$foo` pointing at `$bar` you have to point it at something else, 
eg. <code>$foo =& $somethingElse</code>
or you need to call <code>unset($foo)</code>.

<hr>

Common methods in PHP already use references for performance reasons, such as:

for sorting the elements in the given array
<code>function sort (array &$array, $sort_flags = null) {}</code> (and other array sorting functions)

for randomly re-ordering elements in the given array
<code>function shuffle (array &$array) {}</code>

for iterating over the given array, modifying values as it loops over items in the given array
<code>function array_walk (array &$array, $funcname, $userdata = null) {}</code>

for getting the current element in the given array
<code>function current (array &$array) {}</code>

for adding a variable to the end of the given array
<code>function array_push (array &$array, $var, $_ = null) {}</code>

So all of these methods modify the given array in-situ, rather than returning a modified copy of it.
If you think about it, this makes a lot of sense, because if you want to retain the original array before you modify it, then it's just one line of code to copy the array; but if these functions always returned a copy of the given array, you would have to write your own function to modify the original array in-situ if you wanted the performance gain.

<hr>

Other core functions in PHP use references for passing data back to the calling code, for example:
<pre>
$count = null;
str_replace("_", "-", "ab_cd_12_34", $count);
echo "count: $count"; //says "count: 3"
</pre>
The signature for this function is:
<pre>
function str_replace ($search, $replace, $subject, &$count = null) {}
</pre>
This can be very handy when you want to change an existing function to return more than one variable, but don't want to change all the existing usages of the function in your code. Just tack on an optional reference parameter to the function signature and use that to return your extra data. And maybe design your system better next time :D

<hr>

<h3>you don't need to use references to objects</h3>

When passing objects around, copies of them are <b>NOT</b> created by default. So you don't need to use references in this case, the original object will still be modified by your function. See example:
<pre>
class Foo {
    public $foo;
}
$a = new Foo;
function updateMe($a) { //<-- no &
    $a->foo = "bar";
}
updateMe($a);
echo "Foo: ". $a->foo; // says "Foo: bar"
</pre>
<i>If you use a reference on the parameter here it doesn't give you any warning, because the parameter may accept primitive types as well as objects, which would otherwise be passed `ByVal` instead of `ByRef`.</i>

<hr>

<h3>ye olden days of references in php</h3>

For those who are interested, in PHP's history, parameters to be passed by reference use to have to be specified in the calling code, instead of the function signature, like so:
<pre>
$a = "hello";
function modifyMe($a) { //<-- reference not used here
    $a .= " world!";
}
modifyMe(&$a); //<-- reference used here
</pre>

But if you try to do this today, you get a Fatal error....
<pre>
PHP Fatal error:  Call-time pass-by-reference has been removed;
If you would like to pass argument by reference, modify the declaration of modifyMe().
</pre>

<hr>

<h3>Further Reading</h3>
<a href="http://php.net/manual/en/language.references.php" target="_blank">PHP Manual: References</a>
<a href="http://code.stephenmorley.org/php/references-tutorial/" target="_blank">Stephen Morley: PHP references tutorial</a>
