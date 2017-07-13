Passwords are everywhere, we use them for logging into all sorts of services, and typically we use a very small number of passwords, sometimes with small variations. In this article I explain why and how you should be using a password generator.

<h3>are my passwords strong?</h3>

Using similar passwords everywhere is bad practise - it means if an attacker gets a password for one service you use, they will likely be able to gain access to your other accounts elsewhere.

The reason you don't use lots of different passwords is because it's difficult for us humans to remember them all, especially when we're told to use special characters, numbers, and mixed case letters, and avoid using words, patterns, or repetition.

Because this is what goes into making an OK password, but there is a better way.

<h3>so what is a strong password?</h3>

To create a strong password the only real option is to use a secure password generator. These use a cryptographically-secure random number generator to create the strongest passwords possible with the available limitations, such as length, characters available, etc.

What secure passwords look like:
64 character alphanumeric password:
<code>a40vr2IStSnuSJWYwscjSgX5zKNyxKjn4v0q6mxkCbY7I2wm9FvbZumD2fMEtKL</code>
64 character alphanumeric password with symbols:
<code>aY5v[kHDRo:tET"zIarFn4£4[ZP|%YIH{hR6"xRU[6£n,baS8[tf&d5\20ZhE:2</code>

So you get the idea. Even short passwords (16 characters or less) are much more secure than any password you can actually think of, even mashing your face on the keyboard will result in reproducible patterns. If an attacker mashes their own face into a keyboard for hours and hours, not only is this hilarious for everyone watching, but chances are that they will have generated several passwords that are similar to yours. They can then add these "mashed face" passwords to a database to use in the future, to save themselves the face-pain if nothing else. (This is what password databases are for - they contain billions of passwords to use in cracking systems. I'm not covering <a href="https://en.wikipedia.org/wiki/Rainbow_table">rainbow-tables</a> here, but please feel free to read up on it in your spare time.)

<h3>why are my normal passwords not strong?</h3>

Basically it comes down to the ease at which they can be computationally guessed, given whether the attacker has a hash, hint, or other personal data that you may have used to create your password, such as patterns you've used to create passwords on other sites.

If an attacker has one of your passwords, are your other passwords sufficiently different to prevent the attacker guessing the other ones?

If you use `las4nco7h3jpjse;facebook` as your facebook password, what combinations do you think the attacker might try to use to access your gmail account?
Perhaps `las4nco7h3jpjse;gmail` and `las4nco7h3jpjse;google` would be their first two attempts?

It only takes one of your accounts' passwords to be compromised using this pattern for all of your other account passwords to become guessable.

<h3>how do i generate a strong random password?</h3>

I use a password generator that I wrote myself, which is clean and simple. It is based on principles of generating numbers with high entropy using a secure random number generator function built into Javascript, and available on most (if not all) browsers.
This extension works on Chrome and Firefox at the time of writing, and it doesn't work in Internet Explorer. 
<i>(I would appreciate it if people would tell me via comments if it works in other obscure browsers.)</i>

Drag the following button into your bookmarks bar to use this password generator, the code is also shown a bit further down the page.
<center>
<b><a href="javascript:(function(){var x='0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ',y=x+'!&quot;£$%^&*()_+-={}[]:@~;#?,./|\\',w=window.open('','','height=110,width=600,top=100,left=100,location=no,menubar=no,resizable=no,scrollbars=no,status=no,titlebar=no,toolbar=no'),d=w.document,b=d.body,a='appendChild',C='createElement',B=d[C]('button'),e=d[C]('input'),ctn='createTextNode',f=function(c,ct){var bb = d[C]('button');bb[a](d[ctn]('x'+ct));bb.onclick=function(){var f=new Uint8Array(ct),i=f.length-1,o='';w.crypto.getRandomValues(f);while(i)o+=c[f[i--]%c.length];e.value=o;e.select();};b[a](bb);},m=function(R,t){b[a](d[R](t));};b.style.fontFamily='Tahoma';b.style.background='#cfc';e.style.width=20;b.onclick=function(){e.select();};B[a](d[ctn]('password hidden for security - click to show'));B.onclick=function(){e.style.width=550;B.style.display='none';};m(ctn,'Press Ctrl-C to copy your new password ');m(C,'br');b[a](e);b[a](B);m(C,'br');f(x,16);f(x,32);f(x,64);m(ctn,' a-zA-Z0-9');m(C,'br');f(y,16);f(y,32);f(y,64);m(ctn,' with symbols');}())">Password Generator</a></b>
</center>
Clicking on it in your toolbar should result in a window like this:

<center>
[caption id="attachment_356" align="aligncenter" width="335"]<a href="/wp-content/uploads/2015/12/pwgen.png"><img src="/wp-content/uploads/2015/12/pwgen.png" alt="password generator" width="335" height="177" class="size-full wp-image-356" /></a>[/caption]
</center>

Click the buttons to generate passwords according to the length and complexity required.

<h3>but how am i supposed to remember all these passwords?</h3>

<b>You aren't.</b>

There are several browser extensions and third-party applications that enable the secure storage of passwords, such as <a href="http://keepass.info/">KeyPass</a>, <a href="https://keepersecurity.com/">Keeper</a>, and even as <a href="https://support.google.com/chrome/answer/95606?hl=en-GB">a core part of the Google Chrome browser</a>.

Many of these secure password stores are available as extensions to your existing Firefox, Chrome or IE browsers, and on mobile devices, so you can generate and submit a new password on one device, and it will be available on your other devices automatically.

<h3>but what if someone gets my password manager password?</h3>

Using a password manager does mean you typically need to remember at least one relatively secure master password, but these services do typically offer two-factor authentication, which forces anyone who attempts to log in to your password storage to also have access to your phone or mobile device, so it can send you a text message to confirm the person who owns the password storage is also in control of the registered phone number for the account.

<i><b>Google, Facebook, Twitter and others support two-factor authentication, and you should be using it whenever possible.</b></i>

<h3>the geeky bit</h3>

The bookmarklet is a block of Javascript code that executes on the current page you are browsing, but doesn't send any data to any server at all. This means the password generated only ever exists in the browser window. You have the option in the tool to generate various password sizes, with or without special characters. It is also relatively easy to change the code if you want additional buttons.

From a security perspective there are a couple of other features this password generator makes use of:
<ol>
<li>By default it obscures most of the password from view, so if your screen is being watched (either by shoulder-surfing or by a more nefarious screen watching software utility) then the password is still moderately protected. There is a 'show password' button so you can check the password looks as you might expect, but this defeats the security feature.</li>
<li>If you have a keylogger installed on the computer, the copy-paste action of the password text is not likely to be captured by the keylogger, which it would be if you had to type it in manually. Copy-pasting passwords also reduces the likelihood that you will get the password wrong, and makes you feel more comfortable using absurd-looking long passwords with lots of special characters.</li>
<li>The bookmarklet relies on no third party code, so there are no connections made to any other resource to generate or store your password, or to facilitate any component of this utility.</li>
</ol>

Here's the code for the password generator with a bit of wrapping and white-space to make it more readable.
<pre>
javascript:(function() {
    var x='0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ',
        y=x+'!"£$%^&*()_+-={}[]:@~;#?,./|\\',
        w=window.open('','',
            'height=110,width=600,top=100,left=100,location=no,menubar=no,'+
            'resizable=no,scrollbars=no,status=no,titlebar=no,toolbar=no'
        ),
        d=w.document,
        b=d.body,
        a='appendChild',
        C='createElement',
        B=d[C]('button'),
        e=d[C]('input'),
        ctn='createTextNode',
        f=function(c,ct){
            var bb = d[C]('button');
            bb[a](d[ctn]('x'+ct));
            bb.onclick=function(){
                var f=new Uint8Array(ct),
                    i=f.length-1,
                    o='';
                w.crypto.getRandomValues(f);
                while(i)o+=c[f[i--]%c.length];
                e.value=o;
                e.select();
            };
            b[a](bb);
        },
        m=function(R,t){
            b[a](d[R](t));
        };
    b.style.fontFamily='Tahoma';
    b.style.background='#cfc';
    e.style.width=20;
    b.onclick=function(){e.select();};
    B[a](d[ctn]('password hidden for security - click to show'));
    B.onclick=function(){
        e.style.width=550;
        B.style.display='none';
    };
    m(ctn,'Press Ctrl-C to copy your new password ');
    m(C,'br');
    b[a](e);
    b[a](B);
    m(C,'br');
    f(x,16);
    f(x,32);
    f(x,64);
    m(ctn,' a-zA-Z0-9');
    m(C,'br');
    f(y,16);
    f(y,32);
    f(y,64);
    m(ctn,' with symbols');
}());
</pre>

The code is minified somewhat, as URLs are limited to 4096 bytes - this code is way below that anyway, but it makes it look more exciting if nothing else.


<h3>what is the difference between random and secure-random?</h3>

I'm glad you asked.

Generating random numbers with normal 'random' functions aren't considered secure because although the output of these functions appears random, and the randomness usually stands up to basic statistical analysis, standard random number generators are only random enough to be used for non-security related scenarios. Normal random number generators should only be used where the numbers generated aren't predictable or reproducible.

The last few digits of the current system time in microseconds might be used as a simple generator, but these numbers are selected at intervals from the same source, with little external influence, so reproducing or predicting the next number in the series would be a lot easier than if the numbers were produced by a more secure system. So `current microseconds mod 6` is great for generating dice rolls, but not so great for generating passwords.

To generate random numbers in Javascript that are strong, we use `window.crypto.getRandomValues()`. This is a built-in security component of modern web browsers. If we just need a normal 'random' number, we use `Math.random()` instead, which is quicker, but not secure.


<h3>further reading</h3>
<ul>
<li><a href="https://www.schneier.com/blog/archives/2013/06/a_really_good_a.html">Schneier on Security - A Really Good Article on How Easy it Is to Crack Passwords</a></li>
<li><a href="https://en.wikipedia.org/wiki/Password_strength">Wikipedia - Password Strength</a></li>
<li><a href="http://lifehacker.com/5937303/your-clever-password-tricks-arent-protecting-you-from-todays-hackers">Lifehacker: Your Clever Password Tricks Aren't Protecting You from Today's Hackers</a></li>
<li><a href="https://crackstation.net/">CrackStation.net</a> - a free online password cracker</a></li>
<li><a href="http://www.defaultpassword.com/">DefaultPassword.com</a> - list of default device passwords</li>
</ul>
