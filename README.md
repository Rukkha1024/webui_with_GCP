# Docker 설치, Open WebUI 설치, 방화벽 설정

아래 가이드는 GCP 환경에서 Ubuntu 서버를 기준으로 작성되었습니다. 이 단계를 따라가면 Docker를 설치하고, Docker를 통해 Open WebUI를 구동하고, 방화벽 설정을 완료할 수 있습니다. 이 가이드는 깃허브에 게시하기 적합하도록 다듬어진 형태입니다.

---

## 1. Docker 설치

### 1-1. Docker APT 리포지토리 설정

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

### 1-2. Docker 패키지 설치

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 1-3. Docker 설치 확인

```
sudo docker run hello-world
```

위 명령어는 테스트 이미지를 다운로드하고 컨테이너를 실행합니다. 정상적으로 설치되었다면, "Hello from Docker!" 메시지를 확인할 수 있습니다.

### 1-4. Docker 권한 설정

```
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl restart docker
docker run hello-world
```

정상적으로 "Hello from Docker!"가 표시되면 권한 설정이 완료된 것입니다.

---

## 2. Open WebUI 설치 (Docker 사용)

CPU만 사용하는 예시:

```
sudo docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

- **-p 3000:8080**: 호스트의 3000번 포트를 컨테이너의 8080번 포트에 매핑합니다.
- **--restart always**: 컨테이너가 중단되더라도 자동으로 재시작하도록 설정합니다.
- **-v open-webui:/app/backend/data**: 앱 데이터를 호스트 볼륨에 저장하여, 컨테이너 재생성 시에도 데이터를 유지합니다.

### 2-1. 컨테이너 상태 확인

```
sudo docker ps
```

`open-webui`라는 이름의 컨테이너가 정상적으로 실행 중이면 설치가 완료된 것입니다.

---

## 3. 방화벽 설정 (GCP 콘솔 기준)

Open WebUI를 외부에서 접근하기 위해서는 GCP 방화벽 규칙이 허용되어야 합니다.

### 3-1. 방화벽 규칙 생성

1. GCP 콘솔 접속 후, **VPC 네트워크 > 방화벽 규칙**으로 이동
2. **"만들기"** 버튼 클릭
3. 다음과 같이 설정

   - **이름**: `allow-open-webui`
   - **네트워크**: `default` 또는 사용 중인 VPC 네트워크
   - **우선순위**: `1000` (기본값)
   - **대상 태그**: `open-webui-tag`
   - **소스 IP 범위**: `0.0.0.0/0` (모든 외부 IP 허용)
   - **프로토콜 및 포트**: `tcp:3000`

4. **만들기** 버튼 클릭

### 3-2. VM 인스턴스에 태그 추가

1. **Compute Engine > VM 인스턴스** 에서 해당 인스턴스 **수정**
2. **네트워크 태그**에 `open-webui-tag` 추가
3. **저장**

### 3-3. 방화벽 규칙 및 VM 태그 확인

```
gcloud compute firewall-rules list
gcloud compute instances describe [YOUR_INSTANCE_NAME] --format="get(tags.items)"
```

방화벽 규칙과 VM의 태그가 정상적으로 설정되었는지 확인할 수 있습니다.

---

## 4. 접속 테스트

방화벽 설정이 완료되면 다음과 같이 Open WebUI에 접근할 수 있습니다.

```
http://[YOUR_EXTERNAL_IP]:3000
```

브라우저에서 정상적인 페이지가 표시되면 모든 구성이 성공적으로 완료된 것입니다.

---

이상으로 Docker 설치부터 Open WebUI 구동, 그리고 GCP 방화벽 설정까지의 과정을 마칩니다. 깃허브에 올릴 문서라면, 추가적인 이미지나 예시, 문제 해결 방법 등을 보강하여 다른 사용자들이 쉽게 참고할 수 있도록 해주세요.

