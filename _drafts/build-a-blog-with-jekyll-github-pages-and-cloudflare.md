---
layout: post
title: Build a Blog with Jekyll, GitHub Pages and CloudFlare
---
Blogging을 시작하자.

거의 10년만에 다시 시작하는거라 어색하지만, 작고, 가볍게하자.

내용은 객관적이고 기술적인 것들만 채울 생각이다.

첫번째 글에서는 손풀기 겸으로 이 blog를 어떻게 만들었는지 정리해보려한다.

### 선택

먼저 10년 전에 비해서 지금은 blogging 방법이 다양해서, 어떤걸 선택할지 고민했는데, 아래와 같이 선택했다.

- domain: 원래 소유하고 있던 '[addnull.net](https://addnull.net)'
- domain hosting: 국내외 여러 업체를 써봤지만, 딱히 별 차이를 느끼지 못해서 원래 쓰던 업체([tt.co.kr](https://tt.co.kr)) 유지
- server hosting: [GitHub Pages](https://pages.github.com)
- static site generator: [Jekyll](https://github.com/jekyll/jekyll)
- CDN: [CloudFlare](https://cloudflare.com)

### 만들기

가장 먼저, blog hosting server를 설정해야하는데, [GitHub Pages](https://pages.github.com)에 나온 간단한 guide를 참고해서 git repository를 생성한다. 

그 후에 local(laptop) shell에서 아래의 과정을 거친다.

{% highlight bash linenos %}
cd $PATH_TO_YOUR_PLAYGROUND

git clone $YOUR_GITHUB_PAGES_REPOSITORY
cd $YOUR_GITHUB_PAGES_REPOSITORY

sudo gem install jekyll
sudo gem install jekyll-sitemap

{% endhighlight %}
