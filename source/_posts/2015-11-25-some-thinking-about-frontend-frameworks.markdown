---
layout: post
title: Some thinking about frontend frameworks
date: 2015-11-25
---

Recently I am working on a frontend project, I created it 2 months ago. Kind of urgent... I thought this was a temper project. Actually it's not. There are only three pages. The one really bothers me is that people change the requirements all the time. I wasted some time on this temper project. And I don't want to waste my time anymore.

So I decide to refactor this project and then deliver it to another team to maintain. For now, this project doesn't use any famous frontend frameworks, like Angularjs, emberjs, or reactjs. Just bootstrap and jquery~ And less, gulp, browserify, node, jade.

I dive into the modern frontend frameworks for a while. I used angularjs in a project before. So I want to taste another one. React is cool. But my project doesn't have many dom manuplation. So no.

So how about ember? Ember js for ambitous web app. Sounds good. So I decide to move my project to an ember one. And not successful. Why?

Because my project is way small. It's created with a simple purpose of activity pages. It's even not a web app! I can't use any frameworks of their good parts. For example, a normal web app only has one index page. Right? This makes sense. But this project doesn't have an index thing! Actually, every page is an index page because of nginx proxy

I have to say, all the three frameworks are greate jobs. Sometimes I just think about which one I need. What I can get from the good part? Not the way I can't use.

So for the activity pages. I think just keep it with plain jquery and bootstrap is just fine.
