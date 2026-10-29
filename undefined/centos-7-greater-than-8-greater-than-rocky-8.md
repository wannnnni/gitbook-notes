# centos 7 -> 8 -> rocky 8

<pre><code><strong>1. CentOS 업그레이드 및 reboot
</strong><strong>  $ sudo yum update -y
</strong>  $ sudo reboot

2. EPEL-Release 설치
CentOS-Base 저장소 외부에서 패키지 다운로드를 위해 EPEL-Release 설치
  $ sudo yum install epel-release -y

3. 패키지 설치
  $ sudo yum install -y yum-utils rpmconf

4. 불필요 패키지 정리
설치에 사용되지 않거나, 충돌하는 패키지 제거
  $ sudo rpmconf -a
  Configuration file '/etc/yum/pluginconf.d/langpacks.conf'
  /etc/yum/pluginconf.d/langpacks.conf.rpmnew
  /etc/yum/pluginconf.d/langpacks.conf
  
   ==> Package distributor has shipped an updated version.
     What would you like to do about it ?  Your options are:
      Y or I  : install the package maintainer's version
      N or O  : keep your currently-installed version
        D     : show the differences between the versions
        M     : merge configuration files
        Z     : background this process to examine the situation
        S     : skip this file
   The default action is to keep your current version.
  *** aliases (Y/I/N/O/D/M/Z/S) [default=N] ? 
  Your choice: N

# 불필요 패키지 목록
  $ sudo package-cleanup --leaves

# 중복 패키지 목록
  $ sudo package-cleanup --orphans

# YUM 명령어를 통한 패키지 제거
  $ sudo yum remove -y &#x3C;package>

5. YUM에서 DNF로 업그레이드
CentOS8의 기본 패키지 관리자인 DNF 패키지를 설치합니다.
# DNF 설치
<strong>  $ sudo yum install -y dnf
</strong>
# 기존 YUM 패키지 관리자 제거
<strong>  $ sudo dnf remove -y yum yum-metadata-parser
</strong>
# YUM 구성 제거
  $ sudo rm -Rf /etc/yum

# DNF 리포지토리용 캐시 구성
  $ sudo dnf -y makecache
  Extra Packages for Enterprise Linux 7 - x86_64  71 MB/s |  17 MB     00:00
  CentOS-7 - Base                                 49 MB/s |  10 MB     00:00
  CentOS-7 - Updates                              63 MB/s |  17 MB     00:00
  CentOS-7 - Extras                               9.3 MB/s | 310 kB     00:00
  Metadata cache created.

6. CentOS7 패키지 업그레이드
  $ sudo dnf upgrade -y

7. YUM 저장소 업그레이드
YUM 리포지토리를 CentOS7 -> CentOS8로 변경
  $ sudo dnf install http://vault.centos.org/8.5.2111/BaseOS/x86_64/os/Packages/{centos-linux-repos-8-3.el8.noarch.rpm,centos-linux-release-8.5-1.2111.el8.noarch.rpm,centos-gpg-keys-8-3.el8.noarch.rpm}

8. CentOS7 -> CentOS8 업그레이드
  $ cd /etc/yum.repos.d
  $ mkdir backup
  $ mv CentOS-* backup
  
# Create new for CentOS BaseOS repo
$ sudo tee CentOS-Linux-BaseOS.repo&#x3C;&#x3C;EOM
[baseos]
name=CentOS Linux \$releasever - BaseOS
baseurl=http://vault.centos.org/8.5.2111/BaseOS/\$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
EOM
  
# Create new for CentOS AppStream repo
$ sudo tee CentOS-Linux-AppStream.repo&#x3C;&#x3C;EOM
[appstream]
name=CentOS Linux \$releasever - AppStream
baseurl=http://vault.centos.org/8.5.2111/AppStream/\$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
EOM
  
# EPEL-Release 리포지토리 업그레이드
  $ sudo dnf upgrade -y epel-release  
  
<strong># CentOS 커널 제거
</strong>  $ sudo rpm -e `rpm -q kernel` --nodeps $ sudo rpm -e `rpm -q kernel-devel` --nodeps  
  
# 충돌 패키지 제거
  $ sudo rpm -e --nodeps sysvinit-tools
  
# CentOS8 업그레이드
  $ sudo dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync  
  
  충돌하는 패키지 오류가 발생할 수 있습니다. 충돌이 발생하면 기존 패키지 제거가 필요합니다.  
  패키지 충돌 해결 후 위 명령어 다시 실행  
  아래 패키지에서 충돌날 가능성이 있습니다.  
  - NetworkManager
  - dracut-network
  - python36-rpmconf  

# CentOS8 커널 설치
  $ sudo dnf install -y kernel-core
  
# CentOS8 설치
  $ sudo dnf -y groupupdate "Core" "Minimal Install"
  
# 시스템 재시작
  $ sudo systemctl reboot
  
<strong>** failovermethod 옵션 제거 또는 주석 처리 **
</strong>  $ sudo sed -i 's/failovermethod=priority/#failovermethod=priority/' /etc/yum.repos.d/epel.repo
  $ sudo sed -i 's/failovermethod=priority/#failovermethod=priority/' /etc/yum.repos.d/epel-testing.repo

** CentOS 7용 EPEL 리포지토리 삭제 **
  $ sudo dnf remove epel-release -y

** CentOS 8용 EPEL 리포지토리 설치 **
  $ sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
  $ sudo dnf clean all
  $ sudo dnf update -y

** CentOS 8 → Rocky 8 업그레이드 **
  $ curl -O https://raw.githubusercontent.com/rocky-linux/rocky-tools/main/migrate2rocky/migrate2rocky.sh
  $ chmod +x migrate2rocky.sh
  $ sudo ./migrate2rocky.sh -r -V

</code></pre>
