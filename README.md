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

Note: On Ubuntu 16.04 I had a compiler segfault when building the Chromium webengine when using gcc-7.3. Upgrading to gcc-7.4 worked.
If you have followed the Configuring Build Environment - New](https://github.com/NREL/OpenStudio/wiki/Configuring-OpenStudio-Build-Environments---New#linux) wiki page,
you should be fine since you would have the `ppa:ubuntu-toolchain-r/test` which has gcc-7 at 7.4.0.
