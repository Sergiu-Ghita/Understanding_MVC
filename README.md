# Twitter Streams Example in Erlang

An Erlang/OTP Application that connects to Twitters public streaming API https://stream.twitter.com/1.1/statuses/filter.json
Extended to support API calls and parsing

## For deployment, please follow step by step through the following: Environment configuration, Application configuration, Dependencies, Compiling and Running.

## Environment configuration

For windows 8.1 64 bit:

- Erlang 6.0 is required (otp_win64_17.0 @ https://www.erlang-solutions.com/downloads/download-erlang-otp)
- Rebar is required (https://github.com/basho/rebar) - You might need Microsoft Visual Studio to build rebar
- An internet connection is required

## Application Configuration

Part 1.
Edit src/twitter_parser_server.erl and use the following as an example:

The following may be obtained from your twitter developer account

-define(CONSUMER_KEY, "71YXI8vKwx47895hWJsGFukSg").
-define(CONSUMER_SECRET, "Wa7kcea38j40gF3ZYfv8uBkqk6as30FWKuXwuhI7HpRwBrUO6w").
-define(ACCESS_TOKEN, "2970209080-bmMPRR2bkLU4CBN3eL03DIiuQiuhMT0HX3PuWqb").
-define(ACCESS_TOKEN_SECRET, "sB7xXwgoFUFz3RpjnLjAwpwo4eDkkdCvSjoeHEeAYyRWi").

The following indicates the web server's address for sending parsed tweets
-define(NODEJS_TWEET_POST_URL, "http://192.168.1.34:8080/newTweet").

The following indicates the listening port for receiving API calls (e.g. addition/removal of articlesets)
-define(LISTENING_PORT, 8081).

Part 2.
The folder \config contains configuration files for the distributed nodes (see book on otp distributed). Modify these files, and change the host to your own: e.g. from  c@AlottaFagina you change it to c@YourHostStringThatYourPCHas

Use inet:gethostname/0 to determine the hostname. (http://www.erlang.org/doc/man/inet.html#gethostname-0)

Part 3.
The libs folder contains two modules: keyword_registry and tweet_parser. In each module folder, if missing, create an empty folder ebin. For each module, copy and paste the file tweet_parser/src/tweet_parser.app (or keyword_registry/src/keyword_registry.app) to tweet_parser/ebin folder (or keyword_registry/ebin).

## Dependencies

The only required external dependencies are: oauth and json. External dependencies are specified in the rebar.config file.

To download the dependencies, run in the project root directory:
    
    rebar get-deps

## Compiling

From the project root directory run

    rebar compile

From each of the two module folders located in libs run (The libraries need to be compiled as well)
    
    rebar compile

## Running

For running the application on three nodes, run in the project's root directory

First shell:

    erl -sname a -config config/a -s ssl -s inets -s crypto -pa ebin deps/oauth/ebin deps/json/ebin deps/jsx/ebin deps/jsonpointer/ebin libs/tweet_parser/ebin libs/keyword_registry/ebin

Second shell:

    erl -sname b -config config/b -s ssl -s inets -s crypto -pa ebin deps/oauth/ebin deps/json/ebin deps/jsx/ebin deps/jsonpointer/ebin libs/tweet_parser/ebin libs/keyword_registry/ebin

Third shell:

    erl -sname b -config config/b -s ssl -s inets -s crypto -pa ebin deps/oauth/ebin deps/json/ebin deps/jsx/ebin deps/jsonpointer/ebin libs/tweet_parser/ebin libs/keyword_registry/ebin


Once the shells have been unlocked run on each shell:

    application:ensure_all_started(twitter_parser) 

Notice: The nodes will hang until you execute all the three commands. Otherwise in 30 seconds you will get timed-out.


## Regarding JSON format

To add an article or several send JSON following this format:

```json
{
    "articleSet": [
        {
            "article": "8028s92_mongoDBid_Whatever",
            "keywords": ["Doed"]
        } , 
        {
            "article": "Article6",
            "keywords": ["Smith", "This is a string"]
        }
    ]
}
```
To remove an article, you may only request one removal at a time, use the following format:
```json
{
    "removeArticle" : "Article6"
}
```
You get response codes accordingly (hopefully :D). There is no confirmation for replacing or error non existent article (youll get an ok even though the article was not present).

----

The format with which a parsed tweet is sent to nodejs:

**NOTICE MATCHES IS AN ARRAY, you'll have one or more elements**

```json   
{

    "author" : "Author Name",
    "content" : "The text in the tweet",
    "date" : "the date string twitter format",
    "matches" : [
        {
            "article" : "Article6" ,
            "keywords" : ["test","123"]
        }
    ]

}

```

## Notes:

If you decide to use bson for some reason, try tag v0.2 if it gives you an error. Make sure you have {'bson', ".*", {git, "https://github.com/comtihon/bson-erlang", {tag, "v0.2"}}} on the top of your list in your main rebar.config

---

After downloading pbkdf2 go into deps/pbkdf2/rebar.config and remove the following

%% Ensure the ebin directory exists before we try to put files there.
{pre_hooks, [
    {compile, "mkdir -p ebin"}
]}.

When compiling i was getting this error under Windows 8.1, so that should fix it:
A subdirectory or file ebin already exists.
Error occurred while processing: ebin.
ERROR: Command [compile] failed!

## Credits

Written and extended by team TL;DR, based off Jeremy Howard's example @ https://github.com/fishoutawata

Martin Logan, Eric Merritt and Richard Carlsson for their book **Erlang and OTP in Action**

Tim Fletcher for his OAuth module for Erlang **https://github.com/tim/erlang-oauth**

## License

MIT
