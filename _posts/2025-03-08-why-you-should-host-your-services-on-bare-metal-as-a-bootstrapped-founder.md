# Why you should host your services on bare metal as a bootstrapped (technical) founder

Founding a startup (especially bootstrapped startup) is all but an easy thing to do. There is fire everywhere. On multiple fronts. Constantly. So when someone mentions you that you should move to bare metal it can certainly feel like adding gasoline to the already raging inferno.

As a startup founder both cost and developer velocity should be on your mind constantly. You should find all means possible to stay lean, and on the other hand, you need to have the ability to ship a new feature to production within minutes of finishing it.

Ideally you need to be set up for success right from the start, and as paradoxically as that sounds, bare metal deployments are the way to go.

## Free Credits Bait

Most of the bootstrapped startups usually start with free credits for one of the large cloud providers in the market. All of them offer magical one click deployments, free resources and bunch of premade tools. All of those you’ll wholeheartedly accept of course, because you want to move fast and ship features as soon as possible.

And this works. It’s awesome. Everything is so simple. You don’t need to do anything. And right after setting in into this new awesome infrastructure you set up for yourself, your free credits have run out and you start to notice the cost - But that’s fine, it’s not that bad, you don’t have many users. It’s fine, and it will continue being fine, right? right???

At one point you are starting to get users, everything is going according to plan, but your cost is slowly but surely rising up along with the number of users and very quickly your cloud bill is not so small anymore.

All that while running most likely a single database instance, couple of microservices and a load balancer.

Don’t trust me? Here is the empirical proof:

![Pricing Report](/docs/assets/images/2025-03-08-why-you-should-host-your-services-on-bare-metal-as-a-bootstrapped-founder/image.png)

Graph you are seeing is our Google Cloud bill that starts in January ‘22 and it end on January ‘24. As you can probably see, all was fine and dandy for the first year or so (mostly because we didn’t have any traffic). And then, starting in 2023, cost starts rising, and it’s rising fast. Our bill went from ~10$ a month to ~40$ in the span of one year, and then it again doubled in less than one year. That’s an 8 times the increase. We didn’t have 8 times more users.

At that point, we were running 8 microservices, bucket storage with less than < 1TB of data and a single datastore instance. We also used GCP pub/sub service.

We were also heavily optimizing our costs here. That meant running our services with just enough memory and with just enough instances so we could save as much money as possible. Of course, this caused some problems since sometimes our services would just OOM if there was a spike in traffic, or some larger than usual request. Additionally, we configured our bucket storage to store only last 3 months of data to save on cold storage costs. And that optimization caused us to waste time on it instead of actually building new features / improving existing ones.

## Vendor Lock In

But I’ll just migrate my infrastructure somewhere cheaper once I start getting users, what’s the big deal? - No, you will not.

You’ll very soon realize that all that magical clicking and set up you did *cannot be duplicated easily***.** Because new cloud has it’s own magical clickable stuff, which is different than your current magical clickable stuff. So you need to spend considerable amount of time just to replicate the existing setup, without any other improvements.

You just got vendor locked without realizing it.

It’s also worth noting that all cloud providers have their equivalents of commonly used services (queues, serverless functions, databases, logging mechanisms, etc…), and I can guarantee you that almost all of those are specific to that one provider so you cannot easily switch from Google Cloud functions to AWS lambdas for example.

Use already existing open-source stuff instead, it can be hosted anywhere, and you’ll be a little less locked in.

Just to give you a sense of the time scale needed for the migration. At Notify Me, it took us ~2 months of active work to move away from Google Cloud. And have in mind that 2/3 of our team has extensive DevOps experience needed to do that kind of migration. We are still somehow being billed for 2 cents per day and we have no idea why.

## You’ll become a better engineer

But but but… Shouldn’t we focus on improving our product and delivering value to our customers instead of doing a huge migration of our codebase to a cheaper set up.

You should, but you should also focus on becoming a better engineer and hosting your own stuff is one of the best ways to accomplish that.

You’ll learn that there is no magic required to set up a database instance, or a queue service. Or that there is, if by definition of magical you mean docker containers.

You’ll also get in depth knowledge of all the parts of your tech stack, from bare metal level, up to latest frontend framework(s) you are currently using.

## Stupidly cheap servers

And onto the most imporant thing - Bare metal servers are  **c h e a p**. Seriously. Just find a service that offers VMs for rent. We use [Hetzner](https://www.hetzner.com/cloud/) for that.

Just to give you an idea of what the cost differences are between using the equivalent VM on Google Cloud and Hetzner:

- E2 machine on google cloud, 4vCPU 8gb RAM - $87.64/ mo
- CAX21 machine on hetzner, 4vCPU 8gb RAM - $7.59/ mo

**Price difference is 11x**

But how is this possible? - Because, with Hetzner (or any similar provider), you get only VM with an SSH access and some storage. You don’t get 326 other different offerings and APIs and you are the only one responsible for setting up whatever infra / services you need. Also, you’ll come to realize that you most likely need less than 10 different services for everything you are going to run.

### The usual setup you’ll need

- Way to run your code in a container - go with docker / k8s
- Database - Postgres or Mongo will do just fine
- Queueing mechanism - Redis / RabbitMQ
- Long term storage - S3 buckets of any kind will do (Hetzner has super cheap ones)
- Reverse proxy - ngnix or caddy

All of these are readily available as docker images.

## To Conclude

If you are a technical founder who is willing to put away time to learn how to deploy his own infra, you’ll learn a lot in the process and save some money.

Managed cloud solutions are also completely fine, just try to avoid vendor lock in whenever possible, and be prepared to pay for a bunch of services you probably won’t need. Also, make sure to keep an eye for rising costs when you hopefully start to scale.

Open source solutions are more than enough for everything you might need. There is a reason bunch of open source tooling / infra are the industry standard.

Until next time 🫡
