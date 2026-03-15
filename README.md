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

A policy engine takes these three pieces, evaluates them together, and gives you the answer.

<br/>

If we map that into OPA terminology, it looks like this:
- **Policy** → Anyone with role **Admin** is allowed to **CREATE**
- **Data** → Alice has role **Admin**
- **Input** → Is Alice allowed to **CREATE**?

And that is really the first mental model to build before going deeper into OPA: **OPA itself does not hardcode your business decisions - it simply evaluates the rules you give it against the context you provide.**

That separation is what makes it powerful.

## 🧨 Enough theory - let's see OPA in action
Theory is useful, but policy engines become much easier to understand once you actually see one running.

For this demo, we will run Open Policy Agent inside a Docker container using a `docker-compose.yaml` setup, just to keep things simple and reproducible.

The setup is intentionally lightweight - bring everything up with:
```Bash
docker-compose up
```

If everything is wired correctly, you should see version details printed back, which confirms that OPA is running properly inside the container.

And now that our policy engine is sitting there quietly waiting for instructions, we can finally make it do something useful.

For this demo, instead of inventing policies from scratch, we will borrow one of the well-known examples from the [Rego Playground](https://play.openpolicyagent.org/) - specifically the **Role-Based Access Control (RBAC)** example available in the official playground.



