---
layout: post
title: "Restricting Models with Heimdallr"
date: 2012-07-16 14:28
comments: true
categories: rails
tags: heimdallr
---
{% img right /images/cards.jpg 180 "deck of cards, some have visible faces" %}

I found myself several times implementing the logic for restictions in controllers.
Althought that might be fine, when the things get more complex, this practice
can become hard to maintain. As modeling restrictions belongs to the core logic of
the application, perhaphs they should be placed there..

That's what [heimdallr](http://github.com/roundlake/heimdallr) is supposed to resolve.
This gem provides model level security by wrapping the model classes and
instances by a proxy. The access rights are declared in the model:

{% codeblock card.rb %}
class Card < ActiveRecord::Base
  include Heimdallr::Model

  belongs_to :person

  attr_accessible :suit, :rank, :backside

  restrict do |person, card|

    if person.arbiter?
      # the arbiter can do anything
      scope :fetch, -> {}
    elsif person.playing?
      # if the user is playing, only the card at hand is visible
      scope :fetch, -> do
        where(['person_id = ?', person])
      end
    end
  end
end
{% endcodeblock %}

The `restrict` block can take more parameters, based on these we can decide how
to restrict the model. The last parameter (`card` in the example) is an
instance of the model, if it is `nil` the model class was passed to the restrict
block. The restriction can be thus called on either the model class or on a
concrete instance: `Card.restrict(person)` vs `card.restrict(person)`.

Heimdallr maintains a scope for fetching and deletion. Scope restriction are obviously
useful when restricting the model class. Calling `all` on the proxied model
returns only the cards visible by the person. Wondering how to simply ask
whether a card instance is visible by the person? `card.restrict(person).visible?`
does just that. For testing destroyability of a record there is `destroyable?`.

I covered briefly scope restriction using  heimdallr in this post. However, there
is yet more how heimdallr can help. Finer restrictions, such as attribute visibility
and updatability are often crucial as well. But let's leave this for later.
