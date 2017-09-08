---
layout: post
title:  "Hunting for SQL Injection (SQLi) attacks in Windows IIS Logs"
date: 2017-08-30 11:13:00 CST
author: jd
categories: [splunk windows iis sql sqli injection search]
published: true
comments: true
---

![sql-injection](/images/sql-injection.jpg)

I'll be the first to admit that i don't like Windows or anything to do with SQL. Fortunately, that does not exempt me from having to build use cases and Splunk searches for my "customers". It's fortunate because there is no progress in the comfort zone. Outside of your standard means of mitigating SQLi attacks, such as, using stored procedures, input sanitation, etc. I don't know much else about SQLi attacks.

<!--more-->

So, while working on an SQL Injection (SQLi) use case, one of the guys said there are "filtering rules" that are used to restrict what input can be passed to the database from the web front end. Okay, good to know! According to the Microsoft [docs](https://docs.microsoft.com/en-us/iis/configuration/system.webserver/security/requestfiltering/filteringrules/)  those terms, in XML format, are:

```xml
<requestFiltering>
   <filteringRules>
      <filteringRule name="SQLInjection" scanUrl="false" scanQueryString="true">
         <appliesTo>
            <clear />
            <add fileExtension=".asp" />
            <add fileExtension=".aspx" />
            <add fileExtension=".php" />
         </appliesTo>
         <denyStrings>
            <clear />
            <add string="--" />
            <add string=";" />
            <add string="/*" />
            <add string="@" />
            <add string="char" />
            <add string="alter" />
            <add string="begin" />
            <add string="cast" />
            <add string="create" />
            <add string="cursor" />
            <add string="declare" />
            <add string="delete" />
            <add string="drop" />
            <add string="end" />
            <add string="exec" />
            <add string="fetch" />
            <add string="insert" />
            <add string="kill" />
            <add string="open" />
            <add string="select" />
            <add string="sys" />
            <add string="table" />
            <add string="update" />
         </denyStrings>
         <scanHeaders>
            <clear />
         </scanHeaders>
      </filteringRule>
   </filteringRules>
</requestFiltering>
```

Armed with some new found knowledge, I came up with a quick search that will identify the `cs_uri_query` that contain the strings Microsoft believes should be filtered out.

I believe the search can and should be tuned up a bit but amid the false positives, we've seen some very interesting queries logged. Below you'll find the search I'm currently using:

```
<SPL input for your iis logs>
| regex cs_uri_query="(?i)(?:--|\;|\/\*|\@|\@\@version|char|alter|begin|cast|create|cursor|declare|delete|drop|end|exec|fetch|insert|kill|open|select|sys|table|update)"
| stats count by host c_ip cs_uri_stem cs_uri_query
| rex field=cs_uri_query "(?i)(?<suspect>--|\;|\/\*|\@|\@\@version|char|alter|begin|cast|create|cursor|declare|delete|drop|end|exec|fetch|insert|kill|open|select|sys|table|update)" max_match=0
```

NOTE: The commenting I use below is not supported in SPL, however, it is quicker for me to write. Please base your search on the SPL above.

```
# SPL required to search for IIS logs
<SPL input for your iis logs

# Returns events where "cs_uri_query" matches any of the strings based on list above. The regex matches in a case-insensitive manner.
| regex cs_uri_query="(?i)(?:--|\;|\/*|\@|\@\@version|char|alter|begin|cast|create|cursor|declare|delete|drop|end|exec|fetch|insert|kill|open|select|sys|table|update)"

# Aggregate results and return back the limited data set to the Search Head.
| stats count by host c_ip cs_uri_stem cs_uri_query

# Creates a multi-vaule field with matched terms extracted out. This helps to quickly identify what caused the event to match the "regex" command above.
| rex field=cs_uri_query "(?i)(?<suspect>--
|\;|\/*|\@|\@\@version|char|alter|begin|cast|create|cursor|declare|delete|drop|end|exec|fetch|insert|kill|open|select|sys|table|update)" max_match=0
```

Well, there you have it. The beginnings of ONE way to locate, close with, and destroy the enemy.

Enjoy!
