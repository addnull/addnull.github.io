---
layout: post
title: Build a Blog with Jekyll, GitHub Pages and CloudFlare
modified: 2015-12-13
tags: [cloudflare, github pages, jekyll]
---
/* Last modified at {{ page.modified | date: '%B %d, %Y' }} */

# 환경

이 글의 내용은 아래와 같은 환경에서 확인된 내용이다.

{% highlight bash %}
OS    : Mac OS X (10.11.1)
Xcode : 7.1
Ruby  : 2.0.0p645

Installed Gems:

bigdecimal (1.2.0)
blankslate (2.1.2.4)
CFPropertyList (2.2.8)
classifier-reborn (2.0.3)
coffee-script (2.4.1)
coffee-script-source (1.9.1.1)
colorator (0.1)
execjs (2.6.0)
fast-stemmer (1.0.2)
ffi (1.9.10)
io-console (0.4.2)
jekyll (2.5.3)
jekyll-coffeescript (1.0.1)
jekyll-gist (1.3.4)
jekyll-last-modified-at (0.3.4)
jekyll-paginate (1.1.0)
jekyll-sass-converter (1.3.0)
jekyll-sitemap (0.9.0)
jekyll-watch (1.3.0)
json (1.7.7)
kramdown (1.9.0)
libxml-ruby (2.6.0)
liquid (2.6.3)
listen (3.0.3)
mercenary (0.3.5)
minitest (4.3.2)
nokogiri (1.5.6)
parslet (1.5.0)
posix-spawn (0.3.11)
psych (2.0.0)
pygments.rb (0.6.3)
rake (0.9.6)
rb-fsevent (0.9.6)
rb-inotify (0.9.5)
rdoc (4.0.0)
redcarpet (3.3.3)
rouge (1.10.1)
safe_yaml (1.0.4)
sass (3.4.19)
sqlite3 (1.3.7)
test-unit (2.0.0.0)
toml (0.1.2)
yajl-ruby (1.2.1)
{% endhighlight %}

# 시작

Blogging을 시작하자.

거의 10년만에 다시 시작하는거라 어색하지만, 작고, 가볍게하자.

내용은 객관적이고 기술적인 것들만 채울 생각이다.

첫번째 글에서는 손풀기 겸으로 이 blog를 어떻게 만들었는지 정리한다.

# 선택

먼저 10년 전에 비해서 지금은 blogging 방법이 다양해서, 어떤걸 선택할지 고민했는데, 아래와 같이 선택했다.

| Category | Choice |
|:-|:-|
| domain | 원래 가지고 있던 '[addnull.net](https://addnull.net)' |
| -
| domain hosting | 국내외 여러 업체를 써봤지만, 딱히 별 차이를 느끼지 못해서<br/>원래 쓰던 업체([tt.co.kr](https://tt.co.kr)) 유지 |
| -
| server hosting | [GitHub Pages](https://pages.github.com) |
| -
| static site generator | [Jekyll](https://github.com/jekyll/jekyll) with [neo-hpstr-jekyll-theme](https://github.com/aron-bordin/neo-hpstr-jekyll-theme) |
| -
| CDN | [CloudFlare](https://cloudflare.com) |
| -
{: rules="groups"}

# 만들기

가장 먼저, blog hosting server를 설정해야하는데, [GitHub Pages 첫화면의 간단한 안내](https://pages.github.com)를 따라 Git repository를 생성한다.

나는 Git repo 이름을 "addnull.github.io"라고 정했고, 그렇기 때문에 Git URL이 [https://github.com/addnull/addnull.github.io](https://github.com/addnull/addnull.github.io)로 정해졌다.

그 후에 local(laptop) shell에서 아래의 명령어를 실행한다.

{% highlight bash linenos %}
mkdir -p $YOUR_PLAYGROUND
cd $YOUR_PLAYGROUND

git clone https://github.com/addnull/addnull.github.io
cd addnull.github.io

sudo gem install jekyll
sudo gem install jekyll-sitemap

curl "https://codeload.github.com/aron-bordin/neo-hpstr-jekyll-theme/zip/master" -o "a.zip"
unzip a.zip

# unzip에서 생성된 불필요한 root directory 삭제
mv neo-hpstr-jekyll-theme-master/* neo-hpstr-jekyll-theme-master/.* .
rmdir neo-hpstr-jekyll-theme-master

rm a.zip

# web browser에서 "http://127.0.0.1:4000"에 접속해서 정상 동작 확인한다.
jekyll server

# 'ctrl-c'로 kill Jekyll server

# '_config.yml'등의 내용을 적당히 수정한다.

git add .
git commit -m "..."
git push
{% endhighlight %}

여기까지 하면, "http://addnull.github.io"에 접속해서 정상 동작 확인한다.

# 한걸음 더 나아가기

이제 blogging 하기엔 문제가 없는 상황이지만, 접속 주소를 "http://addnull.github.io" 대신에 내 domain을 쓰고 싶다.

그리고 http가 아니라 https로 접속할 수 있도록 하고 싶기 때문에, 아래와 같은 구조를 선택했다.

<a href="/images/github-pages-with-cloudflare.png">
    <img src="/images/github-pages-with-cloudflare.png" alt="">
</a>

먼저, 아래처럼 Git repo에 'CNAME'이라는 이름의 파일을 추가한다.

{% highlight bash linenos %}
echo "addnull.net" > CNAME
git add CNAME
git commit -m "..."
git push
{% endhighlight %}

이제 "http://addnull.github.io"로 접속하면 "http://addnull.net" 주소로 redirect(HTTP status code 301)된다.

[CloudFlare](https://cloudflare.com)에 접속해서 "addnull.net"에 대한 DNS record(Type A)를 추가한다. (아래 screenshot 참고)

여기서 추가할 A record 값은 [GitHub Pages 문서](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/)에 나온다.

<a href="/images/cloudflare-dns-record-addnull.net.png">
    <img src="/images/cloudflare-dns-record-addnull.net.png" alt="">
</a>

다음으로 domain hosting service(내 경우엔 [tt.co.kr](https://tt.co.kr))에서 name server 정보를 CloudFlare로 바꿔야한다.

CloudFlare의 name server 값은 사용자 계정마다 다를 수 있으므로 CloudFlare에 접속 후, DNS 항목에서 확인한다.

위 screenshot에 나오듯이 내 CloudFlare 계정의 name server는 아래와 같다.

- diva.ns.cloudflare.com
- igor.ns.cloudflare.com

DNS 정보 변경은 다소 시간이 걸리는데, 경험상 늦어도 24시간 내에 반영된다.

마지막으로 다시 CloudFlare에 접속해서 page rules 항목에 http로 접속했을 경우 https로 자동 forwarding하는 rule을 추가한다. (아래 screenshot 참고)

<a href="/images/cloudflare-page-rule-addnull.net.png">
    <img src="/images/cloudflare-page-rule-addnull.net.png" alt="">
</a>

# 마무리

약간의 시간, 노력과 비용(domain 구매)만 들이면, 그럴싸한 blog system을 구축할 수 있다.

이제 꾸준히 blogging 하는 일만 남았다.

<br/>
<br/>
<br/>
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- blog_0000 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-2574234961505557"
     data-ad-slot="6369673644"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
