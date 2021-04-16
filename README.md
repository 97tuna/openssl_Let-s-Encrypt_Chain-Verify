# openssl_Lets_Encrypt_Chain_Verify

> 내가 방법을 까먹을까봐 적어놓는곳

### 요약
openssl verify는 Root CA정보가 포함된 chain에 대해서 검증하는데 Let's Encrypt는 Root CA 정보가 누락된 상태로 Chain 발급이 되므로, Root CA정보를 따로 넣어줘야합니다.

<!-- 이미지 -->
<p align="center">
 <a href="#"><img width="500px" src="https://user-images.githubusercontent.com/50114556/114988304-2588ae00-9ed1-11eb-9c87-74006e9ca0bb.png"></a>
</p>

### 진행 순서

1. Root CA 다운로드
2. pem형식으로 변환
3. Root CA가 포함된 Chain생성
4. Chain 검증

### Lets_Encrypt 발급시 생성 파일

- cert.pem
- chain.pem
- fullchain.pem
- privkey.pem

### subject와 issue를 확인하기

```
$ openssl x509 -noout -in chain.pem -subject -issuer

> subject= /C=US/O=Let's Encrypt/CN=R3
> issuer= /O=Digital Signature Trust Co./CN=DST Root CA X3
```

위와같은 출력이 나온다면 문제 없음


### 인증서 Chain 관계를 파악하기

```
$ openssl verify -CAfile chain.pem cert.pem

> cert.pem: C = US, O = Let's Encrypt, CN = R3
> error 2 at 1 depth lookup:unable to get issuer certificate
```

위와같이 나온다면 Let's_Encrypt 이기에 오류가 발생
> Intermediate CA 정보만 입력된 Chain이 발급되기 때문

### 해결하기

- -untrusted 인자 전달
- Root CA 추가

> 1번 방법의 경우, 임시적으로 -untrusted 인자를 넣어 검증하는 방법입니다.

```
$ openssl verify -untrusted chain.pem cert.pem

> cert.pem: C = US, O = Let's Encrypt, CN = R3
> error 2 at 1 depth lookup:unable to get issuer certificate
> OK
```

이 방법은 임시 방편이므로 서비스에 추가할 수 없음 👉 2번 방법을 사용하여 해결

> 2번 방법의 경우, Let's Encrypt 가 사용하는 Root CA를 찾아 완벽한 Chain 관계를 만들어 주기

1. Root CA 다운로드
```
$ wget http://apps.identrust.com/roots/dstrootcax3.p7c
```

2. pem 형식으로 변환

```
$ openssl pkcs7 -inform der -in dstrootcax3.p7c -out dstrootcax3.pem -print_certs
```

3. Root CA가 포함된 Chain을 생성

```
$ cp fullchain.pem fullca.pem
$ cat dstrootcax3.pem >> fullca.pem
```

4. Chain verification을 진행

```
$ openssl verify -CAfile fullca.pem cert.pem

> cert.pem: OK
```

끝.