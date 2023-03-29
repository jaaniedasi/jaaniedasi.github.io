---
layout: post
title: "Soccer @ HTB"
date: "2023-03-29 21:30: +0200"
author: jne
excerpt: Notes on HackTheBox's Soccer box
---
Hi

This is not a full walkthrough, just some things I noticed that felt interesting enough to share and that I hadn't seen in other guides for this box.

After conducting directory fuzzing, exploiting default credentials on a `tiny` fileserver, and exploiting PHP code injection, you end up discovering a web application with a blind-ish SQL injection vulnerability. While most people would simply set sqlmap loose on the endpoint (as did I, tbh), I wanted to try extracting the information to a file that I could browse to since I had the ability to read files from the `/var/www/html/tiny/uploads` path via the `tiny` fileserver.

There were (at least) two ways to decide if you SQL injection worked or not: via sleeps and error messages (which you can confirm when you gain a foothold on the server). Success resulted in `Ticket Exists.` and failure resulted in `Ticket Doesn't Exist.` For example, using `55259 union select 1,2,3` would give me `Ticket Exists.` and `55259 union (select 1,2)` would give me `Ticket Doesn't Exist.`

Next, I tried writing the result to a file in the world writable directory `/var/www/html/tiny/uploads` by using `5259 union select 1,2,3 into outfile '/var/www/html/tiny/uploads/foo.txt';` but this resulted in a failure. I also attempted to upload to the `/tmp` directory, but the result was the same. This suggested that the `mysql` user did not have privileges to write files altogether.

Later, when I rooted the box, I attempted to do the same via the command line with the hardcoded credentials in the application code. I received an error message that confirmed my suspicion:

`ERROR 1227 (42000): Access denied; you need (at least one of) the FILE privilege(s) for this operation.`

When I attempted to execute the same statement as the `root` user in MySQL, I received another error message: 

`ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement.`

Seems that the appropriate security measures where in place.

That's all. Take care!
