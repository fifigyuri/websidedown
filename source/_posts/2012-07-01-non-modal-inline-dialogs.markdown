---
layout: post
title: "Non-Modal Inline Dialogs: Ask Questions in Awesome Ways"
date: 2012-07-15 18:00
comments: true
categories: javascript, dialogs
---
{% img right /images/modaldialog.png 300 "simple modal dialog with a yes-no question" %}

Asking questions, confirmations from the users is a pretty usual part of any
user interface. Still often I find tackling this simple task solved in
unsatisfying way. The simplest solution is the `confirm('ask your question..');`
dialog. However, this often ruins the otherwise possibly clean design. Confirm
dialogs are modal and therefore users can not proceed further actions without
answering the questions. But what if you do not need immediate answers?
Often it is enough leaving the dialog questions opened. In such situation try
using "inline dialogs".

Let's build a simple scenario. We have three simple events which the user can
trigger: calling a firemen, calling police or ambulance. Obviously triggering
these events could be useful to handle with a simple question before they are
really triggered. If this security question is answered positively, the event
is triggered, otherwise the original _Call the ..._ link is rendered.
The dialog blocks are divided into a trigger and question part.

{% codeblock inline-dialogs.html %}
<DIV CLASS='dialog_block' DATA-MESSAGE='Calling the firemen!'>
  <A HREF="#" CLASS="dialog_trigger">Call the Firemen!</A>
  <DIV CLASS="dialog_question">
    Call? 
    <A HREF="#" CLASS="answer_yes">Yes</A>
    <A HREF="#" CLASS="answer_no">No</A>
  </DIV>
</DIV>
<!-- do something similar for the two other calls -->
{% endcodeblock %}

For handling the dialog actions I use jQuery now. We need only three simple
callbacks for handlers: activation of the question trigger, handler for the yes
and the no answer.

{% codeblock inline-dialogs.js %}
var turnOffDialogQuestion = function(el) {
  $dialog_block = getDialogBlock(el);
  $dialog_block.find('.dialog_question').fadeOut('fast',
    function() { $dialog_block.find('.dialog_trigger').fadeIn('fast'); });
};
var getDialogBlock = function(el) {
  return $(el).parents('.dialog_block');
};
$(document).ready(function() {
  $('.dialog_trigger').click(function(ev) {
    $dialog_block = getDialogBlock(this);
    $(this).fadeOut('fast',
      function() { $dialog_block.find('.dialog_question').fadeIn('fast'); });
    ev.preventDefault();
    return false;
  });
  $('.dialog_question .answer_yes').click(function(ev) {
    alert(getDialogBlock(this).data('message'));
    turnOffDialogQuestion(this);
    ev.preventDefault();
    return false;
  });
  $('.dialog_question .answer_no').click(function(ev) {
    turnOffDialogQuestion(this);
    ev.preventDefault();
    return false;
  });
});
{% endcodeblock %}

The full source codes of this simple example can be downloaded from here:
[HTML](codes/inline-dialogs.html), [javascript](codes/inline-dialogs.js).
By following on the HTML link the example can be also tried out.
