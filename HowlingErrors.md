**NOTE: The Howl application no longer appears to be available online and on the AppStore**

ELMAH ships with a module called `ErrorTweetModule` that was originally designed to post brief error notifications to a Twitter account. It only made sense, in fact, for a Twitter account where updates were locked away from public view. If the account had a following then those people would receive notifications as well.

`ErrorTweetModule` shipped with ELMAH before Twitter service [disabled the use of Basic authentication](http://blog.twitter.com/2010/08/twitter-applications-and-oauth.html). While `ErrorTweetModule` won't work for Twitter anymore, it was developed general enough that you can still use it with custom or other notification services. It's not by chance that the module was named `ErrorTweetModule` instead of `ErrorTwitterModule`; _tweet_, in this context, meaning a brief message or update. In this article, you will see how to use `ErrorTweetModule` to receive notification on an iPhone or an iPad without writing a single line of code!

`ErrorTweetModule` fundamentally takes an unhandled or [signaled](http://code.google.com/p/elmah/wiki/DotNetSlackersArticle#Signaling_errors) exception and turns it into an HTTP form post to any URL. Like most other modules in ELMAH, it can be configured with a few options. Out of the box, `ErrorTweetModule` is setup to post to Twitter such that all that was left to do in the past (while Twitter still supported Basic authentication) was configuring the user name and password of a Twitter account and one was good to go.

In its default state, the module is configured as if you had the following under `elmah` section group in your `web.config`:

```xml

<errorTweet
userName=""
password=""
statusFormat="{Message}"
maxStatusLength="140"
ellipsis="..."
url="http://twitter.com/statuses/update.xml"
formFormat="status={0}" />
```

Following is a breif explanation of the attributes:

| **Attribute**        | **Meaning** |
|:---------------------|:------------|
| `userName`           | User name for (Basic) authentication |
| `password`           | User password for (Basic) authentication |
| `statusFormat`       | Template used to format the tweet, status update or notification message (hereafter referred to as simply _tweet_) |
| `maxStatusLength`    | Maximum length of the tweet |
| `ellipsis`           | The text that is appended to the tweet if it is clipped, that is, if it exceeds `maxStatusLength` |
| `url`                | URL where to post the tweet |
| `formFormat`         | Template used to format the HTTP form posted |

Let's see how configuring these attributes, you can use have `ErrorTweetModule` send you nearly real-time notifications to a device such as an iPhone or an iPad.


<img src='http://howlapp.com/images/howl-128.png' width='128' height='128'>


<a href='http://howlapp.com/'>Howl</a> is a Growl application for iPhone and iPad. It sports a simple <a href='http://howlapp.com/interface.html'>API</a> using which one can emit notifications with HTTP POST and have them subequently deliviered to your device.<br>
<br>
First, you will need to purchase the <a href='http://itunes.apple.com/us/app/howl-a-growl-app/id327646112?mt=8'>Howl application</a> for your device:<br>
<br>
<br>
<img src='http://a1.mzstatic.com/us/r1000/042/Purple/af/60/c2/mzl.kudjygoq.320x480-75.jpg' width='320' height='480' />


Next, setup an account from within the application. Make sure that when you setup your account, you supply a valid e-mail address (marked optional in the application but required for posting notifications). Once your e-mail address has been verified, open the <code>web.config</code> of a web application already using ELMAH. Define the following section under the <code>&lt;sectionGroup&gt;</code> element for <code>elmah</code>:<br>
<br>
<pre><code><br>
&lt;section<br>
name="errorTweet" requirePermission="false"<br>
type="Elmah.ErrorTweetSectionHandler, Elmah" /&gt;<br>
</code></pre>

Next, add the following section under the <code>&lt;elmah&gt;</code> section group:<br>
<br>
<pre><code><br>
&lt;errorTweet<br>
userName="USERNAME"<br>
password="PASSWORD"<br>
formFormat="name=error&amp;amp;title=Error%20%40%20example.com&amp;amp;description={0}&amp;amp;application=ELMAH&amp;amp;icon-name=warning"<br>
url="https://howlapp.com/public/api/notification" /&gt;<br>
</code></pre>

Replace <code>USERNAME</code> and <code>PASSWORD</code> in caps above with your Howl account user name and password. You may also want to replace <code>example.com</code> or other parts of the <code>formFormat</code> attribute to more meaningful values. See the <a href='http://howlapp.com/interface.html'>Howl public notification API</a> for more details and the meaning of the various form fields.<br>
<br>
You will then need to register the HTTP module by adding the following element in the <code>&lt;httpModules&gt;</code> section:<br>
<br>
<pre><code><br>
&lt;add<br>
name="ErrorTweet"<br>
type="Elmah.ErrorTweetModule, Elmah" /&gt;<br>
</code></pre>

If you are using IIS 7 or later then you will need the following entry in the <code>&lt;modules&gt;</code> section:<br>
<br>
<pre><code><br>
&lt;add<br>
name="ErrorTweet"<br>
type="Elmah.ErrorTweetModule, Elmah"<br>
preCondition="managedHandler" /&gt;<br>
</code></pre>

That's it! Next time you get an error, you should see a notification appear on your device, similar to screen captures below!<br>
<br>
<br>
<img width='320' height='480' src='http://wiki.elmah.googlecode.com/hg/howl-iphone-lock.png' />
<img width='320' height='480' src='http://wiki.elmah.googlecode.com/hg/howl-iphone.png' />

<img width='384' height='512' src='http://wiki.elmah.googlecode.com/hg/howl-ipad-lock.png' />
<img width='512' height='384' src='http://wiki.elmah.googlecode.com/hg/howl-ipad.png' />


Note that you can also use ErrorFiltering together with <code>ErrorTweetModule</code>. Just make sure that <code>ErrorTweetModule</code> is registered <i>before</i> <code>ErrorFilterModule</code> in the modules section.<br>
<br>
You can now imagine how you can use <code>ErrorTweetModule</code> to also send error notifications to a custom in-house service!