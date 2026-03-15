<a href="https://www.openpolicyagent.org/"><img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/OpenPolicyAgent_001.png" width=150 height=150 align=right /></a>

# Open Policy Agent Access Control via REST APIs

<img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/OpenPolicyAgent_001.png" width=50 height=50 /> <img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/Docker_001.svg" width=50 height=50 /> <img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/Postman_001.svg" width=50 height=50 />

This repository contains code to demo the way the Open Policy Agent's REST API operate. The codebase has been intentionally kept as plain vanilla as possible.

## 📖 Open Policy Agent (OPA) Introduction

For those who have never heard of Open Policy Agent - which, to be fair, still includes a fairly large chunk of the engineering world - the simplest way to think about it is this: **OPA is a policy engine**.

Now, "policy engine" may sound like one of those terms that appears in architecture diagrams and disappears before anyone explains it properly, so let's simplify it.

At its core, a policy engine is something to which you provide:
- a set of rules that define what is allowed and what is not allowed
- some contextual information about the current situation
- and then a question asking whether a certain action should be permitted

The engine evaluates all of that and returns an answer.

In plain language, you are essentially saying:
> *Here are my rules.*
>
> *Here is the context.*
>
> *Now tell me whether this action should be allowed.*

<br/>

A very simple real-world example would be this:

Suppose we define a rule that says:

> *Anyone with the role **Admin** is allowed to **CREATE** something.*

Now we provide some context:

> *Alice has the role **Admin**.*

And finally we ask:

> *Is Alice allowed to **CREATE**?*




