One of the first, most obvious security practices that users are taught, is not to write down your password.

However, I have often come across code that contains secrets - written by developers who don't know better.

<ul>
<li>A session cookie is a password.</li>
<li>A secure token is a password.</li>
<li>An API key is a password.</li>
<li>Anything called a 'secret' is a password.</li>
</ul>

<b>Don't store these in source control - and NEVER store these anywhere public searchable.</b>

To access AWS (Amazon Web Services), Amazon provides you with an AWS key and secret. Together, these are equivalent to a username and password, used to access your AWS server instances.

If a malicious entity gets hold of your AWS key, they can potentially control your AWS instances, destroy them, re-purpose them, or use them in a CPU farm (which costs you money - potentially a LOT of money).

In the past few years there have been instances of AWS instances being hijacked and used as nodes in CPU farming, either used for distributed crypto attacks, <abbr title="Distributed Denial of Service">DDoS</abbr> attacks, mining for BitCoins, or sold on a timeshare basis on the dark-web. (See <a href="http://readwrite.com/2014/04/15/amazon-web-services-hack-bitcoin-miners-github/">here</a>, <a href="https://news.ycombinator.com/item?id=7490766">here</a>, and <a href="http://www.securityweek.com/how-hackers-target-cloud-services-bitcoin-profit">here</a>.)

<i><b>If you have put any secrets onto GitHub at any point, consider them compromised and reset them as soon as possible. Don't think for a second that just committing more code to replace your credentials will save you in any way - it most likely won't. Your git commit history is still there. Git does that. (Yes there is a way to permanently remove commits from Git history, but it's not a simple process, and your secrets have still been exposed.)</b></i>

Also consider that if a nefarious entity has managed to get hold of your keys for just a minute, would that have given them access to your server, or your database, or even write-access to your codebase, to install some nefarious code for granting them access once again when you change your keys?

Keep your secrets in a separate config file on your server. Keep a copy of this config file somewhere safe, and private. Add this file to your `.gitignore` file so it doesn't accidentally get added to Git.

<i>Keep your secrets, secret.</i>
