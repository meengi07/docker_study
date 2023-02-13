![dockerLOGO](/image/dockerLOGO.png)


# 컨테이너 애플리케이션 구축

먼저 컨테이너를 구축하는 것에 있어서 권장하는 건 하나의 컨테이너에는 하나의 애플리케이션만 동작하도록 구성하는 것이 컨테이너간의 독립성을 보장함과 동시에 애플리케이션의 버전 관리, 소스코드 모듈화 등 다양한 이점을 얻을 수 있으며 도커의 철학이 한 컨테이너에 한 프로세스만 실행하는 것입니다.


```bash
docker run -d \ # 옵션 -d 는 background 로 실행시킴
--name wordpressdb \
-e MYSQL_ROOT_PASSWORD=password \
-e MYSQL_DATABASE=wordpress \
# \(역슬래시) 는 가동성을 위해 명령어의 옵션을 구분할 때 쓰며, 안써도 상관없음
```

기본 옵션은 컨테이너 내부로 진입하도록 attach  가능한 상태라면, -d 옵션은 Detached 모드로 컨테이너를 실행합니다. Detached 모드는 애플리케이션을 백그라운드에서 동작하도록 설정합니다.

> 이 옵션으로 run을 실행하면 입출력이 없는 상태로 컨테이너를 실행합니다.
컨테이너 내부에서 프로그램이 터미널을 차지하는 포그라운드(foreground) 로
실행 돼 사용자의 입력을 받지 않습니다. Detached 모드 인 컨테이너는 반드시
컨테이너에서 프로그램이 실행돼야 하며, 포그라운드 프로그램이 실행되지 않으면
컨테이너는 종료됩니다.
>

위 문장의 뜻을 보자면 백그라운드 모드로 실행된 애플리케이션을 반드시 실행할 프로그램이 있어야하며, 프로그램이 종료되면 자동으로 컨테이너도 종료된다는 의미입니다.

그리고 데이터베이스 이미지를 컨테이너로 만들고 실행하게 되면 Detached 로 동작하도록 설정되어 있어, 포그라운드로 실행된 로그를 볼 수 있습니다.

하지만 exec 명령어를 통해 Detached 모드에서도 컨테이너 내부의 셸을 사용할 수 있습니다.

<aside>
💡 컨테이너는 각기 하나의 모니터를 가지고 있다고 생각하면 이해하기 쉽습니다.

</aside>

-e 옵션은 컨테이너 내부의 환경변수를 설정합니다.

```bash
... -e MYSQL_PASSWORD = password ...
# 컨테이너의 MYSQL_PASSWORD 환경변수를 password로 설정한다는 의미
# 현재 설정된 환경변수를 확인하려면 echo ${ENVIRONMENT_NAME} 로 확인

root@a0897ecfbbae:/\# echo $MYSQL_PASSWORD #컨테이너 내부에서 확인하면
password # 환경변수의 결과값이 출력
```

~~--link 옵션은 A컨테이너에서 B컨테이너로 접근하는 방법 중 가장 간단한 것은 NAT로 할당받은 내부 IP 를 쓰는 것 입니다. B 컨테이너의 IP가 172.17.0.3 이라면 A 컨테이너는 이 IP를 써서 B컨테이너에 접근할 수 있습니다.  그러나 도커 엔진은 컨테이너에게 내부 IP를 172. 17.0. 2, 3, 4...와 같이 순차적으로 할당합니다. 이는 컨테이너를 시작할 때마다 재할당하는 것이므로 매번 변경되는 컨테이너의 IP  로 접근하기는 어렵습니다.~~

--link 옵션은 deprecated 되었으며 docker network 명령어를 사용하는 것을 권장합니다.

---

# 도커 볼륨 volume

도커 볼륨은 도커 컨테이너에서 생성되고 사용되는 데이터를 유지하는 방법입니다.
도커에서 이미지로 컨테이너를 생성하면서 이미지는 읽기 전용이 되고, 컨테이너의 변경 데이터만 별도로 저장해서 관리한다고 했습니다. 하지만 이 데이터 역시 컨테이너가 삭제되면 컨테이너 계층에 저장돼어 있던 데이터도 삭제된다는 점 입니다. 이렇게 컨테이너가 삭제되면 데이터를 복구할 수 없습니다. 이를 방지하기 위해 컨테이너의 데이터를 영속적(Persistent) 데이터로 활용할 수 있는 방법이 몇가지 있는 데 그 중 가장 쉬운 방법이 바로 볼륨을 활용하는 것 입니다.

- 호스트 볼륨으로 공유 : -v 옵션을 사용하여 [호스트의 공유 디렉터리]:[컨테이너의 공유 디렉터리] 형태로 설정할 수 있습니다. 호스트의 디렉터리에 데이터가 있었다면 덮어씌워집니다.

    ```bash
    ex) -v /home/mysql_db:/var/lib/mysql  #-v 대신 -mount 옵션도 동일한 기능
    # 이렇게 볼륨옵션을 통해 생성한 컨테이너를 삭제해도 
    # 컨테이너 계층에 저장된 데이터는 /home/mysql_db 에 저장되어 있습니다.
    ```

  ![volume.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eed016c1-5d98-48d1-b1cb-3c5bf318c485/volume.png)

- 볼륨 컨테이너 : -v 옵션으로 볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유하는 것 입니다.
  컨테이너를 생성할 때 --volumes -from 옵션을 설정하면 -v & --volume 옵션을 적용한 컨테이너의 볼륨 디렉터리를 공유할 수 있습니다. 하지만 이 공유는 직접 볼륨을 공유하는게 아니라 -v 옵션을 적용한 컨테이너를 통해 공유하는 것 입니다.

  ![volumeContainer.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/380110ff-41bb-488e-817f-5575c75654ba/volumeContainer.png)

- 도커 볼륨 : 도커 자체에서 제공하는 볼륨 기능을 docker volume 명령어를 통해 사용할 수 도 있습니다.

    ```bash
    docker volume create --name myvolume #생성할땐 create
    docker volume ls #볼륨 목록 조회
    ```

  볼륨을 사용하는 컨테이너를 생성할때는 기존의 -v 옵션과 다르게 [볼륨 이름]:[컨테이너 공유 디렉터리] 형식으로 입력합니다.

  ![dockervolume.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/51c2a167-a027-45da-8cb8-4ed63e4d0114/dockervolume.png)

  볼륨은 도커엔진이 디렉터리 하나에 상응하는 단위로서 관리하게 됩니다. 호스트 볼륨과 동일하게 호스트에 저장함으로써 데이터를 보존하지만 사용자는 파일이 어디에 저장되어 있는지 알 필요가 없습니다.

    ```bash
    docker inspect #inspect 명령어를 통해 볼륨의 정보를 조회해 볼 수 있습니다.
    docker inspect --type volume myvolume
    [
    	{
    		"Driver": "local", # 볼륨이 사용하는 드라이버
    		"Labels": {}, # 볼륨을 구분하는 라벨
    		"Mountpoint": "/var/lib/docker/volumes/myvolume/_data", # 주소
    		"Name":"myvolume", # 볼륨명
    		"Option": {}, # 옵션
    		"Scope": "local" #스코프
    	}
    ]
    
    docker volume prune # 볼륨을 삭제하는 명령어
    ```


# 도커 네트워크

### 네트워크 구조

도커는 컨테이너의 네트워크 인터페이스에 eth0 , lo 네트워크 인터페이스가 있습니다. 도커는 내부 IP를 순차적으로 할당하며, 이 IP는 컨테이너를 재시작할 때 마다 변경될 수있는 변동IP입니다. 이 IP는 내부 망(호스트)에서만 쓸 수 있는 IP이므로 외부와 연결해야 합니다. 이 과정은 컨테이너를 시작할 때 마다 호스트에 있는 veth 라는 네트워크 인터페이스를 생성함으로써 이루어 집니다. 그리고 veth는 가상 네트워크 인터페이스의 이름이며 컨테이너 마다 생성하여 사용합니다

<aside>
💡 veth : virtual + eth 라는 뜻 입니다.

</aside>

- eth0 : 공인 IP 또는 내부 IP가 할당되어 실제로 외부와 통신할 수 있는 호스트의 네트워크 인터페이스
- docker0 : 브릿지라는 기능이며 각 veth 와 바인딩되어 호스트의 eth0와 이어주는 역할입니다.

  ![docContainer.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/068b09d6-c1ff-41e1-a44a-24fd046bdfb8/docContainer.png)


## 도커의 네트워크 기능

컨테이너를 생성하면 기본적으로 docker0 브릿지를 통해 외부와 통신할 수 있는 환경을 사용할 수 있지만 사

자의 선택에 따라 여러 네트워크 드라이버를 쓸 수 도 있습니다.

- 브리지(bridge)
- 호스트(host)
- 논(none)
- 컨테이너(container)
- 오버레이(overlay)
- 그 외 서드파티 플러그인 솔루션 (third-party)
    - weave
    - flannel
    - openvswitch
    - etc…

```bash
docker network ls
NETWORK ID         NAME       DRIVER       SCOPE
a7d2c538bb49       bridge     bridge       local
e87a830883b0       host       host         local 
2d2c1cd4c9e7       попe       null         local
```

브리지 네트워크는 컨테이너를 생성할 때 자동으로 연결되는 docker0 브리지를 활용하도록 자동설정돼 있습니다. 이 네트워크는 172.17.0.x IP 대역을 컨테이너에게 순차적으로 할당합니다

```bash
docker network inspect bridge # network 정보조회 
# docker inspect --type network 도 동일한 정보조회
```

### 브리지 네트워크 (bridge network)

브리지 네트워크는 docker0이 아닌 사용자 정의 브리지를 생성해 각 컨테이너에 연결하는 구조입니다. 각 컨테이너는 연결된 브리지를 통해 외부와 통신할 수 있습니다.

```bash
docker run -i -t --name mynetwork_container --net mybridge ubuntu:14.04
# 컨테이너 내부에서 ifconfig 를 통해서 네트워크 대역을 확인할 수 있다.
docker network disconnect mybridge mynetwork_container # 기존 브리지 연결을 해제
docker network connect mybridge mynetwork_container # 브리지 재 연결
```

### 호스트 네트워크 (host network)

네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 쓸 수 있습니다. 위의 브리지 드라이버 네트워크와 달리 호스트 드라이버의 네트워크는 별도로 생성할 필요 없이 기존의 host라는 이름의 네트워크를 사용합니다.

```bash
root@docker-host: docker run -i -t --name network_host --net host ubuntu:14.04
```

--net host 옵션으로 호스트를 설정한 컨테이너의 내부에서 네트워크를 확인하면 호스트와 동일한 것을 확인할 수 있습니다. 이는 호스트 머신에서 설정한 호스트 이름도 컨테이너가 물려받기 때문에 컨테이너의 호스트 이름도 무작위 16진수가 아닌 도커엔진이 설치된 호스트 머신의 호스트 이름으로 설정됩니다.

### 논 네트워크 (none network)

none 네트워크는 말 그대로 아무런 네트워크도 사용하지 않겠다는 뜻입니다.

```bash
docker run -i -t --name network_none --net none ubuntu:14.04
```

none 옵션을 설정한 컨테이너 내부에서 네트워크를 확인하면 로컬호스트를 나타내는 lo 외에는 아무것도 존재하지 않습니다.

### 컨테이너 네트워크 (container network)

컨테이너 네트워크는 다른 컨테이너의 네트워크 네임스페이스 환경을 공유할 수 있습니다. 공유되는 속성으로는 내부 IP, interface MAC 주소 등입니다.

```bash
docker run -i -t -d --name network_container_1 ubuntu:14.04
docker run -u -t -d --name network_container_2 --net container:network_container_1 ubuntu:14.04
```

위와 같이 다른 컨테이너의 네트워크 환경을 공유하면 내부 IP를 새로 할당받지 않고 호스트에 veth로 시작하는 가상 네트워크 인터페이스도 생성하지 않습니다.

![스크린샷 2023-02-10 오후 9.19.15.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7fdb8806-f30b-444b-9c87-c595b3d1e7b3/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.19.15.png)

### 브리지 네트워크와 --net-alias

브리지 타입의 네트워크와 run 명령어의 --net-alias 옵션을 쓰면 특정 호스트 이름으로 컨테이너 여러개에 접근할 수 있습니다.

```bash
docker run -i -t -d --name network_alias_container1
--net mybridge **--net-alias alicek106 buntu**: 14.04

docker run -i -t -d --name network_alias_container2 
--net mybridge **--net-alias alicek106ubuntu**: 14.04

docker run -i -t -d --name network_alias_container3 
--net mybridge **--net-alias alicek106ubuntu**: 14.04

docker run -i -t --name network_alias_ping --net mybridge ubuntu:14.04
ping -c 1 alicek106 # return (172.18.0.5)
ping -c 1 alicek106 # return (172.18.0.3)
ping -c 1 alicek106 # return (172.18.0.4)
```

각 컨테이너 3개의 IP로 각각 ping이 전송된 것을 확인할 수 있습니다. 매번 달라지는 IP를 결정하는 것은 라운드 로빈(round-robin) 방식입니다. 이는 도커 엔진에 내장된 DNS가 호스트 이름 alicek106을 —net-alias 옵션으로 설정한 컨테이너로 변환(resolve)하기 때문입니다.

![스크린샷 2023-02-10 오후 9.32.16.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c66bdde4-5369-4052-b56e-46fd0ef53bab/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.32.16.png)

도커의 DNS는 호스트 이름으로 유동적인 컨테이너를 찾을 때 주로 사용됩니다.
dig라는 도구를 사용해서 DNS로 도메인 이름에 대응하는 IP를 조회할 때 쓸 수 있습니다.

### MacVLAN 네트워크

호스트의 네트워크 인터페이스 카드를 가상화해 물리 네트워크 환경을 컨테이너에게 동일하게 제공합니다.
물리 네트워크상에서 가상의 맥(MAC)주소를 가지며, 해당 네트워크와 연결된 다른 장치와의 통신이 가능해집니다. 연결된 컨테이너는 기본 할당IP대역이 아닌 네트워크 장비의 IP를 할당받습니다.

![스크린샷 2023-02-10 오후 9.35.33.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/23164314-ff8c-41d4-8510-87c9ddf71e2d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.35.33.png)

<aside>
💡 MacVLAN 네트워크를 사용하는 컨테이너는 기본적으로 호스트와 통신이 불가능합니다

</aside>

```bash
docker network create -d macvlan --subnet=192.168.0.0/24 
--ip-range=192.168.0.64/28 --geteway=192.168.0.1 
-o macvlan_mode=bridge -o parent=eth0 my_macvlan
```

- -d : 네트워크 드라이버로 macvlan을 사용한다는 것을 명시합니다. —driver와 같습니다.
- --subnet : 컨테이너가 사용할 네트워크 정보를 입력합니다
- --ip-range : MacVLAN을 생성하는 호스트에서 사용할 컨테이너의 IP범위를 입력합니다. node01과 node02의 IP범위가 겹치지 않게 설정해야 합니다.
- --gateway : 네트워크에 설정된 게이트웨이를 입력합니다.
- -o : 네트워크의 추가적인 옵션을 설정을 설정합니다.

### 컨테이너 로깅

- json-file 로그 : 도커는 컨테이너의 표준 출력과 에러 로그를 별도의 메타데이터 파일로 저장하며 이를 확인하는 명령어를 제공합니다.

    ```bash
    docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=1234 mysql:5.7
    # 이렇게 mysql을 실행시키면 logs 명령어로 컨테이너의 내부상태를 알 수 있습니다.
    docker logs mysql
    ```

    - --tail : 마지막 로그줄로 부터 출력한 줄의 수를 설정할 수 있습니다.
    - --since : 유닉스 시간을 입력해 특정 시간 이후의 로그를 확인할 수 있습니다.
    - -t : 타임스탬프를 표시할 수 있습니다.
    - -f : 로그를 스트림으로 확인할 수 있습니다.

- syslog 로그 : syslog는 유닉스 계열 운영체제에서 로그를 수집하는 오래된 표준 중 하나로서 커널, 보안 등 시스템과 관련된 로그, 애플리케이션의 로그등 다양한 종류의 로그를 수집하고 저장합니다.

    ```bash
    docker run -d **-h rsyslog** --name syslog_container 
    --log-driver=syslog ubuntu:14.04 echi syslogtest
    ```

    - **rsyslog** : syslog를 원격으로 중앙 컨테이너에 저장할 수 있습니다.
    - --log-opt : 옵션으로 syslog-facility를 통해 로그가 저장될 파일을 바꿀 수 있습니다. 이 옵션은 로그를 생성하는 주체에 따라 로그를 다르게 저장하여 여러 애플리케이선에서 수집되는 로그를 분류하는 방법입니다. 기본적으론 daemon으로 설정돼어 있지만 kern, user, mail등 다른 종류로 사용할 수 있습니다.

- fluentd 로깅 : 오픈소스 도구로서 도커 엔진의 컨테이너의 로그를 저장할 수 있도록 플러그인 형태로 제공합니다. JSON 포맷을 사용하기 때문에 쉽게 사용가능하며 수집된 데이터들을 AWS S3, HDFS(Hadoop Distributed File System), MongoDB등 다양한 저장소에 저장할 수 있다는 장점이 있습니다.

  ![스크린샷 2023-02-10 오후 11.35.32.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/833c9201-40c5-4bcb-a150-30711c456fca/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-02-10_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.35.32.png)

    ```bash
    root@mongo:/ docker run --name mongoDB -d -p 27017:27017 mongo
    ```

    - fluentd 서버의 호스트에서 파일을 fluent.conf 로 저장합니다.
    - 들어오는 로그 데이터를 몽고DB에 전송하고, access 라는 이름의 컬렉션에 로그를 저장하여 몽고DB 컨테이너의 호스트 주소와 포트를 지정한 것입니다.

    ```bash
    root@fluentd:/ docker run -d --name fluentd -p 24224:24224 
    -v $(pwd)/fluent.conf:/fluentd/etc/fluent.conf -e FLUENTD_CONF=fluent.conf
    alicek106/fluentd:mongo
    ```

    <aside>
    💡 도커 허브의 fluentd 이미지에는 몽고DB에 연결하는 플러그인이 내장돼 있지 않으므로 따로 설치해야 합니다.

    </aside>

- 아마존 클라우드워치 로그 : AWS에서는 로그 및 이벤트등을 수집하고 저장해 시각적으로 보여주는 클라우드워치를 제공합니다. 도커를 AWS EC2에서 사용한다면 별도 설치없이 컨테이너에서 드라이버 옵션을 설정하는 것 만으로 클라우드워치 로깅 드라이버를 사용할 수 있습니다.
    1. 클라우드워치에 해당하는 IAM 권한 생성
    2. 로그 그룹생성
    3. 로그 그룹에 로그 스트림 생성
    4. 클라우드워치의 IAM권한을 사용할 수 있는 EC2 인스턴스 생성과 로그 전송


### 컨테이너 자원 할당 제한

컨테이너를 생성할 때 사용하는 명령어인 create, run 에서 컨테이너의 자원 할당량을 조정하도록 옵션을 입력할 수 있습니다.

<aside>
💡 현재 컨테이너에 설정된 자원 제한을 확인하는 쉬운 방법은 docker inspect 명령어 입니다. 
설정된 컨테이너의 자원 제한을 변경하려면 update 명령어를 사용합니다.

</aside>

- **메모리 제한** : --memory를 지정해 컨테이너의 메모리를 제한할 수 있습니다.
  입력 단위는 m (megabyte) , g (gigabyte)이며 제한할 수 있는 최소 메모리는 4MB입니다.
- **CPU 제한** :
    - **--cpu-shares** : 컨테이너에 가중치를 설정해 해당 컨테이너가 CPU를 상대적으로 얼마나 사용할 수 있는지를 나타내며, CPU를 한 개씩 할당하는 방식이 아닌 시스템에 존재하는 CPU를 얼마나 나눠 쓸 것인지 명시하는 옵션입니다.
    - **--cpuset-cpu** : 호스트에 여러개의 CPU가 있을 때 컨테이너가 특정 CPU만을 사용하도록 설정할 수 있습니다. CPU집중적인 작업이 필요하다면 여러개의 CPU를 사용하도록 설정해 작업을 적절하게 분배하는 것이 좋습니다.

        <aside>
        💡 CPU와 메모리에 고의로 과부하를 줘서 성능 테스트를 하고 싶을 때 stress 라는 명령어를 사용합니다.
        CPU별로 사용량을 확인할 수 있는 대표적인 도구로 htop이 있습니다.

        </aside>

    - **--cpu-period, quota** : 컨테이너의 CFS(Completely Fair Schedler) 주기는 기본적으로 100ms로 설정되지만 이 옵션으로 주기를 변경할 수 있습니다. quota 옵션으로 설정된 period 시간중에 얼마나 CPU스케줄링에 할당할 것인지 정합니다. [quota 값] / [period 값] 만큼 CPU시간을 할당받습니다.
    - **--cpus** : 위 period, quota와 동일한 기능이지만 좀 더 직관적으로 CPU개수를 직접 정할 수 있습니다.
- **Block I/O 제한** : 기본적으로 컨테이너를 생성할 땐 IO옵션을 설정하지 않으면 파일을 읽고 쓰는 대역폭에 제한이 없습니다. 컨테이너가 블록 입출력에 제한을 설정해야 한다면
    - --device-write-bps
    - --device-read-bps
    - --device-write-iops
    - --device-read-iops

  옵션을 지정해 블록 입출력을 제한할 수 있습니다. 단 Direct IO에 경우는 블록 입출력만 제한되며 Buffered IO는 제한이 되지 않습니다.

- **스토리지 드라이버와 컨테이너 저장 공간 제한** : 컨테이너 내부의 저장 공간을 제한하는 기능을 보편적으로 제공하지 않지만, 스토리지 드라이버나 파일 시스템등 특정 조건을 만족하는 경우에만 이 기능을 제한적으로 사용할 수 있습니다.