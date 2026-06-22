+++
title = 'SnipCart on Hugo: Setting up a Local Test Development Environment on Windows 11 and Git Bash'
date = 2026-06-22T12:21:02-07:00
tags = ["hugo", "snipcart", "test-environment", "development"]
draft = false
+++


> WARNING: This method should be done only with a public key for testing purposes. The public key WILL show up in the client-end code. 

Let's say you want to test your Hugo site with SnipCart in a test development environment.

Here is how you can do that. 
## Prerequisites
1. [Git Bash](https://gitforwindows.org/)
2. [Hugo](https://gohugo.io/installation/windows/)
3. [A Hugo projec](https://gohugo.io/getting-started/quick-start/)
4. [A SnipCart account](https://snipcart.com/)

Go to each link and follow the instructions for installation if you have not done so already.
## Set your Test API Key as an Environment Variable
Below are listed two ways to set your test API key as an environment variable in your Windows system:

1. Git Bash: Temporarily Set Your Test API Key as an Environment Variable
2. PowerShell: Permanently Set Your Test API Key as an Environment Variable
### 1. Git Bash: Temporarily Set Your Test API Key as an Environment Variable

> Note: each time you start a new Git Bash session, you need to follow these steps:

1. Open at the root of project. 
2. With the export command, set your API key, where `<your-api-key>` is the test API key:
	1. Note: Start the API key var name with `HUGO`, so that the local Hugo server can recognize the var name (see next step). 
```shell
export HUGO_PUBLIC_TEST_API_KEY=<your-api-key>
```

3. Echo your API key to make sure it set:
```bash 
$ echo $HUGO_PUBLIC_TEST_API_KEY
<your-api-key>
```

[Source](https://discourse.gohugo.io/t/keeping-api-keys-secret-on-github-using-a-env-file/25283/8)

### 2. PowerShell: Permanently Set Your Test API Key as an Environment Variable

> Note: If you want to set an environment variable to persist across multiple sessions, follow these steps instead:

1. Run PowerShell as Administrator
2. Input this command with your variable name and your key, where `<your-api-key>` is the test API key:
```shell
[System.Environment]::SetEnvironmentVariable('HUGO_PUBLIC_TEST_API_KEY', '<your-api-key>', 'Machine')
```
3. Start a new PowerShell session
4. Echo your API key to make sure it set:
```shell
echo $Env:HUGO_PUBLIC_TEST_API_KEY
<your-api-key>
```
5. Restart Git Bash
6. Make sure it is set in Git Bash:
```shell
echo $HUGO_PUBLIC_TEST_API_KEY
<your-api-key>
```

[Source](https://superuser.com/questions/1729958/how-do-i-permanently-set-a-system-variable-system-environmentsetenvironment)
## Set your Newly Created Environment Variable in Hugo

### hugo.toml: 
We want to make sure the local Hugo server identifies the correct key through regular expressions. In the below code with the getenv array, Hugo attempts to find an environment variable that begins with HUGO, which is what we want. 

This is default behavior in Hugo, but add it to your config file in case:

```toml
[security]
  enableInlineShortcodes = true
  [security.funcs]
    getenv = ['^HUGO_', '^CI$']
```
[Source](https://gohugo.io/configuration/security/)
### baseof.html:
You will see this code given in the installation documentation and tell you to put it after the body tag. 
```html
    <script>
        window.SnipcartSettings = {
            publicApiKey: '{{ getenv "HUGO_PUBLIC_TEST_API_KEY" }}',
            loadStrategy: "on-user-interaction",
        };
        (function(){var c,d;(d=(c=window.SnipcartSettings).version)!=null||(c.version="3.0");var s,S;(S=(s=window.SnipcartSettings).timeoutDuration)!=null||(s.timeoutDuration=2750);var l,p;(p=(l=window.SnipcartSettings).domain)!=null||(l.domain="cdn.snipcart.com");var w,u;(u=(w=window.SnipcartSettings).protocol)!=null||(w.protocol="https");var m,g;(g=(m=window.SnipcartSettings).loadCSS)!=null||(m.loadCSS=!0);var y=window.SnipcartSettings.version.includes("v3.0.0-ci")||window.SnipcartSettings.version!="3.0"&&window.SnipcartSettings.version.localeCompare("3.4.0",void 0,{numeric:!0,sensitivity:"base"})===-1,f=["focus","mouseover","touchmove","scroll","keydown"];window.LoadSnipcart=o;document.readyState==="loading"?document.addEventListener("DOMContentLoaded",r):r();function r(){window.SnipcartSettings.loadStrategy?window.SnipcartSettings.loadStrategy==="on-user-interaction"&&(f.forEach(function(t){return document.addEventListener(t,o)}),setTimeout(o,window.SnipcartSettings.timeoutDuration)):o()}var a=!1;function o(){if(a)return;a=!0;let t=document.getElementsByTagName("head")[0],n=document.querySelector("#snipcart"),i=document.querySelector('src[src^="'.concat(window.SnipcartSettings.protocol,"://").concat(window.SnipcartSettings.domain,'"][src$="snipcart.js"]')),e=document.querySelector('link[href^="'.concat(window.SnipcartSettings.protocol,"://").concat(window.SnipcartSettings.domain,'"][href$="snipcart.css"]'));n||(n=document.createElement("div"),n.id="snipcart",n.setAttribute("hidden","true"),document.body.appendChild(n)),h(n),i||(i=document.createElement("script"),i.src="".concat(window.SnipcartSettings.protocol,"://").concat(window.SnipcartSettings.domain,"/themes/v").concat(window.SnipcartSettings.version,"/default/snipcart.js"),i.async=!0,t.appendChild(i)),!e&&window.SnipcartSettings.loadCSS&&(e=document.createElement("link"),e.rel="stylesheet",e.type="text/css",e.href="".concat(window.SnipcartSettings.protocol,"://").concat(window.SnipcartSettings.domain,"/themes/v").concat(window.SnipcartSettings.version,"/default/snipcart.css"),t.prepend(e)),f.forEach(function(v){return document.removeEventListener(v,o)})}function h(t){!y||(t.dataset.apiKey=window.SnipcartSettings.publicApiKey,window.SnipcartSettings.addProductBehavior&&(t.dataset.configAddProductBehavior=window.SnipcartSettings.addProductBehavior),window.SnipcartSettings.modalStyle&&(t.dataset.configModalStyle=window.SnipcartSettings.modalStyle),window.SnipcartSettings.currency&&(t.dataset.currency=window.SnipcartSettings.currency),window.SnipcartSettings.templatesUrl&&(t.dataset.templatesUrl=window.SnipcartSettings.templatesUrl))}})();
    </script>
```
[Source](https://docs.snipcart.com/v3/setup/installation)

However, this code prevents the API key from properly loading and gives a CORS error when serving:
```Shell
Access to fetch at 'https://cdn.snipcart.com/themes/v3.9.0/l10n/en-us.json' from origin 'http://localhost:1313' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

### baseof.html
Ignore the code on the installation site:
```html
	 <script>
        window.SnipcartSettings = {
            publicApiKey: '{{ getenv "HUGO_PUBLIC_TEST_API_KEY" }}',
            loadStrategy: "on-user-interaction",
        };
         (function(){var c,d;(d= etc...
	</script>
```

And dynamically load the API key directly in to the HTML async script tag using the getenv keyword:
```html
    <link rel="stylesheet" href="https://cdn.snipcart.com/themes/v3.0.0/default/snipcart.css">

    <script async type="text/javascript" id="snipcart" src="https://cdn.snipcart.com/themes/v3.0.0/default/snipcart.js" data-api-key="{{ getenv `HUGO_PUBLIC_TEST_API_KEY` }}"></script>
```
This goes after the body tag

[Source](https://github.com/snipcart/snipcart-hugo-integration/blob/e541b772283ba4587ad6c04ec20247281cd4fc66/layouts/partials/footer.html)

### Start your Hugo Server

In Git Bash, start your Hugo server by inputting `hugo serve` and navigate to `localhost:1313` in your browser of choice.

### Some Common Errors and Fixes
An error you may receive if you don't `hugo serve` in Git Bash:
```shell
snipcart.js:1 Uncaught (in promise) Error: A public Test or Live API key is required. You can find it in your Snipcart dashboard (https://app.snipcart.com/dashboard/account/credentials).
```

Another error message you might see if you forget to set your environment variable following the steps above
```shell
Access to fetch at 'https://cdn.snipcart.com/themes/v3.9.0/l10n/en-us.json' from origin 'http://localhost:1313' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
snipcart.js:46  GET https://cdn.snipcart.com/themes/v3.9.0/l10n/en-us.json net::ERR_FAILED 404 (Not Found)
```
### That's It!
If you have no errors in your dev tools console, you are good to go!

Some additional checks you can make for peace of mind:
1. Go to the networks tab in your dev tools and you should see a status code 200 next to your Snipcart requests.
2. Check to see if your public API key is being correctly loaded in sources.