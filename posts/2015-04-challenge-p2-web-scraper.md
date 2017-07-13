The Task:
<blockquote>Write a php application that accepts a URL. Download the page the URL references. The page contents should then be broken into two parts. The first part determines all the different kinds of HTML tags on the page and the frequency counts for each. The second part determines all the different words that aren't part of the HTML on the page and the frequency counts for each. The results from the two parts should be stored in a database.</blockquote>
There are a number of reasons why someone would want to do this. Part of this challenge is to create re-usable code, but the main aim is to use best practice code and style to achieve the task, in an efficient, understandable and coherent manner.

In <a href="/blog/2015/04/challenge-part-one-simple-orm/">part one</a> of this challenge, we built our simple <abbr title="Object Relational Mapper">ORM</abbr> classes for storing well structured entities into a database, and here we are going to build on that, to store the results of our page scraping into a set of tables.
 
<h3>Database Design</h3>
Firstly let's do some database design. We need to store URLs, counts of tags on the page, and counts of words on the page that aren't part of the HTML markup.

Let's assume that we are writing this for a small application, and will only be scraping upto a few hundred sites. This allows us to make some assumptions about database capacity and performance considerations, like column widths, choice of database type, column sizes, etc. We will also begin with the assumption that this scraper will only scrape basic HTML pages - any largely dynamic pages (through Javascript or Flash) will not be processed very well, as they tend to offer less fixed HTML up front, with the focus on the browser enriching the page by making subsequent page requests and modifying the page after the initial load.

<pre>
USE scraperdb;

CREATE TABLE `TPage` (
    id SERIAL,
    title VARCHAR(255) NOT NULL COMMENT 'The title of the page we scraped',
    url VARCHAR(4096) NOT NULL COMMENT 'The URL we scraped',
    `when` DATETIME NOT NULL COMMENT 'When we scraped the page',
    success TINYINT(1) NOT NULL COMMENT 'Whether the attempt to scrape this page worked'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `TType` (
    id SERIAL,
    name VARCHAR(64) NOT NULL UNIQUE COMMENT 'The type of value saw on the page, eg. Tag, Content, etc.'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `TCount` (
    id SERIAL,
    TPageId BIGINT UNSIGNED NOT NULL COMMENT 'The page we scraped when we saw this value',
    TTypeId BIGINT UNSIGNED NOT NULL COMMENT 'The type of value we saw',
    value VARCHAR(64) NOT NULL COMMENT 'The value we saw on the page',
    `count` BIGINT UNSIGNED NOT NULL COMMENT 'The number of times we saw the value',
    CONSTRAINT `c_TCount__page_type_value`
        UNIQUE (TPageId, TTypeId, value),
    CONSTRAINT `c_TCount__TPageId`
        FOREIGN KEY (`TPageId`)
        REFERENCES `TPage` (`id`)
        ON DELETE CASCADE,
    CONSTRAINT `c_TCount__TTypeId`
        FOREIGN KEY (`TTypeId`)
        REFERENCES `TType` (`id`)
        ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
SQL;
</pre>

So we've created a database that will allow us to store URLs, Tags and Content values, and the number of times we have seen each. We've put in some referential integrity constraints to prevent us from doing silly things by accident, such as trying to enter two different counts for one Tag/Scrape instance.
We could go one step further and create a separate table for the values we scrape, thus achieving third-normal form
within our data structure - but at this point it would be overkill and premature optimisation of our system.
Also, I have not added any extra column indexes to the tables, as we don't yet have an idea of how the data will be used - we can add these once we have an established working prototype and want to optimise how we are using the results.

<h3>System Design</h3>
So lets now design our system. Here is some pseudo-code to establish what we're going to do.

<ol>
<li>enter URL to connect to</li>
<li>verify that we want to allow the given URL to be connected to</li>
<li>connect to the URL to check if robots are allowed, abort if not</li>
<li>connect to the URL and download the content in full</li>
<li>run an XML parser to analyse and pull apart our downloaded HTML</li>
<li>save the scraper page details to the database</li>
<li>analyse the tags, save the counts to the database</li>
<li>analyse the tag content, save the counts to the database</li>
</ol>

<h3>Verification</h3>
The URL that has been passed into our system may not be be in a form that we wish to accept - we may wish to prevent users from using IP addresses, or using a URL with embedded username and password to connect to.

<h3>Robots.txt</h3>
You'll have noticed that I've included a check for 'robots' - This is an internet standard that has been around for many years - you can read up more about it on the <a href="http://www.robotstxt.org">robotstxt.org</a> site. I've chosen to not scrape sites that have objected to being automatically scraped, using this method. You will see the code below contains checks for this.

<h3>XML Parser</h3>
We're going to use the built-in <a href="http://php.net/manual/en/class.domdocument.php">DomDocument</a> parser to parse our HTML. This library seems to be the most appropriate library to use for parsing HTML, as there is a <b>lot</b> of bad HTML in the wild, and this library is fairly fault-tolerant, and easy to use. HTML is not always XML compliant, some XML parsers will fail to parse HTML because of trivial shortcuts that programmers make when writing HTML, like not closing tags properly, or embedding attributes within tags that don't have an argument, eg. `&lt;script src="..." async defer&gt;`. This behaviour is not XML compliant.

<h3><abbr title="Object Relational Mapper">ORM</abbr> Entities</h3>
Our system has 3 entity classes, `TPage`, `TType` and `TCount`. These classes will extend the abstract class `AbstractEntity` and will be read from and written to the database using a class named `EntityHandler` - this will take care of the heavy lifting and database interactions.
Lets see what they look like...
<pre>
/**
 * All entities in the system that are storable in the database must extend the AbstractEntity class.
 * @package StampyCode\Scraper
 */
abstract class AbstractEntity
{
    /** @var int The ID of the object instance, generated by the DB */
    public $id;
}
</pre>
<pre>
class TPage extends AbstractEntity
{
    /** @var string The page Title for the URL scraped */
    public $title;

    /** @var string The URL scraped */
    public $url;

    /** @var \DateTime The date/time that the scrape was performed, or attempted */
    public $when;

    /** @var bool Whether the page scrape was successful */
    public $success;

    /** @var TCount[] Collection of  */
    public $tCounts;
}
</pre>
<pre>
class TType extends AbstractEntity
{
    /** @var string The name of the Type */
    public $name;
}
</pre>
<pre>
class TCount extends AbstractEntity
{
    /** @var TPage The scraped page that this count belongs to */
    public $TPage;

    /** @var TType The type of element this count refers to */
    public $TType;

    /** @var string The value of the element this count belongs to */
    public $value;

    /** @var int The number of elements of the given type that were found */
    public $count;
}
</pre>

<h3>The Code</h3>
Here's the class definition for our scraper. It contains all the features we've described above, commented and ready to be used.

<pre>
/**
 * Class Scraper
 *
 * @package Scraper
 */
class Scraper
{
    /** @var string */
    private $url;

    /** @var string */
    private $rawContent;

    /** @var int[][] */
    private $results = [];

    /**
     * @param string $url
     */
    public function __construct($url)
    {
        $this->url = $url;
    }

    /**
     */
    public function scrape()
    {
        $this->checkUrlSanity($this->url);
        $this->checkRobotPermission($this->url);
        $this->rawContent = $this->getHttpContent($this->url);
        $this->getTagSummary($this->rawContent);
    }

    /**
     * @return int[][]
     */
    public function getResults()
    {
        return $this->results;
    }

    /**
     * @param string $url
     * @throws \Exception
     */
    private function checkUrlSanity($url)
    {
        $urlParts = parse_url($url);
        if($urlParts['scheme'] !== 'http') {
            throw new \Exception("Scraper only accepts HTTP URLs");
        }
        if($urlParts['host'] === 'localhost') {
            throw new \Exception("Scraper will not scrape the local machine");
        }
        if(filter_var($urlParts['host'], FILTER_VALIDATE_IP)) {
            throw new \Exception("URLs must be host-based, not IP based");
        }
        if(!strpos($urlParts['host'], '.')) {
            // try to prevent the hostname from being a locally resolvable one
            throw new \Exception("Host names must contain at least a TLD and a gTLD");
        }
        if(isset($urlParts['pass'])) {
            throw new \Exception("Scraper will not accept URLs with username/password parameters");
        }
    }

    /**
     * @see http://www.robotstxt.org/
     * @param string $url
     * @throws \Exception if the given URL is not scrapable by robots.
     */
    private function checkRobotPermission($url)
    {
        $urlParts = parse_url($url);
        $url = $urlParts['scheme'] . '://' . $urlParts['host'] . ':' . $urlParts['port'] . '/robots.txt';

        $robotsTxt = $this->getHttpContent($url);
        if(!$robotsTxt) {
            //robots.txt file not found, so we can proceed :)
            return;
        }
        if($robotsTxt[0] === '<') {
            //we received some kind of other document instead, so not parsing it.
            return;
        }
        $urlPath = $urlParts['path'] ?: '/';
        if($urlParts['query']) {
            $urlPath .= '?'.$urlParts['query'];
        }
        $arr = explode("\n", $robotsTxt);
        $userAgent = '';
        $allowed = true;
        $ruleLength = 0;
        $lastMatchingRule = '';
        foreach($arr as $row) {
            if($row[0] === '#' || $arr === '') {
                continue;
            }
            list($field, $value) = strpos($row, ':') !== false ? explode(':', $row) : $row;
            //remove any trailing comments from the rule path
            if(($pos = strpos($value, '#')) !== false) {
                $value = substr($value, 0, $pos);
            }
            //rule field is case insensitive, value field is not
            $value = trim($value);
            $field = strtolower($field);
            if('user-agent' === $field) {
                $userAgent = $value;
            }
            if($userAgent !== '*') {
                //we are only interested in the generic wildcard robot rules for now.
                continue;
            }
            //todo: add processing for part-matching using extended syntax, eg. * ? and $
            if($field !== 'allow' && $field !== 'disallow') {
                continue;
            }
            //check the path provided is in the allow or disallow list
            $valueLength = strlen($value);
            if(substr($urlPath, 0, $valueLength) !== $value) {
                continue;
            }
            //the rule matches our URL, but we have to use the rule length to see if it is a more specific
            // rule that any previous one - /foo might be disallowed, but /foo/bar might be allowed......
            // (and when allow and disallow both match the url, the latest rule result is used)
            if($ruleLength >= $valueLength) {
                continue;
            }
            $ruleLength = $valueLength;
            $allowed = ($field === 'allow');
            $lastMatchingRule = $value;
        }
        if(!$allowed) {
            throw new \Exception("Robots are not allowed to access path '$lastMatchingRule'");
        }
    }

    /**
     * Retrieve the web content of the URL provided
     *
     * @param string $url
     * @return string
     */
    private function getHttpContent($url)
    {
        $context = stream_context_create(['http' => ['header'=>"Connection: close\r\n"]]);
        return file_get_contents($url, false, $context);
    }

    /**
     * Summarises the given HTML content and stores a count of all seen tag names and non-HTML words used
     *
     * @param string $content
     * @throws \Exception if the parser fails
     */
    private function getTagSummary($content)
    {
        $tags = [];
        $words = [];
        $excludedTags = [];
        $excludedWords = [null,''];

        $charHandler = function($data) use (&$words) {
            $data = preg_replace('|[^a-zA-Z0-9_-]|', ' ', $data);
            $data = explode(' ', $data);
            $words = array_merge($words, $data);
        };
        //todo: exclude text content within Script and Style tags

        $doc = new \DOMDocument();
        $doc->loadHTML($content);
        $this->processDomNodeList($doc->documentElement->childNodes, $tags);

        $charHandler($doc->textContent);

        $tags = array_diff($tags, $excludedTags);
        $words = array_diff($words, $excludedWords);
        $tags = array_count_values($tags);
        $words = array_count_values($words);
        arsort($tags);
        arsort($words);
        $this->results = [
            'Tag' => $tags,
            'Word' => $words
        ];
    }

    /**
     * Iterates over the given \DomNodeList object, identifies the tag name and stores it to the given array
     *
     * @param DomNodeList $list
     * @param string[] $tagList
     */
    private function processDomNodeList(\DomNodeList $list, &$tagList)
    {
        for($i=0; $i<$list->length; ++$i) {
            $item = $list->item($i);
            $tagList[] = $item->nodeName;
            if($item->childNodes instanceof \DomNodeList) {
                $this->processDomNodeList($item->childNodes, $tagList);
            }
        }
    }
}
</pre>

<h3>Saving the Results</h3>
This is a basic working prototype for a web-page scraper. It accepts a URL, processes the given page, and counts the number of Tag types and Words used in the page. In order to store these results, we create instances of our `AbstractEntity` classes, and save them to the database using our `EntityHandler` class.

<pre>
try {
    //establish our database connection, to pass into the EntityHandler class
    $dbConn = new MysqliDbConnection();
    $dbConn->setParameters(
        [
            'user' => 'scraperdbuser',
            'pass' => '<mypassword>',
            'host' => 'localhost',
            'dbname' => 'scraperdb',
            'port' => null
        ]
    );
    $dbConn->connect();
    $entityHandler = new EntityHandler($dbConn);

    // Create a TPage object to associate and collate our results with
    $testPage = new TPage();
    $testPage->title = 'Foo';
    $testPage->url = 'http://stampy.me';
    $testPage->when = new \DateTime();
    $testPage->success = true;

    $scraper = new Scraper($testPage->url);

    try {
        $scraper->scrape();
    } catch (\Exception $e) {
        echo "Scrape Failed - ".$e->getMessage() .' - '. $e->getTraceAsString();
        $testPage->success = false;
    }

    foreach($scraper->getResults() as $tagType => $result) {
        //get the tag type class, if it exists
        $typeObj = $entityHandler->get('TType', ['name' => $tagType]);
        if(!$typeObj) {
            // otherwise, create it!
            $typeObj = new TType();
            $typeObj->name = $tagType;
        }
        foreach($result as $value => $count) {
            //create a new TCount object for each count result
            $tag = new TCount();
            $tag->count = $count;
            $tag->TPage = $testPage;
            $tag->TType = $typeObj;
            $tag->value = $value;
            $testPage->TCountList[] = $tag;
        }
    }

    $entityHandler->set($testPage);

    print_r($scraper->getResults());

} catch(\Exception $e) {
    echo $e->getMessage() . "\n\n" . $e->getTraceAsString();
}
</pre>

<h3>Alternatives</h3>
There's a world of options out there for page scraping - it is after all, how search engines operate. They connect to a website, pull useful information from it, which includes links to other pages or websites, and then connect to those as well. This tutorial was written in a couple of days by a single developer - and the simplicity of the classes reflect that.

Enjoy :)
