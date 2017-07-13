When you're learning about object-oriented code, people use "code deduplication" as one of the reasons for modularising your code, to increase code re-use, and the cut the number of times the same blocks of code are present inside any given system.

The following phrases are some of those that all developers should be thinking:
<ul>
<li><i>"If I want to re-use this block of code, maybe I should put it in a method..."</i></li>
<li><i>"If I want to re-use these methods, maybe I should put them in a separate class..."</i></li>
<li><i>"If I want to re-use this set of classes, maybe I should put them into a bundle..."</i></li>
</ul>

These thoughts all focus on code re-use for the purposes of maintainability, and de-duplication of effort. They focus on code-deduplication at Method, Class, and Module levels, respectively.

What these statements have missed, is that there is one more important reason for deduplication, that of <b>code performance</b>.

<hr>

Code performance is overlooked often in <b><i>so much</i></b> of the code that I see, that it is often the number one reason that I end up spending time refactoring.

Consider the following code. It suffers from code duplication at the line-level that reduces readability, performance, and maintainability.
<pre>
switch ($filters->getDateFilter()->getDateType()) {
    case 'started':
        $searchQueryBuilder->applyFilter(
            new StartDateRangeFilter($filters->getDateFilter()->getFrom(), $filters->getDateFilter()->getTo())
        );
        break;
    case 'ended':
        $searchQueryBuilder->applyFilter(
            new EndDateRangeFilter($filters->getDateFilter()->getFrom(), $filters->getDateFilter()->getTo())
        );
        break;
    case 'created':
        $searchQueryBuilder->applyFilter(
            new CreatedDateRangeFilter($filters->getDateFilter()->getFrom(), $filters->getDateFilter()->getTo())
        );
        break;
    case 'active':
        $searchQueryBuilder->applyFilter(
            new ActiveDateRangeFilter($filters->getDateFilter()->getFrom(), $filters->getDateFilter()->getTo())
        );
        break;
    default:
        throw new Exception("Invalid date_type, valid types are 'started', 'ended', 'created' and 'active'");
}
</pre>
Repeated calls in this statement:
<ul>
<li>`$filters->getDateFilter()` <i>- occurs 9 times</i></li>
<li>`$searchQueryBuilder->applyFilter()` <i>- occurs 4 times</i></li>
<li>`$filters->getDateFilter()->getFrom()` <i>- occurs 4 times</i></li>
<li>`$filters->getDateFilter()->getTo()` <i>- occurs 4 times</i></li>
</ul>
There are several methods in the above code that are repeatedly called, even though we know that the result will always be the same.
<i>(If we know that the result may change between calls, then this should be illustrated in a comment, but only badly written code would prefix a mutator method with 'get')</i>

<pre>
$dateFilter = $filters->getDateFilter();
$from = $dateFilter->getFrom();
$to = $dateFilter->getTo();

switch ($dateFilter->getDateType()) {
    case 'started':
        $filter = new StartDateRangeFilter($from, $to);
        break;
    case 'ended':
        $filter = new EndDateRangeFilter($from, $to);
        break;
    case 'created':
        $filter = new CreatedDateRangeFilter($from, $to);
        break;
    case 'active':
        $filter = new ActiveDateRangeFilter($from, $to);
        break;
    default:
        throw new Exception("Invalid dateType, valid types are 'started', 'ended', 'created' and 'active'");
}
$searchQueryBuilder->applyFilter($filter);
</pre>
In the code above, these repeated calls have each been replaced with a single method call, using a local variable to store the result when appropriate, so it can be referenced without repeating the call. Code that is not necessary to be inside of the `switch` statement has been relocated after the switch statement.
This code could be shrunk further with the use of stringified method names, but in this instance is unnecessary, makes code tracing and maintenance harder, and reduces code readability and possibly also affects performance to a small extent.


Here is an example of how you could shrink the code further:

<pre>
$dateFilter = $filters->getDateFilter();
$dateType = $dateFilter->getDateType();

$types = [
    'started' => 'StartDateRangeFilter',
    'ended' => 'EndDateRangeFilter',
    'created' => 'CreatedDateRangeFilter',
    'active' => 'ActiveDateRangeFilter',
];
if(!array_key_exists($dateType, $types)) {
    throw new Exception("Invalid date_type, valid types are '" . implode("', '", array_keys($types))."'");
}

$searchQueryBuilder->applyFilter(new $types[$dateType]($dateFilter->getFrom(), $dateFilter->getTo()));
</pre>

This shrunk code does have the advantage that it is the most flexible, in that even the Exception message is dynamic, so adding a new type now only involves adding one line of code.
From the perspective of code tracing and maintainability, you could add a PHPDoc `@var` tag to tell any watching IDE that the resultant class could be an instance of any one of the four listed classes from above:
<pre>
/** @var StartDateRangeFilter|EndDateRangeFilter|CreatedDateRangeFilter|ActiveDateRangeFilter $f **/
$f = new $types[$dateType]($dateFilter->getFrom(), $dateFilter->getTo());
$searchQueryBuilder->applyFilter($f);
</pre>
But this just starts to get silly, and we are again repeating the class names from the list for the purposes of IDE integration and code documentation.

<hr>

Some developers would prefer to use an inline formatted layout for the switch statement as below:
<pre>
$dateFilter = $filters->getDateFilter();
$from = $dateFilter->getFrom();
$to = $dateFilter->getTo();

switch ($dateFilter->getDateType()) {
    case 'started': $filter = new StartDateRangeFilter($from, $to);   break;
    case 'ended':   $filter = new EndDateRangeFilter($from, $to);     break;
    case 'created': $filter = new CreatedDateRangeFilter($from, $to); break;
    case 'active':  $filter = new ActiveDateRangeFilter($from, $to);  break;
    default:
        throw new Exception("Invalid dateType, valid types are 'started', 'ended', 'created' and 'active'");
}
$searchQueryBuilder->applyFilter($filter);
</pre>
Recently I have been avoiding this code pattern, as it is against the <a href="http://www.php-fig.org/psr/psr-2/">PSR-2 Code Style Guide</a> layed out by <a href="http://www.php-fig.org/">PHP-FIG</a> - This is one of the standards I follow, and my IDE is configured to auto-format code which prevents me using this pattern. This is one of the (very few) things I dislike about the PSR-2 guide.
