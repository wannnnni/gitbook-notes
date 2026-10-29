# Centos 7 python3.7 -> 3.9 업그레이드

## 1. 컴파일에 필요한 도구 한 번에 설치

yum groupinstall -y "Development Tools" yum install -y openssl-devel bzip2-devel libffi-devel xz-devel wget

## 2. Python 3.9 소스 코드 다운로드 및 압축 해제

cd /usr/src wget https://www.python.org/ftp/python/3.9.19/Python-3.9.19.tgz tar xzf Python-3.9.19.tgz

## 3. 설치 준비 및 컴파일 진행 (기존 3.7을 파괴하지 않기 위해 altinstall 사용)

cd Python-3.9.19 ./configure --enable-optimizations make altinstall

기존 버전 사용 시: python3 또는 python3.7 입력 ➡️ Python 3.7.16 실행 새로운 버전 사용 시: python3.9 입력 ➡️ Python 3.9.19 실행



python3.9 설치위치 : /usr/local/bin/python3.9
