---
layout: post
title: "Heimdallr with active model serializers"
date: 2013-07-07 07:07
comments: true
categories: rails
tags: heimdallr
---
I have introduced [heimdallr](https://github.com/inossidabile/heimdallr) in an [earlier post](/blog/restricting-models-with-heimdallr). It is an excellent tool for securing models. `heimdallr` can create proxies for collections and records, it is possible to define scopes for fetching and deletion, and which messages are permitted to be forwarded to the proxied record. I use `active_model_serializers` for serialization. It provides an elegant way to define serialization rules for the models.

`active_model_serializers` does not work with the collections and records secured by a proxy out of the box. It would be good to know what should happen when a not permitted attribute is accessed on the model. I thought about two options:

1. the serializer could raise a permission error or
2. it serializes a value interpretted as not readable.

I chose the second approach. However, as json does not know about symbols as ruby does, serializing unaccessible attributes as `nil` seems to a proper option. The consumers of the produced json need to agree on the interpretation of a `null` value in the received json as a not accessible attribute.

Luckily the described strategy for serializing secured record is very simple. Heimdallr defines method `implicit` which returns a proxy with a strategy returning `nil` for unaccessible attributes.

I conclude this post with the short code doing the trick described above:

{% codeblock heimdallr_with_active_model_serializers.rb %}
require 'heimdallr'
require 'active_model_serializers'

module Heimdallr
  class Proxy::Record
    delegate :active_model_serializer, to: :@record

    def read_attribute_for_serialization(name)
      self.implicit.send name
    end

  end

  class Proxy::Collection
    include ActiveModel::ArraySerializerSupport
  end
end
{% endcodeblock %}

