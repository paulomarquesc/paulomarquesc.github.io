---
layout: post
title: Hello World - First Post on GitHub Pages!
excerpt_separator: <!--more-->
comments: false
---
{% assign postPath = page.path | remove: '%0A' | remove: '_posts' | remove: '.md' %}

First blog post on GitHub Pages with Jekyll. Contains references on how to get started the way I did and other references as well, e.g. adding to Bing and Google to be crawled. Also, how to add pictures and code (this is for my own reference as well :-) ).

<!--more-->

![helloworld_ps](/assets/{{postPath}}/hello-ps.png)

{% highlight powershell %}
Write-Host "Hello World!" -ForeGround Green
{% endhighlight %}

![helloworld_sh](/assets/{{postPath}}/hello-sh.png)

{% highlight bash %}
echo "Hello World!"
{% endhighlight %}

![helloworld_cs](/assets/{{postPath}}/hello-csharp.png)

{% highlight csharp %}
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
            Console.ReadKey();
        }
    }
}
{% endhighlight %}


Hello world from GitHub Gist.

{% gist paulomarquesc/7f2d3a4ef4e43a4edb40ab4b7b190f09 %}

Testing Jekyll Locally
```
bundle jekyll serve
```

## References

Following links helped me build/maintain this new blog site:

* [Jekyll](https://jekyllrb.com/)
* [Build A Blog With Jekyll And GitHub Pages](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/)
* [Jekyll cheatsheet](https://devhints.io/jekyll)
* [Jekyll Gist Plugin](https://github.com/jekyll/jekyll-gist)
* [How do I use disqus comments in github pages blog (Markdown)?](https://stackoverflow.com/questions/21446165/how-do-i-use-disqus-comments-in-github-pages-blog-markdown)
* [Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll)
* [Could not locate Gemfile or .bundle/ directory](https://forestry.io/docs/troubleshooting/could-not-locate-gemfile-or-bundle-directory/)
* [How to Setup Jekyll Sitemap](https://www.supertechcrew.com/setup-jekyll-sitemap/)
* [Social media Share icons for Jekyll](https://sourabhbajaj.com/blog/2017/10/29/adding-social-media-share-icons-to-jekyll/)
* [Jekyll Compose](https://github.com/jekyll/jekyll-compose) and [fatal: 'jekyll draft' could not be found](https://github.com/jekyll/jekyll-compose/issues/58)

Webmaster tools from search engines:

* [Bing](https://www.bing.com/webmaster)
* [Google](https://www.google.com/webmasters/tools)

Just drop both BingSiteAuth.xml and google`<xyz>`.html files into your not-compiled site (same folder as your _config.yml) and push into your repository. These files will get into _site folder when compiled by GitHub Pages.
To test access to these files just use your browser (they are case sensitive), for my GitHub Pages it will look like this:
* https://paulomarquesc.github.io/BingSiteAuth.xml
* https://paulomarquesc.github.io/google0af3453ea3d6d7ce.html

