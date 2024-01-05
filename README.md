# restful-booker
restful booker API tests using jmeter
API's reference - https://restful-booker.herokuapp.com/apidoc/index.html
Type of API's - GET/ PATCH/ POST/ DELETE

# How to run tests using pre defined npm command and reporting option:
git clone https://github.com/phanipatsa/restful-booker.git
git pull
npm run jmeter_test

# Reports 
1. Logs, reports will be generated under logs folder.
2. Click on the folder for specific execution timestamp and open index.html to review the report.

OR 

# Execute via Command line and override the parameters as needed for control the execution:

./apache-jmeter-5.4.1/bin/jmeter -JPROTOCOL=http -JHOST=restful-booker.herokuapp.com -JNUM_USERS=2 -JUSERS_RAMP_UP_SEC=1 -JREQ_COUNT=2 -JREQ_DELAY_MS=200 -JREQ_COUNT_INCREMENT=1 -n -t booker_api.jmx

# How to pass variables and override jmeter parameters
Supported variables with their default values. Please read this guide first.
https://www.redline13.com/blog/2018/05/guide-jmeter-thread-groups/

NUM_USERS (1) - Number of Threads
USERS_RAMP_UP_SEC (1) - Ramp-up Period in seconds
PROTOCOL (http)
HOST (demo.npm.loc)
PORT (9010)
REQ_COUNT (2) - # of requests to send per user / thread
REQ_DELAY_MS (3000)
REQ_DELAY_OFFSET_MS (400)
REQ_COUNT_INCREMENT (1) - used for dynamic request generation to control request level caching
Example command to override defaults

jmeter -JNUM_USERS=1 -JUSERS_RAMP_UP_SEC=1 -JREQ_COUNT=5 -n -t npm_perf.jmx -l jmeter_log.jtl -j jmeter_log.log -e -o jmeter_dashboard
patterns(NUM_USERS / pattern)(REQ_COUNT / NUM_USERS)

Above example: 215 = 10 Rate requests + 1 Auth Request = 11 requests

General Guide / jmx file details
NOTE: Counter's Start and End value is currently hardcoded such as N = End - Start = 100.

For each pattern there are N number of unique requests that can be made.
This is achieved by making the request dynamic where a numeric range column (amt) is incremented.
This makes the request unique for caching purposes. Because caching works based on the uniqueness of the request body.

REQ_COUNT_INCREMENT can be used to tweak the amount of requests tha would get cached.

Now, if starting - end values for range column is 1-100 and REQ_COUNT_INCREMENT = 1, then there can be 100 unique requests possible.

Let's take this example
patterns(NUM_USERS / pattern)(REQ_COUNT / NUM_USERS)
225 = 20 Rate requests

2 concurrent systems starting at the same time. Each system is responsible for single usage pattern.
Each system has 2 users.
Each user will make 5 requests.

In the above example: 20 unique rate requests would get cached.

What if we adjust the REQ_COUNT_INCREMENT = 5?

In the above example: 20 / 5 = 4 unique rate requests would get cached.

Why is this important?

We want to try scenarios where variance of unique requests made / cached is a percentage of total requests made.

Higher the cached requests, lower the increment value, higher the response time.
Loser the cached requests, higher the increment value, lower the response time.

4 unique requests out of total 20 requests made would yield a much better SLA than all 20 unique requests made.

Control throughput
The main variables involved are NUM_USERS and USERS_RAMP_UP_SEC

And to further fine-tune: use REQ_DELAY_MS.
Don't bother much with REQ_DELAY_OFFSET_MS - it is just an offset to control some randomness in the delay.

eg. NUM_USERS = 10 and USERS_RAMP_UP_SEC = 10
It will take 10 seconds to ramp up all 10 users
- new user joins every second - more requests per second - increased throughput!

eg. NUM_USERS = 10 and USERS_RAMP_UP_SEC = 30

It will take 30 seconds to bring up all 10 users
- so that is 30 / 10 = new user joins every 3 seconds - decreased throughput!
- it is possible that users that have finished their work early are no longer active by the time the last users join
- adjust REQ_DELAY_MS and USERS_RAMP_UP_SEC accordingly if you want to make sure all users are still actively making calls by the time every user has joined the party

eg. NUM_USERS = 10 and USERS_RAMP_UP_SEC = 5