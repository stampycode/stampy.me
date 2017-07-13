<em>This is a helpful reminder to everyone (me included), to always add logging to the system you’re working on.</em>

If a code block is exiting abnormally, or results are not as expected, this obviously needs to be handled in the code, but is often the result of an external problem, and so should be recorded to a log file. In most circumstances this only involves adding one line of code. All logging facilities use different levels of logging output to differentiate the severity of the message, and to disable the more verbose levels in production systems, to save processing, disk space and I/O bandwidth.

Throwing a generic Access Denied Exception can be backed up with a more useful logging message, for analysis<em>:</em>
<pre class="lang:php decode:true">} catch (\Exception $e) {
    $this-&gt;container-&gt;get('logger')-&gt;error(
        "User does not have access to mid $mid"
    );
}</pre>
Exceptions should only be thrown under exceptional circumstances, hence  the name. This situation is worthy of being logged.
<em>Instead of this:</em>
<pre class="lang:php decode:true">} catch (\Exception $e) {}
</pre>
<em>We should be doing this:</em>
<pre class="lang:php decode:true ">} catch (\Exception $e) {
    $this-&gt;container-&gt;get('logger')-&gt;error(
        'Exception caught when fetching access list: " . $e-&gt;getMessage()
    );
}</pre>
Logging is a very useful development aid too, but consider <strong><em>not</em></strong> removing all your debugging output when your code is all working and polished.

I’ve used logging numerous times in the past to diagnose when a system is experiencing Fatal errors, beit from Syntax to SegFaults – using logging on a line-by-line basis, outputting simply the __CLASS__ name and __LINE__ number, and flooding files with these statements is sometimes the only way to track down the fault.

So if the message you’re logging contains no static parts, remember that it will be <u>harder to find</u> in the code later on! So include something like the current class name, or use the special built-in <a href="http://php.net/manual/en/language.constants.predefined.php">magic constants</a> that PHP has to offer:
<pre class="lang:php decode:true">} catch (\Exception $e) {
    $this-&gt;container-&gt;get('logger')-&gt;error(
        __METHOD__ . '#' . __LINE__ . ' : ' . $e-&gt;getMessage()
    );
}</pre>
