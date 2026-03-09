Rate limiting is a strategy for limiting network traffic. It puts a cap on how often someone can repeat an action within a certain timeframe – for instance, trying to log in to an account. It can reduce strain on web servers.

#### What kinds of attacks are stopped by rate limiting?
Rate limiting is often employed to stop malicious actors from negatively impacting a website or application. It can help mitigate:
* Brute force attacks
* DoS and DDoS attacks
* Web scraping
* API overuse

#### How does rate limiting works?
Rate limiting runs within an application, rather than running on the web server itself. Typically, rate limiting is based on tracking the IP addresses that requests are coming from, and tracking how much time elapses between each request.

A rate limiting solution measures the amount of time between each request from each IP address, and also measures the number of requests within a specified timeframe. If there are too many requests from a single IP within the given timeframe, the rate limiting solution will not fulfil the IP address's requests for a certain amount of time.

#### How does rate limiting work with user login?
Users may find themselves locked out of an account if they unsuccessfully attempt to log in too many times in a short amount of time. This occurs when a website has login rate limiting in place.

This precaution exists, not to frustrate users who have forgotten their passwords, but to block [brute force attacks](https://www.cloudflare.com/learning/bots/brute-force-attack/) in which a bot tries thousands of different passwords in order to guess the correct one and break into the account. If a bot can only make 3 or 4 login attempts an hour, then such an attack is statistically unlikely to be successful.

Rate limiting on a login page can be applied according to the IP address of the user trying to log in, or according to the user's username. Ideally it would use a combination of the two, because:

- If rate limiting is only applied by IP address, brute force attackers could bypass this by attempting logins from multiple IP addresses (perhaps by using a [botnet](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-botnet/)).
- If it's only done by username, any attacker that has a list of known usernames can try a variety of commonly used passwords with those usernames and is likely to successfully break into at least a few accounts, all from the same IP address.

Because rate limiting is necessary to prevent these brute force attacks, users who can't remember their passwords may be rate limited along with malicious bots. Users will likely see a "too many login attempts" message of some sort and be prompted to try again within a specified timeframe, or be advised that they are locked out of their accounts altogether.

#### Most common rate limiting algorithms:
##### 1. Fixed Window 
A set number of requests can be made within a predefined time window. Requests increment a counter that’s reset to zero at the start of each window.
**Pros:**
* Simple to implement
* Predictable for users
**Cons:**
- Allows bursts up to 2x the `limit` when requests begin near the end of a window
**Real-world example:**
- GitHub’s API uses a fixed window rate limiter with `limit = 5000`, `windowDuration = 1h`, and `windowStart` set to the start of each wall clock hour, allowing users 5,000 requests per hour.

**A problem with 24 hour window**
```text
If we have a rate limiter which reset every day at midnight, but midnight according to which time zone? One can use UTC, but a user in different time zone can try to request just after midnight and will still see he is blocked. A potential solution can be to use time zone according to users(where they are based). But in this case user can change their time zone manually to gain additional requests. Or if the user is travelling they can gain or lose number of requests according to the direction they are travelling. Suppose they are travelling from east to west they will have more requests allowed, and if they are travelling from west to east they will have less number of requests available to them.
```

**Fixed window with user-defined start**
Instead of fixing the start times to a set interval, each window can be created at the time of the user’s first request within that window. With this approach, it’s especially important to show users the time remaining until the next window once they’re limited since there’s no set time that aligns each window.

Sources:
[Cloudflare](https://www.cloudflare.com/en-gb/learning/bots/what-is-rate-limiting/)
[Smudge](https://smudge.ai/blog/ratelimit-algorithms)