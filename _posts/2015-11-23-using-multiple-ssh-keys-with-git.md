---
layout: post
title: Using Multiple SSH Keys with Git command
modified: 2015-11-24
tags: [ssh, git]
---
/* Last modified at {{ page.modified | date: '%B %d, %Y' }} */

# 환경

이 글의 내용은 아래와 같은 환경에서 확인된 내용이다.

{% highlight bash %}
OS  : Mac OS X (10.11.1)
Git : 2.4.9
Zsh : 5.0.8
{% endhighlight %}

# 시작

여러 Git repository('GitHub', 'Bitbucket' 그리고 사설 Git server들)를 동시에 사용하다보면, 보안을 위해서라도, repository 별로 다른 'ssh key'를 쓰고 싶어진다.

또는 어떤 Git repository에서는 'ssh key' import를 지원하지 않기 때문에, 어쩔 수 없이 다른 'ssh key'를 사용해야할 때도 있다.

하지만 아쉽게도, 이렇게 설정하기 위해선, 단순히 Git 설정 값 몇 개를 추가하는 정도가 아닌, 별도의 shell script를 작성해야한다.

# 해결

'~/.zshrc'에 아래 내용을 추가한다. 만약 'Zsh' 대신에 'bash'를 쓴다면, 일반적으로 Max OS X에서는 '~/.bash_profile' 그 외는 '~/.bashrc'에 추가한다.

{% highlight bash linenos %}
export GIT_SSH="/PATH/TO/YOUR/PLAYGROUND/git_ssh.sh"
{% endhighlight %}

이제 'git_ssh.sh' 내용을 아래와 같이 작성한다.

{% highlight bash linenos %}
#!/bin/bash

# git-upload-pack ...
# git-receive-pack ...

KEY_PATH=/PATH/TO/YOUR/KEY/DIRECTORY/     # your private key file directory
LOG=/PATH/TO/YOUR/PLAYGROUND/log_git      # for debugging

echo -e "==========" > $LOG
echo -e "\nnumber of parameters:\n\t"$# >> $LOG
echo -e "\nparamters:\n\t"$* >> $LOG
echo -e "\nR:" >> $LOG

###############################################################################
REGEX[0]="git@bitbucket.org git-.*-pack '.*.git'$";    KEY_FILE[0]="bitbucket.private";   MSG[0]="Bitbucket"
REGEX[1]="git@github.com git-.*-pack '.*.git'$";       KEY_FILE[1]="github.private";      MSG[1]="GitHub"

i=0
while [ $i -lt 2 ]; do
	echo -e "\t"${REGEX[i]} >> $LOG
	if [[ $* =~ ${REGEX[i]} ]]; then
		ssh -i $KEY_PATH/${KEY_FILE[i]} $*
		echo -e "\nuse key:\n\t"${MSG[i]} >> $LOG
		exit
	fi
	let i=i+1
done

###############################################################################
# others
ssh -i ~/.ssh/id_rsa $*
echo -e "\nuse key:\n\tunknown repository" >> $LOG
{% endhighlight %}

위의 shell script 내용을 보면, 'git clone' 할 때, 관련된 parameter 전체(문자열)를 regular expression으로 검사해서, 해당되는 case의 'ssh key'를 사용하게 만든 것이다.

혹시나 의도한대로 잘 동작하지 않았을 경우, debugging을 위해서, 추가로 'log_git'에 진행 내용을 기록하도록 했다.

# 예제

이제 'Bitbucket', 'GitHub' 그리고 'git_ssh.sh'에 명시되지 않은(unknown repository)에서 'git clone' 했을 때, 각각의 'log_git'을 보자.

### 'git clone git@bitbucket.org:aaaa/bbbb.git'

{% highlight bash linenos %}
==========

number of parameters:
	2

paramters:
	git@bitbucket.org git-upload-pack 'aaaa/bbbb.git'

R:
	git@bitbucket.org git-.*-pack '.*.git'$

use key:
	Bitbucket
{% endhighlight %}

### 'git clone git@github.com:aaaa/bbbb.git'

{% highlight bash linenos %}
==========

number of parameters:
	2

paramters:
	git@github.com git-upload-pack 'aaaa/bbbb.git'

R:
	git@bitbucket.org git-.*-pack '.*.git'$
	git@github.com git-.*-pack '.*.git'$

use key:
	GitHub
{% endhighlight %}

### 'git clone git@11.22.33.44:aaaa/bbbb.git'

{% highlight bash linenos %}
==========

number of parameters:
	2

paramters:
	git@11.22.33.44 git-upload-pack 'aaaa/bbbb.git'

R:
	git@bitbucket.org git-.*-pack '.*.git'$
	git@github.com git-.*-pack '.*.git'$

use key:
	unknown repository
{% endhighlight %}

# 마무리

여러 'ssh key'를 사용하기 위해서 다소 복잡한 과정이 필요했다.

하지만, 한번 설정해두니, 필요할 때마다 추가, 수정해서 만족스럽게 쓰는 중이다.

끝으로 궁금한 건, 'Git' 자체적으로 여러 'ssh key'를 사용할 수 있는 설정이 없다는게 좀 의아스럽다. (필요가 없거나, 아님 내가 못찾았거나.)
