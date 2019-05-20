---
templateKey: 'blog-post'
title: 'Reactive Subscriptions In Meteor'
date: 2016-07-25
featuredpost: true
description: >-
  Reactive Subscriptions In Meteor
author: Hari Krishna
tags:
  - Meteor Development
  - Reactive subscription
---

Publication and subscription concepts are as important as its getting misunderstood. Recently my friend Tarun wrote a tutorial on[ Meteor publications](https://codebrahma.com/meteor-publications-and-subscriptions/), Now let’s dive deeper into subscriptions, but in a simpler way. This article shows how to write reactive subscriptions using Meteor’s autorun for effective sync between remote data sources and the view.

Publication sends data to the client on subscribing to that end point. The data gets merged into MiniMongo collections. But there will be considerable amount of latency in receiving
data after subscribing to it. We can use __subscription handlers__ provided in subscription function as callbacks. ```onReady()``` callback, is called when data is received and available in MiniMongo, then we query collection in MiniMongo and load data into the view. ```onStop()``` callback is called when subscription is stopped. In overview we will have a subset of mongoDB available on client side as MiniMongo.

Code to show how data has to be fetched from MiniMongo collection through subscribing to ```listItems``` publication.
```
if (Meteor.isClient) {
  // create a new instance of MiniMongo collection into which data received
  // from publication will be merged.
 var Items = new Mongo.Collection('items');

 Meteor.subscribe('listItems', {

  onReady: function () {
   // called when data is ready to be fetched from MiniMongo collection.
   var data = Items.find().fetch();
   // pass data to the view (React components or Blaze templates)
  },

  onStop: function () {
   // called when subscription is stopped
  }
 });
}
```
However, this is not reactive. When the data changes in remote MongoDB, it will to be pushed into MiniMongo but will not be loaded into the view. Meteor does magic in keeping server and client collections in sync through DDP communication. But how will client get to know if new data has been received and it should query from MiniMongo again for updated data and load into the the view?. Let’s take a look at how it could be done.

Meteor provides a reactive method ```.ready()``` which returns Boolean regarding the status of data received. It returns false until data is received by the client, then it returns true. In previous example we load data into view inside ```onReady()``` callback, now we do it other way.

```
if (Meteor.isClient) {

 var Items = new Mongo.Collection('items');

 // subscribing to listItems
 var itemSub = Meteor.subscribe('listItems');
 
 // fetch data from collection when .ready() returns true
 if (itemSub.ready()){
   var data = Items.find().fetch();
 }
}
```
This means whenever subscription handler ```.ready()``` is true, we load data into view. So when data changes in server, publication returns updated cursor which changes reactive data source, ```.ready()```. We need to keep watching ```.ready()``` function, whenever it changes we have to load the updated data to refresh the view. This process is more or less called as reactivity. View is in sync with remote source.

## Meteor’s autorun tracker
To keep watching ```.ready()```, Meteor provides autorun function. Basically, autorun runs the callback function, whenever the reactive data source used inside it changes. In this case, the reactive data source is ```.ready()```. If ```.ready()``` changes, we load the data into view. This is as shown below.

Thus subscription is reactive and updates data into view automaticaly.

```
 var itemSub = Meteor.subscribe('listItems');
 
 Tracker.autorun(() => {
   // fetch data from collection when .ready() returns true
   if (itemSub.ready()){
    var data = Items.find().fetch();
   }
 });
```
## Stopping subscription when not in use
We need to stop the subscription when we are no longer need of its data in MiniMongo.``` .stop() ```handler stops the subscription and removes data from MiniMongo. If we don’t use autorun. we need to call``` .stop() ```manually.

```
 var itemSub = Meteor.subscribe('listItems');
 
 // fetch data from collection when .ready() returns true
 if (itemSub.ready()){
   var data = Items.find().fetch();
 }
 // stopping the subscription when data is no longer needed.
 itemSub.stop();
```
But if we use autorun it takes care of stopping it when needed. Better explanation on functionality of autorun will be done in upcoming article.

## Changing subscription arguments
Sometimes we need to pass arguments along with subscription call, such as number of items you want or any filter. Publication returns data based on these arguments. Sometimes arguments keep changing. Good examples are pagination, filters etc., In these cases, a new subscription with other set of arguments is needed for new data. When we subscribe to same publication again, but with different arguments, and if it returns different data back then the previous subscription will be stopped and new request is sent to server. If data received by new subscription is same as previous one, then new one is stopped and old one is reused. The autorun works like charm in this case of re-subscribing with other arguments and updates data under the hood.

```
  Tracker.autorun(() => {

    // any means of receiving count or filter, either from
    // router params or from state
    let count = LocalState.get('listCount');
    var itemSub = Meteor.subscribe('listItems', count);

    // fetch data after subscription is ready
    if (itemSub.ready()) {
      var data = Items.find().fetch();
    }
  });
}
```
This is all we need to know about how to make subscriptions reactive. Advanced concepts, sequence of stopping and starting of new subscription, exploring the autorun will be featured in next posts. Stay tuned!

