Встановлення необхідних інструментів та бібліотек:
Встановіть всі залежності для збірки, включаючи інструменти для Python.
Bash

###
sudo apt install -y build-essential cmake pkg-config git
sudo apt install -y libjpeg-dev libtiff-dev libpng-dev libwebp-dev libopenexr-dev
sudo apt install -y libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libdc1394-22-dev
sudo apt install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev
sudo apt install -y gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl
sudo apt install -y libgtk-3-dev
sudo apt install -y libatlas-base-dev gfortran
sudo apt install -y python3-dev python3-pip
sudo apt install -y libqtgui4 libqtwebkit4 libqt4-test
sudo apt install -y libhdf5-dev libhdf5-103 # Переконайтеся, що версія hdf5 відповідає доступній у репозиторіях
sudo apt install -y libprotobuf-dev protobuf-compiler
sudo apt install -y libgoogle-glog-dev libgflags-dev libgphoto2-dev libtesseract-dev

2. Завантаження вихідного коду OpenCV

Завантажте вихідний код OpenCV та opencv_contrib. Рекомендується використовувати останню стабільну версію.
Bash

cd ~
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git

Якщо ви хочете конкретну версію (наприклад, 4.9.0):
Bash

git clone --branch 4.9.0 https://github.com/opencv/opencv.git
git clone --branch 4.9.0 https://github.com/opencv/opencv_contrib.git

3. Створення віртуального середовища Python та встановлення build

Використання віртуального середовища є обов'язковим для створення .whl файлу. Нам також знадобиться пакет build для пакування.
Bash

pip install virtualenv virtualenvwrapper
echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> ~/.bashrc
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
source ~/.bashrc
mkvirtualenv cv -p python3
workon cv

Після активації віртуального середовища (ви побачите (cv) перед командним рядком), встановіть build:
Bash

pip install build

4. Компіляція та створення .whl файлу

Створіть каталог для збірки та перейдіть до нього:
Bash

cd ~/opencv
mkdir build
cd build

Виконайте команду cmake. Важливо включити опцію WITH_GSTREAMER=ON та вказати, що ми хочемо створити Python-модуль.
Bash

cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
    -D ENABLE_V4L=ON \
    -D WITH_GSTREAMER=ON \
    -D WITH_LIBV4L=ON \
    -D BUILD_opencv_python3=ON \
    -D BUILD_opencv_python2=OFF \
    -D PYTHON_DEFAULT_EXECUTABLE=$(which python3) \
    -D BUILD_EXAMPLES=OFF \
    -D WITH_QT=ON \
    -D WITH_OPENGL=ON \
    -D BUILD_TESTS=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D INSTALL_PYTHON_EXAMPLES=OFF \
    -D INSTALL_C_EXAMPLES=OFF \
    ..

Важливі зміни в cmake для .whl:

    -D PYTHON_DEFAULT_EXECUTABLE=$(which python3): Це гарантує, що cmake використовуватиме Python з вашого активного віртуального середовища, що є ключовим для правильного пакування.

Після успішного виконання cmake, починайте компіляцію. Це займе значний час (кілька годин). Використовуйте опцію -j з кількістю ядер вашого процесора (на Raspberry Pi 5 це зазвичай 4):
Bash

make -j4

Тепер, замість sudo make install, ми будемо використовувати модуль build для створення .whl файлу. Переконайтеся, що ви все ще в каталозі ~/opencv/build.
Bash

cd .. # Поверніться до кореневого каталогу opencv (тобто ~/opencv)
python3 -m build --wheel

Ця команда запустить процес збірки та пакування, і результатом буде .whl файл у каталозі dist/ всередині вашої директорії opencv.

Приклад імені файлу: opencv_python-4.9.0-cp39-cp39-linux_aarch64.whl (версія Python та архітектура можуть відрізнятися).
5. Перевірка та використання .whl файлу

    Знайдіть ваш .whl файл:
    Він буде в ~/opencv/dist/.
    Bash

ls ~/opencv/dist/

Перенесіть .whl на інший пристрій (або встановіть локально):
Ви можете скопіювати цей файл на USB-накопичувач, завантажити його через мережу або використовувати scp.

Встановлення .whl на іншому Raspberry Pi 5:
На цільовому пристрої (переконайтеся, що він теж Raspberry Pi 5 з тією ж архітектурою та версією Python), створіть віртуальне середовище, якщо потрібно, а потім встановіть файл:
Bash

# Приклад: якщо файл в поточній директорії
pip install opencv_python-4.x.x-cpXX-cpXX-linux_aarch64.whl

Замініть opencv_python-4.x.x-cpXX-cpXX-linux_aarch64.whl на фактичне ім'я вашого файлу.

Перевірка встановлення:
Абсолютно так само, як і після компіляції:
Bash

workon cv # Якщо використовуєте віртуальне середовище
python3

У консолі Python:
Python

import cv2
print(cv2.__version__)
print(cv2.getBuildInformation())

У виводі cv2.getBuildInformation() знову шукайте GStreamer: YES.
