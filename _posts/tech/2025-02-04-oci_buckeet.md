---
layout: post
title: "Oracle Cloud Infrastructure(OCI)로 이미지 호스팅하기"
date: 2025-02-04 00:00:00 +0900
categories:
  - tech
---

OCI가 다른 클라우드 서비스보다 좀 더 나은 free tier allowance를 제공해 OCI를 애용하고 있다. 개인 linux 서버도 몇 개씩 굴려볼 수 있다. CPU 4core (특정 아키텍쳐만 지원), 24GB RAM, 200GB block volume을 지원한다. 자세한 free tier 정책은 [OCI Documentation](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm)에서 확인.


### OCI Bucket
---
OCI Bucket은 객체 스토리지 서비스로 다양한 멀티미디어를 업로드하고 관리할 수 있다. [Free Tier로는 20GB까지 지원](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm#objectstorage)되며, 별도의 볼륨을 생성할 필요는 없다.



### 버킷 생성하기
---
스토리지 > 오브젝트 스토리지 및 아카이브 스토리지 > 버킷

버킷 생성 패널에서, 이름만 원하는대로 수정 후 별도의 설정 없이 버킷 생성 클릭

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/oci_bucket/oci_bucket (1).png)



### 이미지 업로드하기
---
![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/oci_bucket/oci_bucket (2).png)

업로드 버튼 클릭. 작업 더 보기에서 새 폴더를 생성할 수도 있다.

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/oci_bucket/oci_bucket (3).png)

파일을 드래그해서 올려놓고 업로드 클릭.




### 이미지 호스팅하기
---

폴더 또는 객체 끝에 있는 점 3개 -> 사전 인증된 요청 생성

![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/oci_bucket/oci_bucket (4).png)


![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/oci_bucket/oci_bucket (5).png)

객체 그 자체일 경우 `객체`에 표시되고, 특정 폴더일 경우 `접두어가 있는 객체` 항목에 표시된다.

`이름`은 원하는대로 수정.

`액세스 유형`은 `객체 읽기 허용` 으로 선택.

`만료일`은 본인이 원하는 대로 설정.



![](https://axqxbktknqat.objectstorage.ap-chuncheon-1.oci.customer-oci.com/p/x3c6dl2qfNZsDPc-JZrqIhRn3xzFhMvEz_7wHM1FFXpkxE8_wMXctQZCts4NVm76/n/axqxbktknqat/b/image_bucket/o/blog/oci_bucket/oci_bucket (6).png)


이렇게 하면 접근 가능한 URL이 나온다. 

**URL은 생성 시에만 보여주고 창을 끄면 보여주지 않으니 주의.** 

OCI는 하단에 있는 링크를 사용할 것을 권장한다. 

그리고 URL을 복사해, 끝에 있는 /o/ 다음을 디렉토리처럼 사용하면 된다. 예를 들어 myfolder 아래 있는 a.png에 접근하고 싶다면,

> ~/o/myfolder/a.png

로 접근하면 이미지를 호스팅할 수 있다.   


사전 인증된 요청은 좌측 패널의 `사전 인증된 요청` 항목에서 관리할 수 있다.