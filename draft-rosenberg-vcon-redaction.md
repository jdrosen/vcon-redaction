---
title: "Use Cases and Requirements for Redaction in VCON"
abbrev: "VCON Redaction"
category: info

docname: draft-rosenberg-vcon-redaction-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Virtualized Conversations"
keyword:
 - VCON
 - AI
 - redaction
venue:
  group: "Virtualized Conversations"
  type: "Working Group"
  mail: "vcon@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/vcon/"
  github: "jdrosen/vcon-redaction"
  latest: "https://jdrosen.github.io/vcon-redaction/draft-rosenberg-vcon-redaction.html"

author:
 -
    fullname: "Jonathan Rosenberg"
    organization: jdrosen.net
    email: "jdrosen@jdrosen.net"

normative:

informative:

  VCON: I-D.draft-ietf-vcon-vcon-core

  BIRK: I-D.draft-birkholz-verifiable-agent-conversations

  HOWE: I-D.draft-howe-vcon-agent-session

  VAC: I-D.draft-birkholz-verifiable-agent-conversations

  RESTRUCTURE: I-D.draft-rosenberg-vcon-restructure



--- abstract

The Virtualized Conversations (VCON) specification defines a standardized object format for representing multimedia conversations between users and AI agents. Recent work has expanded VCON to enable it to been used to capture sessions with AI Agents as well, including tool calls, reasoning steps, model configuration and more. This has also expanded the set of use cases and requirements for redaction of sensitive information from a VCON. This document outlines use cases and requirements for redaction in VCONs. This document is meant for discusssion purposes. All of this document was written by the author and not by an LLM. 


--- middle

# Introduction {#intro}

The Virtualized Conversations (VCON) specification [VCON] defines a standardized object format for representing multimedia conversations between users and AI agents. Recent work [HOWE] [VAC] [RESTRUCTURE] has expanded VCON to enable it to be used to capture sessions with AI Agents as well, including tool calls, reasoning steps, model configuration and more. This has also expanded the set of use cases and requirements for redaction of sensitive information from a VCON. This document outlines use cases and requirements for redaction in VCONs. This document is meant for discusssion purposes. No AI was used in the creation of this document.


# Redaction Support in VCON

The VCON specification itself provides support for redaction in a singular fashion. It allows a VCON to reference a prior instance of a VCON which was unredacted. This is accomplished through the "redacted" object defined in Section 4.1.8 of [VCON]. 

Beyond this, the specification says that specific methods for redaction of text, audio, and video are out of scope for the document. 

This document considers what those methods could be.

## Redaction Properties

There are several different properties of redaction systems. These properties largely define what information still remains after the redaction has ocurred. 

### Signaled or Unsignaled

Firstly, redaction can be signaled or unsignaled. In signaled redaction, there is some kind of indication - meant for processing by software - which indicates that redaction has occurred. In unsignaled, there is no such indication meant for processing by software. The distinction of "meant for processing by software" is important in this definition. It means that there is something like a field in a JSON document, a flag or tag or other piece of information which conveys the fact that redaction has occurred. In the current VCON specification, the redacted field is the property which provides a way for redaction to be signaled.

### Positioning and Participant Indications

Redaction operations can provide a positioning indication. A positioning indication is information that remains post-redaction that indicates where the redaction occurred. For audio and video streams, this is the start and stop timestamps where the redaction ocurrred. For textual streams, it is the exact text which was redacted in the chat sequence. For events, like a tool call, it is the exact position in the event - the tool call input or output for example - which was removed. 

Redaction operations can provide a participant indication. A participant indication is relevant only for conversational elements - audio, video or chat - and it indicates the participant that was redacted. 

By definition, if a VCON has positioning or participant indications, the VCON has signaled redaction. 

What are the use cases where we want to redact sensitive information but retain positional and/or participant indicators? in something like a contact center call, we want to remove PCI, but wish to retain positional and participant indicators for troubleshooting purposes. It is very common in contact centers for managers to review calls after the fact, and use the conversation to adjust many aspects of how both human and AI agents support customers. Another use case is for PII redaction, where we want to anonymize the conversation but don't need to hide the fact that redaction has ocurred. In such cases, we would want positional indicators, but not participant indicators. 


### Detectable and Undetectable Redaction

In the case of audiovisual content, redaction can be characterized as detectable or undetectable, which indicates whether a human being (or perhaps a model which is processing the content as a human might) - through observation of that content - would be able to determine that redaction has occurred.  For example, if a user speaks their social security number, and it is "beeped" over within the recording itself, then a human could determine that redaction has occurred by listening to the audio, and noting the beep as a clear indicator that there has been information redaction. In video content, there might be rapid fadeout to black. In image content, there might be a black box pasted over an image. 

To make audiovisual redaction undetectable, it needs to be substituted with something which sounds or looks appropriate to a human observer. This is generall quite difficult to do. For example, if a user speaks their social security number to an agent, one could use a text-to-speech model to generate an alternative, fake social security number, using the voice of the user. In other words, apply a "deepfake" technology to alter the content in such a way that it is generally undetectable. 

There are different use cases for how much information we want to remove, and how much we want to retain in the document. For highly sensitive redactions, such as when a witness needs to be protected and their presence utterly removed from a conversation - it is desirable to make the redaction undetectable, with no remaining positional or participant indications. 


### Information Tagging vs. Redaction

A close cousin to redaction is information tagging. In information tagging, sensitive information - such as PII - are tagged explicitly in the document or object. By tagging it, downstream consumers of the document can hide the information - or not - depending on the user that is viewing it. This relies on trust between the transmitter of the document, and the recipient. Namely, it requires that the transmitter trust the recipient to perform the information hiding appropriately based on the user that is viewing it. 

A common use case for this is in the contact center. A normal human contact center agent reviewing a call may not have permissions to see the name and address of the user in the transcript of the call, should they view that transcript. However, someone from the security and compliance team, might have such permission. 



# Redaction Methodologies for VCON Audio and Video Dialogs

This section discusses the techniques which can be applied to a VCON to achieve different types of redaction. Redaction methodologies for audio and video are more complex than those for text, due to the obvious difficulities in manipluation of the content. 


## Full Dialog Object Removal

The simplest way to redact audiovisual information in a VCON is to remove the dialog object containing that information entirely from the VCON. In this approach, there is no trace whatsoever that this dialog existed in the VCON. This removal changes the index for any other dialogs in the VCON, which might require updating of references to that dialog from other objects, such as an analysis or participant. 

Full object removal only makes sense if the VCON was structured as a sequence of dialogs, each representing a temporal segment of the conversation. For example, if each turn of a conversation was represented in a singular dialog. If a user spoke their social security number in a specific turn, the dialog representing that turn could be removed from the VCON.

Full object removal may result in a "hole" in the sequence of dialogs where no audiovisual information is present, making it potentially detectable. Whether it is detectable or not depends on how the dialogs were structured and on the nature of the conversation being captured. If the VCON had a set of dialogs, each representing a one-minute interval of audio mixed from all participants, then removing of one of the dialogs results in a VCON in which redaction is easily detectable - there will be a notable silence in the audio. However, if the VCON had a series of dialogs, each representing the audio contribution of a single speaker for a one-minute portion of time (or perhaps a single turn), then it might be detectable, or might not. If it was a conversation between a user and an AI agent, and the user's utterance is removed, the conversation would clearly sound like it was missing something. For example - 


AI Agent: What is your social security number?
AI Agent: Got it, thanks

A user listening to this audio sequence would know something was missing, making the redaction detectable.

However, in a less structured meeting, where it is possible there was no direct response to a user's utterance of redacted information, it may not be detectable. 

Full dialog object removal does not have positional or participant indications, making this approach problematic for cases where positional and participant indications are desirable. 

## Dialog Object Content Removal

In this approach, the content of the Dialog object - present either in the body element or the url and content_hash - is removed from the VCON, but the Dialog object itself and its meta-data (including parties, start and duration), remain. We call this a Dialog object "shell". 

Like full dialog object removal, this approach makes sense of the VCON was structured as a sequence of dialogs, each representing a temporal segment of the conversation. For example, if each turn of a conversation was represented in a singular dialog. If a user spoke their social security number in a specific turn, the dialog representing that turn could be converted to just a shell.

Unlike full dialog object removal, there is no change in the index of the dialog that was redacted, eliminating the need to potentially update references elsewhere in the VCON. 

Also unlike full dialog object removal, content removal can be accompanied by positional indicators (if the start and duration elements are preserved) and participant indicators (if the parties element is preserved). This makes this approach useful for many applications. However, it has a hard requirement that the VCON be structured in a way facilitating removal of content from a single dialog. For calls between a consumer and an AI agent, it would require per speaker, per-turn dialog elements. Even then, redaction of an entire turn might cause loss of information which is desirable for downstream troubleshooting and management purposes. Consider this conversation - 

AI Agent: How can I help you today?
User: Yeah, so my name is Jonathan Rosenberg and my account number is 12345. I ordered a widget 2000 like a month back and it still hasn't shown up. The website is showing it shipped but it has said that for a while and there is no Fedex tracking number. What's going on?

In this case, the user turn contains PII information (the user's name and account number), but it also contains critical information - the intent of the caller and context of the removal. If the dialog content is removed, the audio is lost. Now, this might be acceptable if there was an accompanying transcript in an analysis object. The entire audio turn could be redacted, and then just the two PII elements from the transcript. The alternative is to perform content substitution, noted below. 


## Dialog Object Content Substitution

In this approach, the content of the dialog object is replaced with something else. For audio, this might be a "beep" sound. For video, it might be a recorded video of a black screen. More interestingly, the content can be substituted for one in which the "beep" or video overlay only occur in a portion of the content. For example, consider a Dialog containing recorded audio covering 20 minutes of a call. At timestamp 3:41 the user said their social security number and it took 3 seconds to speak it. During the interval 3:41 to 3:44, the audio is replaced by an audible "beep". 

Content substitution is detectable. However, in the current VCON specification, there is no explicit indication that the substitution has happened. There is no positional or participant indicators either. 

This is a gap in the specification. 

# Redaction of Text

Text content can be present in dialog objects, containing emails or chat messages. It can also be present in events, such as a tool call request, a tool call response, or even in reasoning outputs. It can be present the Analysis object, inside of a transcript or summary of a dialog element. 

There are several ways to perform redaction of text within VCON. 

## Well-Known Character Substitution

The simplest technique is to just substitute the sensitive information with some kind of alternative well-known characters - "XXX" being quite common. Sometimes, the word "REDACTED" is used as well. 

Well Known Character substitution is effective and easy to implement. It is detectable. 

It's main drawback is that, as a positional indicator, it can lead to false positives. Consider the following interesting conversation - 

AI Agent: What is your three letter confirmation code?
User: Yes, it is XXX. 
AI Agent: Thanks, let me look it up.

Here is the interesting question - was the user's confirmation code actually XXX, or was it something else, and there was a redaction operation which substituted the real confirmation code? 

This perhaps seems contrived, but it is even more complex in cases where the redaction occurred over text that was the result of speech recognition. Consider this example - 

AI Agent: What is your three letter confirmation code?
User: Hang on, umm, OK. Its XXX, then YKZ. 
AI Agent: Thanks, let me look it up.

In this case, is the user's confirmation code XXXYKZ, or has some kind of partial redaction taken place? There is no way to know. 

This is perhaps improved by using a word like REDACTED, but again can lead to confusion if the participants in a conversation actually use that word! Consider this conversation - 

User 1: Did you try reading the document? 
User 2: Yes I did, but the account number was redacted.

In this conversation - did user 2 speak the account number, but then it was subsequently redacted? Or did they say the word "redacted"?

There is another problem that the word "redacted" is in English, making it difficult to parse in VCON represented conversations in a different language.

All of these problems stem from the fact that character substitution is not actually part of the schema of the VCON at all - its a hint. As a hint, is ambiguous in some cases. If we want software to be able to know that redaction has ocurred, we want to do better. 


## Information Substitution

An alternative solution is to replace a piece of information - like a social security number or name - with an alternative that is incorrect, but plausible. For example, using the name "John Smith" or a social security number like "183-13-3948". To perform this substitution, software requires knowlede about the type of information element, and needs some kind of algorithm to perform substitution.

This approach is similar to well-known character substitution. However, when done well, it can produce redaction which is undetectable. In some use cases, that is highly desirable. In others, it may be very bad. Using a contact center call as an example, consider a user that chats with a contact center agent to get a refund. The account number is recorded into the VCON for the call at the end, and then replaced using information substitution. The user calls back the next day. A human contact center agent reviews the transcript of the prior chat session, and observes the account number in the prior transcript. Not knowing that this is actually NOT the correct account number, the agent uses the wrong account number and applies the refund to the wrong account. 


# Requirements for VCON Extensions

Based on the discussion in this document, there are a number of gaps in the VCON specification which would be good to address. Two stand out. 

## Explicit Positional and Participant Redaction Indicators

In cases where audiovisual dialog content has undergone substitution (e.g., the "beep" sound in an audio track), it would be desireable to have a property within the dialog object that explicity indicates the points in time (start and duration) that contain these beeps. That would address the limitation that, without this, there is no clear positional or participant indicators. THose indicators are highly useful for troubleshooting purposes. 

## Explicit Text Redaction Element

Instead of using XXX or the word REDACTED or similar, it would be good to standardize a way for any text string - present anywhere in the body of the VCON - to have a standardized way of indicating that some kind of redaction has ocurred. A good way to do this is by adding the idea of a redaction manifest, which allows a string anywhere in the document to be replaced by a reference to a standardized object which supports a mix of text and redacted information. Consider a simple VCON showing a single event - a tool call - which closes an account (using the event concept defined in [RESTRUCTURE]). 

~~~
{
  "events" : [
    {
      "type" : "tool-call",
      "tool_name" : "close_account_by_id",
      "input-params" : [
        {
          "input-name" : "account_id",
          "input-value" : "293827344"
        }
      ]
    }
  ]
}
~~~


You can see that this contains a sensitive account ID. Here is a version that is now redacted.

~~~
{
  "events" : [
    {
      "type" : "tool-call",
      "tool_name" : "close_account_by_id",
      "input-params" : [
        {
          "input-name" : "account_id",
          "input-value" : "__REDACTED_REF_001"
        }
      ]
    }
  ],
  "redactions" : [
    "name" : "__REDACTED_REF_001",
    "type" : "account_number"
  ]
}
~~~

You can see that, we've added a redactions element into the JSON which contains the list of redacted fields, including specification of their types. 

## Explicit Information Identication Element

There is no way in VCOn to take a piece of information - like a first name or social security number - and then explicitly tag it as a piece of sensitive information, without removing it from the document per se. 

This can be done along much the same lines as the text redaction element noted above. 

# AI Authorship Considerations

All of the text in this document was written by hand, without AI assistance. AI was used to analyze options for redacting text strings in a JSON document without needing to modify the schema of the document. 

# Security Considerations {#security}

This document is entirely about security - focused on information privacy through redaction. 


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO.


