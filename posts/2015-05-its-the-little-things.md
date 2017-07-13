When in a rush to deploy new applications, people rarely consider the impact of working on additional little features. I'm not talking about fixing bugs or highlighting browser incompatibilities, I'm talking about little additional extras that normal people dont always notice, but technically savvy people will.

It's the little things in an application that make the difference between being quite good, and being brilliant.

<hr>
In an interactive web application, consider adding context menus to interface components. Being able to right-click on something and perform an action on it that would otherwise require you to go to a new page to perform the same action, is a little thing, but a useful, time saving, bandwidth reducing addition to your application.

<hr>
If you have paginated results of things that is limited to a specific number, say 20, why not check first if the actual list size is only marginally over that number before sending back only 20 items?
If you have 20 items on page 1, and there are 2 pages, the user may be disappointed when they click on the 'next page' button when only 1 more result comes back.
If there are only marginally more results than the pagination is limited to, just show them all and hide the pagination features.

<hr>
Make use of HTML5 form features - use the new types! 
Instead of using <code><input type="text" name="phone"></code> 
you should be using <code><input type="tel" name="phone"></code>. 
On mobile devices, this restricts the available keys on the displayed keyboard to only those used in phone numbers, adding an extra layer of validation, and also making the user feel like they might actually be using a modern website.
There are more in-depth validation options you can use here, there is a great article here: <a href="http://www.html5rocks.com/en/tutorials/forms/html5forms/">Making Forms Fabulous with HTML5</a>.

<hr>
Make sure any part of your site that submits personal information, like your name, email address, and especially login details, always submit this information using HTTPS.
<sub>Actually this isn't a little thing, it is quite a big thing, but I mention it here anyway because it's good to remind developers as often as possible that they <i>really should be doing this</i>.</sub>

<hr>
Make sure your site has a `favicon` :) Remember that any user that opens more than a few tabs may only be able to see the favicons and not the page names in their tabs list. A picture paints a thousand words. Adding this small image creates a strong bond between the colours and shapes of the image, and your website. This helps users identify which website is yours in their tabs list, and hopefully in their list of bookmarks too.

<hr>
If you have several simple pages on your site that users visit regularly, consider combining them into a single page, even if this means having another page, it might just cause a reduction in unnecessary traffic.
Maybe show a portion of each page instead, so users can see part of the pages at a glance, and can still click through to them to see more detail.

<hr>
Use hover/popup/help text. If you have an abbreviation on a page, and your user doesn't know what it means, they might feel a little frustrated that you are expecting them to know already. Don't force them to Google it - use the `&lt;abbr&gt;` tag. When it isn't an abbreviation but a link displayed only as an icon, then make sure you use the `title` attribute of the image - otherwise your users may not even know what the button does!
<sub>In fact it is a generally recommended accessibility rule anyway - those with impaired visibility might have other less common mechanisms to interpret your website, and these devices can't be expected to describe what your icons look like if you haven't bothered to give them any description.</sub>
<hr>

Happy tinkering :)
