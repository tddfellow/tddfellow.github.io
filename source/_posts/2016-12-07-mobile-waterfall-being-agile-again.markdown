---
layout: post
title: "Mobile Waterfall. Being Agile Again"
description: "Have you ever worked with mobile platforms, such as Apple Store or Google Play? How much time it takes to release a new version of the application? They have ~2-4 days manual review of your mobile application. What do you think that means? - You can release your application to production no often than two times a week..."
twitter_image: "/images/waterfall.jpg"
date: 2016-12-07 19:54:07 +0100
comments: true
categories:
- microservices
- mobile
- ios
- android
- dsl
- waterfall
- agile
---

Have you ever worked with mobile platforms, such as Apple Store or Google Play? How much time it takes to release a new version of the application? They have ~2-4 days manual review of your mobile application. What do you think that means? - You can release your application to production no often than two times a week. It became much better around this year - before it was 1-1.5 weeks.

![waterfall](/images/waterfall.jpg)

<!-- more -->

Would you like to deploy a bug fix? - Half a week. Unless it makes your application crash all the time. Then you can get it going in half a day or one day. Which is still pretty slow.

Would you like to make a canary release to 1% of your users and test an assumption quickly using your analytics tools? - Forget it. Waiting for three days to get results on your assumption negates the benefits of the Canary release. You should be getting your feedback in minutes, not days!

At such disadvantage deploy takes half a week and makes software development teams switch to much more defensive mode:

- more extensive end-to-end testing (slow), and
- more extensive manual QA before the release.

That is Waterfall. Right there.

## Being Agile Again

In that environment, how could we get our quick feedback back? - What if we had the ability to send logic in the form of Domain-Specific Language (DSL) from our server, that we can deploy to whenever we wish to? What if our mobile application could have interpreted this DSL and could have updated itself every time user starts the application while connected to the Internet?

We would be able to get a quick feedback! It would make fast deploys possible and also it would allow us to do canary releases, which can enable us to be LEAN again - To be Agile again.

This approach has a few problems:

- Reviewer can reject the application release. In this case, the developer has to negotiate with reviewers and explain that their business process requires such capabilities for their application to be quickly deployable. Exchange of few email and you can be Agile again.
- You would have to implement that DSL. Both: design the DSL and write the interpreter on the mobile application side. That is quite a significant investment of time. It is reasonable to implement only tiny part of such DSL and apply it only in places that need to be often changed, such as UI and business logic that everyone tries to fiddle with to optimize the KPIs of the mobile application.
- The approach does not guarantee quick deployability of 100% of the changes you need to make. Some changes will require an extension of DSL, and its interpreter, and native bindings. Surely, such change will have to be deployed via regular submit -> review -> wait four days -> release cycle. The tricky challenge is to balance what gets implemented in a DSL and what gets implemented in the native bindings for that DSL to achieve high enough portion of changes to be quickly deployable: about 90-95%; and keeping
the DSL and interpreter complexity as low as you can.

If you take the idea of such DSl and the interpreter to the extreme, you will wound up with the programming language. There is already such programming language and platform that has the same characteristics - Javascript + React Native. Nowadays, application stores allow to download javascript code updates from your server, but you have to deploy all native bindings via application store with a manual review; also, one can not change the essence of their application.

Why would you want to go with your DSL instead of React Native?:

- You already have a native mobile application, and you have identified places that change just too often.
- The performance of the application is critical.

## Example of the DSL

DSL might be as simple as Abstract Syntax Tree represented in JSON. Let's imagine that we have an application, where users can buy some items and now we want to contact the recommendation service and present them a NEW view with the list of recommendations. Normally, you would have to do a full development and full deployment via application store. With DSL you might end up just writing some JSON:

```javascript
// chunk of logic A
{
  "subscribe": {
    "event": "user_has_liked_item",
    "actions": [
      {
        "request": "/api/v1/recommendations/${event.item_id}",
        "publish": "received_recommendations"
      }
    ]
  }
}

// chunk of logic B
{
  "subscribe": {
    "event": "received_recommendations",
    "actions": [
      {
        "render": {
          "view": "item_list",
          "data": "${event.response.items}"
        }
      }
    ]
  }
}
```

Of course, parser and interpreter for this JSON, and actions (such as: `request` and `render`) are written in the native code and, therefore, have to be deployed via application store every time you change them or add a new capability. One can implement views in the native code, or represent them in DSL (aka simplified HTML + CSS).

## Bottom Line

As an industry, we understand why different application store vendors want to review every release of every application:

- To maintain quality of the applications and avoid possible malware, and
- To preserve their revenue streams intact (paid application payments have to go through vendors, so they will be able to take a commission).

Nevertheless, we need to reject manual review as a bad practice and aim for fully automated deployments; that deliver our new application releases to users in minutes. As an industry, we need to push companies running application stores to improve their process to enable us to do that.

## Thanks

Thank you for reading, my dear reader. If you liked it, please share this article on social networks and follow me on twitter: [@tdd_fellow](https://twitter.com/tdd_fellow).

If you have any questions or feedback for me, don’t hesitate to reach me out on Twitter: [@tdd_fellow](https://twitter.com/tdd_fellow).
