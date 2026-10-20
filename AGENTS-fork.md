# AGENTS-fork.md

이 저장소는 `binwiederhier/ntfy`의 포크다. 이 문서는 포크에만 있는 파일과 운영 규칙을 다룬다.

## 포크 전용 파일

| 파일 | 역할 |
|---|---|
| `railway/Dockerfile` | Railway용 빌드. `Dockerfile-build`에서 BuildKit 캐시 마운트와 문서 빌드를 뺐다 |
| `railway/server.yml` | `ntfy.zz.gg` 서버 설정. 이미지의 `/etc/ntfy/server.yml`로 복사된다 |

## 배포

- Railway 개인 워크스페이스의 `ntfy` 프로젝트, `ntfy` 서비스에서 운영한다. 리전은 Singapore다.
- Railway가 Config as Code(`railway.toml`)를 deprecated 처리했으므로, 빌드·배포 설정은 저장소 파일이 아니라 서비스 설정에 둔다.
  - Dockerfile 경로 `railway/Dockerfile`, 헬스체크 `/v1/health`(타임아웃 300초), 재시작 정책 `ON_FAILURE`(최대 10회)
- 도메인은 `https://ntfy.zz.gg`이다. Cloudflare `zz.gg` 존에 DNS only CNAME으로 연결한다.
- `main`에 push하면 Railway가 `railway/Dockerfile`로 빌드해서 바로 배포한다. upstream 동기화 머지도 push하는 순간 배포된다.
- 메시지 캐시와 사용자 DB는 `/var/lib/ntfy` 볼륨에 있다.
- 비공개 서버다(`auth-default-access: deny-all`). 사용자와 토큰은 볼륨의 DB에 있으므로 컨테이너 안에서 CLI로 관리한다. `railway ssh`를 쓰려면 Railway에 SSH 키가 등록되어 있어야 한다(`railway ssh keys add`).
  - 사용자 목록: `railway ssh --service ntfy -- ntfy user list`
  - 사용자 추가: `railway ssh --service ntfy -- sh -c 'NTFY_PASSWORD=<password> ntfy user add <user>'`
  - 토큰 발급: `railway ssh --service ntfy -- ntfy token add <user>`

## upstream 동기화 때 확인할 것

- `Dockerfile-build`의 빌드 단계(Go·Node 버전, Makefile 타깃)가 바뀌면 `railway/Dockerfile`에도 반영한다.
- upstream `server/server.yml`에서 쓰고 있는 설정 키의 이름이나 의미가 바뀌면 `railway/server.yml`도 고친다.
