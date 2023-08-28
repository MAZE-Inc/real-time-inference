# A-MAZE: real-time-inference

- [IoB + 딥러닝] 카페 내 현황(**좌석 점유율**, **고객 외형**, **고객 행동**, **주문 메뉴**)을 실시간 추론   
- IoT 장치(Raspberry Pi 4B)에 탑재를 위한 모델 경량화 + 정확도 및 속도 유지   
- 실시간 영상 분석을 통한 카페 내 현황 실시간 데이터 마이닝(**4종의 모델**)   

|추론 항목|비고|
|----|---|
|좌석 점유율|시간대별 좌석 점유현황 실시간 파악|
|고객 성별|익명성 고려 성별 파악|
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
설치를 진행 하기 전 아래 명령어를 먼저 진행 후 vim 코드 편집기를 설치해 줍니다.   
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
<img width="619" alt="image" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044040/4b606b9e-44c8-4140-ab5e-2d4774202eb5">

vim 편집기로 파일을 열고 <**/CONF_MAXSWAP**> 을 통해서 CONF_MAXSWAP이 있는 줄로 이동이 가능합니다.   
해당 줄로 이동하여 <**i**> 키를 눌러 insert 모드로 변경합니다.   
<**CONF_MAXSWAP=4096**> 으로 변경 후   
<**esc**> 키를 입력 후 <**:wq!**> 를 입력하여 저장합니다.   

아래 명령어를 통해서 /etc/dphys-swapfile 내에도 있는 파일을 변경해 줍니다.

```shell
# 위와 동일한 방법으로 메모리 확장 (CONF_SWAPSIZE)
$ sudo vim /etc/dphys-swapfile
```

<img width="619" alt="image" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044040/229d47b2-7105-421f-af67-709b3d21e23a">

확장을 마친 후 재부팅을 진행해 줍니다.
``` shell
# 재부팅 명령어
$ sudo reboot
```

재부팅 진행 후 아래 코드를 통해서 메모리를 확인 할 수 있습니다   
``` shell
# 메모리 확인 명령어
$ free -m
```

![image](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/3d93b0dd-59d5-4c83-8a4b-d0b125dfd39a){: width="619"}

RAM이 1GB인 경우 4096이 아닌 5120으로 스왑 메모리를 확장 해야합니다.   
4GB도 마찬가지로 2048GB의 스왑 메모리로 진행 하면 충분 합니다. 4GB의 경우 /sbin/dphys-swapfile 에서 CONF_MAXSWAP를 수정 할 필요가 없습니다.   
모든 설치가 끝난 후 스왑 메모리를 원래 크기로 복원해 놓는 것이 좋습니다.   
굳이 불필요하게 SD 카드를 낭비 할 필요가 없습니다.   

다음으로 진행 할 부분은 아래 그림을 참고하여 GPU 메모리를 128 Mbyte로 변경해 주세요.   
![image](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/8656d751-e6a3-48eb-a384-2b946977b6b0){: width="619"}

메모리 확장을 다 진행 했다면 이제 OpenCV 4.8 버전을 설치 스크립트 작성을 진행합니다.  
전체 설치 과정은 1시간 30분이 소요되며 스왑 메모리(5.8 GB)가 충분한지 확인해 주세요 !

``` shell
# 메모리를 확인합니다.
$ free -m
# 전체 메모리가 5.8 GB 이 필요합니다. 만약 5.8 GB 보다 작다면 위의 과정을 참고하세요 !
# 아래는 OpenCV 설치 명령어입니다.
$ wget https://github.com/Qengineering/Install-OpenCV-Raspberry-Pi-64-bits/raw/main/OpenCV-4-8-0.sh
$ sudo chmod 755 ./OpenCV-4-8-0.sh
$ ./OpenCV-4-8-0.sh
```

위 과정을 진행 후 종속성 설치가 잘 진행 되었는지 확인합니다.   
설치가 안되서 생기는 문제보다 한번 더 확인 하는 것을 추천합니다.   

``` shell
# 업데이트 확인
$ sudo apt-get update
$ sudo apt-get upgrade
# 종속성 확인.
$ sudo apt-get install build-essential cmake git unzip pkg-config
$ sudo apt-get install libjpeg-dev libpng-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo apt-get install libgtk2.0-dev libcanberra-gtk* libgtk-3-dev
$ sudo apt-get install libgstreamer1.0-dev gstreamer1.0-gtk3
$ sudo apt-get install libgstreamer-plugins-base1.0-dev gstreamer1.0-gl
$ sudo apt-get install libxvidcore-dev libx264-dev
$ sudo apt-get install python3-dev python3-numpy python3-pip
$ sudo apt-get install libtbb2 libtbb-dev libdc1394-22-dev
$ sudo apt-get install libv4l-dev v4l-utils
$ sudo apt-get install libopenblas-dev libatlas-base-dev libblas-dev
$ sudo apt-get install liblapack-dev gfortran libhdf5-dev
$ sudo apt-get install libprotobuf-dev libgoogle-glog-dev libgflags-dev
$ sudo apt-get install protobuf-compiler
```

OpenCV 설치 
``` shell
# 메모리 체크 먼저 진행합니다.
$ free -m
# 5.8 GB가 충분히 있는지 확인하세요.
# 다음에 아래 명령어를 진행 합니다.
$ cd ~
$ git clone --depth=1 https://github.com/opencv/opencv.git
$ git clone --depth=1 https://github.com/opencv/opencv_contrib.git
```

아래 과정을 통해 빌드를 진행합니다.
``` shell
$ cd ~/opencv
$ mkdir build
$ cd build

$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
    -D ENABLE_NEON=ON \
    -D WITH_OPENMP=ON \
    -D WITH_OPENCL=OFF \
    -D BUILD_TIFF=ON \
    -D WITH_FFMPEG=ON \
    -D WITH_TBB=ON \
    -D BUILD_TBB=ON \
    -D WITH_GSTREAMER=ON \
    -D BUILD_TESTS=OFF \
    -D WITH_EIGEN=OFF \
    -D WITH_V4L=ON \
    -D WITH_LIBV4L=ON \
    -D WITH_VTK=OFF \
    -D WITH_QT=OFF \
    -D WITH_PROTOBUF=ON \
    -D OPENCV_ENABLE_NONFREE=ON \
    -D INSTALL_C_EXAMPLES=OFF \
    -D INSTALL_PYTHON_EXAMPLES=OFF \
    -D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-packages \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D BUILD_EXAMPLES=OFF ..
```

순조롭게 진행되었을 때 아래와 같은 스크린샷이 나옵니다.   
![image](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/121e6760-7cf4-4452-90d5-a7a2868daad4){: width="619"}

위와 같은 결과창이 나오면 아래 명령어를 입력합니다.   
``` shell
$ make -j4
```

아래와 같은 결과가 나오면 성공입니다.   

![image](https://github.com/MAZE-Inc/real-time-inference/assets/129044040/b0fa9da5-e20c-4629-9320-95d61e8bb9fd){: width="619"}

다음 명령어를 입력하여 모든 패키지를 설치합니다.
``` shell
$ sudo make install
$ sudo ldconfig
# cleaning 과정. 300KB의 여유 공간이 필요합니다.
$ make clean
$ sudo apt-get update
```

아래 그림처럼 확인이 가능합니다.   
<img width="619" alt="image" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044040/35c071df-f4a5-4438-8d7b-3e4eab56093a">

다음같이 진행이 되면 이제 위에 말했던대로 스왑 메모리를 낮춰줍니다.   
SD 메모리 카드는 제한된 수의 사이클만 사용 할 수 있는데 최소한으로 유지를 해야 SD 카드의 마모를 막을 수 있습니다.
``` shell
$ sudo nano /sbin/dphys-swapfile
# set CONF_MAXSWAP=2048 with the Nano text editor
$ sudo nano /etc/dphys-swapfile
# set CONF_SWAPSIZE=100 with the Nano text editor
$ sudo reboot
```

그리고 1.5GB 공간을 더 확보할 수 있는 방법으로 아래 명령어를 사용하면 됩니다.   
sudo make install을 통해 이미 원본을 복사해 놨기 때문에 지워도 됩니다.
``` shell
# 공간을 절약하기 위한 팁
$ sudo rm -rf ~/opencv
$ sudo rm -rf ~/opencv_contrib
```

이제 OpenCV 설치를 마쳤습니다.   

## 실행
이제 코드를 실행하기 위해 복제를 진행합니다.   

``` shell
# git 코드 복제
$ git clone https://github.com/MAZE-Inc/real-time-inference.git
```
<img width="619" alt="clone" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044938/80010479-c739-48ae-ab6b-4f2adee57d27">

```shell
# 패키지 설치.
$ pip3 install -r requirements.txt
```
<img width="619" alt="install" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044938/2014e20b-c42d-4ec3-813d-25578288c01c">

패키지 내에 버전은 아래와 같습니다.   
``` shell
Python >= 3.7
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
```
torch의 경우 [pytorch](https://pytorch.org/get-started/locally) 링크로 이동하여 다운로드 받으면 됩니다.
<https://pytorch.org/get-started/locally>


# 실시간 추론 4개의 모델
## 좌석 점유율 실시간 추론
- 아래 이미지를 보면 6개의 좌석에 4번 테이블과 6번 테이블에 사람이 앉아 있으므로 점유율은 2/6으로 33.3%가 된다.
- 좌석에 앉은 사람이 해당 좌석에 해당하는지 판단하는 부분은 객체 인식 기술과 메이즈 알고리즘을 통해 실시간 추론을 진행한다.
<img width="619" alt="스크린샷 2023-08-29 오전 1 13 18" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044938/7a7f61fc-3521-442a-93f2-6db358246dbf">

## 고객 성별 / 연령대 실시간 추론
<img width="619" alt="image" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044040/89e3a736-8437-4b6d-be8c-bfc5b24d6e30">

## 고객 행동 패턴 실시간 추론
<img width="619" alt="스크린샷 2023-08-29 오전 12 36 52" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044938/2b47d5ad-ec43-474c-8d4e-5c5405fb5f0b">

## 주문 메뉴 / 매출 실시간 추론
<img width="621" alt="스크린샷 2023-08-29 오전 1 00 31" src="https://github.com/MAZE-Inc/real-time-inference/assets/129044938/f730ccbe-f1c6-498a-9b59-47f14c17705a">

