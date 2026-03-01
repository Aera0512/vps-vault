# 로컬 OpenClaw Git 자동 동기화 설정

> 이 문서를 로컬 OpenClaw의 `HEARTBEAT.md` 또는 `AGENTS.md`에 추가하세요.

---

## HEARTBEAT.md 추가 내용

```markdown
## 옵시디언 볼트 자동 동기화

Heartbeat 시 자동으로 Git 동기화 실행:

### knowledge-vault (개인 볼트)
```bash
cd /path/to/knowledge-vault
git pull origin master --rebase
if [ -n "$(git status --porcelain)" ]; then
    git add .
    git commit -m "Local sync: $(date '+%Y-%m-%d %H:%M')"
    git push origin master
fi
```

### vps-vault (VPS 볼트)
```bash
cd /path/to/vps-vault
git pull origin master
```

**주의:**
- 로컬 경로를 실제 볼트 위치로 수정하세요
- 충돌 방지를 위해 `--rebase` 사용
- 변경사항이 있을 때만 커밋
```

---

## AGENTS.md 추가 내용

```markdown
## Git 볼트 동기화 규칙

### 원칙
1. **로컬 우선**: 로컬에서 변경한 내용은 즉시 커밋
2. **주기적 풀**: Heartbeat마다 원격 변경사항 확인
3. **충돌 방지**: rebase로 깔끔한 히스토리 유지

### 동기화 대상 볼트

| 볼트 | 경로 | 용도 | 방향 |
|------|------|------|------|
| knowledge-vault | `~/Obsidian/knowledge-vault` | 개인 노트 | 양방향 |
| vps-vault | `~/Obsidian/vps-vault` | VPS 작업물 | pull 전용 |

### 자동 동기화 트리거

- **Heartbeat**: 30분마다 자동 동기화
- **파일 변경 감지**: 옵시디언 종료 시 자동 커밋
- **수동 요청**: "볼트 동기화해줘" 명령

### 충돌 해결

충돌 발생 시:
1. 로컬 변경사항 백업
2. 원격 버전 우선 적용
3. 사용자에게 알림
```

---

## 설정 스크립트 (sync-vault.sh)

로컬에 저장 후 실행 권한 부여:

```bash
#!/bin/bash
# 로컬 옵시디언 볼트 동기화

# === 설정 ===
KNOWLEDGE_VAULT="$HOME/Obsidian/knowledge-vault"
VPS_VAULT="$HOME/Obsidian/vps-vault"
LOG_FILE="/tmp/local-vault-sync.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

# === knowledge-vault (양방향) ===
sync_knowledge() {
    log "Syncing knowledge-vault..."
    cd "$KNOWLEDGE_VAULT" || return

    # 원격 변경사항 가져오기
    git fetch origin
    git pull origin master --rebase 2>&1 >> "$LOG_FILE"

    # 로컬 변경사항 커밋 & 푸시
    if [ -n "$(git status --porcelain)" ]; then
        git add .
        git commit -m "Local auto-sync: $(date '+%Y-%m-%d %H:%M')"
        git push origin master 2>&1 >> "$LOG_FILE"
        log "knowledge-vault: Pushed local changes"
    else
        log "knowledge-vault: No local changes"
    fi
}

# === vps-vault (pull 전용) ===
sync_vps() {
    log "Syncing vps-vault..."
    cd "$VPS_VAULT" || return

    git pull origin master 2>&1 >> "$LOG_FILE"
    log "vps-vault: Pulled remote changes"
}

# === 실행 ===
sync_knowledge
sync_vps

log "Sync complete"
```

### 설치 방법

```bash
# 1. 스크립트 저장
nano ~/sync-vault.sh
# 위 내용 붙여넣기

# 2. 실행 권한
chmod +x ~/sync-vault.sh

# 3. 경로 수정
# KNOWLEDGE_VAULT, VPS_VAULT를 실제 경로로 변경

# 4. 테스트
~/sync-vault.sh
```

---

## HEARTBEAT.md 완전 예시

```markdown
# HEARTBEAT.md

# Keep this file empty (or with only comments) to skip heartbeat API calls.

# Add tasks below when you want the agent to check something periodically.

---

## 자동 동기화

Heartbeat 시 볼트 자동 동기화 실행:

```bash
~/sync-vault.sh
```

### 동기화 대상
- **knowledge-vault**: 양방향 (push + pull)
- **vps-vault**: 단방향 (pull only)

### 주기
- Heartbeat마다 실행 (약 30분)
- 변경사항이 있을 때만 커밋

---

## 체크할 것들

- [ ] 이메일 확인
- [ ] 캘린더 확인
- [ ] 날씨 확인 (필요시)
```

---

## 빠른 설정 가이드

### 1단계: 로컬에서 볼트 클론

```bash
cd ~/Obsidian  # 또는 원하는 위치
git clone https://github.com/Aera0512/knowledge-vault.git
git clone https://github.com/Aera0512/vps-vault.git
```

### 2단계: 동기화 스크립트 설치

```bash
# 위의 sync-vault.sh 내용을 ~/sync-vault.sh로 저장
chmod +x ~/sync-vault.sh
```

### 3단계: HEARTBEAT.md 수정

로컬 OpenClaw의 `HEARTBEAT.md`에 위 내용 추가

### 4단계: 테스트

```bash
# 수동 실행
~/sync-vault.sh

# 로그 확인
cat /tmp/local-vault-sync.log
```

---

## 문제 해결

### 충돌 발생 시

```bash
cd ~/Obsidian/knowledge-vault
git status
git mergetool  # 또는 수동으로 충돌 해결
git rebase --continue
```

### 권한 문제

```bash
chmod +x ~/sync-vault.sh
```

### 경로 문제

스크립트 내 `KNOWLEDGE_VAULT`, `VPS_VAULT` 경로 확인

---

**생성일:** 2026-03-01
**작성자:** 유용현 (OpenClaw VPS)
