### Backend Project 4

| Group 8          |
| ---------------  |
| Ajinkya Bhalerao |
| Joshua Popp      |
| Nolan O'donnell  |
| Akhil Chirra     |

##### HOW TO RUN THE PROJECT

```c
      // NOTE: Please make sure you have all the dependencies. 
```
1. Copy the contents of our [nginx config file](https://github.com/Ajinkya-Bhalerao/cpsc449-project4) into a new file within `/etc/nginx/sites-enabled` called `default`. Assuming the nginx service is already running, restart the service using `sudo service nginx restart`.

Nginx Config:

```
server {
    listen 80;
    listen [::]:80;

    server_name tuffix-vm;

    location /registration {
        proxy_pass http://127.0.0.1:5000/registration;
    }

    location /newgame {
        auth_request /auth;
        proxy_pass http://gameservice;
    }

    location /addguess {
            auth_request /auth;
            proxy_pass http://gameservice;
    }

    location /allgames {
            auth_request /auth;
            proxy_pass http://gameservice;
    }

    location /onegame {
        auth_request /auth;
        proxy_pass http://gameservice;
    }


    location = /auth {
           internal;
           proxy_pass http://127.0.0.1:5000/login;
    }
    
    location /webhook {
        proxy_pass http://gameservice/webhook;
    }

}

upstream gameservice {
    server 127.0.0.1:5100;
    server 127.0.0.1:5200;
    server 127.0.0.1:5300;
}
```

2. Initialize the folder stucture within the project folder and install redis if you have not previously installed

   ```c
      // step 1. give the script permissions to execute
      chmod +x ./bin/folder.sh

      // step 2. run the script
      ./bin/folder.sh
   ```

   ```c
      // step 3. install redis
      pip install redis
   ```

3. Start the API

   ```c
      foreman start
      // NOTE: There will be an sql error just continue doing the next steps
   ```

4. Initialize the databases within the project folder

   ```c
      // step 1. give the script permissions to execute
      chmod +x ./bin/init.sh

      // step 2. run the script
      ./bin/init.sh
   ```

5. Populate the word databases

   ```c
      python3 dbpop.py
   ```

6. Restart the server to resolve the error


    ```c
      foreman start
      // NOTE: Do this to register the callbackUrl
      // sometimes foreman will not start so please restart the system and then run foreman start
     ```

7. Test all the endpoints using httpie
   - user
      - register account: `http POST http://tuffix-vm/registration username=Ajinkya password=Aj123`

       Sample Output:
       ```
      {
         "id": 1,
         "password": "Ajinkya",
         "username": "Aj123"
      }
      ```
     - login {Not accesible}: 'http --auth Ajinkya:Aj123 GET http://tuffix-vm/login'
     Sample Output:
     ```
      HTTP/1.1 404 Not Found
      Connection: keep-alive
      Content-Encoding: gzip
      Content-Type: text/html
      Date: Fri, 18 Nov 2022 21:04:31 GMT
      Server: nginx/1.18.0 (Ubuntu)
      Transfer-Encoding: chunked

      <html>
      <head><title>404 Not Found</title></head>
      <body>
      <center><h1>404 Not Found</h1></center>
      <hr><center>nginx/1.18.0 (Ubuntu)</center>
      </body>
      </html>
      ```
   - game

      - create a new game: `http --auth Ajinkya:Aj123 POST http://tuffix-vm/newgame`

      Sample Output:
      ```
      'http --auth Ajinkya:Aj123 POST http://tuffix-vm/newgame'
      {
         "answerid": 577,
         "gameid": "559ddf5a-7e6a-11ed-a301-a93a28a3300d",
         "username": "Ajinkya"
      }
      ```
      Note - this will return a `gameid`
    - add a guess: `http --auth Ajinkya:Aj123 PUT http://tuffix-vm/addguess gameid=559ddf5a-7e6a-11ed-a301-a93a28a3300d word=money`

    Sample Output:
    ```
      http --auth Ajinkya:Aj123 PUT http://tuffix-vm/addguess gameid="b0039f36-6784-11ed-ba4a-615e339a8400" word="amigo"
     {
        "Accuracy": "XXXO✓",
        "guessedWord": "money"
     }
     ```
    - display your active games: `http --auth Ajinkya:Aj123 GET http://tuffix-vm/allgames`

    Sample Output:
    ```
      http --auth Ajinkya:Aj123 GET http://tuffix-vm/allgames
      [
         {
            "gameid": "559ddf5a-7e6a-11ed-a301-a93a28a3300d",
            "gstate": "In-progress",
            "guesses": 1
         }
      ]
      ```
    - display the game status and stats for one game: `http --auth Ajinkya:Aj123 GET http://tuffix-vm/onegame?id=gameid`
       - example: `.../onegame?id=559ddf5a-7e6a-11ed-a301-a93a28a3300d`
    Sample Output:
    ```
      http --auth Ajinkya:Aj123 GET http://tuffix-vm/onegame?id="559ddf5a-7e6a-11ed-a301-a93a28a3300d"
      [
         {
             "gameid": "559ddf5a-7e6a-11ed-a301-a93a28a3300d",
            "gstate": "In-progress",
            "guesses": 1
          },
          {
             "accuracy": "XXXO✓",
             "guessedword": "money"
          }
      ]
      ```
8. Test leaderboard using http docs: http//127.0.0.1:5400/docs

    GET/leaderboard
    Sample output:
    ```
      ('Ajinkya', 6.0)
      ('test1', 3.0)
      ('myName', 2.0)
      ('Avni', 2.0)
    ```

9. To start crontab run the following command:

   $ crontab -e 
   
   enter the script in the file which opens (example : /tmp/crontab.81GywU/crontab):

   */10 * * * * run-one rq reque -all --queue default
   
