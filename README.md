# real-time-inference

- 메뉴/매출 추론
    - 메뉴 정보는 Item 테이블에서 가져오는 걸로 (얘는 실제 코드도 그렇게 작성해야 함)
    - 매출 추론은 menu tracking_id로 예측 한다고 적는 걸로.
 
# 핵심기술 (사업 계획서 p.11 부터)

1. [IoB + 딥러닝(CNN)] 카페 내 현황(**좌석 점유율**, **고객 외형**, **고객 행동**, **주문 메뉴**)을 실시간 추론 
    
    달성과제: IoB장치(Raspberry Pi 4B)에 탑재를 위한 CNN모델 경량화 + 정확도 및 속도 유지 
    
    수행기능: 실시간 영상 분석을 통한 카페 내 현황 실시간 데이터 마이닝(**4종의 CNN 모델**) 
    
|추론 항목|비고|
|----|---|
|좌석 점유율|시간대별 좌석 점유현황 실시간 파악|
|고객 성별 / 연령대|익명성 고려 성별 / 연령대 파악|
|고객 행동 패턴|카페 내 행동패턴 (코딩, 대화, 회의 등)|
|주문 메뉴 / 매출|주문메뉴 추론을 통한 매장 매출 추정|

# 하드웨어 정보
 - Raspberry Pi 4 Model B Rev 1.5
   
| CPU | Broadcom BCM2711, Quad core Cortex-A72 (ARM v8) 64-bit |
| --- | --- |
| RAM | 2GB LPDDR4-3200 SDRAM |
| HDD | 32GB |
| GPU | Broadcom VideoCore VI |

# 소프트웨어 정보
| Distributor ID | Debian |
| --- | --- |
| Description | Debian GNU/Linux 11 (bullseye) |
| Release | 11 |
| Codename | bullseye |

# 사전 조건
## Raspberry Pi [OpenCV 설치 Qengineering 링크](https://qengineering.eu/install-opencv-on-raspberry-64-os.html)
위 링크에서 Method 3을 참고 하였습니다.   
Raspberry Pi에서 OpenCV를 설치하는 방법 중 가장 좋은 방법은 소스 코드를 컴파일 하는 방법이기에 Method 3을 참고 하였습니다.   
항상 최신 버전으로 사용이 가능하며, C++ 라이브러리랑 Python 바인딩을 생성합니다.   
빌드를 시작 하기 전에 스왑 메모리를 늘려야 합니다. OpenCV에는 최소 총 5.8GB의 메모리가 필요합니다.   
설치를 진행 하기 전 아래 명령어를 먼저 진행 후 vim 코드 편집기를 설치 해 줍니다.   
``` shell
# 설치 진행 하기 전 업데이트 및 업그레이드 확인.
$ sudo apt-get update
$ sudo apt-get upgrade
# vim 코드 편집기 설치.
$ sudo apt-get install vim
```
아래 코드를 이용하여 스왑 메모리를 늘릴 수 있습니다.   
``` shell
# 최대 스왑 메모리 확장 ( CONF_MAXSWAP )
$ sudo vim /sbin/dphys-swapfile
```
<img width="653" alt="image" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044040/4b606b9e-44c8-4140-ab5e-2d4774202eb5">

![/sbin/dphys-swapfile](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/e11c6eec-f61d-439b-a7e7-196635985495)

nano를 사용해서 코드를 수정 합니다.

이 명령은 /sbin/phys-swapfile 시스템 파일이 있는 매우 가벼운 텍스트 편집기인 Nano를 엽니다 . 화살표 키를 사용하면 새 값 4096을 입력할 수 있는 CONF_MAXSWAP 라인 으로 커서를 이동할 수 있습니다 . 그런 다음 <Ctrl+X> 키 조합을 사용하여 세션을 닫습니다. <Y>와 <Enter>를 누르면 변경 사항이 저장됩니다. 동일한 절차를 사용하여 /etc/dphys-swapfile 에서 CONF_SWAPSIZE를 변경할 수 있습니다 .
슬라이드 쇼에서는 절차를 명확하게 보여줍니다.

``` shell
# 위와 동일한 방법으로 메모리 확장 (CONF_SWAPSIZE)
$ sudo nano /etc/dphys-swapfile
# 위 방법을 전부 진행 후 재부팅
$ sudo reboot
```
![/etc/dphys-swapfile](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/2774e6ea-e8f0-4055-9a61-cf269379d84e)

![image](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/0731d748-b5cb-4be2-9fcf-5a2eca1fb9bd)

``` shell

# git 코드 복제
$ git clone https://github.com/MAZE-Inc/real-time-inference.git
# 패키지 설치.
$ pip3 install -r requirements.txt
```
| 사전조건 | Python >= 3.7
gitpython>=3.1.30   
matplotlib>=3.3   
numpy>=1.18.5   
opencv-python>=4.1.1   
Pillow>=7.1.2   
psutil   
PyYAML>=5.3.1   
requests>=2.23.0   
scipy>=1.4.1   
thop>=0.1.1   
torch>=1.7.0  # see https://pytorch.org/get-started/locally (recommended)   
torchvision>=0.8.1   
tqdm>=4.64.0   
ultralytics>=8.0.100 
onnxruntime
|
| --- | --- |

    **➡️ 4개의 CNN 모델**
## 좌석 점유율 실시간 추론


## 고객 성별 / 연령대 실시간 추론

## 고객 행동 패턴 실시간 추론

## 주문 메뉴 / 매출 실시간 추론
