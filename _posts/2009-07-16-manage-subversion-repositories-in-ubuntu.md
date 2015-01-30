---
layout: post
title: "우분투에서 서브버전 저장소 운영하기"
tags: [ Subversion,  Ubuntu ]
---

회사에서 사용하는 버전 관리 시스템을 Git으로 이전하기 전에, 나중에 또 언젠가 다시 필요할 지 모르는 일이므로, 7년동안 운영해 왔던 Subversion 서버 구성과 운영 방식을 기록해 둡니다. 물론, 우분투 리눅스 환경이고 로컬 / 외부 네트웍 모두에서 svn.example.com 주소로 어디에서든 서브버전에 접근할 수 있도록 환경을 구성하는데 목적이 있습니다.

제일 먼저 서브버전 패키지를 설치합니다.

    $ sudo apt-get install subversion

우분투처럼 데비안(Debian GNU/Linux) 기반 리눅스는 대부분 저장소(repository) 기본 디렉토리가 `/var/lib/svn` 디렉토리입니다. 배포판 규칙을 어기는게 괜히 꺼림칙하므로 이 디렉토리를 지우고 다른 디스크 또는 파티션에 있는 디렉토리를 심볼링 링크로 연결합니다. (여기서는 `/opt/svn` 입니다)

    $ sudo rm /var/lib/svn
    $ sudo ln -sf /opt/svn /var/lib/svn

HTTP 방식으로 서브버전을 운영하기 위해 설정합니다. 서브버전이 동작하는 서버는 외부 인터넷에 공개되지 않은 내부 서버입니다. 하지만 내부 서버는 아파치 뿌락찌(proxy) 기능을 이용해 외부에서도 접근 가능하도록 합니다.

어쨌듯 아파치(Apache) 웹 서버에서 HTTP 방식으로 접근할 수 있도록 서브버전 아파치 모듈을 설치하고 활성화합니다.

    $ sudo apt-get install libapache2-svn
    $ sudo a2enmod dav_svn

서브버전 아파치 모듈 설정(`/etc/apache2/mods-available/dav_svn.conf`)을 다음과 같이 수정합니다.

    <Location /svn>
    DAV svn

    SVNParentPath /var/lib/svn

    AuthType Basic
    AuthName "Subverion Repositories"
    AuthUserFile /etc/subversion/dav_svn.passwd
    Require valid-user
    Order allow,deny
    Allow from all

    AuthzSVNAccessFile /etc/subversion/dav_svn.authz
    </Location>

이 설정은 `http://svn.example.com/svn/` URL을 시작으로 여러 서브버전 저장소를 두도록 합니다.

이제 사용자 인증 관련 설정을 해야하는데 우선 `/etc/subversion/dav_svn.authz` 파일을 다음과 같이 설정합니다. 이 파일을 각 저장소별로 접근할 수 있는 사용자를 그룹지어 접근 권한을 제어하는데 사용합니다.

    # Groups configuration
    [groups]
    developers = lethean,user1,user2,user3
    project1 = user1,user2
    project2 = user2,user3
    guests = guest

    # Repositories configuration
    [project1:/]
    @project1 = rw

    [project2:/]
    @developers = rw
    @project2 = r

    # All repositories
    [/]
    @developers = rw
    * =

이제 실제 사용자 목록을 담고 있는 `/etc/subversion/dav_svn.passwd` 파일을 만듭니다. 이 파일은 `htpasswd` 프로그램을 이용해 갱신하는데, 여기서는 'Basic' 인증 방식으로 사용하므로 '-m' 옵션을 추가해야 합니다.

    $ sudo touch /etc/subversion/dav_svn.passwd
    $ sudo htpasswd -m /etc/subversion/dav_svn.passwd lethean
    $ sudo htpasswd -m /etc/subversion/dav_svn.passwd user1

이제 사용자 관리와 웹서버 관련 설정은 끝났으니 서브버전 저장소 설정을 할 차례입니다.

서브버전 저장소를 HTTP 웹 프로토콜을 통해 무리없이 접근하려면 먼저 사용자 권한을 수정해야 합니다. 우분투의 기본 웹서버 접근 계정은 `www-data` 이므로, 이를 `src` 그룹에 추가합니다.

    $ sudo addgroup www-data src

그리고 서브버전 저장소의 권한을 변경하여 `src` 그룹이 서브버전 저장소에 접근할 수 있도록 합니다.

    $ sudo chgrp -R src /var/lib/svn
    $ sudo chmod -R g+rws /var/lib/svn

아파치 웹서버를 재시작합니다.

    $ sudo /etc/init.d/apache2 restart

이제 서브버전 저장소를 설정합니다. 이 작업은 모든 저장소에 각각 해야 하므로 여기서는 'hello' 프로젝트라고 가정합니다. 제일 먼저 소스가 제출(commit)되었을때 메일이 발송되고, 버그질라(Bugzilla) 댓글(comments)에 자동으로 추가되도록 `/var/lib/svn/hello/hooks/post-commit` 파일을 다음과 같이 편집합니다. 여기서는 모든 소스 코드, 문서가 UTF-8 인코딩을 사용한다고 가정하고, svn-commit-log@example.com 주소로 메일을 전송합니다.

    #!/bin/sh
    REPOS="$1"
    REV="$2"

    REPOS_NAME=`basename $REPOS`

    # uncomment below if system locale is not UTF-8
    #export LANG=ko_KR.UTF-8

    /usr/lib/subversion/hook-scripts/commit-email.pl 
      --from "svn-commit@example.com" 
      -s "SVN: $REPOS_NAME" 
      -h example.com 
      "$REPOS" "$REV" svn-commit-log@example.com

    /usr/lib/subversion/hook-scripts/svn2bugzilla.rb 
      "$REPOS" "$REV"

그리고 실행 권한을 줍니다.

    $ sudo chmod +x /var/lib/svn/hello/hooks/post-commit

참, 버그질라로 메일을 보내기 위해 루비를 설치하고,

    $ sudo apt-get install ruby

`/usr/lib/subversion/hook-scripts/svn2bugzilla.rb` 파일을 다음 내용으로 채운뒤,

    #!/usr/bin/ruby
    require 'rubygems'
    require 'active_record'
    require 'set'
    require 'fileutils'

    # If your Subversion usernames are not the same as your
    # Bugzilla usernames, map them here.
    USER_MAP = {"lovemetender" => "lovemetender@example.com"}
    USER_DOMAIN = "example.com"

    # Location of svnlook binary. Change as necessary.
    SVNLOOK = "/usr/bin/svnlook"

    # Configure your AR connection here.
    # Bugzilla supports both MySQL and PostgreSQL.
    AR_CONFIG = {:adapter => 'mysql',
     :host => 'database.example.com',
     :port => 3306,
     :database => 'bugzilla',
     :username => 'bugmaster',
     :password => 'bugpassword' }

    # You should not have to change anything below this line.

    if ARGV[0].nil? || ARGV[1].nil?
     puts "Usage: svn2bugzilla.rb repos_path revision"
     puts "To be used as a subversion post-commit hook."
     exit
    end

    REPOS_PATH = ARGV[0]
    REVISION = ARGV[1]

    ActiveRecord::Base.establish_connection(AR_CONFIG)

    ActiveRecord::Base.connection.execute 'set character set utf8' 

    # These are the three Bugzilla tables we'll be dealing with.
    # It'd probably be less code just to query the database directly,
    # bug using ActiveRecord is more fun!

    class Bug < ActiveRecord::Base
     set_primary_key "bug_id"
     # longdescs has a column named 'type' which doesn't play well with AR.
     # select the columns we need manually.
     has_many :longdescs, :select => "comment_id, bug_id, who, bug_when, thetext"
    end

    # longdescs is the comments table.
    class Longdesc < ActiveRecord::Base
     set_primary_key "comment_id"
     belongs_to :bug
     belongs_to :profile, :foreign_key => "who"
    end

    # profiles is the user table
    class Profile < ActiveRecord::Base
     set_primary_key "userid"
    end

    class Commit
     def initialize(repository_path, revision_number)
     @repository = File.basename repository_path
     @revision_number = revision_number
     @log_message = `#{SVNLOOK} log #{repository_path} -r #{revision_number}`.strip
     @date = `#{SVNLOOK} date #{repository_path} -r #{revision_number}`.strip
     @files_changed = `#{SVNLOOK} changed #{repository_path} -r #{revision_number}`.strip
     @author = `#{SVNLOOK} author #{repository_path} -r #{revision_number}`.strip
     end

     def message
     <<MESSAGE
    #{@log_message}

    Repository: #{@repository}
    Revision: #{@revision_number}
    Author: #{@author}
    Date: #{@date}
    Changes:
    #{@files_changed}

    http://svn.example.com/cgi-bin/viewvc.cgi/#{@repository}?revision=#{@revision_number}&view=revision
    MESSAGE
     end

     def author
     if USER_MAP[@author].nil?
     return "#{@author}@#{USER_DOMAIN}"
     end

     USER_MAP[@author]
     end

     # return a Set of unique bug numbers in the commit message
     def bug_numbers
     bugs = Set.new
     @log_message.scan(/bugD{1,3}(d+)/i).each do |match|
     bugs << match[0]
     end

     bugs
     end
    end

    # Do the actual work of submitting the comment to the database

    commit = Commit.new(REPOS_PATH, REVISION)
    commit.bug_numbers.each do |bug|
     bug = Bug.find_by_bug_id(bug)

     next if bug.nil?

     user = Profile.find_by_login_name(commit.author)

     next if user.nil?

     bug.longdescs.create(:who => user.id,
     :thetext => commit.message,
     :bug_when => Time.now)
    end

이 스크립트 소스를 조금 살펴보시면 버그질라 MySQL 데이터베이스 서버에 대한 접근 정보를 설정하는 부분과, 메일 텍스트 본문에 ViewVC 웹인터페이스를 직접 연결하는 부분이 있으므로, 필요에 맞게 수정해야 합니다. 이제 스크립트에 실행 권한을 줍니다.

    $ sudo chmod +x /usr/lib/subversion/hook-scripts/svn2bugzilla.rb

자 이제 마지막, 외부로 공개된 서버에서 접근시 내부 서브버전 서버로 연결하기 위해 공개 서버의 `/etc/apache2/sites-available/svn.example.com` 파일을 다음과 같이 편집합니다. 여기서는 공식 서브버전 서버 도메인 이름이 svn.example.com이고 (물론 네임서버에서  IP  주소는 외부로 공개된 서버와 동일합니다) 실제 서브버전 서버 IP는 192.168.0.141 이라고 가정합니다.

    <VirtualHost *>
      ServerAdmin lethean@example.com
      ServerName svn.example.com
      ErrorLog /var/log/apache2/svn.example.com-error.log
      CustomLog /var/log/apache2/svn.example.com-access.log combined
      UseCanonicalName Off
      ProxyVia On
      ProxyRequests Off
      ProxyPreserveHost On
      ProxyPass / http://192.168.0.141:80/
      ProxyPassReverse / http://192.168.0.141:80/
      ProxyPassReverse / http://svn.example.com:80/
    </VirtualHost>

그리고 이 호스트를 활성화하고 아파치를 재시작합니다.

    $ sudo a2ensite svn.example.com
    $ sudo /etc/init.d/apache2 restart

음... 더 이상 생각나는게 있으면 채워 나가야겠지만, 일단 오늘은 여기까지... :)
