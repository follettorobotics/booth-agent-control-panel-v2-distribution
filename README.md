# Control panel distribution

Folletto 부스 컨트롤 패널의 서명된 업데이트 파일입니다. 빌드된 앱만 있고 소스 저장소는 여기 없습니다.
부스의 컨트롤 패널이 이 저장소를 직접 읽습니다. 여기 있는 파일은 손으로 고치지 않습니다.

Signed updates of the Folletto booth control panel. The built app only: the source repository is not here.
The control panel on each booth reads this repository by itself; nothing here is edited by hand.

```
latest.json                  what booths are offered
latest.json.sig              its signature (ECDSA P-256 / SHA-256, base64 of the DER signature over the exact bytes)
release v<X.Y.Z>             control-panel-<X.Y.Z>.asar      the update: the panel's app code, about 1 MB
                             ControlPanel-Setup-<X.Y.Z>.exe  the Windows installer, only on some releases
```

## 부스 PC에 설치하기

처음 설치할 때와 설치 프로그램이 필요한 업데이트(화면에 "이 버전은 설치 프로그램으로 설치해야 합니다"라고 나올 때)만
손으로 설치합니다. 그 밖의 업데이트는 컨트롤 패널의 설정 > 일반 설정 > 소프트웨어 업데이트에서 합니다.

1. 주문을 받지 않는 시간에 합니다. 설치 프로그램은 실행 중인 컨트롤 패널을 닫습니다.
2. [Releases](../../releases)에서 `ControlPanel-Setup-<버전>.exe`가 있는 가장 최신 릴리스를 찾아 내려받습니다.
3. 컨트롤 패널을 평소에 실행하는 Windows 계정으로 로그인한 상태에서 실행합니다. 관리자 권한은 필요 없습니다.
   "Windows의 PC 보호" 창이 나오면 추가 정보 > 실행을 누릅니다(서명되지 않은 설치 프로그램입니다).
4. 설치가 끝나면 컨트롤 패널이 열립니다. 설정 > 일반 설정에서 버전과 부스 프로그램(main.exe) 경로를 확인합니다.
   경로가 없으면 화면 위쪽에 빨간 안내가 나옵니다. 같은 화면에서 이 부스의 설정 비밀번호도 만듭니다.
5. 그다음부터는 컨트롤 패널이 새 버전을 스스로 확인합니다.

화면이 깨져서 아무것도 누를 수 없으면: 컨트롤 패널을 닫고 시작 메뉴의 "Control Panel (safe mode)"로 연 다음
설정 > 일반 설정에서 이전 버전으로 되돌립니다. 설치 프로그램을 다시 실행해도 됩니다.

## Installing on a booth PC

Only the first install, and an update the panel shows as "needs the installer", are done by hand. Every other
update is done from the panel: Settings > General Settings > Software update.

1. Do it while the booth takes no orders: the installer closes a running control panel.
2. On [Releases](../../releases), take the newest release that has a `ControlPanel-Setup-<version>.exe`.
3. Run it under the Windows account that normally runs the control panel. It needs no administrator rights.
   The installer is not code-signed: on "Windows protected your PC" choose More info > Run anyway.
4. The panel opens when the install ends. Check the version and the booth program (main.exe) path on
   Settings > General Settings; a red bar at the top says so while the path is missing. Create this booth's
   settings password on the same page.
5. From then on the panel looks for new versions by itself.

A broken screen where nothing can be pressed: close the panel, start "Control Panel (safe mode)" from the Start
menu and roll back on Settings > General Settings. Running the installer again also works.

## How a booth uses this repository

A panel reads `https://raw.githubusercontent.com/follettorobotics/booth-agent-control-panel-v2-distribution/main/latest.json`
and `latest.json.sig`, verifies the signature with the public keys built into it, downloads the `.asar` the
manifest names, checks its size and sha256, and starts it when staff press Update, or by itself when the booth is
set to automatic or scheduled updates and the screen is not in use. A panel never installs a lower version, a file
for another program, or anything the signature does not cover. The booth PC must reach `raw.githubusercontent.com`,
`github.com` and `release-assets.githubusercontent.com` over HTTPS.

## Rules

- **Never edit `latest.json` or `latest.json.sig`.** Any change breaks the signature and every booth stops seeing
  updates. `.gitattributes` (`* -text`) keeps git from changing a byte of them.
- **Never replace the `.asar` of a version `latest.json` has named**: booths pin its sha256.
- Releases are made only by the release workflow of the private source repository, from a tag `v<X.Y.Z>`.

## Withdrawing a release

Revert the commit that published it. The previous two files come back byte for byte and still verify, so booths
that have not installed the withdrawn version are no longer offered it.

```
git clone https://github.com/follettorobotics/booth-agent-control-panel-v2-distribution.git
cd booth-agent-control-panel-v2-distribution
git log --oneline -- latest.json          # find "Release v<X.Y.Z>"
git revert <that commit>
git push origin main
```

Then delete the withdrawn release (assets and tag). Only the newest release can be withdrawn this way. A booth that
already installed the withdrawn version stays on it: staff press Roll back on the booth, or the fix arrives as a
higher version. A withdrawn number is never used again.
