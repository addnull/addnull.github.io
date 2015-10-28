---
layout: post
title: Wildcard Subdomains with DNSMasq
modified: 2015-10-29
tags: [subdomain, dnsmasq]
---
/* Last modified at {{ page.modified | date: '%B %d, %Y' }} */

# 환경

이 글의 내용은 아래와 같은 환경에서 확인된 내용이다.

{% highlight bash %}
OS      : Ubuntu 14.04.3
         (http://cloud-images.ubuntu.com/vagrant/trusty/ 에서 20151021)
DNSMasq : 2.68
{% endhighlight %}

# 시작

최근에 OpenStack Swift로 AWS S3와 호환되는 object storage를 구축한 적이 있다.

여기서 지원해야하는 AWS S3의 기능 중에 'Subdomain Calling Format'이라는 bucket 이름의 기반한 public URL을 생성해주는 기능이있다.

즉, AWS S3에서는 아래 두 URL이 동일한 의미를 가진다.

| Calling Format | URL |
|:-|:-|
| Subdomain | http(s)://[bucket].s3.amazonaws.com/[object] |
| -
| Ordinary | http(s)://s3.amazonaws.com/[bucket]/[object] |
| -
{: rules="groups"}

# 문제

DNS에 domain을 등록할 경우 'Subdomain Calling Format' 지원에 큰 어려움은 없다.

하지만 개발 환경(DV phase)의 경우, local(laptop)에 Vagrant VM을 띄워서 내부에서만 사용하는 dummy domain을 '/etc/hosts/'에 등록해서 사용하고 있었고, 이 방법은 subdomain을 지원하지 않기 때문에, 가능한 모든 bucket 이름마다 '/etc/hosts'에 등록해줘야하는 문제가 있다.

예를 들면, 아래와 같다.

{% highlight bash linenos %}
#
# /etc/hosts
#
127.0.0.1 aaaa.dummy_domain.com
127.0.0.1 bbbb.dummy_domain.com
127.0.0.1 cccc.dummy_domain.com
...
{% endhighlight %}

# 해결

해결방법으로, local에 내부 DNS를 구축하는 것도 해결 방법이겠지만, 더 간단하게 'DNSMasq' service로도 해결할 수 있다.

설정 방법도 '/etc/dnsmasq.conf'에 'address=/.dummy_domain.com/127.0.0.1'를 추가하고 service restart만하면 된다. (아래 script 참고)

{% highlight bash linenos %}
#!/usr/bin/env bash
set -e

apt-get -y install dnsmasq

cp /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
echo 'address=/.dummy_domain.com/127.0.0.1' >> /etc/dnsmasq.conf

service dnsmasq restart
{% endhighlight %}

# 마무리

알고 보면 간단한 내용이지만, 나중에 필요할 때 바로 쓸 수 있도록, 내 [GitHub](https://github.com/addnull/miscellaneous/blob/master/wildcard_subdomains_with_dnsmasq/)에 Vagrantfile과 bash script를 올려두었다.

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
