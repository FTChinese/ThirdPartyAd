# Serve DoubleClick in China

Nowadays, online publishers usually don't serve ads, they serve HTML/JavaScript which are provided by third parties such as DoubleClick. Like this: 

    <SCRIPT language='JavaScript1.1' SRC="http://ad.doubleclick.net/ddm/adj/N5324.139612.6251545501621/B9229605.1252916080;sz=300x600;ord=[timestamp]?"></SCRIPT>

In China, this creates a huge problem for publisher, because DoubleClick can only successfully deliver to around 70% of users in China. This means when an advertiser books 100 impressions on publishers' own ad system, only around 70 impressions gets recorded in DoubleClick. 

When we called DoubleClick China they wouldn't admit this. But we have very solid proof. 

## Measuring DoubleClick's Success Rate
We use Google Analytics to measure DoubleClick's success rate. Here are the steps: 

### Load Start
Before the DoubleClick code, we send a request to Google Analytics. This gets 26,508 in one day. 

### Load Success
But how do we measure success? 

First, we cannot make any change to the code provided by DoubleClick. 

Second, there isn't be any JavaScript events to listen to because we have absolutely no control or idea what goes into each third party code. 

So there's only one way to measure success, by checking the ad DOM. We write our code to check every second if the ad DOM has either at least one image, iframe, object or video. If the check comes back as success within 8 seconds, we send a request to Google Analytics to record a success. 

We also try to access a fallback image provided in the DoubleClick's noscript tag, if the script tag doesn't come back in 8 seconds. In reality, we found out if script tag fails, fallback image almost always fails too. 

In the same example, we recorded 19,567 successes, a success rate of 73.8%. 

Please check out [Here](https://github.com/FTChinese/ThirdPartyAd/blob/master/index.html) for the full code. 

## What can we do about it? 
### Avoid Underdelivery
If we know DoubleClick will only delivery around 70% of our inventory, we need to set a higher buffer in our own delivery system. Like it or not, advertisers prefer to blame publisher rather than DoubleClick for any problem. It's not fair, but there's no other way. 

### Reduce Waste
The DoubleClick problem is basically a problem with their server accessibility. It makes sense to assume that if a user can load DoubleClick ad successfully, it's very likely he/she will be able to load successfully next time (in 30 minutes), and vice versa. To verify this assumption, we set cookie duration of 30 minutes and segment users into three categories: reachable, unreachable and unknown. Here's the result. 

| Reachability   | Description                 | Success Rate                               |
|----------------|-----------------------------|--------------------------------------------|
| Reachable      | when last attempt succeded  | 93.5%                                      |
| Unreachable    | when last attempt failed    | 19.0%                                      |
| Unknown        | first attempt in 30 minutes | 72.6%                                      |

As you can see, if we target DoubleClick ads to the Reachable category, we can raise the success rate to over 90%. Similarly, we should avoid delivering DoubleClick tag to a user if last attempt fails, because the success rate would only be 19%. 

We need to mark as many as possible users to either Reachable or Unreachable, otherwise the overall success rate will not be able to be improved markedly. One way to do it is to send an ajax request to DoubleClick for all Unknown category users before loading any ad, assuming DoubleClick accessibilities are consistent across different clients and campaigns. To do this, we need to get a DoubleClick script tag, which doesn't affect any other client. Ideally, this should be a house ad or just a 1x1 image dot. 
