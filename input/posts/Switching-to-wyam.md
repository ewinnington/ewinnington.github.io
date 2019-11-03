Title: Switching to Wyam
Published: 23/10/2019
Tags: Wyam
---
After many years with a half abandoned Wordpress blog, I'm switching to [Wyam](https://wyam.io/) for a static site generator. 

This blog was generated as follows: 

```
dotnet tool install -g wyam.tool
wyam new -r blog
wyam -r blog -t Phantom -p -w
```

The last command makes wyam host the blog using the recipe blog, the theme Phantom (-t), in preview mode (-p) so that you can access it locally and in Watch mode (-w), so that you can see changes without having to re-run the build line. 

I also exported the previous posts from Wordpress and will be adding those that are not too outdated back here. 