---
title: Target "_blank" vulnerability
updated: 2016-10-01 18:14
---


External Links being opened up in a new tab now all have noopener / noreferrer due to the security issues with window.opener.

[via Google](https://sites.google.com/site/bughunteruniversity/nonvuln/phishing-with-window-opener)
> Over the past few months, we have received a significant number of reports about a "reverse tabnabbing" attack, where a foreground tab opened from a trusted application, and displaying an attacker-controlled website, uses window.opener.location.assign() to replace the background tab with a malicious document. Of course, this action also changes the address bar of the background tab - but the attacker hopes that the victim will be less attentive and will blindly enter their password or other sensitive information when returning to the background task.

> Unfortunately, we believe that this class of attacks is inherent to the current design of web browsers and can't be meaningfully mitigated by any single website; in particular, clobbering the window.opener property limits one of the vectors, but still makes it easy to exploit the remaining ones. 

```js
// External Links = New Tab
function addTargetBlank() {
	var internal = location.host.replace("www.", "");
    internal = new RegExp(internal, "i");
      
	var a = document.getElementsByTagName('a');
	for (var i = 0; i < a.length; i++) {
		var href = a[i].host; 
		if( !internal.test(href) ) {
			// https://dev.to/ben/the-targetblank-vulnerability-by-example
			// https://sites.google.com/site/bughunteruniversity/nonvuln/phishing-with-window-opener
			a[i].setAttribute('rel', 'noopener noreferrer'); 
			a[i].setAttribute('target', '_blank'); 
		}
	}
};
```