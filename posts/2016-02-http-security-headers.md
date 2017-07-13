There is a new breed of HTTP response headers, the sole purpose of which is to improve the security of your site for your readers.
<br />
In the following post, I'm going to describe the most popular headers, what they do, and why you should use them.

<pre>
Content-Security-Policy: connect-src 'none' ;
    font-src 'self' https://fonts.gstatic.com https://s0.wp.com ;
    form-action 'none' ;
    frame-ancestors 'none' ;
    child-src https://ghbtns.com https://widgets.wp.com https://platform.twitter.com
        https://www.facebook.com https://staticxx.facebook.com ;
    img-src 'self' data: https://i1.wp.com https://i0.wp.com https://stackexchange.com
        https://www.paypalobjects.com https://secure.gravatar.com https://pixel.wp.com
        https://www.facebook.com https://syndication.twitter.com ;
    media-src 'none' ;
    object-src 'none' ;
    script-src 'self' https://s1.wp.com https://connect.facebook.net
        https://platform.twitter.com https://s0.wp.com https://secure.gravatar.com
        'unsafe-inline' ;
    style-src 'self' https://secure.gravatar.com https://fonts.googleapis.com https://s0.wp.com ;
    default-src 'none'
Strict-Transport-Security: max-age=31531337; includeSubDomains; preload
Upgrade-Insecure-Requests: 1
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1 ; mode=block
Public-Key-Pins: pin-sha256="abcabcabcabcabcabcabcabcabcabcabc=="; max-age=31531337 ; includeSubdomains;
</pre>

<h3><code>Content-Security-Policy (CSP)</code></h3>
This header tells the browser where it is allowed to fetch extra content from. It basically helps thwart code injection attacks that would normally fetch and run code from a remote site hosting malicious code.
This injected code could be unwittingly served from your website, or could be injected in mid-air <abbr title="man in the middle">MITM</abbr> style.

<h3><code>Strict-Transport-Security (HSTS)</code></h3>
This header should <em>ONLY</em> be included in your headers when served over HTTPS, otherwise it is a breach of the specification. This is pretty much because this header is saying "Always Use HTTPS" and to put it in the HTTP header enables a <abbr title="man in the middle">MITM</abbr> attack to remove the header entirely, which provides you as a site owner with a false sense of security that you've used the header in the first place.
<br />
You should be upgrading all HTTP requests to HTTPS and setting this header on all subsequent HTTPS responses - this way any browser that re-connects to your site won't attempt to connect using HTTP, it will remember to only use HTTPS. <em>This is helpful for when you have an old site with lots of links around the place referring to your site using the HTTP protocol.</em>

<h3><code>Upgrade-Insecure-Requests</code></h3>
This header is somewhat redundant with site-wide usage of HSTS, but if portions of your site are still HTTP and are being migrated to HTTPS, this is the header you should use in your HTTP (not HTTPS) responses to tell the browser that all links in the content being served should be treated as if they were `https://` instead of `http://`.

<h3><code>X-Content-Type-Options</code></h3>
The only value for this header is `nosniff`. It basically prevents the browser from trying to guess the content type of a fetched resource.
This adds a useful stop-gap in the event that one of the resources being referenced from your site changes from something benign to something malicious - if an image hosted on your server is replaced somehow with malicious JavaScript, but the header type being returned along with the javascript is still `Content-Type: image/png` or similar, then the browser should try to interpret that JavaScript as an image, and if it fails, then ignore it, raise an error, and move on.
<br />
Historically, where the Content-Type of data returned from a request doesn't match what the browser is expecting, the browser takes it upon itself to try to guess the content type to use the content anyway, which lead to security issues.

<h3><code>X-Frame-Options</code></h3>
Prevent your site from being served from within someone else's `iframe`. This has been a popular way for hackers and clickjackers to make you think you are browsing one site, but instead you are just browsing the site through an invisible frame in a different site. This enables mouse  or keyboard actions on the iframe (making it look like you're clicking on your site) to be hijacked and/or recorded.
<br />
The `SAMEORIGIN` policy as used above, ensures any iframes linking to [parts of] your site are themselves hosted on the same domain as your site itself. So you can have parts of your own site in your own iframes, but no other domains are allowed to host an iframe pointing to [parts of] your site.

<h3><code>X-XSS-Protection</code></h3>
This header is largely redundant, in that <abbr title="Cross Site Scripting">XSS</abbr> protection should be enabled by default in all browsers anyway, but this serves to forcefully turn it on if it isn't already.
<br />
<br />
Cross-Site Scripting can happen in several ways, but generally involves scripts executed on a different site to send requests from that site to your site, but to make it look like the user is doing it deliberately. A common scenario is that a request launched via JavaScript from `http://evil.example.com` makes a `POST` request to your site, e.g. to perform some action that only that user can perform, such as update their password... Or transfer money from their bank account...

<h3><code>Public-Key-Pins (PKP)</code></h3>
This is a way of "pinning" the <abbr title="Secure Socket Layer">SSL</abbr> certificate your site is using to your domain, so if a server tries to impersonate your site, the <abbr title="Secure Socket Layer">SSL</abbr> connection will be rejected, as they won't have the site certificate they need.
<br />
These days there are a lot more <abbr title="Certificate Authorities">CAs</abbr> than there ever used to be, and recently there have been some emerging that offer free <abbr title="Secure Socket Layer">SSL</abbr> certificates - like <a href="https://letsencrypt.org">Let's Encrypt</a> for example.
<em>(Incidentally, I wholly recommend this site and will be switching my site over to using it very soon.)</em>
<br />
In the event that the domain is <abbr title="Domain Name System">DNS</abbr> hijacked, your browser may try to make an <abbr title="Secure Socket Layer">SSL</abbr> connection to a new server... if this happens for a long enough time period, an SSL certificate could be systematically issued to this new server, so when you connect to this new server, you'd see the pleasing green address bar (assuming their very new certificate is valid), and the domain name matches what you originally typed in.
At this point, before you've even typed anything in, your browser has sent this malicious site your protected session cookies, and possibly even entered your username and password into the login dialog on your behalf, if you've got a password manager that does this for you.
<br />
Your browser doesn't know not to do this! It's a legitimate site that you've logged into before, it's just got a different <abbr title="Secure Socket Layer">SSL</abbr> certificate, but who cares, right? As long as it's valid?!.....
<br />
<br />
Public Key Pinning means that when you connect to a legitimate site with <abbr title="Secure Socket Layer">SSL</abbr>, the browser will remember the signature of that certificate until it expires <em>(with some extra conditions to enable the certificate to be updated)</em>.
<br />
So when a <abbr title="man in the middle">MITM</abbr> attack does happen and your browser tries to connect to this new IP with HTTPS, it will be automatically rejected and you should receive a nice big red warning page telling you the site certificate is not what it expected.
This thwarts an attack that is getting progressively easier and cheaper to carry out.
<br />
<a href="https://stampy.me/wp-content/uploads/2016/02/icon-256x256.png" rel="attachment wp-att-394"><img src="https://stampy.me/wp-content/uploads/2016/02/icon-256x256.png" alt="icon-256x256" width="256" height="256" class="alignright size-full wp-image-394" /></a>
<b>* &nbsp; * &nbsp; *</b>
<br />
At last count there were 176 Root <abbr title="Certificate Authorities">CAs</abbr> in the list bundled within Mozilla Firefox.
<br />
These authorities are trusted implicitly - any valid certificate signed by one of these is trusted by your browser, at least unless it violates the <abbr title="Public Key Pinning">PKP</abbr> header.
<br />
These CAs sign certificates for "Intermediate Certificate Authorities" who in turn can sign certificates for other intermediate <abbr title="Certificate Authorities">CAs</abbr>, as well as the usual end-user domains.
<br />
This all means that there is a whole heap of delegated trust out there, just one of those <abbr title="Certificate Authorities">CAs</abbr> or intermediate <abbr title="Certificate Authorities">CAs</abbr> needs to be compromised in order to allow the hacker to sign certificates for any domain they like, and carry out highly successful <abbr title="man in the middle">MITM</abbr> attacks without detection.
<br />
<b>/!\ If you install your own root <abbr title="Certificate Authority">CA</abbr> certificate into your system, you'd better make sure that the private key for that certificate is kept somewhere *really really safe* - because if anyone else gets hold of it, they can perform a <abbr title="man in the middle">MITM</abbr> attack against you on <u>any site</u> and you probably won't notice.</b>
<br />
Companies unfortunately use internal root <abbr title="Certificate Authority">CA</abbr> certificates quite a bit, primarily so that they can easily manage secure connections within the company intranet, but it also enables them to intercept all your other <abbr title="Secure Socket Layer">SSL</abbr> connection setup requests at the gateway, and effectively perform a <abbr title="man in the middle">MITM</abbr> attack against you at your work's gateway, albeit with less malicious intentions... <i>(you hope.)</i> So do you trust your company to handle this certificate responsibly? If it gets into the wrong hands, a person with malicious intentions could intercept and read <u><b>all</b></u> of your communications made on the company network. 
<br />
<u>All of them.</u> <i>(Except the ones protected with <abbr title="Public Key Pinning">PKP</abbr>!)</i>
<br />
This is why <a target="_blank" href="http://www.theregister.co.uk/2015/09/21/symantec_fires_workers_over_rogue_certs/">it's a big deal</a> when owners of root <abbr title="Certificate Authority">CA</abbr> certificates misbehave - the consequences could be catastrophic.
<br />
Similarly, when CAs are compromised - it will undoubtedly <a target="_blank" href="https://threatpost.com/final-report-diginotar-hack-shows-total-compromise-ca-servers-103112/77170/">end badly</a> for the company responsible, or they will at least have <a target="_blank" href="https://www.us-cert.gov/ncas/current-activity/2015/11/24/Dell-Computers-Contain-CA-Root-Certificate-Vulnerability">the wrath of governments</a> to deal with. These incidents undermine the security that is fundamental to the Internet.
<br />
<br />
According to <a target="_blank" href="https://scotthelme.co.uk/security-headers-alexa-top-million/">Scott Helme's research</a>, only 0.0265% of sites in the top 1 million sites on the web (according to <a href="http://www.alexa.com">Alexa</a> rankings) were using <abbr title="Public Key Pinning">PKP</abbr>. This is a little concerning, given what we've just learned - and although this number has increased by ~68% in 6 months, it's still a tiny number. Companies aren't doing enough, but maybe 
<br />
<br />
I leave the choice up to you which security headers you use on your site. <i><b>I suggest you use them all.</b></i>
<br />
<br />
<h3>WordPress Plugin</h3>
I have written a <a href="https://en-gb.wordpress.org/plugins/headit/">WordPress plugin</a> that enables the owner to add their own custom HTTP response headers to their WordPress site. I wrote it because I couldn't find any existing plugin that enabled me to do this, at least not with the flexibility I wanted.
This is the method I used to add <i>(nearly)</i> all of the security headers described in this blog, to my site.
<br />
Check it out here: <a href="https://en-gb.wordpress.org/plugins/headit/">Headit</a>.
<h3>Further Reading</h3>
<ul>
<li><a target="_blank" href="https://developer.mozilla.org/en-US/docs/Web/Security">Mozilla.org - Security Headers</a></li>
<li><a target="_blank" href="https://www.owasp.org">OWASP - Free and Open Security Information</a></li>
<li><a target="_blank" href="https://scotthelme.co.uk/security-headers-alexa-top-million/">Scott Helme - Alexa Top 1 Million Sites Security Header Analysis</a></li>
<li><a target="_blank" href="https://securityheaders.io">Security Headers site checker</a></li>
<li><a target="_blank" href="https://en-gb.wordpress.org/plugins/headit/">Headit by StampyCode</a></li>
</ul>


