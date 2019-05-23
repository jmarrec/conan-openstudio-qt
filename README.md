# Conan-OpenStudio-Qt

The goal is to setup CMake to build conan-qt with the right options and be able to find the Qt components that are required to link against [NREL/OpenStudioApplication](https://github.com/NREL/OpenStudioApplication).

The conan-qt recipe installs a few system dependencies for you, assuming you set up SUDO_ASKPASS on Ubuntu to provide your password.
```
echo "echo 'mypassword'" > askpass.sh
chmod +x askpass.sh
export SUDO_ASKPASS=./askpass.sh
make
```

But it doesn't get you 100% there and you will likely need to install a few dependencies manually, including the ones for Qt Webengine:
see [Building QtWebengine](https://wiki.qt.io/QtWebEngine/How_to_Try) for the list.
