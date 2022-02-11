## Automated Release Pipeline for my Blogs

I frequently blogs on two platforms medium and hashnode, links to my blogs are
- [ksingh7.medium.com](https://ksingh7.medium.com/)
- [dev.to/ksingh7](https://dev.to/ksingh7)

This repository is the single source of truth for all my blogs

## How i Automated my blogs

- Copy the `template.md` file under the posts directory and give it a sane name
- Write content in your new `blogs/automated-release-pipeline-for-my-blogs.md`
- Git add/commit/push
```
git add . ; git commit -am "blog : Automated release pipeline for my blogs" ; git push origin main
```
- Head over to [Github Actions UI](https://github.com/ksingh7/blogs/actions)
- Let the Github Action Complete
- Head over to your blog properties (login required)
  - [ksingh7.medium.com](https://medium.com/me/stories/public)
  - [dev.to/ksingh7](https://dev.to/dashboard)

## Blog Template

```
---
title: Your Title Goes Here
description: Your Description goes here
tags: 'productivity, beginners, test'
# cover_image: ./assets/cat.jpeg (currently does not work)
canonical_url: null
published: false
---
![Cover Image](https://images.unsplash.com/photo-1644333192059-10ec15101699)

Some random text with a [link](https://code.visualstudio.com).

## Serious title

Add some text here and there!

![and some pictures too](./assets/cat.jpeg)

## Some Code Snippet

```python

def myFunction()
  return true
  
```
