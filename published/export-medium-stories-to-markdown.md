## Introduction
If you are like me, I mean a frequent tech blogger. You would like to keep all your blog posts in Markdown format. If you have not thought about this until now, then I invite you to give a serious thought. 

Nothing could beat, a blog post in Markdown format. It is a simple format that is easy to read and write. It is also a very good format for blogs, and brownie points to store those in a git repository.

I have been writing on Medium for a while now, and was using its inbuilt editor. Lately I have started to use hashnode and dev.to. I thought, why not move my Medium blog posts to dev.to and hashnode (both of them supports markdown). However, there was no easy way provided by Medium to export them to markdown.

## mediumexporter to the rescue
- Install node on your computer
- Then install mediumexporter node package
```
npm install -g mediumexporter
```
- Finally, export your medium stories to Markdown format using mediumexporter

```
mediumexporter https://ksingh7.medium.com/kubernetes-endpoint-object-your-bridge-to-external-services-3fc48263b776 > exported-blog.md
```
- This will create `exported-blog.md` file in your present working directory, which is the markdown file you were looking for.

### Summary
Like everything, this is not a perfect solution. But it is a good start. You might want to review the code/image formatting, before you can re-port this blog to other platforms.

Special Thanks to the creator of  `markdownexporter` that I used to export my Medium stories to Markdown format. Thanks a lot and well done my friend :)


