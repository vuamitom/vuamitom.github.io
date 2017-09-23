---
layout: post
title: Create a Ghost blog for free
date: '2015-11-29 05:51:53'
category: trivial
tags:
- life
---

My blog on Blogger was created roughly a year ago. Blogger was a good quick-start for beginner blogger. It has most things setup only a few mouse clicks away. But over the year, it became clear that Blogger is not a good fit for my need. A ancient and clumsy web editor (which, to certain extend, can be remedied by [Stackedit](https://stackedit.io/)) , non eyes-pleasing default look and feel. And above all, Blogger is a hosted blogging platform, which means, all the posts I have written over the year resides in some Google databases that I have no control over. Not to mention the little customisation option other than a single theme file. 

Long story short, I decided to have my self-hosted blog, which I can poke at, tweak, stuffs that engineers often do. Among the options that was considered, e.g Jekyll + github, Wordpress, Ghost ... Ghost came across as the most suitable candidate, due to its minimalist style by default, comes with markdown editor. 

The guys at [Ghost.org](http://ghost.org) offers to host your blog for a monthly fee of $8, which is reasonable for most. Yet, for infrequent writer like myself, I don't want that recurring cost for something that has not yet grown into a habit. And as I said earlier, full control is what I want. Fortunately, the Ghost team has been graceful enough to open-source their blogging platform. Technically, anyone can get the source code and run their own Ghost server. 

Though setting up a server may sound very involving technically, there are only a few steps that you don't have to learn programming in order to do it. 

#### 1. Sign up for OpenShift

[Openshift](https://www.openshift.com/) is a free-for-starter PaaS solution. Once you go to their website, sign up, you will be given a web console. Select **Add Application...**, and then select **Ghost** in the list of Instant App. 

![](/content/images/2015/11/Screen-Shot-2015-11-29-at-14-37-36.png)

Enter your sub-domain. This is the url where people can access your blog. 

![](/content/images/2015/11/Screen-Shot-2015-11-29-at-14-41-33.png)

Leave other configuration default. Click **Create Application**, sit tight and wait for the process to finish. Once it's done, your blog is readily available at `http://<yoursubdomain>-homegrown.rhcloud.com`. 

#### 2. Setup an admin password
The above step gets your blog up and running. Like most blogs, only certain people have the rights to post, edit, and do stuffs. Since your blog is fresh out of the oven, you should claim to be its rightful owner before anyone else does. 

To do that, just go to `http://<yoursubdomain>-homegrown.rhcloud.com/ghost/`. You will be prompted to enter your admin email and password. This is a one time process only. And after that, nobody can dispute your ownership of the site. 

#### 3. Custom domain
If you don't like the url that Openshift gives you, you can purchase a custom domain and point that to your blog. However, this process requires a little code editing. Since I decided this post to be non-technical, and someone else has written about that, you can find instructions [here](https://www.insidersbyte.com/use-a-custom-domain-for-your-azure-ghost-blog/). 

Happy blogging! 