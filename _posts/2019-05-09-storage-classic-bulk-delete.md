---
layout: post
title:  Oracle Storage Classic의 Object 지우기 - Bulk Delete REST API 이용
categories: Cloud
tags: [Oracle Cloud, Object Storage, REST API, Python] 
---

Object Storage에서 Container를 지우고 싶을 경우, Container 내에 Object들이 존재한다면 **먼저 Object들을 다 지우고 Container를 지워야 한다.**
오라클 Cloud에서는 이 작업을 Cloud Console을 통해서 수행할 수 있지만, Object가 수천개가 넘어 간다면 이를 일일이 수동으로 삭제하는 것이 거의 불가능하다.

따라서 이 경우 REST API를 이용하여서 Object를 지워줘야 하는데, 이를 위해서 **Bulk Delete**하는 REST API를 제공한다.
이를 통해서 금방 다 지울 수 있을 것 같지만 지우려고 하는 Object가 수천개라면 수천 개 List를 Bulk Delete API에 한방에 보내서 처리할 수가 없다.
테스트 해 본 결과 안정적으로 처리 후 응답을 받을 수 있는 Object 갯수는 70개 내외였다. 

> bulk-delete의 input object 수를 상황에 따라 적절히 조절하면서 스크립트를 수행한다.

Object List를 70개 내외로 잘라내서 매번 REST API를 돌려주는 것도 상당히 번거로운 일이기 때문에, 다음과 같이 정해진 수의 Input을 받아서 Bulk Delete API를 호출하는 스크립트를 만들어 주면 작업은 훨씬 간편해 진다.

# bulk-delete.sh

스크립트의 내용은 다음과 같다.

```bash
#!/bin/bash

# Connection Info
container="ContainerName"
iddomain="Identity Domain Name"
username="User Name"
password="Password"

# get AuthToken
curl -I -X GET -H "X-Storage-User: Storage-${iddomain}:${username}" \
     -H "X-Storage-Pass: ${password}" \
      https://${iddomain}.us.storage.oraclecloud.com/auth/v1.0 > output.txt 

token=`grep AUTH output.txt | head -n1 |  awk '{print $2}'`
echo ${token}

## Looping Count는 원하는 대로
for (( c=1; c<=5; c++ ))
do
echo "========= $c ======================="
cp /dev/null object1.txt
cp /dev/null object2.txt

# Get Object List
curl -H "X-Auth-Token: ${token}" https://${iddomain}.us.storage.oraclecloud.com/v1/Storage-${iddomain}/${container} -o object1.txt

python3 ./object.py ${container}

# Bulk Delete
curl -X DELETE \
     -H "X-Auth-Token: ${token}" -H "Content-Type: text/plain" \
     -T object2.txt \
     "https://${iddomain}.us.storage.oraclecloud.com/v1/Storage-${iddomain}/?bulk-delete"
done
```

# object.py
스크립트 내부에서 **Python** 파일을 수행하는데, 이것이 하는 역할은 Object 이름 앞에 Container의 이름을 달아 주는 것이다.
여기서 bulk delete에 사용할 input 값의 크기를 조정하면 된다. 예제에서는 70으로 설정하였다.

```py
import glob
import os
import sys

fr = open("object1.txt", 'r', encoding='utf8')

fw = open("object2.txt", 'a')

container=str(sys.argv[1])

linenum = 0


for line in fr:
    linenum = linenum +1 
    ''' print("line", line) '''
    fw.write(container+"/" + line)
    if linenum > 70:
       break
fr.close()
fw.close()
```

# 사용법

bulk-delete.sh의 Connection 관련 정보와 Container 명을 수정한 후 스크립트만 돌려주면 된다.

```
> bulk-delete.sh
```

루핑을 돌면서 잘 지워지고 있다.

![](/assets/images/storage/01_result.png)

Object가 다 지워지고 나면 Container는 콘솔에서 삭제하면 된다.

# 참고 자료

- [Deleting Multiple Objects in a Single Operation](https://docs.oracle.com/en/cloud/iaas/storage-cloud/cssto/deleting-multiple-objects-single-operation.html)

