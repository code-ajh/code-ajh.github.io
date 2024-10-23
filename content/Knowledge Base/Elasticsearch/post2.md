---
title: Elasticsearch 설치 및 설정
description: Rocky Linux 9에 Elasticsearch SSL 적용하여 Standalone 으로 설치
aliases:
  - Elasticsearch + kibana 설치
tags:
  - ELK
  - Elasticsearch
  - Kibana
  - "#Knowledge-Base"
draft: 
date: 2024-06-27
---

  

Elasticsearch(이하 es)는 2010년대 중후반부터 한국에서 많이 사용되기 시작한 검색 및 분석 엔진입니다. Elastic license 아래에서 무료로 사용할 수 있기 때문에 기업의 규모와 상관없이 용도에 따라서 많이 사용되는 기술입니다.

Elasticsearch 는 문서화가 잘되어있지만 여러 페이지를 참고해야되는 경우가 있습니다. 때문에 어려움을 느끼는 분들을 위해 공식 문서에서 필요한 부분만 정리하여 작성합니다.

---
# 1. 설치  

## 1.1 dnf repo 설정  

Docker 사용하여 컨테이너로 띄우는 법도 있지만 프로덕션 레벨에는 적합하지 않다고 공식 문서에서 서술하고 있습니다. 여기선 공식 패키지를 사용하여 설치합니다.
 

> [!note]
> 다른 버전이 필요한 경우에는 8.x 부분을 수정해서 원하는 버전으로 변경해주세요.


```bash
[elasticsearch]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

위의 내용을 `/etc/yum.repos.d/` 의 경로에 `elasticsearch.repo` 라는 이름의 파일을 만들어 저장합니다.

## 1.2 패키지 설치
 

```bash
dnf install --enablerepo=elasticsearch elasticsearch kibana
```

  
[1.1에서](#11-dnf-repo-설정) 설정한 repository를 사용하여 elasticsearch 와 kibana를 함께 설치합니다.

  
정상적으로 설치가 완료되면 아래와 같은 문구가 출력됩니다.

``` bash
--------------------- Security autoconfiguration information ----------------------

Authentication and authorization are enabled.
TLS for the transport and HTTP layers is enabled and configured.  

The generated password for the elastic built-in superuser is : {랜덤 패스워드}  

If this node should join an existing cluster, you can reconfigure this with
'/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>'
after creating an enrollment token on your existing cluster.

You can complete the following actions at any time:  

Reset the password of the elastic built-in superuser with
'/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'.  

Generate an enrollment token for Kibana instances with
 '/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'.
 
Generate an enrollment token for Elasticsearch nodes with
'/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node'.
-----------------------------------------------------------------------------------
```

  
# 2. 설정

## 2.1 인증서 관련 파일 생성

아래의 스크립들은 필요에 따라 생략하거나 할 수 있으며 스크립트의 실행의 모든 과정은 생략합니다. 해당 스크립트의 자세한 옵션들은 [공식 문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-basic-setup-https.html)를 참조해주세요.

  > [!note]
  > 아래의 모든 동작은 사용자의 필요에 경로나 이름등이 변경될 수 있습니다.

**CA 인증서 생성** 
```bash
/usr/share/elasticsearch/bin/elasticsearch-certutil ca
# elastic-stack-ca.p12 생성
```


**Elasticsearch SSL 인증서 생성** 
```bash
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
# elastic-certificates.p12 생성
```

  
**통신용 SSL 인증서 생성**
```bash
/usr/share/elasticsearch/bin/elasticsearch-certutil http
# elasticsearch-ssl-http.zip 생성
```

  **인증서 만들면서 설정한 비밀번호 저장**
  ```bash
# 통신용 SSL 인증서 비밀번호
elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
  
# Elasticsearch SSL 인증서 비밀번호
elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password
```

**인증서 경로 정리**
```bash
# es 인증서
# http.p12 는 elasticsearch-ssl-http.zip 에 압축되어 있음
mv {경로}/elastic-certificates.p12 /etc/elasticsearch/certs/
mv {경로}/http.p12 /etc/elasticsearch/certs/
  

# kibana 인증서
# elasticsearch-ca.pem 는 elasticsearch-ssl-http.zip 에 압축되어 있음
mkdir /etc/kibana/certs
mv {경로}/kibana/elasticsearch-ca.pem /etc/kibana/certs/
```

  **권한 설정**
```bash
# es 인증서 권한 변경
chown root:elasticsearch /etc/elastcsearch/certs/*.p12
chmod 640 /etc/elasticsearch/certs/*.p12  

# kibana 인증서 권한 변경
chown root:kibana /etc/kibana/certs/elasticsearch-ca.pem
chmod 640 /etc/kibana/certs/elasticsearch-ca.pem
```

## 2.2 설정 파일 수정  

이 단계에서는 실행에 필요한 일부분만 수정하였습니다. 다른 자세한 옵션의 경우에는 [공식 문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)를 참조해주세요.  

  
**elasticsearch.yml**
```yml
# 모든 서버에서 접속 가능하도록 변경
network.host: 0.0.0.0

# 인증서 설정
# 경로는 인증서가 있는 위치를 작성해주세요.
xpack.security.http.ssl:
  enabled: true
  keystore.path: {인증서 경로}/http.p12  

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: {인증서 경로}/elastic-certificates.p12
  truststore.path: {인증서 경로}/elastic-certificates.p12
```

  
**kibana.yml**

kibana 접속에 필요한 토큰 or 비밀번호 먼저 생성합니다.

```bash
# 토큰 생성

/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

# 계정 + 비밀번호 생성

/usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

```


해당 값을 가지고 kibana.yml 을 수정합니다.


```yml
# 모든 서버에서 접속 가능하도록 변경
server.host: "0.0.0.0" 

# kibana 접속 웹에 사용할 SSL 인증서
# 위에서 생성한 인증서와는 다르므로 따로 준비
server.ssl.enabled: true
server.ssl.certificate: {인증서 경로}
server.ssl.key: {인증서 키 경로}

# elasticsearch 와 통신에 사용되는 인증서 및 주소
elasticsearch.hosts: ["https://{ip 주소}:9200"]
elasticsearch.ssl.certificateAuthorities: [ "{인증서 경로/elasticsearch-ca.pem}" ]

# elasticsearch 연결을 위한 설정
# 아래에서 토큰이나 계정/비밀번호 둘중 하나만 설정해도 접속 가능
elasticsearch.serviceAccountToken: "{토큰 값}"

# OR

elasticsearch.username: "kibana_system"
elasticsearch.password: "{랜덤 비밀번호 입력}"
```

  
# 3. 실행

서버가 실행될 경우에는 자동으로 실행되도록 설정합니다.

```bash
systemctl enable elasticsearch
systemctl enable kibana  

systemctl start elasticsearch
systemctl start kibana
```
  
실행에 약간의 시간이 필요할 수 있습니다.

---

# **참조**

>Set up Elasticsearch - Elastic Documentation\
><https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html>

> elasticsearch-8.4.0 (8.x) 인증서 생성하여 설치 - Tistory\
> <https://realkoy.tistory.com/entry/elasticsearch-840-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EC%83%9D%EC%84%B1%ED%95%98%EC%97%AC-%EC%84%A4%EC%B9%98> - 

>\[ElasticSearch] 클러스터 구성 (단일 구성) - velog.io\
><https://velog.io/@jhchoi94/%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0-%EA%B5%AC%EC%84%B1-%EB%8B%A8%EC%9D%BC-%EA%B5%AC%EC%84%B1> - 