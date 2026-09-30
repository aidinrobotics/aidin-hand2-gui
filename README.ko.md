<div align="center">

<a href="https://www.aidinrobotics.co.kr/"><img height="240" src="docs/assets/aidin_hand2_logo.webp" alt="AIDIN Hand Gen2 — AIDIN Robotics"></a>

<h1>AIDIN Hand Gen2 GUI</h1>

촉각 센서가 통합된 로봇 핸드 AIDIN Hand Gen2용 GUI입니다. SDK나 ROS 2를 통해 로봇 핸드에 연결합니다.

[![SDK](https://img.shields.io/badge/SDK-0.7.x-blue)](https://github.com/aidinrobotics/aidin-hand2-sdk) [![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange)](#1-system-requirements)

[Install](#2-install-the-dependencies) | [Tutorial](#4-tutorial) | [Official Site](https://www.aidinrobotics.co.kr/) | [English](README.md) | 한국어

</div>

## Contents

&nbsp;&nbsp;[**1. System requirements**](#1-system-requirements)<br>
&nbsp;&nbsp;[**2. Install the dependencies**](#2-install-the-dependencies)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[2.1 Install the SDK](#21-install-the-sdk)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[2.2 Install the ROS 2 wrapper](#22-install-the-ros-2-wrapper)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[2.3 Set up the real-time kernel](#23-set-up-the-real-time-kernel)<br>
&nbsp;&nbsp;[**3. Install the app**](#3-install-the-app)<br>
&nbsp;&nbsp;[**4. Tutorial**](#4-tutorial)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[4.1 Basics](#41-basics)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[4.2 Connect through ROS 2](#42-connect-through-ros-2)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[4.3 Connect through the SDK](#43-connect-through-the-sdk)<br>
&nbsp;&nbsp;[**5. Close the app**](#5-close-the-app)<br>
&nbsp;&nbsp;[**6. Troubleshooting**](#6-troubleshooting)<br>
&nbsp;&nbsp;[**7. Uninstall**](#7-uninstall)<br>
&nbsp;&nbsp;[**8. Related repositories**](#8-related-repositories)<br>
&nbsp;&nbsp;[**9. License**](#9-license)

## 1. System requirements

| Component | Requirement |
|---|---|
| Operating System | Ubuntu 22.04 (amd64 · arm64) |
| SDK | `aidin_hand2` 0.7.x, `/usr/local`에 설치 |
| CAN interface | SDK로 로봇 핸드를 구동할 때 USB CAN-FD adapter (SocketCAN), nominal 1 Mbit/s, data phase 5 Mbit/s |
| ROS 2 | ROS 2로 연결할 때 로봇 핸드를 연결한 PC에 ROS 2 Humble과 [aidin-hand2-ros2](https://github.com/aidinrobotics/aidin-hand2-ros2) |

## 2. Install the dependencies

앱은 화면과 로봇 핸드를 잇기 위해 HTTP·WebSocket 통신 기반의 web bridge 서버(`aidin_hand2_web_bridge`)를
함께 실행합니다. `web bridge` 프로필(Standalone)이면 web bridge가 SDK로 로봇 핸드를 직접 구동하고, `ros2`
프로필이면 rosbridge를 거쳐 로봇 핸드에 연결합니다. 어느 쪽이든 SDK 라이브러리를 쓰므로, 앱을 실행하는 PC에는 SDK를 설치합니다.
ROS 2로 연결하는 PC와 로봇 핸드가 없는 PC도 마찬가지입니다.

2.2와 2.3은 로봇 핸드를 연결한 PC(아래에서 로봇 PC)에서 필요할 때만 합니다.

### 2.1 Install the SDK

SDK 빌드에 필요한 패키지를 설치하고 SDK 0.7 release를 받습니다.

```bash
sudo apt update
sudo apt install -y build-essential cmake git libspdlog-dev can-utils
git clone --branch v0.7.0 https://github.com/aidinrobotics/aidin-hand2-sdk.git
cd aidin-hand2-sdk
```

> [!IMPORTANT]
> **AIDIN Hand Gen2에는 hand type A·B·C가 있고, SDK를 빌드할 때 type을 선택합니다.** type에 따라
> kinematics가 다르게 계산됩니다. type은 로봇 핸드 전달과 함께 알려드립니다. `mock`만 쓴다면
> 기본값 `a`로 빌드해도 됩니다.

아래 명령의 `a`를 전달받은 type(`a`·`b`·`c`)으로 바꿔 실행하고, 출력에서 선택한 type을 확인합니다.

```bash
cmake -S cpp -B cpp/build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DAIDIN_HAND2_HAND_TYPE=a
```

```text
-- aidin_hand2: hand type = a
```

> [!IMPORTANT]
> SDK는 `/usr/local`에 설치하십시오. 앱은 `sudo ldconfig`로 등록된 라이브러리만 찾으므로, SDK 문서에
> 있는 사용자 prefix(`~/.local`) 설치로는 앱이 실행되지 않습니다.

빌드하고 설치합니다.

```bash
cmake --build cpp/build -j"$(nproc)"
sudo cmake --install cpp/build
sudo ldconfig
```

라이브러리가 등록되었는지 확인합니다. arm64 PC에서는 괄호 안이 `libc6,AArch64`로 표시됩니다.

```bash
ldconfig -p | grep libaidin_hand2.so.0.7
```

```text
	libaidin_hand2.so.0.7 (libc6,x86-64) => /usr/local/lib/libaidin_hand2.so.0.7
```

테스트와 제거 방법은 SDK의
[SDK build & install](https://github.com/aidinrobotics/aidin-hand2-sdk/blob/main/docs/ko/06_sdk_build_and_install.md)에
있습니다.

### 2.2 Install the ROS 2 wrapper

ROS 2로 연결할 때만 필요합니다. 앱은 로봇 PC에서 실행해도 되고, 같은 network의 다른 PC에서 실행해도
됩니다. 앱을 실행하는 PC에는 ROS 2가 없어도 되고, 2.1의 SDK와 앱만 있으면 됩니다.

로봇 PC에 ROS 2 Humble과 wrapper를
[Installation](https://github.com/aidinrobotics/aidin-hand2-ros2/blob/main/docs/ko/03_installation.md)대로
설치하십시오. 앱이 접속하는 rosbridge_server도 이 절차의 `rosdep install`이 함께 설치합니다.

### 2.3 Set up the real-time kernel

선택 사항입니다. 500 Hz 제어·통신 루프를 실시간으로 스케줄링하려면 로봇 PC에서
[Real-time kernel setup](https://github.com/aidinrobotics/aidin-hand2-sdk/blob/main/docs/ko/04_real_time_kernel_setup.md)대로
PREEMPT_RT kernel과 실시간 실행 권한을 설정합니다. 권한이 없으면 SDK는 경고를 남기고 실시간
스케줄링 없이 동작합니다.

## 3. Install the app

[Releases](https://github.com/aidinrobotics/aidin-hand2-gui/releases)에서 PC의 아키텍처에 맞는 `.deb`
파일을 받습니다. x86_64 PC는 `amd64`, Jetson 같은 arm64 PC는 `arm64` 파일입니다. 아키텍처는
`dpkg --print-architecture`로 확인할 수 있습니다.

받은 파일이 있는 폴더에서 설치합니다. `<version>`은 받은 파일의 버전으로 바꾸십시오.

```bash
sudo apt install ./aidin-hand2-gui_<version>_amd64.deb
```

설치하면 앱 메뉴에 **AIDIN Hand GUI**가 생기고, 터미널에서는 `aidin-hand2-gui`로 실행할 수 있습니다.
한 PC에서 앱은 하나만 실행됩니다. 이미 실행 중일 때 다시 실행하면 열려 있는 창이 앞으로 옵니다.

## 4. Tutorial

튜토리얼은 앱을 한 단계씩 따라 하게 합니다. 누를 곳을 밝혀 주고, 그 동작을 앱이 확인하면 무엇이
바뀌었는지 보여 줍니다. **Next**를 누르면 다음 단계로 넘어갑니다.

1. 앱 메뉴에서 **AIDIN Hand GUI**를 실행하거나 터미널에서 `aidin-hand2-gui`를 실행합니다.
2. 시작 화면 오른쪽 위에서 펼친 책 모양의 튜토리얼 버튼을 누릅니다. 튜토리얼 목록이 열립니다.

   <img src="docs/assets/tutorial-button.png" alt="시작 화면 오른쪽 위, 창 버튼 앞에 있는 펼친 책 모양의 튜토리얼 버튼" width="214">
3. 1장·2장·3장 중 하나의 **Start**를 누르거나, 그 아래의 Guide 하나를 눌러 그 Guide부터 시작합니다.

앱을 처음 쓴다면 1장 Basics부터 시작하십시오. **Quit tutorial**을 누르면 어느 단계에서든 튜토리얼을
끝냅니다.

> [!WARNING]
> 2장과 3장은 마지막에 로봇 핸드를 원점 복귀하며, 이때 손가락이 움직입니다. 로봇 핸드 주변을
> 비워 두십시오.

### 4.1 Basics

1장은 연습용 손으로 진행하므로 앱만 있으면 됩니다. 무엇을 눌러도 로봇 핸드에는 가지 않으며, 1장에서
만든 프리셋과 시퀀스는 튜토리얼이 지웁니다.

| Guide | 다루는 내용 |
|---|---|
| Hand states | 상단바에 표시되는 손의 상태, 연결, 실행과 원점 복귀, 정지, 진단 창, Faulted에서 복구 |
| Sending commands | State 패널, Joint Position과 Preview, Set command와 Live, 프리셋, zero target과 capture pose, controller 설정과 max effort, Iso·Front 시점 |
| Sequences | Home·Wait·command·Loop 블록으로 시퀀스 만들기, 재생, 블록 편집 단축키 |
| Tactile sensing | 촉각 패널, Raw·Normalized·Index 보기, Calibrate, 3D 손 위의 heatmap |

### 4.2 Connect through ROS 2

2장에는 로봇 핸드와, 로봇 PC에 [2.2 Install the ROS 2 wrapper](#22-install-the-ros-2-wrapper)로 설치한
wrapper가 필요합니다. 2장은 로봇 PC에서 rosbridge(`gui_bridge.launch.py`)와 bringup(`aidin_hand2.launch.py`)을
실행하는 명령과 `ros2` 프로필 만들기를 안내하고, 로봇 핸드의 원점 복귀를 마치면 끝납니다.

> [!WARNING]
> rosbridge는 인증과 TLS를 제공하지 않습니다. 기본값으로 실행하면 같은 network의 누구나 로봇 핸드에
> command를 보낼 수 있습니다. 앱을 로봇 PC에서만 실행한다면 `gui_bridge.launch.py` 명령에
> `address:=127.0.0.1`을 붙여 이 PC의 접속만 받으십시오.

| Guide | 다루는 내용 |
|---|---|
| Prepare the PC | ROS 2 workspace 확인, CAN-FD 설정, CAN interface에서 응답하는 손 찾기 |
| Start ROS 2 | rosbridge와 bringup 실행. 실행되었는지는 앱이 확인합니다 |
| Connect the GUI | `ros2` 프로필 추가, topic·service 이름 확인, ENTER, 로봇 핸드 원점 복귀 |

2장의 bringup 명령은 손 하나를 실행합니다. 양손과 그 밖의 launch 인자는 wrapper의
[2. Robot hand](https://github.com/aidinrobotics/aidin-hand2-ros2/blob/main/docs/ko/04_bringup.md#2-robot-hand)에
있습니다.

### 4.3 Connect through the SDK

3장에는 이 PC에 연결한 로봇 핸드가 필요합니다. 3장은 CAN-FD 명령과 `web bridge` 프로필 만들기를
안내하고, 로봇 핸드를 연결해 원점 복귀를 마치면 끝납니다. CAN-FD 명령의 자세한 설명은 SDK의
[CAN-FD setup](https://github.com/aidinrobotics/aidin-hand2-sdk/blob/main/docs/ko/05_can_fd_setup.md)에
있습니다.

| Guide | 다루는 내용 |
|---|---|
| Prepare the PC | SDK의 hand type 확인, CAN-FD 설정, 응답하는 손 찾기 |
| Create a profile | 손·CAN interface·`auto_home`을 담은 `web bridge` 프로필 추가 |
| Connect and home | ENTER, 로봇 핸드 연결, 실행과 원점 복귀 |

## 5. Close the app

창에는 제목 줄이 없고, 상단 바 오른쪽 끝의 버튼이 그 역할을 합니다. **−**는 창을 최소화하고, 전체 화면
버튼은 화면을 가득 채웁니다. 메인 화면의 **✕**는 프로필 화면으로 돌아가고, 프로필 화면의 **✕**는 앱을
닫습니다. 창을 옮기려면 상단 바의 빈 곳을 끕니다.

`web bridge`나 `ros2` 프로필로 들어간 뒤 앱을 닫았다가 다시 열면, 프로필 화면을 거치지 않고 그
프로필로 바로 들어갑니다. 다른 프로필로 바꾸려면 **✕**로 프로필 화면으로 돌아갑니다.

앱을 닫을 때 로봇 핸드가 어떻게 되는지는 프로필의 `source`에 따라 다릅니다.

| Source | 앱을 닫으면 |
|---|---|
| `web bridge` | 앱이 web bridge를 종료한 뒤 끝납니다. 로봇 핸드는 actuator를 정지하고 통신을 끊습니다 |
| `ros2` | 로봇 핸드는 로봇 PC에서 계속 실행됩니다. 끝내려면 로봇 PC에서 실행한 `aidin_hand2.launch.py`와 `gui_bridge.launch.py`를 Ctrl+C로 종료합니다 |

## 6. Troubleshooting

web bridge를 실행하지 못하면 창에 사유와 web bridge의 출력이 표시됩니다.

| Message | 조치 |
|---|---|
| `The AIDIN Hand SDK (libaidin_hand2) is not installed on this PC.` | SDK 0.7.x를 찾지 못했습니다. 다른 버전만 설치되어 있어도 이 메시지가 나옵니다. [2.1 Install the SDK](#21-install-the-sdk)를 마친 뒤 앱을 다시 실행합니다 |
| `Another program holds the web bridge port.` | 다른 프로그램이 포트 26350을 쓰고 있습니다. `sudo ss -ltnp \| grep 26350`으로 어떤 프로그램인지 확인합니다.<ul><li>`users:` 뒤의 이름이 `aidin_hand2_web`이면 전에 실행한 web bridge가 멈춘 채 남아 있는 것입니다. `pid=` 뒤의 번호로 `kill <pid>`를 실행해 종료한 뒤 앱을 다시 실행합니다. 끝나지 않으면 `kill -9 <pid>`를 씁니다</li><li>계속 써야 하는 다른 프로그램이면 터미널에서 `aidin-hand2-gui --port 26360`처럼 앱을 다른 포트로 실행합니다</li></ul> |
| `The web bridge keeps stopping.` | web bridge가 연달아 다섯 번 종료되었습니다. 창에 표시된 출력으로 원인을 확인합니다 |

## 7. Uninstall

```bash
sudo apt remove aidin-hand2-gui
```

프로필과 앱에서 저장한 설정은 `~/.local/share/aidin-hand2-gui`에 남으므로, 다시 설치하면 그대로
쓸 수 있습니다. 모두 지우려면 이 폴더와 `~/.config/AIDIN Hand GUI`를 삭제합니다. SDK를 제거하는
방법은 SDK의 [5. Uninstall](https://github.com/aidinrobotics/aidin-hand2-sdk/blob/main/docs/ko/06_sdk_build_and_install.md#5-uninstall)에
있습니다.

## 8. Related repositories

- [aidin-hand2-sdk](https://github.com/aidinrobotics/aidin-hand2-sdk) — C++ SDK
- [aidin-hand2-ros2](https://github.com/aidinrobotics/aidin-hand2-ros2) — ROS 2 wrapper

## 9. License

AIDIN Hand Gen2 GUI는 [Apache License 2.0](LICENSE)으로 배포됩니다. 라이선스가 하위 배포자에게
유지하도록 요구하는 고지 사항은 [NOTICE](NOTICE) 파일에 있습니다.

`.deb` 패키지에는 Electron과 페이지에 들어간 라이브러리 같은 서드파티 소프트웨어도 들어 있습니다.
구성 요소별 라이선스와 소스를 받을 수 있는 곳은 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)에
정리했고, 라이선스 전문은 [licenses/](licenses) 디렉터리에 두었습니다. 패키지는 이 파일들을 앱과 함께
`/opt/AIDIN Hand GUI/`에 설치합니다.
