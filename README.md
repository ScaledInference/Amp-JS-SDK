#
# Amp-Browser Client Integration Instructions

By Scaled Inference

[​1.​ Introduction](#1-introduction)
* [1.1.​Purpose](#11-purpose)
* [​1.2.​Amp.ai](#12-ampai)
* [1.3.​Amp Browser Client](#13-amp-browser-client)

​[2.​ Getting Started](#2-getting-started)

​[3.​ Usage](#3-usage)
* [​3.1.Initialization](#31-initialization)
  + [​3.1.1.​ Initialization Choices](#311-initialization-choices)
  + [3.1.2.​ Configurations](#312-configurations)
  + [3.1.3.​ Advanced Settings](#313-advanced-settings)

* [3.2.​ Observe](#32-observe)
  + ​[3.2.1.​ Options](#321-options)
  + [3.2.2.​ Callback](#322-callback)

* ​[3.3.​ Decide](#33-decide)
  + [3.3.1.​ Options](#332-options)
  + [3.3.2.​ Policies](#333-policies)
  + [3.3.3. Serialization / Deserialization](#334-serialization--deserialization)

[​4.​ Troubleshooting](#4-troubleshooting)


## ​1.​ Introduction
### 1.1.​ Purpose

This document will provide you with an overview of integrating your web-application with _Amp.ai_ using our Amp Browser Client.  We also provide clients for your mobile iOS and Android applications as well as your server-side Java, Node and Python applications.

### 1.2.​ Amp.ai

_Amp.ai_ is a cloud AI platform that can enhance any software application that integrates with it  by making intelligent, goal-driven, context-sensitive decisions. Think of it as A/B testing on steroids.  While A/B testing provides information about what choice to take across all users, Amp.ai will provide you with context-aware decisions that continue to improve and provide you with the best decision based on the context they are in.

### ​1.3.​ Amp Browser Client

The _Amp browser client_ consists of a set of functions that enables a web application to integrate with Amp to report observations and request intelligent decisions.

## ​2.​ Getting Started

The following snippet illustrates a typical web-application integration with Amp.

``` javascript
<script src="https://amp.ai/libs/PROJECT_KEY.js"></script> 
<script>  
  amp.observe("userInfo",{ lang:"en", country:"china", premium:false });   
  let decision = amp.decide("pickStyle",{
    color:["red","green","yellow"],
    font:["bold","italic","regular"]
  });  // use "decision.color" and "decision.font" to render your component
</script> 
```

**Initialization.** The first line initializes the amp instance that represents a single session in the Amp project corresponding to the project key PROJECT_KEY. Also, it enables the built-in observe events, e.g., AmpSession, AmpClick, and AmpScroll (see Section 3.1.2 for details). In the simplest case, this line can be all it takes (a.k.a., the _one-line_ integration) to integrate your web-application with Amp!

**Observe.** Most likely, you will also want to include custom observe events to capture metrics (that measures how well your application is to your business goals) or the contexts that are relevant to them.

**Decide.** Learning to make decisions to improve your metric is the key value Amp provides. Simply list the candidates, e.g., the color and font of some key text, that are likely to make a difference for your metric, and Amp will help you learn from your data and make the best decision!

## ​3.​ Usage

The browser client consists of three key components: `initialization`, `observe`, and `decide`. Initializing Amp creates an amp instance that represents a single session in a project. The amp instance then provides two key methods; amp.observe (to make observations) and amp.decide (to make decisions). To support cross-platform integration, amp also provides serialize and deserialize methods.

### ​3.1.​ Initialization

To support the needs of various applications, there are different ways to

- Initialize an amp instance
- Set its configurations
- Set its advanced settings.

#### 3.1.1.​ Initialization choices

There are three initialization choices: `_synchronous_` and `_asynchronous_`.

**Synchronous initialization.** This initialization is illustrated in Section 2. It's main benefit is simplicity, since the availability of the amp instance is guaranteed upon the next javascript statement. The flipside is that the application needs to wait while the amp instance is being initialized. This usage is recommended in applications where a moderate latency (typically within 50ms) can be tolerated.

``` javascript
<script src="https://amp.ai/libs/PROJECT_KEY.js">
</script>
```

**Asynchronous initialization.** For latency-sensitive applications, the amp instance should be initialized as follows:

``` javascript
<script>
"use strict";function _defineProperty(e,t,r){return t in e?Object.defineProperty(e,t,{value:r,enumerable:!0,configurable:!0,writable:!0}):e[t]=r,e}!function(e){function t(e,t){try{var r=Object.prototype.toString.call(e).slice(8).slice(0,-1).toLowerCase();return 2===arguments.length?"string"==typeof t&&(t=t.toLowerCase(),r==t):r}catch(e){return!1}}function r(e){try{if(!t(e,"Object"))return[];var r=Object.keys(e);if(!r.length)return[];for(var n=[];r.length;)!function(){var i=r.pop(),a=e[i];a&&!t(a,"Array")&&(a=[a]),a.length&&(n=0===n.length?a.map(function(e){return _defineProperty({},i,e)}):n.map(function(e){return a.map(function(t){return Object.assign(_defineProperty({},i,t),e)})}).reduce(function(e,t){return e.concat(t)},[]))}();return n}catch(e){return[]}}if(!window.amp){e=e||{},Object.assign(e,window.ampConfig||{});for(var n=window.amp={replay:[],v:"1.0.2",ts:(new Date).getTime()},i="observe log".split(" "),a=0;a<i.length;a++)!function(e){n[e]=function(){for(var t=arguments.length,r=Array(t),i=0;i<t;i++)r[i]=arguments[i];n.replay.push([e,r,(new Date).getTime()])}}(i[a]);n.decide=function(e,i,a,o){return t(i,"object")&&(i=r(i)),(i=t(i,"array")?i.slice(0,1):[]).length>50?(o&&"function"==typeof o&&o(new Error("Candidate length must be less than or equal to 50."),i[0]),i[0]):(n.replay.push(["decide",arguments,(new Date).getTime()]),i[0])};var o=document,c=o.getElementsByTagName("script")[0],u=o.createElement("script");c.parentNode.insertBefore(u,c),u.type="text/javascript",u.async=!0,u.src=e.scriptSrc||(e.domain||"https://amp.ai")+"/libs/"+(e.key||"amp.min")+".js"}}({key:"<AMP-PROJECT-KEY>"});
</script>
<script>  
  amp.observe("userInfo",{lang:"en", country:"china", premium:false});   
  const decision = amp.decide("pickStyle",{
    color:["red","green","yellow"],
    font:["bold","italic","regular"]
  });    
  const color = decision.color;  const font = decision.font;
</script>
```

Effectively, this initialization puts all usages of the amp instance into a replay queue, which are replayed after it is initialized. Note that in this case, the timeout specified for `amp.observe()` and `amp.decide()` needs to include the waiting time while they are in the replay queue.

#### 3.1.2.​ Configurations

For each amp instance, the following configurations are available (i.e., in amp.config).

| **Name** | **Default Value** | **Data Type** | **Details** |
| --- | --- | --- | --- |
| domain | [https://amp.ai](https://amp.ai/) | String | The domain to which an amp instance sends requests to |
| apiPath | /api/core/v1 | String | The API path to which an amp instance sends requests to |
| key | Empty String | String | Project Key |
| userId | 5-character random string | String | Unique User Id |
| samplingRate | 1 | [0-1] | Throttles session- level requests sent to Amp |
| timeout | 30000 | Milliseconds | Maximum time allotted to make session-level requests.  Requests that timeout will send an EARLY_TERMINATION error in the callback. |
| debug | warn | (Boolean\|String) | error, warn, info, debug, true, false |
| builtinEvents | ["AmpConnectionLatency1", "AmpConnectionLatency2", "AmpPageEnd", "AmpSession", "AmpClick", "AmpScroll", "AmpFocus", "AmpHooks"] | Array | See Section 3.1.3 |
| instanceName | Empty String | String | Custom name for Amp.ai instance |
| sessionTTL | 30 * 60 * 1000 | Milliseconds | Time when session will expire after inactivity |
| sessionLifetime | 24 * 60 * 60 * 1000 | Milliseconds | Defines the max session time 
| syncInterval | 30 * 60 * 1000 | Milliseconds | The interval to sync the configurations / policies. See Section 3.3.3 for more information. |
| hooks | [] | Array | Observe all events from supported libraries.  Allowed values: "ga", "optimizely", "segment"|

There are three ways to specify these configurations, i.e., before, during, and after the initialization.

**Before initialization.** When amp does not yet exist, specify configurations using window.ampConfig. For example:

``` javascript
window.ampConfig ={
  sessionTTL: 30 * 60 * 1000
}
```

When amp is being initialized, the properties in window.ampConfig will be used.

**During initialization.** When amp is being initialized, specify configurations using attributes of the `<script>` tag:

``` javascript
<script sessionTTL=1800000 src="https://amp.ai/libs/PROJECT_KEY.js">
</script>
```

Note that specifying configurations this way (i.e., during initialization) only supports the configurations of types string and number (see the 2nd / 3rd column above).

A configuration value specified here overrides the value specified in window.ampConfig.

**After initialization.** After amp is initialized, specify configurations using `amp.config.set(key, value)`:

``` javascript
amp.config.set("sessionTTL", 1800000);
```

This overrides the configuration values specified during initialization. Of note, you can inspect the current value(s) of configurations using:

``` javascript
amp.config.get("sessionTTL");
amp.config.getAll();
```

#### ​3.1.3.​ Advanced settings

**builtinEvents**

The following events are supported by default with the Browser client:

1. AmpSession: this is the event we send when session created, along with all properties we extract from the browser and page;
2. AmpClick: this is the event we will send when any click happens in the page (only those we can detect);
3. AmpScroll: this is the event we will send when page scroll depth increase more than 100px with a 1s debounce;
4. AmpConnectionLatency1: this event measures the latency of the first event sent;
5. AmpConnectionLatency2: this event is sent to check the network latency after the page has had time to load;
6. AmpPageEnd: this is the event we will send when page refresh / close / go next or previous... etc (only those we can detect);
7. AmpFocus: this is the event we will send when user change their focus into an iframe (mainly for ad click detection);
8. AmpHooks: this enables the events specified inconfig.hooks to be caught by Amp.

These can be overridden if you choose not to observe some or any of these by setting the `hooks` configuration.

**Multiple Amp Instances**
If you have a use case where two or more projects overlap, you need to specify a unique instance of Amp for each one beyond the default instance you receive when bringing down the Amp.ai script.

_Example:_
``` javascript
window.ampConfig = {
  // project configuration here
};

<script src='https://amp.ai/libs/PROJECT_KEY.js'></script>

window.ampConfig = {
  // project configuration here and
  instanceName: 'amp2'
};

<script src='https://amp.ai/libs/PROJECT_KEY_2.js'></script>

<script>
  // default Amp instance from synchronous script above
  amp.observe('ObserveEvent', { props }); 
  var decision = amp.decide('DecideEvent', { candidates });
  
  // overlapping project requiring a defined instance of Amp from `instanceName` config
  amp2.observe('ObserveEvent2', { props });
  var decision2 = amp2.decide('DecideEvent2', { candidates });
</script>
  
```

### 3.2.​ Observe

The amp.observe method has the following signature (i.e., in JsDoc):

``` javascript
/**
  * observe, send observation with event name and properties   
    related to observation
  *
  * @name     observe
  * @memberOf session
  * @param    {String} name - required
  * @param    {Object} properties - optional
  * @param    {Object} options - optional
  * @param    {Number} options.timeout - time allowed to make request
  * @callback callback - optional
  * @param    {Error} err
  *
  * @example  
  * 
  * session.observe("userInfo", {country: "china", lang: "zh"},  {timeout: 500});
  *
  */ 
  void observe(name, properties, options, callback(err))
```

which makes an observation using a named _event_ with properties attached to it.

For example, the statement:

``` javascript
amp.observe("userInfo", { lang: "en", country: "china", premium: false });
```

sends the event userInfo with 3 properties `lang`, `country` and `premium`.

#### 3.2.1.​ Options

`amp.observe()` only supports a single option timeout, which uses `amp.config.timeout` by default. To override, use something like:

``` javascript
// override timeout to be 500 ms
amp.observe("userInfo", { lang: "en", country: "china", premium: false },{ timeout:500 });
 ```

#### ​3.2.2.​ Callback

Use callback to handle errors:

``` javascript
// print out the error if any is returned
amp.observe("userInfo", { lang: "en", country: "china", premium: false }, {}, function(err){  
  if(err) {    
    console.log(err);  
  }
});
```

### 3.3.​ Decide

The amp.decide method has the following signature (i.e., in JsDoc)

``` javascript
/**
 * decide, request a decision using a named event and a list of candidates
 *
 * @name decide
 * @memberOf amp
 * @param {String} name - required
 * @param {Object} candidates - required
 * @param {Object} options - optional
 * @param {Number} options.timeout - time allowed to make request
 * @param {Number} options.ttl - the length of time this decision will be cached
 * @callback callback - optional
 * @param {Error} err
 * @param {Object} decision
 */
 Object decide(name, candidates, options, callback(err, decision))
 ```

For example, the statement:

``` javascript
let decision = amp.decide("textStyle", {
  color:["red", "green", "blue"],
  font: ["bold", "italic", "regular"]
}); // Use decision.color and decision.font to take action on page elements
```

requests a decision with the event name `textStyle`, which asks Amp: "which of the (three) candidate(s) is the best for my metrics". The decision returned can then be used to change the behavior of your application.

#### 3.3.1.​ Options

`amp.decide()` supports two options: `timeout`, and `ttl`. To set, use something like:

``` javascript
let decision = amp.decide("textStyle", {
  color:["red", "green", "blue"],
  font: ["bold", "italic", "regular"]
}, { timeout: 500, ttl: 20 * 60 * 1000});
```

**timeout.** Specifies the maximal amount of time between the time when amp.decide() is called and the time when the callback is called. When the asynchronous custom initialization(see Section 3.1) is used, this will include the time when the request is waiting in the replay queue. When `amp.decide()` times out, it returns the _default_ decision (see Section 3.3.3).

**ttl.** The amount of time that the decision returned is cached.  By default, the `ttl` is cached for the duration of the session.

#### 3.3.2.​ Policies

`amp.decide()` makes decision through two types of policies: _`Default`_ and _`Amped`_.

- The _`Amped policy`_ aims to optimize the business metrics you setup via the console, i.e., by using the patterns learned automatically from your data.
- The _`Default policy`_, on the other hand, aims to serve as a conservative fallback by simply truncating the candidates to be of length 1. The decisions made this way are referred as the default decision(s).

The proportion of sessions that uses the `Amped` vs the `Default` policies, a.k.a. the _`policy allocation`_, can be controlled from the Amp console. Furthermore, both the policy allocation and the most current Amped policy (continuously being updated by Amp) will be automatically _`synced`_ to the client, such that they are honored by the decisions made at the client. An amp will do _`policy syncing`_ when an amp instance is created, if the time specified by `syncInterval` has passed since the last syncing. If the instance stays alive past the time specified by `syncInterval`, it will sync again.

#### 3.3.3. Serialization / Deserialization

Sessions can be serialized and deserialized by setting the `sessionTTL` in the Amp instance configuration.  This allows you to set the duration of a session in your application.  Some sessions are defined by user which can have a very long TTL, while others may be defined by each visit to the application, which have a much shorter TTL.  When setting the TTL on the session, it is also recommended to set the TTL on the `decide()` call to the same amount for shorter session definitions.  This will allow for a cohesive user experience.

Another use case for serializing the session may be that you want to persist a session across platforms, for example, from you web application to your mobile application.

**Serialization**

By serializing your session and persisting it on a server, your mobile application can pull it down and use it to continue the session on the mobile device.

**Deserialization**

Upon pulling down the persisted session, you can use the `deserialize()` on the session instance to continue where you left off.

## ​4.​ Troubleshooting

1. What if I pass in some arbitrary options when creating an amp instance ?
  1. Only defined options will be recognized and all other options will be ignored.


2. ​What if I want to keep using the same session across usage ?
  1.  We support sessionTTL as one of the amp level options in order to specify the lifetime of a session. If it's not outdated, it will be resumed; otherwise, a new session will be created.  The session is still available if you refresh the page or navigate to another page as long as sessionTTL has not expired.

3. What if I call bserve or decide without a name ?
  1. If in debug mode, you will be notified with the error, otherwise, it will fail silently and the events won't be able to be sent to us. For decide, you will still get the candidate(s) back, but with default order.

4. ​Will observe or decide affect my latency ?
  1. No. For bserve, we make it purely async. Both calls will fire and forget immediately. If you want to do some actions right after the bserve request being sent, then you can pass in a callback into bserve and it will be called after we hear back from the request. For decide, we will return the decision through the callback, which will be generated by our offline policies, and then report to our server.

5. Will your library break my web site ?
  1. Amp will not break your site as it is in a self-contained closure wrapped in a try-catch block to catch any errors coming from the framework.  All service calls are non-blocking asynchronous calls.

6. What is the compatibilities of Amp ?
  1. We support back to IE11, Firefox 44 on Windows and Firefox 40 on Mac, Chrome 40 and Safari 8 on OSX Yosemite.  For these versions, you will have a fully functional amp instance.  For unsupported versions, amp will use a fallback snippet and will only support amp.observe and amp.decide calls that return the default decision.
  2. The AmpFocus event has a dependency on the browser being able to detect `onBlur` events on an `iframe`.  If the browser does not support this, then AmpFocus events will not be fired.
  3. The AmpPageEnd event has a dependency on the browser supporting `onbeforeunload`.  If the browser does not support this, then AmpPageEnd events will not be fired and any current processing bserve or decide events will be lost.
  4. For mobile browsers:
    1. iOS - Google Analytics and cached decisions are not supported
    2. Android - AmpFocus, decision callback and config syncing are not supported

7. Who can I contact if we have any issues ?
  1. You can contact [support@scaledinference.com](mailto:support@scaledinferene.com) with any questions about Amp.

8. Will Amp conflict with any other libraries ?
  1. We wrap Amp into a self-contained closure and only expose Amp and amp…. we honor ampConfig
9. What if Amp fails to load in my browser?
  1. If Amp fails to load, the failure is caught and uses a fallback snippet.  This code only provides support for amp.observe and amp.decide that returns the default decision.

10. If I have overridden global functions in my code, will Amp still work with my application?
  1. It is STRONGLY ADVISED TO NOT OVERRIDE GLOBAL FUNCTIONS such as Math.random(), as you can expect to get unpredictable results.
