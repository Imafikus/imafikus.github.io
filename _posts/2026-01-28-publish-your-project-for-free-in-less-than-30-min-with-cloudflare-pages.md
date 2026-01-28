## Introduction

Over the years I tried a bunch of different free (or almost) free hosting services, and noticed that people somehow slept on Cloudflare as a hosting provider.

While you might know Cloudflare as a company that specialized in internet infrastructure, they have a pretty decent cloud offering as well.

One of those offerings is called [cloudflare pages](https://pages.cloudflare.com/). It automatically deploys your code to Cloudflare CDN network (makes site load fast) and it gives you _.pages.dev_ domain for whatever you want to deploy.

## How to setup

- Create a Cloudflare account if you already don't have one
- In the upper left corner, there is _+ ADD_ button, click on it and choose _Pages_
- Choose _Import an existing Git repository_ option
- Choose the language / framework and build settings that are needed
- That's it, your code will be automatically deployed on every push, domain will be _name-of-the-repo.pages.dev_
- You are now ready to show your site to the world
- ???
- Profit

**Important:** You can connect your existing domain directly to your project. If you already have domains on Cloudflare, this is just click of a button, otherwise, simply follow the steps for your specific provider.

## Pro Tips

Since I'm running pretty much every single site / project I have this way, here are some tips I picked up along the way.

### Testing your changes before merging to master

By default, every commit on every branch will have its own deployment (awesome, right), so you can quickly test out if everything is working

### Setting up staging and prod environments

If you have complete staging setup and you want a stable URL to the testing page - Simply create another cloudflare pages project, and import your repo again, just this time redirect to a different subdomain.

**Important:** For this to work, you need to set up a custom domain.

## Where it can get complicated?

- If you have a backend, you'll need to figure out the best way to set that up, since it cannot be hosted this way, but [cloudflare workers](https://workers.cloudflare.com/) might be of help.
- Depending on the framework, you might have a harder time getting the build to pass - Next.js specifically requires some tinkering

## Conclusion

This is a fast and simple way of deploying your project to the outside world, especially since you can send the link to people and get feedback immediately.

If you already have your git repo, and cloudflare account, you'll be off to the races in no time.

_Hope this helps!_

_Until next time o7_
