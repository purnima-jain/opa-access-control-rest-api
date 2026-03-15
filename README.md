<a href="https://www.openpolicyagent.org/"><img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/OpenPolicyAgent_001.png" width=150 height=150 align=right /></a>

# Open Policy Agent Access Control via REST APIs

<img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/OpenPolicyAgent_001.png" width=50 height=50 /> <img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/Docker_001.svg" width=50 height=50 /> <img src="https://github.com/purnima-jain/miscellaneous-public/blob/master/icons/Postman_001.svg" width=50 height=50 />

This repository contains code to demo the way the Open Policy Agent's REST API operate. The codebase has been intentionally kept as plain vanilla as possible.

This article is published [here](https://medium.com/@purnima.jain/open-policy-agent-opa-understanding-policies-data-and-decisions-e1ce7c89435f)

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

<br/>

## 🧨 Enough theory - let's see OPA in action
Theory is useful, but policy engines become much easier to understand once you actually see one running.

For this demo, we will run Open Policy Agent inside a Docker container using a `docker-compose.yaml` setup, just to keep things simple and reproducible.

The setup is intentionally lightweight - bring everything up with:
```bash
docker-compose up
```

Once the container is up, it is always a good idea to verify that OPA is actually alive and healthy before doing anything else.

You can jump into the container and check the installed version using:
```bash
docker exec -it opa_container opa version
```

If everything is wired correctly, you should see version details printed back, which confirms that OPA is running properly inside the container.

And now that our policy engine is sitting there quietly waiting for instructions, we can finally make it do something useful.

For this demo, instead of inventing policies from scratch, we will borrow one of the well-known examples from the [Rego Playground](https://play.openpolicyagent.org/) - specifically the **Role-Based Access Control (RBAC)** example available in the official playground.

<br/>

### Injecting our first policy into OPA
Now that Open Policy Agent is up and running, the first real thing we need to do is give it a policy to evaluate.

OPA does not magically know your rules - you have to explicitly load them into the engine.

For this demo, we will take the policy available in the Rego Playground (RBAC example) policy panel and push it into OPA using its REST API.

And yes, for quick experimentation, Postman makes this extremely convenient.

We will send a `PUT` request to:

```
http://localhost:8181/v1/policies/example1
```

Using:
- **HTTP Method** → `PUT`
- **Body Type** → `raw`
- **Format** → `Text`

Then simply copy the policy content from the left-hand **Policy** panel of the Rego Playground and paste it into the request body.

<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/001_Inject_Policy.JPG" width=100% height=100% />

<br/>

A small but important detail here: the name at the end of the URL - in this case `example1` - becomes the policy identifier inside OPA.

So effectively, with this request, we are telling OPA:

> *Here is a policy. Store it under the name `example1`.*

Once the request succeeds, OPA compiles and stores that policy immediately, and it becomes available for evaluation.

This is one of the nice things about OPA during development: you can inject, replace, and iterate on policies very quickly without restarting anything.

<br/>

### Reading back the policy (because trust, but verify)
Once we push a policy into Open Policy Agent, it is always worth checking whether OPA actually accepted it the way we expected.

A successful `PUT` response is reassuring, but I still prefer verifying what is sitting inside the engine before moving further.

OPA gives us another REST endpoint for exactly that.

Send a `GET` request to:

```
http://localhost:8181/v1/policies
```

Using:
- **HTTP Method** → `GET`

<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/002_Read_Policies.JPG" width=100% height=100% />

<br/>

This returns the list of policies currently loaded into OPA.

If everything worked correctly, you should see the policy we inserted earlier - `example1` - along with its stored source content.

And now that the policy is safely inside OPA, the next obvious question is:

> *A policy alone is just a rule. Where does OPA get the actual contextual information from?*

That is where **data injection** comes in.

<br/>

### Injecting data: giving OPA some context
At this point, our policy is loaded, but on its own it still cannot answer anything meaningful.

Why? Because a policy defines only the rule - not the real-world context in which that rule has to be evaluated.

And that contextual information is what Open Policy Agent calls **data**.

Think of it this way:
- the **policy** says what should be allowed
- the **data** tells OPA what currently exists in the system

So next, we need to push some data into OPA as well.

For that, send a `PUT` request to:

```
http://localhost:8181/v1/data
```

Using:
- **HTTP Method** → `PUT`
- **Body Type** → `raw`
- **Format** → `JSON`

Then copy the contents from the **Data** panel in the Rego Playground and paste it into the request body.

<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/003_Inject_Data.JPG" width=100% height=100% />

<br/>

Without data, OPA has rules but nothing to apply them to.

With data loaded, the engine now has both:
- the rulebook
- the supporting facts

<br/>

### Reading back the data
Just like with policies, it is a good habit to verify that the data actually landed inside Open Policy Agent exactly the way we intended.

To read the currently stored data, send a `GET` request to:

```
http://localhost:8181/v1/data
```

Using:
- **HTTP Method** → `GET`

<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/004_Read_Data.JPG" width=100% height=100% />

<br/>

OPA will return the full data document currently available inside the engine.

At this point, OPA now has both major ingredients loaded:
- **Policy** → the rules
- **Data** → the contextual facts

And that means we are finally very close to asking the most interesting part:

> *Given this policy and this data, is a particular action allowed?*

That final piece is where **input** comes in.

<br/>

### Asking OPA the actual question
Now comes the part where everything finally connects.

We already loaded:
- the **policy** (the rules)
- the **data** (the supporting context)

The only missing piece is the actual request we want OPA to evaluate.

In OPA terminology, this is called **input**.

This is the dynamic part - the thing that changes from one authorization request to another.

For example: Is Alice allowed to create?

For our demo, send a `POST` request to:

```
http://localhost:8181/v1/data/app/rbac
```

Using:
- **HTTP Method** → `POST`
- **Body Type** → `raw`
- **Format** → `JSON`

Now copy the contents of the **Input** panel from the Rego Playground, but there is one important adjustment:

Wrap that JSON inside an attribute called `input`.

So instead of sending plain JSON directly, the body should look like:

```json
{
  "input": {
    ...
  }
}
```

That wrapper matters because OPA expects runtime input under the `input` key during evaluation.

One detail that is easy to miss here - and worth paying close attention to - is the URL path:

```
/v1/data/app/rbac
```

This path is derived directly from the **package name inside your policy**.

If your policy contains:

```
package app.rbac
```

Then OPA converts that package path into:

```
/app/rbac
```

In other words:
- dots (.) become slashes (/)
- the package path becomes part of the API endpoint

This is one of those small OPA conventions that makes complete sense once you notice it, but can be confusing the first time you hit it.


<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/005_Query_Or_Input.JPG" width=100% height=100% />

<br/>

And once you send the request, the response returned by OPA should match exactly what you see in the **Output** panel of the Rego Playground.

That is your final evaluated decision.

<br/>

### Updating data: changing the decision without touching the policy
This is where Open Policy Agent starts showing why separating policy from data is actually powerful.

So far, our policy says who is allowed to do what, and our data tells OPA who currently has which role.

Now let's change just the data and watch the decision change immediately - without touching the policy at all.

Suppose we want to revoke Alice's admin access and assign her a different role, say `non-admin`.

For that, send a `PUT` request to:

```
http://localhost:8181/v1/data/user_roles/alice
```

Using:
- **HTTP Method** → `PUT`

And the request body:

```json
[
  "non-admin"
]
```

<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/006_Update_Data.JPG" width=100% height=100% />

<br/>

What we are doing here is updating only Alice's role assignment inside the data store.

The policy itself remains exactly the same.

That means the rule still says:

> *Admins are allowed.*

But the supporting fact has changed:

> *Alice is no longer an admin.*

A nice follow-up step is to read the data again using the same `GET /v1/data` endpoint we used earlier, just to confirm that Alice's role has actually been updated.

And once that looks correct, run the exact same query again - the same input, the same policy path, everything unchanged.

<img src="https://github.com/purnima-jain/opa-access-control-rest-api/blob/master/images/007_Query_Or_Input_After_Update.JPG" width=100% height=100% />

<br/>

This time, the output changes.

And no surprises there: Alice is no longer allowed.

That moment is actually one of the clearest demonstrations of OPA's design philosophy:
- **policy stays stable**
- **data changes frequently**
- **decisions adapt automatically**

In real systems, this becomes incredibly useful because role changes, permissions, entitlements, and user context change all the time, while the authorization logic often remains structurally consistent.

Instead of rewriting application code every time something changes, you simply update the underlying data and let OPA re-compute the decision.

That separation is where a lot of operational elegance comes from.

## 🎁 Wrapping up

Before closing, one important clarification: this is obviously not how Open Policy Agent would typically be integrated in a real production system.

In the real world, policies are not pushed ad hoc through REST APIs every time. They usually live as `.rego` files in version control, go through the normal engineering lifecycle of review and approval, and are deployed into OPA in a controlled way - very often through a CI/CD pipeline. Likewise, application data would continue to live in your database or source systems; you would not simply dump everything into the OPA engine.

The intent of this write-up was much simpler: to understand OPA in isolation, at a conceptual level, without immediately getting pulled into programming languages, microservice frameworks, application servers, databases, caches, or remote calls to other systems. Sometimes it helps to look at a tool without the surrounding ecosystem first.

And for that purpose, this small hands-on flow gives a pretty good first feel for how OPA works. Once you see policy, data, and input flowing separately, the overall model becomes much easier to appreciate. What makes it interesting is how changing just the data can immediately change the decision without touching the policy itself - and that is exactly where its real strength starts to show in larger systems. For anyone curious about externalizing authorization logic and keeping access control cleaner, OPA is definitely worth exploring further.
