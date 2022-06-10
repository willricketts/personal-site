---
layout: post
title: Obsidian Changed Everything for Me
description: Obsidian Changed Everything for Me
---

### What is Obsidian?

The [Obsidian website](https://obsidian.md) describes it as **"A second brain for you, forever."** While this certainly sounds enticing,
its actual description is a bit more useful:

> ...a powerful knowledge base on top of a local folder of plain text Markdown files.

To put it even more plainly, Obsidian is both a collection of folders and markdown files on your local filesystem and a piece of
software that views them and can interconnect them in a really slick way. Their thesis is that traditional forms of note taking and conventional
knowledgebases are typically highly linear and in that regard, very much unlike the human brain, which is graph-like in nature. Obsidian
leans into this idea so heavily that it ships with a graph viewer of your entire knowledgebase, referred to as a "vault."

<sub>_An example from their site:_</sub>

![](https://obsidian.md/images/screenshot.png)

### Why I needed Obsidian

The way I stored information was a total mess and I was constantly forgetting about things due to not being able to find them. [Trello](https://trello.com) was a game-changer for me for a long time, and I happily spent a very long time as a paid-tier customer, but over time I found it to be lacking. There wasn't a great deal of support for interconnecting cards from different boards. Sure it can do this, but being able to zoom out to a ten thousand foot view and see your entire data set as a graph is next-level visibility. Obsidian gives me the tools to be able to map out my thoughts as I do in my head, which in my view is immeasurably powerful.

On top of the features it ships with, it's extensible with custom plugins, and some of the open-source plugins created by the community are phenomenal. My absolute favorite plugin is [obsidian-kanban](https://github.com/mgmeyers/obsidian-kanban) by [mgmeyers](https://github.com/mgmeyers). This plugin enables you to turn any plain old markdown document into a full Kanban board, essentially a full Trello clone, except there's no central server. The entirety of this board lives in your local filesystem and can thus be committed to source control such as [Git](https://git-scm.com). Inside each Kanban board in your vault, each card can be just a simple blurb of text or a link to an entire markdown document, the relationship of which will be displayed in the graph.

<sub>_From the obsidian-kanban Github repo:_</sub>

![](https://camo.githubusercontent.com/51a7f221ca5af26447a2924866adc9e467ba95f595af665a0ef8a0f8739e1296/68747470733a2f2f6d6174746865776d6579652e72732f6f6273696469616e2d6b616e62616e2f4173736574732f53637265656e25323053686f74253230323032312d30392d3136253230617425323031322e35382e3232253230504d2e706e67)

### Why you need Obsidian

It's simple-- because it provides an entirely new way to think about how your data, knowledge, memories, and important documents are stored and related.

_Wait what? Important documents?_

Yep. We're going to cover that too once we talk about `git-crypt`.

An Obsidian vault is highly flexible and versatile. You can use it to store virtually anything from collections of recipes to an internal blog, or something like daily standup meeting notes which is something I use it for daily so that I keep a historical record of my daily standup posts on Slack (I'm a fully remote worker, so our standup updates are posted in a dedicated channel in slack every morning). Additionally, as part of devops / security ops work I've been doing, I send regular reports containing compiled lists of application exceptions with triage notes, etc. Obsidian helps me keep track of all of my application triage notes throughout the day so that they're very easily composable into a report at the day's end. I feel like nearly everyone out there especially in tech has some kind of use case similar to these.

Obsidian is free. There's no central service, no connection-required functionality, no subscription, and no login to remember. Your vault is 100% local and isolated to your machine unless you decide to replicate it to another machine or mobile device using something like Git. The only paid functionality offered by Obsidian is a "sync" service that enables it to synchronize your vault across multiple devices. I haven't even considered paying for this as it's not a big deal to me to manually update my vault on my various devices as Git makes that super easy. Plus, I feel like anyone who gains real value from looking at a graph to remember things probably isn't the optimal demographic for cheeky paid convenience features.

That being said, Obsidian costs nothing to try and doesn't require you to wrangle yet another login to yet another service that might leak your data in yet another data breach. 

### How you should set up Obsidian

So you've decided to take the plunge and give it a go. Firstly, you can create a new vault or convert an existing directory into an Obsidian vault. This can be really useful for migrating your existing notes into Obsidian. Once your vault is created, you'll be able to drag and drop your existing markdown notes around into folders and a structure you can create within the Obsidian application.

![](/assets/img/obsidian-new-vault.png)

If you're creating a brand new vault from scratch and not converting an existing directory into a vault, you'll need to specify a name and location on the filesystem for it to live. This name will be important when we secure the vault for git.

![](/assets/img/obsidian-create-vault.png)

### How you should store Obsidian

My wife and I recently returned from Iceland, and while I was there, I used Obsidian exclusively to store, track, and manage our boarding passes, itinerary, hotel / bus / rental booking confirmations, etc. I keep my vault in a private Git repository, but I also encrypt all of the data within my vault such that not even the source control service I use can peer inside and invade my privacy. This is super easy.

We're going to use [git-crypt](https://github.com/AGWA/git-crypt) to privately store our Obsidian vault on Github. We'll begin by installing `git-crypt` as described in the [installation documentation](https://github.com/AGWA/git-crypt/blob/master/INSTALL.md) found in its [Github repo](https://github.com/AGWA/git-crypt/). 

For this, you're going to need a PGP keypair. If you're unfamiliar with PGP, [this video](https://www.youtube.com/watch?v=DMGIlj7u7Eo) should get you sorted out.

If you're running MacOS, `git-crypt` can be installed very simply with [Homebrew](https://brew.sh/):

```
brew install git-crypt
```

Then initialize your Obsidian vault as a Git repository:

```
cd YourVaultName && git init
```

Once `git-crypt` is installed, we'll need to create a `.gitattributes` file at the top level of your Obsidian vault directory that looks like the following, replacing `YourVaultName` with the name of your vault as it appears in the filesystem:

```
secretfile filter=git-crypt diff=git-crypt
*.key filter=git-crypt diff=git-crypt
YourVaultName/** filter=git-crypt diff=git-crypt
YourVaultName.md filter=git-crypt diff=git-crypt
.obsidian/** filter=git-crypt diff=git-crypt
```

Once this is created, `cd` into your Obsidian vault, initialize `git-crypt`, and add yourself as an authorized user with the email you used to create your PGP keypair:

```
cd YourVaultName
git-crypt init
git-crypt add-gpg-user myname@myemail.com
```

At this point, everything should be configured for your vault to be PGP encrypted at rest on Github. Once you commit and push to Github, you'll notice that all of the data is rewritten. This is `git-crypt` doing its job. Once you've pushed up to Github, take a look around the repo in the Github web interface and ensure that all of the markdown documents in your vault are unable to be rendered by Github.

### That wasn't so bad huh?

🎉 We're done. Now you have a free, portable, resilient, encrypted, and version controlled digital brain that you can teleport anywhere in the world. Since you're leaning against PGP, you can be confident in storing even sensitive documents within your Obsidian vault.

I really hope you get the same kind of value I do out of Obsidian. I can only hope it's as good a fit as it has been for me. Good luck!

