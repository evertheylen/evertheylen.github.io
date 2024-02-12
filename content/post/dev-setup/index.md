---
title: My Desktop+Tablet Development Setup
date: 2024-02-11
image: tablet_with_desktop.jpg
tags:
    - linux
    - hardware
---


# The problem

My requirements are as follows:

  1. My main development environment has to be Linux. (Not just WSL, I want a proper Linux desktop environment.)
  2. I like to sketch stuff and draw diagrams. Paper works, but it gets disorganized really quickly. A digital device allows me to keep my notes organized and you can have an infinite canvas with nice colors and edit history. (Note: I don’t think there are good apps like this on Linux.)
  3. I want to build my own desktop. You can get a lot more performance for your money, and I can hand-pick each component to my liking (e.g. a good CPU, a mid-tier GPU, stellar Linux support).
  4. While I spend most of my time working from home, I do have to work in other places sometimes. I don’t want to maintain duplicate environments.

Requirement 2 suggests a tablet, 3 suggests a desktop, and 4 suggests a laptop. A windows laptop-tablet hybrid may be a compromise solution but I tried it and did not like it [^1].

[^1]: I once had an [HP ZBook Studio G5](https://support.hp.com/us-en/document/c06010885), which seemed like a powerful laptop with a nice large pen-supported touchscreen. One problem, Linux support was really sketchy. I really tried to make it work, even manually compiling chromium to include Wayland support. Even then I couldn’t find a good note-taking app. So I settled on Windows with WSL but just plain didn’t like it. Then one day I opened my laptop to a big advert for some XBox service and that settled it -- no more Windows.


# The solution

I have two devices:

  - A desktop I built myself, which runs Linux.
  - A Samsung Tab S7+, on which I run OneNote.

This setup satisfies requirements 1, 2 and 3. But how do I satisfy requirement 4? It’s a combination of three things:

  - [code-server](https://github.com/coder/code-server), which lets you run Visual Studio Code in any browser.
  - [DeX mode](https://www.samsung.com/us/apps/dex/) on the Samsung tablet, although not technically required, gives me a traditional desktop environment. I also have the keyboard accessory.
  - A bunch of networking configuration so I can reach my desktop from the internet. I only expose a single SSH port, and use SSH tunnels for anything else. (I use [duckdns](https://www.duckdns.org/) and [JuiceSSH Pro](https://juicessh.com/) to make it work).

This allows me to remotely connect with code-server (running on my desktop) through an SSH tunnel. In the end I’m able to work using the exact same editor as if I were on my desktop. This isn’t rocket science of course, it’s kinda like a home-built version of [Github Codespaces](https://github.com/features/codespaces) where the server lives in your house.

This is fine for “simple” development where you never leave your IDE or terminal. Web development is a little harder because you really want to debug your frontend using devtools. While forwarding the ports to [vite](https://vitejs.dev/) (or whatever you want to use) is no problem, most browsers on Android assume you do all your debugging work remotely via USB or the network. The solution is [Kiwi Browser](https://kiwibrowser.com/), which has full devtools available.

# Conclusion

I’ve been using this setup for more than 2 years now and I really like it. It’s not for everyone as it requires a little work to set up and maintain, but for me it combines the best of Android with the best of (Desktop) Linux leading to a very smooth experience.

If you have any questions about this setup, feel free to [contact me](mailto:evertheylen@gmail.com)!
