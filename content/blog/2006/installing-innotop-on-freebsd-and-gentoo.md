---
title: Installing innotop on FreeBSD and Gentoo
date: "2006-11-08"
url: /blog/2006/11/08/installing-innotop-on-freebsd-and-gentoo/
categories:
  - Databases
  - Open Source
---
I recently got a message letting me know FreeBSD users will soon be able to install the innotop MySQL and InnoDB monitor through ports. Gentoo GNU/Linux users can find innotop in Portage.

### FreeBSD adds innotop to ports

When this is finalized, FreeBSD users will be able to install innotop with the following commands:

```
cd /usr/ports/databases/innotop
make all install
```

This is great news. It makes innotop easier to find, install and use. It also means a lot to me that a FreeBSD maintainer thinks innotop is worth including in ports.

You can track the status of the PR at [ports/104722: New port: databases/innotop](http://www.freebsd.org/cgi/query-pr.cgi?pr=ports/104722).

### innotop ebuilds for Gentoo

I've had an [open request to add innotop on the Gentoo bug system](http://bugs.gentoo.org/show_bug.cgi?id=147600) for a long time, and it appears to have stagnated. Since I'm no Gentoo developer, please do give feedback if you have any.

Since the Gentoo bug doesn't seem to be getting any attention, I've also opened an [innotop new-ebuild bug report with the fine folks at breakmygentoo](https://bugs.breakmygentoo.net/view.php?id=257). This might result in innotop being included in their Portage overlay.

**Edit** One of the Gentoo developers saw this post and added innotop to the main Portage tree! Thank you! Gentoo users can now install innotop like this:

```
emerge innotop
```

Yay!


