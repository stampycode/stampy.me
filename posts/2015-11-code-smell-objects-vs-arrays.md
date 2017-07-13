<a href="https://en.wikipedia.org/wiki/Code_smell">"Code Smell"</a> is a term that I have heard used in the past, and regularly use myself to describe code that may work fine, but just doesn't seem as good or clean as it could be. Maybe there's some repeated code, or unnecessary over-use of strings. <i>Basically the code smells bad.</i>

<b>Use objects instead of associative arrays for working with collections of values.</b>

Here's why...

In Javascript, it is now commonplace to see a single object passed to a method as a collection of parameters. 
This is for code flexibility, so you can call the method with as many or as few values in the object as you like, and you can build the list of values to be passed to a method over a larger area of code. It also means you can simply forward the parameters object to another method, perhaps modifying some of the parameters between calls.

So the community has accepted this is convenient way to pass parameters around - I'm not saying this should be done everywhere, in fact this should only be done when it smells right to do so... Currently one commonly used option is to use associative arrays to accomplish this. Here are some unexpected drawbacks:
<ul> 
<li>`$arr['foo']` can't be described using `/** @var */` (or PHPStorm doesn't support it).</li>
<li>`$arr['SomeVariebleName']` won't show the typo when you meant `$arr['SomeVariableName']`</li>
<li>`$arr['bar']` might be not set, and you'd have to use `isset()` or similar function to determine if it has been set.</li>
<li>`$arr['foo']` is slower than `$arr->foo`. Not by a lot, but it is.</li>
<li>finding all the places in your code that `$arr['foo']` is used when your array keeps getting passed around, is hard work.</li>
<li>you have no dedicated place to put functions for interacting with this array of parameters.</li>
</ul>

To fix some of these issues you might convert your array to an object using a cast:
<pre>
$arr = ['foo'=>'a', 'bar'=>'b']; 
$object = json_decode(json_encode($arr), FALSE);
//or 
$object = (object) $arr;
</pre>
However, you still have no dedicated space to describe the object - `$arr->foo` still has no built-in PHPDoc, no default value, and if you are likely to use the same object in multiple classes, you end up reproducing blocks of code, which also smells bad.

<h3>Here are some solutions...</h3>
<pre>
class HelloWorldParams
{
    /** @var int */
    public $foo = 1;

    /** @var string */
    public $bar = 'foo';

    /** @var string */
    public $ran = 'bar';
}

$f = new HelloWorldParams();
$f->foo = 1;
$f->bar = 'Hello';
$f->ran = 'World!';
</pre>
So here we've completely replaced using the array with using a custom object, in the same way that we would use a STRUCT in C. No methods, just a class that contains a bunch of properties. Using this object makes reading the code easier, tracing code easier, and refactoring code easier. And all it takes is a tiny little class that can be re-used elsewhere in your project.

<blockquote>
What about when we have an array and we need to convert it?
</blockquote>
Good question. We can add a constructor method to our class to make this easier:
<pre>
class HelloWorldParams
{
    /** @var int */
    public $foo = 1;
    
    /** @var string */
    public $bar = 'foo';
    
    /** @var string */
    public $ran = 'bar';
    
    /**
     * HelloWorldParams constructor.
     *
     * @param mixed[] $arr
     */
    public function __construct(array $arr=[])
    {
        foreach($arr as $k => $v) {
            $this->$k = $v
        }
    }
}

$arr = [
    'foo' => 1,
    'bar' => 'Hello',
    'ran' => 'World!'
];
$f = new HelloWorldParams($arr);
</pre>
<i>This technique demonstrates replacing an associative array with an object, to improve performance, maintainability, readability and to reduce code duplication. That is all.</i>

Some folks might say
<blockquote>you should always have setters and getters to access object properties</blockquote>
but these people are wrong. <abbr title="Keep It Simple, Stupid!!">KISS!!</abbr>

There is a time and a place for getters and setters <i>(or accessors and mutators if you're in academia)</i> and this is not it. You can add them if you like, but <abbr title="in my humble opinion">IMHO</abbr> you're just adding unnecessary code. Remember that you've already improved your code with this simple step above, there's no need to go overboard.

This is a simple class, and if you want to add complex functionality like input validation (which is one of the main reasons for using setters), then you're better off using a validation class that validates the input data, and a hydrator class that populates the object, which is beyond the scope of this blog entry.

<hr>

It is possible to reduce the code a little using PHPDoc instead of explicit properties - I advise against this as it reduces code clarity, but I'll describe what I mean here anyway, so I can show you a slightly more unusual PHPDoc use-case:
<pre>
/**
 * @property int $foo
 * @property string $bar
 * @property string $ran
 */
class HelloWorldParams{}
$f = new HelloWorldParams();
</pre>
So this happens in PHPStorm:
<center>
<img style="width:414px; height:218px" src="/wp-content/uploads/2015/11/phpdoc-property-usage1.png" alt="PHPStorm recognises @property values" />
</center>
So the only practical difference between this and the previous code above, is that we can't use default values for the properties unless we add them as real properties of the class. Also, parsing annotations in this class won't work as effectively as it would for the previous example, because we are describing properties within the class docblock, instead of using dedicated docblocks for each property. 

<i>(A docblock is a `/** ... */` , not to be confused with a multi-line comment `/* ... */`.)</i>

<hr>

All the examples above allow you to add any property to the class, so by default it doesn't prevent you from setting a property to the object that it doesn't expect to have.
PHPStorm will warn you that you're using an unexpected field, but it will still let you set the property:
<center>
<img style="width:656px;height:144px;" src="/wp-content/uploads/2015/11/field-not-found.png" alt="PHPStorm Field not found" />
<img style="width:530px;height:156px;" src="/wp-content/uploads/2015/11/field-declared-dynamically.png" alt="PHPStorm field declared dynamically" />
</center>

So here's one way to prevent custom dynamic fields being set in your object, that still retains most of the above functionality:
<pre>
class HelloWorldParams
{
    public $foo;
    public $bar;
    public $ran;
    /**
     * @param string $name
     * @param mixed $value
     */
    public function __set($name, $value)
    {
        if(!property_exists($this, $name)) {
            throw new \InvalidArgumentException(
                "Property '$name' not present for object " . __CLASS__
            );
        }
    }
}
$f = new HelloWorldParams();
$f->abcd = "bar";
</pre>
We're using the <a href="http://php.net/manual/en/language.oop5.overloading.php#object.set">`__set()` magic method</a> to throw an exception if we try to set a value to a property that doesn't exist. Simple.
