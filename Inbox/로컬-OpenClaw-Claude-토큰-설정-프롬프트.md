# 로컬 OpenClaw → VPS Claude Code 토큰 설정 프롬프트

> 이 내용을 로컬 OpenClaw에게 복사해서 전달하세요.

---

## 프롬프트

```
VPS에 Claude Code CLI가 설치되어 있는데 인증이 필요합니다.

다음 단계를 실행해주세요:

1. SSH로 VPS 접속
   ssh root@vmi3116112.contaboserver.net

2. Claude 토큰 설정 명령 실행
   export PATH="$HOME/.local/bin:$PATH"
   claude setup-token

3. 나오는 안내에 따라 토큰 입력

완료되면 인증 상태 확인:
   claude auth status

성공하면 "loggedIn": true가 나와야 합니다.
```

---

## SSH 접속 정보

- **Host:** vmi3116112.contaboserver.net
- **User:** root
- **명령:** `claude setup-token`

---

## 토큰 획득 방법

1. https://claude.ai/ 접속 (로그인 필요)
2. 로그인 후 토큰 발급 페이지로 이동
3. 토큰 복사

또는 `claude setup-token` 명령 실행 시 안내되는 URL에서 토큰 획득

---

## 완료 확인

로컬 OpenClaw가 다음을 실행해서 확인:

```bash
ssh root@vmi3116112.contaboserver.net "export PATH=\$HOME/.local/bin:\$PATH && claude auth status"
```

**성공 응답:**
```json
{
  "loggedIn": true,
  "authMethod": "subscription",
  "apiProvider": "firstParty"
}
```

---

**생성일:** 2026-03-01
**작성자:** 유용현 (VPS PM)
