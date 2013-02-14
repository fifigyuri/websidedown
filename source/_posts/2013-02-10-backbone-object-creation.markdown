---
layout: post
title: "Backbone forms: How to get ids for objects created on the client side?"
date: 2013-02-10 22:22
comments: true
categories: backbone, forms
---
I am building an application which is built with Backbone.js. It contains several forms, which make it possible to create sub-items in _collections_. I run into the issue of getting the id for the recently created items. Let me explain the scenario:

1. a new model instance is created with and an empty `id`
2. the form gets saved triggering a POST or PUT request
3. the server receives and saves the new and updated items
4. the new items get the `ids` filled out
5. all items are sent back with updated server-side `ids`

The issue is with mapping the new `ids` to the instances which are on the client side without an `id`.

Backbone.js generates a unique client-side `cid` for every object. The idea was to serialize the `cid` and send it back from the server with the server side `id`. As server I use rails. I added new virtual attribute `cid` to the model and serialized it. For the client models serializing (`toJSON`) and parsing needed to be updated. Method `toJSON` was the easy part:

{% codeblock tojson.js %}
toJSON: function() {
  return _.extend(toJSON.__super__.constructor.apply(this, arguments),
      { cid: this.cid })
}
{% endcodeblock %}

Bit trickier is handling of the response coming from the server. It's necessary to identify the existing item using the `cid` and update it with the new server side `id`. After exploring several possible ways, how to do it wrong, I was [advised](github.com/documentcloud/backbone/pull/2177) to override the `parse` method.

{% codeblock parse.js %}
parse: function (resp) {
  return _.map(resp, function (attrs) {
    var cid = attrs.cid;
    delete attrs.cid;
    var existing = this.get(cid);
    return existing ? existing.set(attrs).attributes : attrs;
  }, this);
}
{% endcodeblock %}

That simple definition of the `parse` method showed me the elegance of backbone. Searching for a simple solution proved to be worthwhile.
